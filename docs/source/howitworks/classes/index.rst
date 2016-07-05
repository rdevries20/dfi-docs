dfi.api
-------

**ApiListener.py**

	Contains all the web API calls. It uses Flask.

dfi.blacklist
-------------

**abstracts.py**
	
	Contains the abstract classes Rule and BlacklistType. These are used to
	implement new rules and types.

**ScoreBoard.py**

	Contains the class ScoreBoard. This class is used to keep track
	of the score of a single report under a certain type. It stores a score
	and the reasons list. This information is grouped under a rule's group ID.
	
	If no group ID is given to a rule. It will be stored under group 0.

dfi.blacklist.rules
-------------------

**This path contains all implemented rules.**
	
	If you are going to implement a new rule by implementing the Rule abstract class, place
	your class in this path.

dfi.blacklist.types
-------------------

**This path contains all implemented types.**
	
	If you are going to implement a new type by implementing the BlacklistType abstract class, place
	your class in this path.

**HashType.py**
	
	Contains class HashType. This class contains the logic that creates a ScoreBoard for a certain report, applies the rules and actually adds
	the value to a blacklist or not.

dfi.cuckoo
----------

**CuckooCalls.py**

	Contains the class CuckooCalls. This class contains a set of methods to ask a Cuckoo server for task statuses, submit binaries, download
	reports etc.

**CuckooReport.py**

	Contains the class CuckooReport.py. This class is used to stored a filtered version of the Cuckoo report in a single dictionary.
	This class contains the format of the report that will be stored in Elasticsearch. This class is used by the report_hoarder.
	
	This class contains all the fields that are stored in Elasticsearch. If you want to store new fields, add them here first.

**FilteredCuckooReport.py**
	
	Contains the class FilteredCuckooReport. This class is used to store a filtered Cuckoo report that was retrieved from the Elasticsearch
	server. Each value of the report is stored in a seperate attribute. This class also adds an extra value 'ip_wl_filtered' which contains
	a list of IP addresses used by the binary minutes the IPs that are in the whitelisted subnets. The original list is also viewable under 'ip'.

dfi.database
------------

**ESManager.py**
	
	Contains the class ESManager. This class is used to add and retrieve all kinds of information from the Elasticsearch server.
	Upon creating an instance of this class for the first time, a small check will be done to see if the Elasticsearch server is
	online.

dfi.rule
--------

**RuleFactory.py**
	
	Contains the class RuleFactory. This class contains the dictionary that matches rule keys used in the rule file to the rule class with the logic
	for that rule. This class is used by Type classes(like HashType) to ask for an instance of a rule class by providing the rule key from
	the rule file.
	
	If you implement a new rule you need to add the name/rule key of your rule to this dictionary together with the matching class.

**TypeFactory.py**

	Contains the class TypeFactory. This class contains the dictionary that matches type keys used in the rule file to the type class with the logic
	for that type. This class is used by the BlacklistingTask class to ask for an instance of a type class by providing the type key from
	the rule file.
	
	If you implement a new type you need to add the name/type key of your type to this dictionary together with the matching class.

**RuleLoad.py**

	Contains the class RuleLoad. This class contains the dictionary with all rules that are used. Each rule is stored under its corresponding type.
	Upon calling 'load_all_rules', it will read all files in the rules directory and verify is they are valid. If they are valid, the rules are added
	to the dictionary.

**Whitelist.py**
	
	Contains the class Whitelist. This class contains an IPSet object containing all the whitelist IP subnets.
	These subnets are refreshed every hour. This class also contains the logic to filter out whitelisted IPs
	from a list.

dfi.task
--------

**BlacklingTask.py**
		
		Contains the class BlacklistingTask. This class downloads 'processed: False' Cuckoo reports from Elasticsearch and applies
		the rules on each report by creating instances of the implemented types. After finishing, it sets the reports to 'processed: True'
		
		This class is used by the blacklist_processor.py script.

**CuckooBinarySubmitTask.py**

	Contains the class CuckooBinarySubmitTask. This class reads binaries from the binaries folder, calculates their sha256 hashes, checks if they have ever been
	submitted before and submits them to the Cuckoo server. After that the sha256 hash of the binary is added to the 'hashes' indice and the return Cuckoo task ID from
	Cuckoo is added to the 'queue' indice. When this all succeededs, the binary is deleted from the directory.
	
**ReportProcessTask.py**

	Contains the class ReportProcessTask. This class downloads task IDs from the 'queue' indice, asks Cuckoo for their status, downloads the reports if they are reported,
	filters out the fields by using the CuckooReport class, and stores the report in Elasticsearch with the extra value 'processed: False'.

dfi
---

**Config.py**

	Contains the class Config. This class contains all the config variables used by all the other classes.
	This class tries to load a 'config.cfg' file, if it is not present, a file will be generated.
	
	After loading the files, the value each config variable in this class is replaced by the corresponding value
	in the config file.
	
**Logger.py**
	
	Contains the class Logger. This class creates new logs for the given name in setup_logger. It uses the logging level stated in the config file.
	The format of the logger is set here.
	
	The logging levels of the 'elasticsearch' and 'urllib3' loggers are set to the ERROR level here. This is done because they both log almost all
	activity under warning. For example: when checking if a value exists in Elasticsearch, it will generate a 'WARNING 404!'. These checks are performed
	very often. These warnings would clog your logfile.
