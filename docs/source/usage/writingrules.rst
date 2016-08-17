Writing rules for a rules file
------------------------------

A rule file should be a JSON file with the keys "info" and "rules"
at its root.

The info key contains information about this rule file in form of a dictionary. At this moment the only
information that it must always contain is a 'type'. This indicates for what the rules
are. The type is the value that will be added to the blacklist. Currently, the only supported type
is hash.

The rules key should contain a dictionary of all rules keys that you want to use.

Example of a rules file without any rules for the type hash.

 .. code-block:: javascript
    :linenos:

	{
		"info": {
		"	type": "hash"
		},
		"rules": {
		}
	}

We are now read to start adding rules that we want to use.

We will start by adding the 'cuckoo_score' rule. Each rule in rules at the root, is made up
of a json object list. **Each of these objects has a "rules" and a "score" key. The value of the "rules" key
is always a list and the value of the "score" key is always a float.**

 .. code-block:: javascript
    :linenos:

	{
		"info": {
		"	type": "hash"
		},
		"rules": {
			"cuckoo_score": [
				
			]
		}
	}
	
We will no add a value to it. We want to increment the total score by 0.5 if the Cuckoo score is from 0.0 to 2.0.
If the score if from 2.1 to 4.0, we want to increment the total score by 1.0, and for scores from 4.1-20.0 we want to increment
the total score by 2.0.

 .. code-block:: javascript
    :linenos:

	{
		"info": {
		"	type": "hash"
		},
		"rules": {
			"cuckoo_score": [
				{
					"rules": ["0.0-2.0"],
					"score": 0.5
				},
				{
					"rules": ["2.1-4.0"],
					"score": 1.0
				},
				{
					"rules": ["4.1-20.0"],
					"score": 2.0
				}
			]
		}
	}
	
Now let's add another rule. "dir_removed". We want to increment the score with 2.0 if
the binary removed any directories in the "C:\Windows\system32" or the "C:\Windows\SysWOW64" directory.

 .. code-block:: javascript
    :linenos:

	{
		"info": {
		"	type": "hash"
		},
		"rules": {
			"cuckoo_score": [
				{
					"rules": ["0.0-2.0"],
					"score": 0.5
				},
				{
					"rules": ["2.1-4.0"],
					"score": 1.0
				},
				{
					"rules": ["4.1-20.0"],
					"score": 2.0
				}
			]
		},
		"dir_removed": [
			{
				"rules": ["C:\Windows\system32*", "C:\Windows\SysWOW64*"],
				"score": 2.0
			}
		]
	}


Grouping rules together
-----------------------

It might happen that you want a rule to match only if another rules also matches, this is where
rule grouping can be used. It is possible to add the key "group_id" to a rule and give it a positive integer value and
1 or higher.

By doing this, the total blacklisting score will only be incremented if all rules in a certain group were matched.


 .. code-block:: javascript
    :linenos:

	{
		"info": {
		"	type": "hash"
		},
		"rules": {
			"cuckoo_score": [
				{
					"rules": ["0.0-2.0"],
					"score": 0.5
				}
			],
			"ip_in_subnet": [
				{
					"rules": ["8.8.0.0/16", "23.116.11.0/24"],
					"score": 0.8
				},
				{
					"rules": ["44.22.1.0/24", "119.23.0.0/16"],
					"score": 1.0,
					"group_id": 3
				}
			],
			"dir_removed": [
				{
					"rules": ["C:\Windows\system32*", "C:\Windows\SysWOW64*"],
					"score": 2.0,
					"group_id": 3
				}
			]
		}
	}

	
The rules are now ready for use.