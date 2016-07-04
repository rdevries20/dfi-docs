How it works
============
The blacklisting system consists of four parts. Each of these parts has a
seperate run script.

Binary submitting
-----------------

Script name: **submit_binary.py**

This script reads the binaries from the configured directory and calculates a sha256 hash.
It will then check if this hash is in the 'hashes' indice in the Elasticsearch server.
This indice is used to keep track of all earlier submitted binaries to prevent analysing the same
binary more then once.

If not present in this indice, it will then proceed to submit it to the Cuckoo Sandbox server and only
after a successful submission, add the the hash, the Cuckoo task ID, and a timestamp to the 'queue' indice.
This indice contains all binaries submitted to Cuckoo Sandbox and is used to keep track of the status of
each task.

After submitting and adding it to the queue, the hash of a binary will be added to the 'hashes' indice
so we later know it has been analysed already. As a last step **the binary is deleted** from the configured
directory.

If we encounter a binary that has already been analysed, a counter for this binary is incremented
in the 'hashes' indice.


Report downloading and filtering
--------------------------------

Script name: **report_hoarder.py**

The goal of this script is to download reports from the Cuckoo Sandbox server and filter out
all values which we do not use and change to structure so we can easily store it in the Elasticsearch
server.

This script starts by asking how many VMs the Cuckoo Sandbox server has. It will then use this amount to process
the reports in the queue in groups of the number of the VMs.

We do this to because a Cuckoo task can take some time, which means we will have to wait for it to finish.

*Say there are four VMs. If we have downloaded all tasks with the status 'reported' and there currently are
four tasks still running, we now we wait for a minute before asking again. This way we only introduce a wait
if we actually need to wait.*

After we know the amount of VMs, the next step is to ask Elasticsearch for items from the 'queue' indice.
A smaller subqueue, the size of the amount of VMs, is filled with Cuckoo task IDs. The 'queue' indice
will be emptied 'first in, first out'

When it has some task IDs, it will ask Cuckoo for the status of each task. A task can have multiple statuses of
which these are especially important:

- Pending - This means it has not been analysed yet
- Running - This means the binary is being executed
- Completed - This means the binary has been executed and the results are now being analysed and written to disk
- Analysis failed - This means something went wrong with the analysis and it could not be reported.
- Reported - This means the report can be downloaded

To prevent the report downloader from waiting forever a seperate dictionary of waiting times is being kept.
When a task ID is added to the waiting dictionary, it is also removed from the 'queue' indice. This is to prevent
getting the same ID again.

A task will be added to a waiting dictionary after Cuckoo first reports it as running. Then only when it is waiting
on all tasks in the subqueue, it will wait for 30 seconds before asking for a status and store that it has waited this amount 
of time.

When the waiting time for a task exceeds 10 minutes, the task will be skipped because it is likely that an error occured with this
task and Cuckoo is not telling this with a status.

A task is also skipped when the status 'analysis failed' is returned.

When Cuckoo returns the status 'reported', the report will be downloaded. After downloading,
a new empty report will be created in the format that Elasticsearch can handle and this report
will be filled with the values we want from the actual Cuckoo report.

*The dfi.cuckoo.CuckooReport class contains the values that are read from the Cuckoo report.*

The filtered report will then be stored in Elasticsearch with an extra value that tells our system
that the report has not been processed by the blacklisting system's rules yet. This value is called 'processed' and has
a boolean value.

If the task ID is still in the 'queue' indice, it will now be removed.

When there are no more task IDs in the 'queue' indice, it will wait for 5 minutes before checking again.
