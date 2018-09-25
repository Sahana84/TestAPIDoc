---
title: Tutorial 1
keywords: documentation theme, jekyll, technical writers, help authoring tools, hat replacements
last_updated: September 20, 2018
tags: [Overview]
summary: "This is a test page!"
sidebar: mydoc_sidebar
permalink: mydoc_tutorial1.html
folder: mydoc
---


# How to configure 10 Ethernet sessions on each of the two ports
This tutorial shows you how to configure 10 Ethernet sessions on each of the two ports.
Follow this tutorial to set up IxNetwork API and to use the Wish Console to proceed with your configuration.

## Step 1: Start Wish Console
To start a Wish Console session, do the following:
* Click Start.
* Click All Programs and navigate to the IxNetwork build.
* Click Wish Console for IxNetwork. 
The Wish Console window appears.
<div><img src="{{ "/images/IxN.png" | absolute_url }}" alt="github octocat" style="width:100%;" ></div> 
 
## Step 2: Define ports
To define the ports on which the Ethernet sessions will be configured, run the following code snippet:

```python
namespace eval ::py {
	     set ixTclServer 10.212.111.211
	     set ixTclPort   8009
	     set ports       {{10.212.111.180 3 1}}{{10.212.111.180 4 1}}
	}
  
  ```

## Step 3: Source the IxNet library
To source the IxNet library, type the following command:

```python
package req IxTclNetwork
```

## Step 4: Connect to IxNet client
To connect to IxNet client, run the following command:

```python
ixNet connect $::py::ixTclServer -port $::py::ixTclPort -version 8.50
```

## Step 5: Clean up IxNetwork
To clean up IxNetwork, run the following:

```python
puts "Cleaning up IxNetwork..."
	ixNet exec newConfig
```

## Step 6: Add ports to configuration
To add ports to the configuration, run the following:

```python
puts "Adding ports to configuration"
	set root [ixNet getRoot]
	ixNet add [ixNet getRoot] vport
	ixNet add [ixNet getRoot] vport
	ixNet commit
	set vPorts [ixNet getList [ixNet getRoot] vport]
	set vport1 [lindex $vPorts 0]
	set vport2 [lindex $vPorts 1]
```

## Step 7: Configure Ethernet
To configure Ethernet, run the following code snippet:

```python
puts "Add topologies"
	ixNet add [ixNet getRoot] topology
	ixNet add [ixNet getRoot] topology
	ixNet commit
	

	set topo1 [lindex [ixNet getList [ixNet getRoot] topology] 0]
	set topo2 [lindex [ixNet getList [ixNet getRoot] topology] 1]
	

	puts "Add ports to topologies"
	ixNet setA $topo1 -vports $vport1
	ixNet setA $topo2 -vports $vport2
	ixNet commit
	

	puts "Add device groups to topologies"
	ixNet add $topo1 deviceGroup
	ixNet add $topo2 deviceGroup
	ixNet commit
	

	set dg1 [ixNet getList $topo1 deviceGroup]
	set dg2 [ixNet getList $topo2 deviceGroup]
	

	puts "Add Ethernet stacks to device groups"
	ixNet add $dg1 ethernet
	ixNet add $dg2 ethernet
	ixNet commit
	

	set mac1 [ixNet getList $dg1 ethernet]
	set mac2 [ixNet getList $dg2 ethernet]
```	

## Step 8: Assign ports
To assign ports, run the following:

```python
set vPorts [ixNet getList [ixNet getRoot] vport]
	puts "Assigning ports to $vPorts"
	::ixTclNet::AssignPorts $py::ports {} $vPorts force
```

## Step 9: Start all Protocols
To start all protocols, run the following code snippet:

```python
puts "Starting All Protocols"
	ixNet exec startAllProtocols
	puts "Sleep 30sec for protocols to start"
	after 30000
```

{% include read_time.html %}

<script src="https://utteranc.es/client.js"	
		repo="Sahana84/IxNetworkAPIDoc"
		branch="master/gh-pages"
		issue-term="url"
		async>
		</script>
		
