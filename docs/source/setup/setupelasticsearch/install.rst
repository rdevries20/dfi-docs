Installation
============

This entire tutorial is based on ElasticSearch, running on Ubuntu Server 16.04 LTS and VMWare ESXi 6.

Due to the use of different modules and software required for the entire installation, 
a script can be used to deal with the complete installation.

The script is called 'ElasticSearchSetup.bash' and can be found in the 'cuckoo-setup' directoy in the project
root directory.

Execute the script on a base install of Ubuntu Server 16.04 LTS::

	$ bash ElasticSearchSetup.bash

The ElasticSearchSetup script has various different modules. Each of these modules has a specific name and function.
This will allow the user to specify which modules is needed. By using these various modules, you will have a complete modular setup.

During the execution of this script you will be prompted to press “**Enter**” to agree with Oracle Java’s policy

The scripts will install the following pieces of software:

	- Python 2.7.11
	- Python PIP 8.1.2
	- Oracle Java 8
	- ElasticSearch 2.3.3
	- Elastic HQ 2.0.3
	- Various dependencies