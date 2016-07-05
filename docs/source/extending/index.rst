Adding new rules and types
==========================

It is possible to add new rules and types.
Currently the only added type is 'hash'.

There are a lot of rules.

Here it will be explained how to add new types and rules by implementing
the correct classes. All implementable classes are in dfi.blacklist.abstracts.

Adding new rules
----------------

This is the easy one to do.

Say we want to add a rule that checks if the given string from the rule json file in /rules
matches a string in the stringlist in the Cuckoo report.

Start by creating a new file in dfi.blacklist.rules and give it a description name.

We will create the file 'StringRule.py'

Start by importing the abstract class Rule like so:

 .. code-block:: python
    :linenos:
	
	from dfi.blacklist.abstract import Rule

After import, create the class that will implement the Rule abstract class like so:

Start by importing the abstract class Rule like so:

 .. code-block:: python
    :linenos:
	
	from dfi.blacklist.abstract import Rule
	
	class StringRule(Rule):

Ok, now what? We only have to implement a single method called 'match_rule'. This
method gets one parameter called 'rule_data'.

**It is important to understand that your rule will be called for EACH value in the rules list
in each rule object belonging to your rule key.**

Example:

 .. code-block:: javascript
    :linenos:
 
	{
		"string": [
			{
				"rules": ["BuildTime", "Somevaluethatiswildcarded*", "!REGEX![0-9]"],
				"score": 0.2
			},
			{
				"rules": ["CRYPTOLOCKER"],
				"score": 1.5
			}
		]
	}
	
The above is an example of how your rule could be used in the rule file. Here
we assumed we called it "strings"(How to do this is explained at the end).
This rule will be called **4** times. Each time your rule is called, 
'rule_data' is one of the values in the 'rules' lists.

The abstract class Rule contains a set of helper methods which can support in
writing your rules.

The methods that you can use are:

**search_wildcard_regex(self, pattern, string)**

	Accepts a pattern and a string to search through.

	The pattern string can contain a regex or wildcards, in this case the
	pattern should have the "!REGEX!" prefix.

	If it does not have the prefix it is treated as a normal string
	with wildcards in it. Available wildcards are: * and ?

	returns True on a found match

	\* = any amount of characters
	? = single character

	Throws sre_constants.error on invalid regex


**number_in_float_range(self, numrange, number)**

	Checks if the given float number falls within the float
	range numrange. The change should be a string formatted like this:
	'2.4-5.0' and always be from low to high or two of the same numbers

**number_in_int_range(self, numrange, number)**

	Checks if the given int number falls within the int
	range numrange. The change should be a string formatted like this:
	'2-5' and always be from low to high or two of the same numbers
		
**validate_number_range(self, numrange)**

	Validates if the given range is formatted like:  1-5 or 1.0-5.0.
	from small to large.
	
	Returns False if the range is invalid, True if valid.
		

We want to match strings in the Cuckoo report. For this we can use the 'search_wildcard_regex' method.

Ok, so how do we get the values from the Cuckoo report? Simple, the report is an attribute of your rule instance.
So you can call self.cuckoo_report. The attribute of the Cuckoo report we need is called 'strings' and is a list.

*What each attribute in the Cuckoo report is can be viewed in FilteredCuckooReport.py*

We need to stored match somewhere. So we will store them in a list called 'matches'

 .. code-block:: python
    :linenos:
	
	from dfi.blacklist.abstract import Rule
	
	class StringRule(Rule):
	
		def match_rule(self, rule_data):
		
			matches = []
			
			for string in self.cuckoo_report.string:
				
				if self.search_wildcard_regex(rule_data, string):
					matches.append(string)

Now, the helper method 'search_wildcard_regex' uses the an re lib method which can throw the 'sre_constants.error'.
We need to catch that error and log it.

 .. code-block:: python
    :linenos:
	
	import sre_constants
	import logging
	
	from dfi.blacklist.abstract import Rule
	
	logger = logging.getLogger(__name__)
	
	
	class StringRule(Rule):
	
		def match_rule(self, rule_data):
		
			matches = []
			
			for string in self.cuckoo_report.string:
				try:
				
					if self.search_wildcard_regex(rule_data, string):
						matches.append(string)
					
				except sre_constants.error as e:
					logger.error("Invalid regex %s. Error: %s", rule_data, e)

We are almost done! The last thing we need to know is what to return.

This is very important! **If no match was found, you must return None. If a match was found, you MUST return a list,
even if it only has 1 entry.**

Your list of returned matches will be used in the 'reasons' field in the blacklist indice if the report ends
up exceeding the max score.

 .. code-block:: python
    :linenos:
	
	import sre_constants
	import logging
	
	from dfi.blacklist.abstract import Rule
	
	logger = logging.getLogger(__name__)
	
	
	class StringRule(Rule):
	
		def match_rule(self, rule_data):
		
			matches = []
			
			for string in self.cuckoo_report.string:
				try:
				
					if self.search_wildcard_regex(rule_data, string):
						matches.append(string)
					
				except sre_constants.error as e:
					logger.error("Invalid regex %s. Error: %s", rule_data, e)
			
			if len(matches) > 0:
				return matches
			
			return None

The above would be the end result of the implementation of the rule.

**There is one more step before you can actually use the rule**
You have to give your rule a rule key name. This is done by adding your
rule to the dfi.rule.RuleFactory class in the RULE_CLASS_MATCH dictionary.

Start by adding a new line at the imports that wil import your rule. After importing,
make up a key which you want to use in your rule file and add the name of the class as the
value for the dictionary key.

 .. code-block:: python
    :linenos:
	
	from dfi.blacklist.rules.StringRule import StringRule
	
	
	class RuleFactory(object):
	
		RULE_CLASS_MATCH = {
			"string": StringRule,
		}
	

You can now use your newly created rule.


Adding a new type
-----------------

Adding a new type is somewhat more complex. You will need to added this class on multiple spots
and completely determine the logic for your new type.

Start by adding a new file to dfi.blacklist.types. Now import BlacklistType from dfi.blacklist.abstracts and
implement the method 'handle_rules'.

 .. code-block:: python
    :linenos:
	
	import logging
	import sys
	
	from dfi.blacklist.abstract import BlacklistType
	
	logger = logging.getLogger(__name__)
	
	
	class SomeType(BlacklistType):
	
		def handle_rules(self):
		
		# check if the rules for your type exist
		 if not self.type_name in RuleLoad.rules:
            logger.error("No rules for type name: %s Cannot run.",
                         self.type_name)
            sys.exit("Exiting..")
			
		# Create a ScoreBoard instance
		
		scoreboard = ScoreBoard(self.type_name,
                                self.cuckoo_report.%The value you want to blacklist%)
								
		# Your code that will call the rules that you want to call
		
		for rule_key in RuleLoad.rules[self.type_name]:

			# Check if the rule key is valid
			if not RuleFactory.rule_key_exists(rule_key):
				logging.error("Invalid rule key \"%s\". Skipping it.")
				continue

			rule_data_set = RuleLoad.rules[self.type_name][rule_key]
			
            # Ask the factory to create a rule object for the given key
            rule = RuleFactory.get_rule_for_key(rule_key, self.cuckoo_report,
                                                scoreboard, rule_data_set)
		
		# Check the total score in the ScoreBoard object
		if self.blacklist_if_blacklistable(scoreboard):
			logger.info("Blacklist score reached."
				 " Adding md5 hash %s to blacklist",
				 self.%Value you want to blacklist%)
		
For a full example of an implemented type, see dfi.blacklist.types.HashType

After creating a new type, you need to add it to a type key name/class matching dictionary in
dfi.rule.TypeFactory.

 .. code-block:: python
    :linenos:

	from dfi.blacklist.types.SomeNewType import SomeNewType

	class TypeFactory(object):

		TYPE_CLASS_MATCH = {
			"your_type": SomeNewType
		}

As a last step you must add your type to dfi.task.BlacklistingTask.
In this class, add your line at this spot:

 .. code-block:: python
    :linenos:

	for report in reports:

		filtered_report = FilteredCuckooReport(report)

		hashtype = TypeFactory.get_type_for_key("hash",
												filtered_report)	
		yourtype = TypeFactory.get_type_for_key("yourtype",
												filtered_report)
		
		if hashtype.handle_rules() and yourtype.handle_rules():
                    processed_reports.append(filtered_report.sha256)

You can now start using your new type in rule files. A rule file can only be of 1 type and
this type should be entered in the 'info' section, like so:

 .. code-block:: javascript
    :linenos:
	
	"info": {
		"type": "yourtype"
	},
	"rules": {
		"string": [
		{
			"rules": ["BuildTime", "Somevaluethatiswildcarded*", "!REGEX![0-9]"],
			"score": 0.2
		},
		{
			"rules": ["CRYPTOLOCKER"],
			"score": 1.5
		}
		]
	}
