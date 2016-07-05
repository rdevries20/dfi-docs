Supported rules
---------------

All currently supported rules will be described here.
For each rule a small example of how to use it is included.

Regular expressions and wildcards
---------------------------------

Some rules support the usage of wildcards or regular expressions in their
rule values.

**It is important to understand that you can you either wildcards or a regular expression. Not both
at the same time.**

Supported wilcards are:

- '?'     for a single character
- \'*'    for unlimited characters

**Regular expressions**

The regular expressions are implemented using the Python 2.7 re lib. Your regular expressions
must therefore be valid for that lib!

When using a regular expressions, you MUST use the '!REGEX!' prefix infront of it. No spaces after it.
This string is used to recognize that you have used a regular expression.


+-----------------------+----------------------------------------------------------+
| **Rule key**          | **Short description**                                    |
+-----------------------+----------------------------------------------------------+
| command_line_         | Matches command line commands used                       |
+-----------------------+----------------------------------------------------------+
| cuckoo_score_         | Matches the Cuckoo score of a task                       |
+-----------------------+----------------------------------------------------------+
| dead_host_amount_     | Matches the amount of non responsing hosts               |
+-----------------------+----------------------------------------------------------+
| dir_created_          | Matches the names of directories that were created       |
+-----------------------+----------------------------------------------------------+
| dir_enumerated_       | Matches the names of directories that were enumerated    |
+-----------------------+----------------------------------------------------------+
| dir_removed_          | Matches the names of directories that were removed       |
+-----------------------+----------------------------------------------------------+
| file_deleted_         | Matches the names of files that were removed             |
+-----------------------+----------------------------------------------------------+
| file_opened_          | Matches the names of files that were opened              |
+-----------------------+----------------------------------------------------------+
| file_read_            | Matches the names of files that were read                |
+-----------------------+----------------------------------------------------------+
| file_written_         | Matches the names of files that were written             |
+-----------------------+----------------------------------------------------------+
| http_path_            | Matches HTTP POST and GET paths.                         |
+-----------------------+----------------------------------------------------------+
| http_user_agent_      | Matches a HTTP user-agent string                         |
+-----------------------+----------------------------------------------------------+
| ip_in_range_          | Matches an IP in the specified range                     |
+-----------------------+----------------------------------------------------------+
| ip_in_subnet_         | Matches an IP in the specified subnet                    |
+-----------------------+----------------------------------------------------------+
| mutex_                | Matches created mutex                                    |
+-----------------------+----------------------------------------------------------+
| process_name_         | Matches the name of a created process                    |
+-----------------------+----------------------------------------------------------+
| registry_key_deleted_ | Matches a registry key that was deleted                  |
+-----------------------+----------------------------------------------------------+
| registry_key_opened_  | Matches a registry key that was opened                   |
+-----------------------+----------------------------------------------------------+
| registry_key_read_    | Matches a registry key that was read                     |
+-----------------------+----------------------------------------------------------+
| registry_key_written_ | Matches a registry key that was written                  |
+-----------------------+----------------------------------------------------------+
| signature_name_       | Matches the triggered Cuckoo signature names             |
+-----------------------+----------------------------------------------------------+
| signature_severity_   | Matches the severity of the triggered Cuckoo signatures  |
+-----------------------+----------------------------------------------------------+
| string_               | Matches strings found in a binary                        |
+-----------------------+----------------------------------------------------------+
| tcp_port_             | Matches used TCP ports                                   |
+-----------------------+----------------------------------------------------------+
| 3rd_party_blacklist_  | Matches IPs that are in the third_party_blacklist indice |
+-----------------------+----------------------------------------------------------+
| udp_port_             | Matches used UDP ports                                   |
+-----------------------+----------------------------------------------------------+
| vt_score_             | Matches the Cuckoo score                                 |
+-----------------------+----------------------------------------------------------+

.. _command_line:

**command_line**
	
	Here rule value should be a string that resembles a command .
	This rule will search through all commands used to
	see if any of them match with rule value

	the rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes
	
	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"command_line": [
					{
						"rules": ["vssadmin.exe delete shadows /all /Quiet"],
						"score": 2.0
					}
				]
			}
		}

.. _cuckoo_score:		

**cuckoo_score**

	The rule value is a float range here. The seperator is a '-'. Examples: 1.0-4.0, 0.5-2.0 etc

	This rule matches if the Cuckoo score falls within
	the specified range in the rule value
	
	- Wilcards: No
	- Regular expression: No
	
	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"cuckoo_score": [
					{
						"rules": ["1.0-3.5"],
						"score": 1.0
					}
				]
			}
		}

.. _dead_host_amount:		
		
**dead_host_amount**

	The rule value is an integer range here. The seperator is a '-'
	examples: 10-20, 1-1, 5-15 etc

	This rule matches if the amount of dead hosts falls
	with the int range specified in rule values
	
	- Wilcards: No
	- Regular expression: No

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"dead_host_amount": [
					{
						"rules": ["1-3"],
						"score": 0.5
					}
				]
			}
		}

.. _dir_created:

**dir_created**

	Here the rule value should be a string that resembles a directory.
	This rule will search through all created directories to
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"dir_created": [
					{
						"rules": ["C:\\Users\\*", "!REGEX![A-Z]"],
						"score": 0.5
					}
				]
			}
		}

.. _dir_enumerated:

**dir_enumerated**

	Here the rule value should be a string that resembles a directory.
	This rule will search through all enumerated directories to
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"dir_enumerated": [
					{
						"rules": ["C:\\secret\\*", "!REGEX![A-Z]"],
						"score": 0.5
					}
				]
			}
		}


.. _dir_removed:

**dir_removed**

	Here the rule value should be a string that resembles a directory.
	This rule will search through all removed directories to
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"dir_removed": [
					{
						"rules": ["C:\\secret\\*", "!REGEX![A-Z]"],
						"score": 0.5
					}
				]
			}
		}


.. _file_deleted:

**file_deleted**

	Here the rule value should be a string that resembles a file.
	This rule will search through all deleted files to
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"file_deleted": [
					{
						"rules": ["*privatekey.key", "!REGEX![A-Z]"],
						"score": 0.5
					}
				]
			}
		}
		
.. _file_opened:

**file_opened**

	Here the rule value should be a string that resembles a file.
	This rule will search through all opened files to
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"file_opened": [
					{
						"rules": ["C:\\secret\\*", "!REGEX![A-Z]"],
						"score": 0.5
					}
				]
			}
		}

.. _file_read:

**file_read**

	Here the rule value should be a string that resembles a file.
	This rule will search through all read files to
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"file_read": [
					{
						"rules": ["*privatekey.key", "!REGEX![A-Z]"],
						"score": 0.5
					}
				]
			}
		}

.. _file_written:

**file_written**

	Here the rule value should be a string that resembles a file.
	This rule will search through all written files to
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"file_written": [
					{
						"rules": ["*.exe", "*.dll"],
						"score": 0.5
					}
				]
			}
		}

.. _http_path:

**http_path**

	Retrieves all HTTP requests paths from a Cuckoo report, function
	checks with a regular expression if it comes across a match.
	
	The rule value is an HTTP path
	
	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::

		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"http_path": [
					{
						"rules": ["/wp-*", "/Content*"],
						"score": 0.5
					}
				]
			}
		}
	
.. _http_user_agent:

**http_user_agent**

	Retrieves all HTTP User-Agents from a Cuckoo report, function
	checks with a regular expression if it comes across a match.
	
	The rule value is an HTTP user agent.
	
	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::

		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"http_user_agent": [
					{
						"rules": ["Mazilla/*", "cryptolocker*"],
						"score": 0.5
					}
				]
			}
		}	


.. _ip_in_range:

**ip_in_range**

	Checks if a given IP Address is in a certain range.
	Returns one IP if the range is a single IP, returns
	
	The rule value should be an IP range like so: 192.168.1.4-192.168.1.49 or be a single ip like so 192.168.1.19.
	
	**IPs that are in the subnets added to the 'whitelist' indice will never be matched!**
	
	- Wilcards: No
	- Regular expression: No
	
	Example::

		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"ip_in_range": [
					{
						"rules": ["144.241.11.10-144.241.11.235"],
						"score": 0.8
					}
				]
			}
		}	


.. _ip_in_subnet:

**ip_in_subnet**

	Checks if a given IP Address is in a certain subnet.
	
	The rule value should be an IP address subnet in CIDR notation.
	Like so: 192.168.0.0/16
	
	**IPs that are in the subnets added to the 'whitelist' indice will never be matched!**
	
	- Wilcards: No
	- Regular expression: No
	
	Example::

		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"ip_in_subnet": [
					{
						"rules": ["8.8.0.0/16", "23.116.11.0/24"],
						"score": 0.8
					}
				]
			}
		}		

.. _mutex:

**mutex**

	Here the rule value should be a string that resembles a mutex created by a binary.
	This rule will search through all the created mutexes to 
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"mutex": [
					{
						"rules": ["__wretw_w4523_345"],
						"score": 0.5
					}
				]
			}
		}

.. _process_name:

**process_name**

	Here the rule value should be a string that resembles a process name.
	This rule will search through all the started processes to 
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"process_name": [
					{
						"rules": ["evil.exe"],
						"score": 0.5
					}
				]
			}
		}

.. _registry_key_deleted:

**registry_key_deleted**

	Here the rule value should be a string that resembles a registry key.
	This rule will search through all the deleted registry keys to 
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"registry_key_deleted": [
					{
						"rules": ["HKLM\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate\\AU"],
						"score": 0.5
					}
				]
			}
		}


.. _registry_key_opened:

**registry_key_opened**

	Here the rule value should be a string that resembles a registry key.
	This rule will search through all the opened registry keys to 
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"registry_key_opened": [
					{
						"rules": ["HKLM\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate\\AU"],
						"score": 0.5
					}
				]
			}
		}


.. _registry_key_read:

**registry_key_read**

	Here the rule value should be a string that resembles a registry key.
	This rule will search through all the read registry keys to 
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"registry_key_read": [
					{
						"rules": ["HKLM\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate\\AU"],
						"score": 0.5
					}
				]
			}
		}

.. _registry_key_written:

**registry_key_written**

	Here the rule value should be a string that resembles a registry key.
	This rule will search through all the written registry keys to 
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"registry_key_deleted": [
					{
						"rules": ["KEY_CURRENT_USER\\Software\\Microsoft\\Windows\\CurrentVersion\\Run"],
						"score": 0.5
					}
				]
			}
		}

.. _signature_name:

**signature_name**

	The rule value here is the name of a Cuckoo signature.
	This rule will search all triggered Cuckoo signatures to see
	if it finds a match with the given rule value.
	
	If you are looking for the names of these signatures, these can be found in the source
	code of the signatures module in the cuckoo/community repository.
	
	Or simply `click here  <https://github.com/cuckoosandbox/community/tree/master/modules/signatures>`_ for the correct repository folder.
	
	- Wilcards: No
	- Regular expression: No

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"signature_name": [
					{
						"rules": ["ransomware_files", "modifies_files"],
						"score": 0.5
					}
				]
			}
		}

.. _signature_severity:

**signature_severity**

	The rule value here is a severity integer. Cuckoo signatures have severities
	from 1-5. 3-5 are mostly signatures for specific malware, highly malicious actions
	or very suspicious actions.
	
	This rule will search all triggered Cuckoo signatures and see what their severity is.
	If a sverity matching the rule value is found, the rule matches.
	
	If you are looking for the severities of all signatures, these can be found in the source
	code of the signatures module in the cuckoo/community repository.
	
	Or simply `click here  <https://github.com/cuckoosandbox/community/tree/master/modules/signatures>`_ for the correct repository folder.
	
	- Wilcards: No
	- Regular expression: No

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"signature_severity": [
					{
						"rules": [2],
						"score": 0.1
					},
					{
						"rules": [3,4,5],
						"score": 0.5
					}
				]
			}
		}

.. _string:

**string**

	Here the rule value should be a string that resembles string embedded in a binary.
	This rule will search through all the strings found in the binary to 
	see if any of them match with the rule value

	The rule value can have wildcards -> or <- be a regular expression in the
	python lib re format.
	
	- Wilcards: Yes
	- Regular expression: Yes

	Example::
	
		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"string": [
					{
						"rules": ["somestring"],
						"score": 0.5
					}
				]
			}
		}


.. _tcp_port:

**tcp_port**

	The rule data value is a TCP port number here.
	This rule checks if the TCP port number in the rule value was connected to.
	It searches the TCP traffic object list in the Cuckoo report.
	
	**IPs that are in the subnets added to the 'whitelist' indice will never be matched!**
	
	- Wilcards: No
	- Regular expression: No

	Example::

		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"tcp_port": [
					{
						"rules": [80],
						"score": 0.1
					}
				]
			}
		}

.. _3rd_party_blacklist:

**3rd_party_blacklist**

	Checks if an single IP exists in the IP blacklist
	database which is build from external sources.

	The rule value is the name of the doc_type within the indice third_party_blacklist.
	This means that you can add any rule value and use this rule to check if it exists.
	
	To refresh/build a new list of IPs, use the ThirdPartyBlacklist.py tool. See the Utilities documentation for more information.
	By default the 'third_party_blacklist' indice is empty it is empty!

	Currently, the only type of IP blacklists that are being collected are malware command and control server IPs.
	
	The currently support rule data types are:

	- C&C
	
	- Wilcards: No
	- Regular expression: No

	Example::

		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"3rd_party_blacklist": [
					{
						"rules": ["C&C"],
						"score": 2.0
					}
				]
			}
		}

.. _udp_port:

**udp_port**

	The rule data value is a UDP port number here.
	This rule checks if the UDP port number in the rule value was connected to.
	It searches the UDP traffic object list in the Cuckoo report.
	
	**IPs that are in the subnets added to the 'whitelist' indice will never be matched!**
	
	- Wilcards: No
	- Regular expression: No

	Example::

		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"udp_port": [
					{
						"rules": [53],
						"score": 0.1
					}
				]
			}
		}

.. _vt_score:

**vt_score**

	The rule value is an integer range like: 1-5 or 10-20.
	
	This rules checks how many Virustotal hits the submitted binary has.
	
	It is advised to give a high score to more than 10 matches.
	
	At this time, Virustotal has about 58 scanners.
	
	- Wilcards: No
	- Regular expression: No

	Example::

		{
			"info": {
			"	type": "hash"
			},
			"rules": {
				"vt_score": [
					{
						"rules": [1-5],
						"score": 0.5
					},
					{
						"rules": [6-58],
						"score": 5.0
					}
				]
			}
		}
