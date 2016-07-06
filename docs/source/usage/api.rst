The API
=======

The web API contains all calls used to retrieve the blacklist, manage the whitelist, get statistics, etc

+--------------------------------------+-------------------------------------------------------------------+
| **API calls**                        | **Description**                                                   |
+--------------------------------------+-------------------------------------------------------------------+
| GET :ref:`get_hash_since`            | Returns a list of blacklisted hashes                              |
+--------------------------------------+-------------------------------------------------------------------+
| GET :ref:`get_count_indice_doc_type` | Counts how many records are known in a given                      |
|                                      | indice and have a certain doctype.                                |
+--------------------------------------+-------------------------------------------------------------------+
| GET :ref:`get_stats_hashes`          | Returns the average amount of hashes                              |
|                                      | blacklisted each hour.                                            |
+--------------------------------------+-------------------------------------------------------------------+
| GET :ref:`blacklist_exist`           | Checks if an ID for the given doc type was is blacklisted         |
+--------------------------------------+-------------------------------------------------------------------+
| GET :ref:`blacklist_create`          | Creates blacklist item based on the give do type, id, and reason. |
+--------------------------------------+-------------------------------------------------------------------+
| GET :ref:`blacklist_delete`          | delete                                                            |
|                                      | blacklist items based on a given doc type and unique identifier.  |
+--------------------------------------+-------------------------------------------------------------------+
| GET :ref:`cuckoo_get_hour`           | Returns the amount of Cuckoo reports that                         |
|                                      | get saved per hour                                                |
+--------------------------------------+-------------------------------------------------------------------+
| POST :ref:`whitelist_subnet_add`     | Add an IP subnet and the corresponding owner to the whitelist     |
+--------------------------------------+-------------------------------------------------------------------+
| POST :ref:`whitelist_subnet_delete`  | Delete an IP subnet from the whitelist                            |
+--------------------------------------+-------------------------------------------------------------------+
| GET :ref:`health_check`              | Returns JSON with the availability of all running                 |
|                                      | processes to keep the system operational.                         |
+--------------------------------------+-------------------------------------------------------------------+

.. _get_hash_since:

/hash/get
---------

	Usage: /hash/get/<sinc>
	
	Returns a list of blacklisted hashes

	Method: GET
	
	Mandatory arguments:
		- Since
			- Integer: Returns the blacklisted items in the last 'since' seconds for the hash type. Example 60.
			- All: Returns all the blacklisted items for the hash type
	
	Examples::
	
		$ curl http://localhost:8081/hash/get/all
		["cf48bae14f7973aaecb9a978266b9c0b", "99d473dcb245acbfae1197f370fdecb3"]
		
		$ curl http://localhost:8081/hash/get/60
		["99d473dcb245acbfae1197f370fdecb3"]

.. _get_count_indice_doc_type:

/count
------

	Usage: /count/<indice>/<doc_type>
	
	Counts how many records are known in a given indice with the given doc_type
	
	Method: GET
	
	Mandatory arguments:
		- Indice
			- The name of the indice you want to check
		- doc_type
			- The name of the doc_type you want to check
	
	Example::
	
		$ curl http://localhost:8081/count/whitelist/subnet
		0

.. _get_stats_hashes:		

/stats/hashes
-------------

	Usage: /stats/hashes
	
	Returns the average amount of hashes blacklisted per hour
	
	Method: GET
	
	Example::
	
		$ curl http://localhost:8081/stats/hashes
		0


.. _blacklist_exist:

/blacklist/exist
----------------

	Usage: /blacklist/exist/<doc_type>?id=<blacklist_value>
	
	Checks if the given blacklist value exists in the blacklist indice of the given type.
	
	Method: GET
	
	Mandatory arguments:
		- doc_type
			- The name of the doc_type you want to check
		
	Mandatory parameters:
		- id
			- The value of something that can be blacklisted. An MD5 hash for example
	
	Example::
	
		$ curl http://localhost:8081/blacklist/exist/hash?id=99d473dcb245acbfae1197f370fdecb3
		True

.. _blacklist_create:

/blacklist/create
-----------------

	Usage: /blacklist/create/<doc_type>?id=<blacklist_value>&reason=<reason>
	
	Adds the given id to the blacklist doc_type with the reason specified
	
	Method: GET
	
	Mandatory arguments:
		- doc_type
			- The name of the doc_type you want to check
		
	Mandatory parameters:
		- id
			- The value of something that can be blacklisted. An MD5 hash for example
		- reason
			- A string describing why this value was blacklisted
	
	Example::
	
		$ curl http://localhost:8081/blacklist/create/hash?id=99d473dcb245acbfae1197f370fdecb3&reason=Stuff
		True

.. _blacklist_delete:

/blacklist/delete
-----------------

	Usage: /blacklist/delete/<doc_type>?id=<blacklist_value>
	
	Deletes the given id to the blacklist doc_type given
	
	Method: GET
	
	Mandatory arguments:
		- doc_type
			- The name of the doc_type you want to check
		
	Mandatory parameters:
		- id
			- The value of something that can be blacklisted. An MD5 hash for example
	
	Example::
	
		$ curl http://localhost:8081/blacklist/delete/hash?id=99d473dcb245acbfae1197f370fdecb3
		True

.. _cuckoo_get_hour:

/cuckoo/get/hour
----------------

	Usage: /cuckoo/get/hour
	
	Returns the amount of Cuckoo reports that get saved per hour
	
	Method: GET
	
	
	Example::
	
		$ curl http://localhost:8081/cuckoo/get/hour
		3

.. _whitelist_subnet_add:

/whitelist/subnet/add
---------------------

	Usage: /whitelist/subnet/add
	
	Add an IP subnet and the corresponding owner to the whitelist
	
	Method: POST
	
	Mandatory form fields:
		- subnet
			- Contains an IP subnet in CIDR format. Example: 192.168.0.0/16
		- owner
			- An owner or description of the subnet. Example: Private-c
	
	Example::
	
		$ curl --data "subnet=192.168.0.0/16&owner=private-c" http://localhost:8081/whitelist/subnet/add
		{'message': 'success'}

.. _whitelist_subnet_delete:

/whitelist/subnet/delete
------------------------

	Usage: /whitelist/subnet/delete
	
	Add an IP subnet and the corresponding owner to the whitelist
	
	Method: POST
	
	Mandatory form fields:
		- subnet
			- Contains an IP subnet in CIDR format. Example: 192.168.0.0/16
	
	Example::
	
		$ curl --data "subnet=192.168.0.0/16" http://localhost:8081/whitelist/subnet/delete
		{'message': 'success'}

.. _health_check:

/health_check
-------------

	Usage: /health_check
	
	Returns JSON with the availability of all running processes for the system.
	If a process is not running, it is listed as False, True if it is running.
	
	Method: GET
	
	Example::
	
		$ curl localhost:8082/health_check
		{'CuckooBinarySubmitTask': 'True', 'ReportProcessTask': 'True', 'BlacklistingTask': 'True', 'APIListener': 'True'}
