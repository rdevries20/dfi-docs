Installation
============

This entire tutorial is based on Cuckoo, running on Ubuntu Server 16.04 LTS and VMWare ESXi 6.

Due to the use of different modules and software required for the entire Cuckoo installation, a script can be used to deal with the complete installation.

The script is located in the 'cuckoo-setup' folder in the root of the project and is called 'CuckooSetup.bash'

Execute the script on a base install of Ubuntu Server 16.04 LTS::

	$ sudo bash CuckooSetup.bash

The CuckooSetup script has various different modules. Each of these modules has a specific name and function.
This will allow the user to specify which modules is needed. By using these various modules, you will have a complete modular setup.

During the execution of this script you will be prompted to press “Enter” to install Suricata for 
internal network traffic capture off the malware analysis virtual machines (i.e. cuckoo1)

This scripts takes 15 minutes to complete.

The scripts will install the following pieces of software:

- Cuckoo 2.0-dev
	- Commit: 9d16025d5e6b264ce8db3aeccfc37b3dcedb7f43 
- Python 2.7.11
- Python PIP 8.1.2
- TCPDump 4.7.4
- PyDeep 0.4
- SSDeep 2.13
- Volatility 2.5
- Suricata 3.1RC1
- Suricata Updater 0.7
- Yara 3.4.0
- Yara Python 3.4.0
- Tesseract 3.04.01
- LibVirt 1.3.5
- ISC-DHCP-Server (Optional) 
- Various dependencies
