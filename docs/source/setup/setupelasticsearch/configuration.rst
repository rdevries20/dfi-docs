Configuration
=============

The installation and configuration is quite extensive, therefor it is divided in six sections that are divided in various subsections. 
This will keep you in control of the environment creation and able to make changes matching your own setup.


Change cluster name
-------------------

Open the elasticsearch.yml file and set the cluser.name to DFI-Cluster. This will will clearify the cluster name::

	$ sudo vi /etc/elasticsearch/elasticsearch.yml

::

	[Cluster]
	cluster.name: DFI-Cluster

Change node name
----------------

Open the elasticsearch.yml file and set the node.name to DFI-Node-1. This will make it easier to 
see which node is connected to which cluster::

	$ sudo vi /etc/elasticsearch/elasticsearch.yml

::

	[Node]
	node.name: DFI-Node-1


Enable Node master
------------------

Open the elasticsearch.yml file and add the line node.master: true.
By enabling the node.master, this will appoint the node as the manager of the cluster::

	$ sudo vi /etc/elasticsearch/elasticsearch.yml

::

	[Node]
	node.master: true

Enable Node data
----------------

Open the elasticsearch.yml file and set the node.data: true. 
The data nodes that have this option enabled hold data and perform data related operations::

	$ sudo vi /etc/elasticsearch/elasticsearch.yml

::

	[Node]
	node.data: true

Enable memory lock
------------------

Open the elasticsearch.yml file and uncomment the bootstrap.mlockall: true. 
This will allow ElasticSearch to allocate memory from the startup::

	$ sudo vi /etc/elasticsearch/elasticsearch.yml

::

	[Memory]
	bootstrap.mlockall: true

Increase max query size
-----------------------

Open the elasticsearch.yml file and add the line max_query_size: 100000.

Because our various scripts need to interact with ElasticSearch, we sometimes need to retrieve a huge amount of documents, 
by setting the max_query_size to 100000 this will increase the maximum number of documents that will be downloaded from ElasticSearch in a single query::

	$ sudo vi /etc/elasticsearch/elasticsearch.yml
	
::

	max_query_size: 100000
	
Change bound IP
---------------

Open the elasticsearch.yml file and set the network.host to 0.0.0.0 or the external IP of the server.

The ElasticSearch server needs to be reachable over the network.

Changing the listen IP to 0.0.0.0 will allow ElasticSearch to be reachable via all interfaces::

	$ sudo vi /etc/elasticsearch/elasticsearch.yml

::

	[Network]
	network.host: 0.0.0.0

Allow cross-origin resource sharing
-----------------------------------

Open the elasticsearch.yml file and set http.cors.enabled: true. If this line is not present, add it.

Because we are using an external ElasicSearch cluster/node, we need to enable Cross-origin resource sharing.
To make sure our traffic doesn’t get blocked::

	$ sudo vi /etc/elasticsearch/elasticsearch.yml

::

	http.cors.enabled: true
