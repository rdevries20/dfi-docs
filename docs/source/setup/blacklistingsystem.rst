Setting up the blacklisting system
==================================

Setting up the blacklisting system consists of multiple steps.

Downloading the system
----------------------

Find a directory on your system where you would like to download the system to.
The current link to the Github repository is https://github.com/TomWestenberg/dfi.git
This might change in the future. Verify before downloading.

download the repository by using::

	$ git clone https://github.com/TomWestenberg/dfi.git



Installing all dependencies
---------------------------

The system depends upon a variety of Python packages, these need to be installed. All of these packages
are listed in a .txt file included in the repository. The file is called requirements.txt
You can install them by doing the following::

	$ cd dfi
	  pip install -r requirements.txt
	  
Configuring the system
----------------------

After installing all dependencies, it is time to start editing the configuration file.
The file is called 'config.cfg' and is in the root directory of the project.

You will need to configure the IP address and the API port of the Cuckoo server and the IP address and API
port of the Elasticsearch server.
Open the file in your favorite editor and start editing the lines under the 'Servers' section::

	[Servers]
	# The Cuckoo server where binaries will be submitted to
	cuckoo_server_ip = your-cuckoo-server-ip-or-domain.com

	# The Cuckoo server's API port
	cuckoo_api_port = 8090

	# The Elasticsearch server used to stored all data for this application
	# Queues, blacklists, filtered Cuckoo reports, list of hashes etc
	elasticsearch_server_ip = your-elasticsearch-server-ip-or-domain.com

	# The Elasticsearch API port
	elasticsearch_port = 9200

Creating the required Elasticsearch indices
-------------------------------------------

When you are done with editing, go to the 'utils' folder.
Before you can use the system, your Elasticsearch server needs to be prepared by creating
all indices used by the system. The tool 'IndiceManagement.py' will do this for you.

The indice that will be created are: "blacklist", "cuckoo", "hashes", "queue", "third_party_blacklist", and "whitelist".
**--> Please verify that these indice do not exist yet! This tool will overwrite them. <--**

In the 'utils' folder. Run the IndiceManagement.py script with the help parameter::

	$ ./IndiceManagement.py --help
	  usage: Third-Party Blacklist Builder [-h] [-da] [-ba]

	  optional arguments:
	    -h, --help         show this help message and exit
	    -da, --delete-all  Delete's all indices
	    -ba, --build-all   Posts all indices to ElasticSearch

You will need the indices, so you need to run the script with the --build-all parameter::

	$ ./IndiceManagement.py --build-all
	  Indices created

Configuring the binaries directory
----------------------------------

The system needs to know where you want it to get the binaries to submit to the Cuckoo server.
This can also be configured in the 'config.cfg'. The default configured path is a folder 'binaries' in the root
folder of the project. The binaries directory does not exist be default.

Find this section in the config.cfg::

		# The directory where the binary submitter will look for binaries
		# to submit to the Cuckoo server
		binary_directory = binaries

You can set binary_directory to any directory on your system. **Be sure to use the full path!**

**--> Caution: after a binary has been submitted, it will be deleted from the configured directory!**

The blacklisting score
----------------------
The blacklisting score is the score on which a value gets blacklisted. This score is determined by the matched rules.
Each rule has its own score.

If the total score of all matching rules is equal to or higher than your blacklisting score, the value gets blacklisted.

The default score is 2.0. The score you need depends on the amount of rules you intend to use. When using it in production and with
more than 1 or 2 rules, it is suggested that you use a score of 3.0 at least::

	# The score at which a value will be added to the blacklist.
	# This score is determined using the rules in the rules/ directory
	blacklist_score = 2.0


The API and logging
-------------------

There are three more settings: the logging level, the API listen IP and the API listen port.

By default the API listens on port 8085 and on all interfaces by listening op 0.0.0.0.

The default logging level is DEBUG. Setting the level to INFO is recommended after the system has ran for a while.
While configuring, DEBUG is recommended to find the cause of any crashes, errors::

	# The IP this application's API will listen on
	api_listen_ip = 0.0.0.0

	# The port this application's API will listen on
	api_listen_port = 8081
	
	# The minimum logging level. Debug shows a lot of information.
	# You only want to enable debug if you actually are debugging.
	# It is recommended to use an info level in a production environment
	# Possible levels: debug, info, warning, error
	log_level = debug

Populating the third part blacklist indice
------------------------------------------

This indice is used in the rule '3rd_party_blacklist'. Before you can use this rule, you need to populate the indice.

You can do this by using the 'ThirdPartyBlacklist.py' tool in the 'utils' directory. See the `Utilities doc <../usage/utils.html>`_ 
for more information on how to do this.

Add subnets to the whitelist
----------------------------

The blacklisting system can ignore IP subnets in rules. This is called the whitelist. It is stored
in Elasticsearch under the 'whitelist' indice. You can add subnets to the whitelist and an owner of the subnet by
using `the web API <../usage/api.html#whitelist-subnet-add>`_.
The 'owner' field will be used as a description for the whitelisted IP subnet.

**It is recommended that you add all the private IP subnets and your own subnets to the whitelist**

Creating rules for the system
-----------------------------

These is an important, if not the most important part about the system.

The rules do the matches in the data and determine the total endscore.

All your rule files are in the rules directory at the root of the project. By default, this
directory contains one rules file as an example. The file is called 'hashrules.json'.

This file is a meant as an example and should NOT be used in production. It contains all kinds of bogus values.
However, all rule keys used in the file as keys that you can actually use.

To see how to create rules go the `Using the blacklisting system <../usage/index.html>`_ part of the documentation. This will explain how to write rules,
what rules exist, how to use these, and the utilities for the system.
