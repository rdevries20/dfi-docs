Configuration
=============

The installation and configuration is quite extensive, therefor it is divided in six sections that are divided in various subsections.
This will keep you in control of the environment creation and able to make changes matching your own setup

Cuckoo
------

**Enable ESXi**

Depending on the setup of your own environment you can changes values where needed.
Edit the esx.conf file and change the DSN URL to the correct ESXi server IP.
Change the username and password to the correct credentials for the ESXi hypervisor.
Next, change the interface for network traffic to the secondary interface, we have two interfaces ens160 and ens192 where ens192 our second interface is.

During this setup the format for naming the machines will be cuckoo<instance number>.

::

	$ sudo vi /etc/cuckoo/conf/esx.conf

The machine info will look somewhat like the following::

	[cuckoo1]
	label = cuckoo1
	platform = windows
	snapshot = Snapshot
	ip = 192.168.56.2

Next, we will edit the cuckoo.conf file, and will set the machinery to esx::

	$ sudo vi /etc/cuckoo/conf/cuckoo.conf

::
	
	machinery = esx

**Enable Suricata**

Open the file processing.conf and enable Suricata.
Set enabled to yes under the Suricata section::

	$ sudo vi /etc/cuckoo/conf/processing.conf

::

	[suricata]
	enabled = yes
	
**Enable MongoDB**

Open the reporting.conf file and set enable to yes under the MongoDB section::

	$ sudo vi /etc/cuckoo/conf/reporting.conf
	
::

	[mongodb]
	enabled = yes

Keep the file opened for the next step.

**Disable JSON API calls**

Open the reporting.conf file and set no to calls in the jasondump section::

	[jsondump]
	calls = no

**Enable screenshot**

To enable the automatic screenshot function edit the processing.conf file and set enabled to yes in the screenshot section::

	$ sudo vi /etc/cuckoo/conf/processing.conf

::

	[screenshot]
	enabled = yes

**Update signatures & agents**

To update the various signatures and agents of Cuckoo to the most recent versions,
execute the following commands from the utils directory in the cuckoo folder::

	$ cd /etc/cuckoo/utils
	  sudo python community.py -wafb monitor
	  sudo python community.py -wafb 2.0
	  sudo python community.py --signatures –force
	  sudo python community.py -a

**Using screen**

To allow Cuckoo to run simultaneous alongside the API and the WEB interface,
screen will be used. The reason we are using screen is so you can view the output of the programs as they happen,
for debugging purposes::

	$ sudo vi /etc/rc.local

::

	screen -S cuckooPY -d -m /etc/cuckoo/cuckoo.py
	screen -S cuckooAPI -d -m /etc/cuckoo/utils/api.py --host 0.0.0.0 --port 8090

These will allow the main Cuckoo program to run in a separate screen session, the same goes for the API.

The WEB interface has some issues when dealing with the templates that Django uses.
Therefor a session has to be created by hand::

	$ sudo screen -S cuckooWEB
	  cd /etc/cuckoo/web
	  python manage.py runserver 0.0.0.0:8080

After those commands have been issued, and the web interface is running the screen can be detached.

Enable NAT
----------

**Edit rc.local**

Because we use two interfaces, one with internet and one without internet,
we need to NAT the interfaces to allow internet to the virtuals controlled by the host.
The interface ens192 is the secondary interface whereas ens160 is the primary interface that has the internet connection.
To do this, edit the rc.local file and add the following lines::

	$ sudo vi /etc/rc.local

::

	sudo ifconfig ens192 192.168.56.1 netmask 255.255.255.0 up
	sudo iptables -A FORWARD -o ens160 -i ens192 -s 192.168.56.0/24 -m conntrack --ctstate NEW -j ACCEPT
	sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
	sudo iptables -A POSTROUTING -t nat -j MASQUERADE
	sudo sysctl -w net.ipv4.ip_forward=1


Suricata
--------

**Copy default Suricata configuration for Cuckoo**

To create a Suricata rule to capture the Cuckoo traffic execute the following command::

	$ sudo cp /etc/suricata/suricata.yaml /etc/suricata/suricata-cuckoo.yaml

**Create rule file**

To create a Suricata rule to capture the Cuckoo traffic execute the following command::

	$ echo "alert http any any -> any any (msg:\"FILE store all\"; filestore; noalert; sid:15; rev:1;)"  | sudo tee /etc/suricata/rules/cuckoo.rules


**Edit Suricata configuration for Cuckoo**

Edit Suricata-cuckoo.yaml::

	$ sudo vi /etc/Suricata/Suricata-cuckoo.yaml
	
Uncomment these::

	#HOME_NET: "any"
	#EXTERNAL_NET: "any"

Add the following under 'rule-files:'::

	rule-files:
	– cuckoo.rules


Set the following options in the yaml file::

	– fast: 
		enabled: no

	- file-store: 
		enabled: yes

	- file-store:
		force-md5: yes

	– file-log:
		enabled: yes

	reassembly:
		depth: 0

	libhtp
		request-body-limit: 0
		response-body-limit: 0


**Suricata Updater**

Create cronjob for updating
To keep Suricata up to date, the updater should run at every hour. To do this automatically, edit the crontab::

	$ sudo crontab -e

Add the following line to update every hour::

	0 * * * * /usr/sbin/etupdate

**Create DHCP range**

Open the dhcpd.conf file to configure the dhcp settings::

	$ sudo vi /etc/dhcp/dhcpd.conf

Comment the following lines::

	option domain-name
	option domain-name-servers

Create a new range at the bottom of the file::

	subnet 192.168.56.0 netmask 255.255.255.0 {
	  range 192.168.56.5 192.168.56.100;
	  option domain-name-servers 8.8.8.8;
	  option subnet-mask 255.255.255.0;
	  option routers 192.168.56.1;
	  default-lease-time 600;
	  max-lease-time 7200;
	}

**Create MAC reservation**

To make deploying easier, we can create MAC reservations. 
To create a reservation, edit the dhcpd.conf file, fill in the right instance numbers and paste the configuration lines at
the end of the file, under the subnet range section::

	$ sudo vi /etc/dhcp/dhcpd.conf

::

	host cuckoo<instance id> {
		hardware ethernet <MAC ADDRESS>;
		fixed-address 192.168.56.<number>;
	}
	
**ISC-DHCP-Server restart**

If the DHCP server needs to be restarted to introduce the changes, execute the following command::

	$ /etc/init.d/isc-dhcp-server restart


Deploying on ESXi
-----------------

**Duplicate Cuckoo-VM**

Log onto the ESXi server using the web interface or a different VMWare client. 
Browse the datastore and create a new folder with the name of the new machine for example cuckoo2.

Then browse to the content of the Cuckoo base VM and copy the content of that folder and paste them in the newly created folder.
Right click the “\*.vmx” file and select “Add to inventory”. Select the appropriate name. When starting the machine for the first time, 
you will be asked if you moved or copied it. Select “**I copied it**”

Note: The name for submitting samples to Cuckoo. Also make sure you run it after copying it. 
This resets all unique identifiers like MAC and UUID.

**Creating a new virtual machine**

When creating a new virtual machine, there are a few steps that need to be taken to make sure Cuckoo can work with it.

When creating a new machine, set the network adapter to the “**Internal Cuckoo Network**” and make sure you note the associated MAC address and remember it. 

Look at the “Adding a new machine to Cuckoo” and the “Adding the new machine to DHCP” to enable it, so it can be used.

Registering new virtual machine
-------------------------------

**Adding a new machine to Cuckoo**

To add a new machine to Cuckoo it must be added to the esx.conf file located in the conf directory of the Cuckoo folder. 
The machines tag must be updated with the new machine. At the bottom of the file the following configuration must be placed::

	$ sudo vi /etc/cuckoo/conf/esx.conf

The machine info will look somewhat like the following::

	[cuckoo<id>]
	label = cuckoo<id>
	platform = windows
	snapshot = Snapshot
	ip = 192.168.56.<number>

**Adding the new machine to DHCP**

When a new machine has been added, the machine also needs to be configured to the DHCP server, if the choice was made to use it.
The MAC address can be found via the settings menu of the virtual machine via various ESXi clients.
After filling in the numbers, edit the dhcpd.conf file in the dhcp directory. Add the lines at the end of the file.
When ISC-DHCP-Server is reloaded, the changes take effect and the virtual machine has network connection::

	$ sudo vi /etc/dhcp/dhcpd.conf

::

	host cuckoo<instance id> {
		hardware ethernet <MAC ADDRESS>;
		fixed-address 192.168.56.<number>;
	}

Restart the DHCP server::

	$ /etc/init.d/isc-dhcp-server restart

**Enabling Cuckoo agent**

When a new machine has been deployed, and the IP settings have been set either by static or using DHCP. Reboot the virtual machine. 
After the reboot has been completed check the IP address, if it is what it should be create a snapshot with the name “**Snapshot**”. 
This way the agent running on the machine has been associated with the correct at, allowing the Cuckoo host and agent to communicate with each other.
