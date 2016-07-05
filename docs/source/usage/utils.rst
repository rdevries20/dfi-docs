Utilities
---------

**ThirdPartyBlacklist.py:**

The function of the third party module is to check external sources for bad hosts or known IP addresses which are involved in distributing malware or can either be a command and control servers. 

To use this feature, the URL list within the ThirdPartyBlacklist.py, which can be found within the ‘utils’ folder, needs to be adjusted to the correct external sources. With a given URL and category which the URL lists an indice can be build in ElasticSearch. Each record in this indice contains the bad IP, category and the URL location where the record was based on.

To build the indice the script can be called with the ‘-r’ parameter.  This can also be used to re-build the indice without deleting the previous batch, ElasticSearch should ignore/overwrite all existing records. 

To build an entirely new indice: one should first clear the indice with can be done using the ‘-c’ parameter. Clearing an indice can also be done using the IndiceManagement utility. 

Usage::

	$ ./ThirdPartyBlacklist.py –r 
		Fetch URL’s and update indice.
	
	$ ./ThirdPartyBlacklist.py -c	
		Clears indice

**IndiceManagement.py:**

IndiceManagement can be found in the ‘utils’ folder and can be used to manage all indice’s used by the analysis system. A failsafe is built in to make sure that it won’t delete any indices other than this list. When adding an indice for any reason, the name of this indice should be added to this list otherwise IndiceManagement is not able to delete the newly created indice.

When using the ‘-ba’ parameter, which builds all indices, the program will check the ‘es-mappings’ folder for any JSON files. These JSON files are supposedly mappings for ElasticSearch; any other JSON file, which is not a mapping, will wreck the ElasticSearch instance. 

The ‘-da’ parameter will drop all indices it knows. This parameter should only be used to  

Usage::

	$ ./IndiceManagement.py –ba 
		Build all indices
	
	$ ./IndiceManagement.py –da
		Drop all indices