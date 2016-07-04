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

