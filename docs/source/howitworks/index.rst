How it works
============
The blacklisting system consists of four parts. Each of these parts has a
seperate run script. The scripts use the indices in Elasticsearch.

The existing Elasticsearch indices
----------------------------------
There are six different indices. Each of these indices stores
other data.

The list below will show the name of the indice, the corresponding mapping file, and
the document type name used to store items under the indice.

queue
	**File:** queue.json - **Document type name:** item

	This indice stores a pieces of info on a binary submitted to Cuckoo.
	As the name suggests, it is used as a queue so the blacklisting system knows
	what was submitted to Cuckoo.

hashes
	**File:** hashes.json - **Document type name:** item

	This indice stores sha256 hashes of all submitted binaries. It is used
	to see if a binary was submitted before.

cuckoo
	**File:** cuckoo.json - **Document type name:** report

	This indice stores all Cuckoo reports that were filtered by the report_hoarder.py.

blacklist
	**File:** blacklist.json
	
	**Document type name:** This depends on the used 'type'. Entries in the blacklist indice
	are stored under their type name. At this moment only the 'hash' type is supported.
	So for that type the document name is hash.

	This indice stores values that have been blacklisted by the blacklist_processor.py. It also
	stores the reason something was blacklisted and the score for the report to which the blacklisted
	value belongs. For the hash' type, an MD5 hash of the binary is stored.

whitelist
	**File:** whitelist.json - **Document type name:** subnet

	This indice stores IP subnets that should be excluded from counting if a rule matches.
	The subnets are stored in CIDR notation. If you would add a new subnet, you will have to provide
	something like '192.168.0.0/16' and an owner, which would be 'private' in this case.

	When a Cuckoo report is loaded into memory from the cuckoo indice, a new object attribute will be added which
	will only reside in memory and is called 'ip_wl_filtered'. This is a list which contains all IPs from the Cuckoo report
	minus IPs that are in one of the whitelisted subnets.

	IP subnets can be added and removed through the API.

third_party_blacklist
	**File:** third_party_blacklist.json - **Document type name:** C&C

	This indice is used to store lists of known malware command and control servers.
	The indice can be filled by using the tool 'ThirdPartyBlacklist.py' in the utils folder.

	The values in the indice are used by the '3rd_party_blacklist' rule.

Binary submitting
-----------------

Script name: **submit_binary.py**

This script reads the binaries from the configured directory and calculates a sha256 hash.
It will then check if this hash is in the 'hashes' indice in the Elasticsearch server.
This indice is used to keep track of all earlier submitted binaries to prevent analysing the same
binary more then once.

If not present in this indice, it will then proceed to submit it to the Cuckoo Sandbox server using the extra option 'json.calls=0'

*'json.calls=0' means that we ask Cuckoo not to store a list of all seperate API calls and their arguments in the report.
We do not use these at this time and it is about 90% of the report. We do this to prevent getting reports of 500MB+* 

Only after a successful submission, add the the hash, the Cuckoo task ID, and a timestamp to the 'queue' indice.
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

The filtered report will then be stored in Elasticsearch in the 'cuckoo' indice with an extra value that tells our system
that the report has not been processed by the blacklisting system's rules yet. This value is called 'processed' and has
a boolean value.

If the task ID is still in the 'queue' indice, it will now be removed.

When there are no more task IDs in the 'queue' indice, it will wait for 5 minutes before checking again.


Blacklisting values by applying rules
-------------------------------------

Script name: **blacklist_processor.py**

The goal of this script is to apply rule logic on the filtered reports in the Elasticsearch server. This logic is called rules
and the system has multiple of those. The rules that will be applied are in the JSON files in the 'rules' directory.
Each of these rules is a dictionary entry containing an object list with the values "rules" and "score".
The "rules" key contains a list
with values that are matched with values in the processed Cuckoo report. The score field contains the weight of the match. If a match
for a rule is found, the total score is incremented by the value of the score field.

The script starts by loading all rule JSON files, verifying if the rules exist, and sorting the rules under the corresponding type.
The type of rule is also stored in the rule file. A type is something that can be blacklisted.
At this time the only existing type is 'hash'.

*New types van be added by implementing the abstract class dfi.blacklist.abstracts.BlacklistType*

After the rules are loaded, it starts asking the Elasticsearch server if it has any reports with the 'processed' value False. If it does, it will start downloading these in
groups of eight. It does not download to many at one time because these reports can be up to 40MB in size.

After downloading the set of reports it will apply all rules for all types(currently only 'hash') by doing first
creating an instance for the supported types and then asking this type to handle all rules.


The handling of the rules means that an object(a scoreboard) to store the score for the current report is created and 
each loaded rule in memory under the corresponding type will be applied to the report.
If a match is found, the score belonging to this rule will be added to the total score stored in the scoreboard for this report. Besides
adding the score, the rule that match, together with the data that matched it will be added to a list of 'reasons'.

When all rules are processed the control is transferred to the type which will ask the scoreboard for the total score. If
the total score is **higher or equal** than the configured 'blacklist score' in the config.cfg, the value for the corresponding type
will be 'blacklisted' by storing the type and the reasons list in the Elasticsearch server in the 'blacklist' indice. The document type
name will be equal to the typename.

*for the type hash the value to be blacklisted is an MD5 hash. This would be stored in 'blacklist/hash/<MD5>'*

After a report has been processed, the sha256 hash from the binary belonging to the report is added to a list of 'processed reports'.

When all reports have been processed, Elasticsearch is asked to bulk update all 'processed' field to True in the 'cuckoo' indice for the given list of IDs(sha256 hashes).

The API
-------

Script name: **api_start.py**

Upon starting this script it will start a webserver listing on the configured IP and port.

The goal of this API is to give a user the ability to ask for a list of blacklisted items, manage the whitelist, ask for
stats, and ask if all processes of the blacklisting system are running.

For a complete list of API calls, see the API documentation.

