---
title: "Multi Data Center Replication: With NAT"
project: riakee
version: 1.4.0+
document: cookbook
toc: true
audience: intermediate
keywords: [mdc, repl, nat]
---

Riak Enterprise Advanced Replication supports replication of data on networks that use static NAT.

This can be used for replicating data over the internet where servers have both internal and public IP addresses (see [[Riak REPL SSL|Multi Data Center Replication SSL New]] if you replicate data over a public network).

### Requirements:
In order for Replication to work on a server configured with NAT, the NAT addresses must be configured **statically**.


### Configuration

NAT rules can be configured at runtime, from the command line.

* **nat-map show**

	Shows the current NAT mapping table

* **nat-map add \<externalip\>[:port] \<internalip\>**
	
	Adds a NAT map from the external IP, with an optional port, to an internal IP. The port number refers to a port that is automatically mapped to the internal  `cluster_mgr` port number.

* **nat-map del \<externalip\>[:port] \<internalip\>**

	Deletes a specific NAT map entry.

#### Applying changes at runtime: 

* Realtime NAT replication changes will be applied once realtime is stopped and started using the following command:

	* **riak-repl realtime stop <clustername>**
	* **riak-repl realtime start <clustername>**

* Fullsync NAT replication changes will be applied on the next run of a fullsync, or you can stop and start the current fullsync.

	* **riak-repl fullsync stop <clustername>**
	* **riak-repl fullsync start <clustername>**
	

### Example


* Cluster_A is the **source** of replicated data.

* Cluster_B and Cluster_C are the **sinks** of the replicated data.

##### **Cluster_A setup**

A node from Cluster_A is setup with a single **internal** IP address: 

  * *192.168.1.20*

##### **Cluster_B setup**

A node from Cluster_B will be configured as follows:

  * *192.168.1.10* (internal)
  * *50.16.238.123:5555* (public) 

	In this example, the `cluster_mgr` port number is the default of *9080*, while the configured NAT port listens on *5555*.

##### **Cluster_C setup**

A node from Cluster_C is setup with *static NAT*, configured with the following IP addresses:

  * *192.168.1.10* (internal)
  * *50.16.238.200:5566* (public)

	In this example, the `cluster_mgr` port number is the default of *9080*, while the configured NAT port listens on *5566*.


```   
	# on any node of Cluster_A
	riak-repl clustername Server_A

	# on any node of Cluster_B
	riak-repl clustername Server_B

	# on any node of Cluster_C
	riak-repl clustername Server_C

	# on a node of Cluster_C with the following IP's:
	nat-map add 50.16.238.123:5555 192.168.1.10

	# Connect replication from Cluster_A to Cluster_B:
	# on any node of Cluster_A
	riak-repl connect 50.16.238.123:5555
	# Cluster_B will receive a connection from Cluster_A at the public IP:port combo *50.16.238.123:5555*

	# Connect replication from Cluster_A to Cluster_C:
	# on any node of Cluster_A
	riak-repl connect 50.16.238.200:5566
	# Cluster_C will receive a connection from Cluster_A at the public IP:port combo *50.16.238.200:5566*
	
	# on any node from Cluster_A
	riak-repl realtime enable Cluster_B
	riak-repl realtime enable Cluster_C
	
	riak-repl realtime start Cluster_B
	riak-repl realtime start Cluster_C
```
