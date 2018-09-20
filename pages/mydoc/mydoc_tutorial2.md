---
title: Tutorial 2
keywords: documentation theme, jekyll, technical writers, help authoring tools, hat replacements
last_updated: September 20, 2018
tags: [Overview]
summary: "This is a test page!"
sidebar: mydoc_sidebar
permalink: mydoc_tutorial2.html
folder: mydoc
---

# How to configure RFC 2544 Throughput/Latency over NGPF
This tutorial shows you how to configure RFC 2544 Throughput/Latency over NGPF.

## Step 1: Start Wish Console
To start a Wish Console session, do the following:
* Click Start.
* Click All Programs and navigate to the IxNetwork build.
* Click Wish Console for IxNetwork. 
The Wish Console window appears.
<div><img src="{{ "/images/IxN.png" | absolute_url }}" alt="github octocat" style="width:100%;" ></div>

## Step 2: Define ports
To define the ports, run the following code snippet:

```python
namespace eval ::py {
	    #client domain name or ip address
	    set ixTclServer PC-name 
	    #client Tcl port
	    set ixTclPort   8009    
	    #chassis ip address/domain name ; card number ; port number	
	    set ports       {{192.168.1.1 1 1}} {{192.168.1.1 1 2}} 
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

## Step 6: Run the Quicktest
Run the Quicktest using the following code snippet:

```python
set FAILED 1
	set PASSED 0
	

	puts "Step 2 -> Add virtual port"
	ixNet add [ixNet getRoot] vport
	ixNet add [ixNet getRoot] vport
	ixNet add [ixNet getRoot] topology
	ixNet add [ixNet getRoot] topology
	ixNet commit
	

	puts "Step 3  -> Add topologies"
	set topo1 [lindex [ixNet getList [ixNet getRoot] topology] 0]
	set topo2 [lindex [ixNet getList [ixNet getRoot] topology] 1]
	set vPorts [ixNet getList [ixNet getRoot] vport]
	

	set vport1 [lindex $vPorts 0]
	set vport2 [lindex $vPorts 1]
	

	puts "Step 4  -> Add device groups"
	ixNet add $topo1 deviceGroup
	ixNet add $topo2 deviceGroup
	ixNet commit
	

	set dev1 [ixNet getList $topo1 deviceGroup]
	set dev2 [ixNet getList $topo2 deviceGroup]
	

	puts "Step 5 -> Add values for deviceGroup and Topologies"
	ixNet setA $dev1 -multiplier 5
	ixNet setA $dev2 -multiplier 5
	ixNet setAttr $topo1 -name "ipv4_topo1"
	ixNet setAttr $topo2 -name "ipv4_topo2"
	ixNet commit
	

	puts "Step 6 -> Add mac to device group"
	ixNet add $dev1 ethernet
	ixNet add $dev2 ethernet
	ixNet commit
	

	set mac1 [ixNet getList $dev1 ethernet]
	set mac2 [ixNet getList $dev2 ethernet]
	

	puts "Step 7  -> Add ipv4 to device group"
	ixNet add $mac1 ipv4
	ixNet add $mac2 ipv4
	ixNet commit
	

	set ip1 [ixNet getList $mac1 ipv4]
	set ip2 [ixNet getList $mac2 ipv4]
	

	puts "Step 8 -> Add virtual ports to topology"
	set tPort1 [lindex [ixNet getList $topo1 port] 0]
	set tPort2 [lindex [ixNet getList $topo2 port] 0]
	

	ixNet setAttr $topo1 -vports $vport1
	ixNet setAttr $topo2 -vports $vport2
	ixNet commit
	

	puts "Step 9 -> Setting multiple values for ipv4 addresse"
	ixNet  setMultiAttr [ixNet getAttr $ip1 -address]/counter -start 192.168.1.2 -step 0.0.0.1 -direction increment
	ixNet  setMultiAttr [ixNet getAttr $ip2 -address]/counter -start 192.168.2.2 -step 0.0.0.1 -direction increment
	ixNet  setMultiAttr [ixNet getAttr $ip1 -gatewayIp]/counter -start 192.168.2.2 -step 0.0.0.1 -direction increment
	ixNet  setMultiAttr [ixNet getAttr $ip2 -gatewayIp]/counter -start 192.168.1.2 -step 0.0.0.1 -direction increment
	ixNet  setMultiAttr [ixNet getAttr $ip1 -resolveGateway]/singleValue -value true
	ixNet  setMultiAttr [ixNet getAttr $ip2 -resolveGateway]/singleValue -value true
	ixNet commit
	

	puts "Wait 5 sec"
	after 5000
	

	puts "Step 10 -> Creating Traffic Item"
	set trafficItem [ixNet add /traffic trafficItem]
	ixNet setMultiAttribute $trafficItem \-name "RFC2544Traffic" \-roundRobinPacketOrdering false \-routeMesh oneToOne \-trafficType ipv4\
	

	set endpointSet [ixNet add $trafficItem endpointSet]
	ixNet setMultiAttribute $endpointSet \-sources $topo1 \-destinations $topo2
	ixNet setMultiAttribute $trafficItem /tracking \-trackBy [list flowGroup0 trackingenabled0] 
	ixNet commit
	

	puts "Step 11  -> Creating QT"
	set test [ixNet add /quickTest "rfc2544throughput"]
	ixNet commit
	

	set test [lindex [ixNet remapIds $test] 0]
	ixNet setMultiAttribute $test/testConfig \-frameSizeMode fixed \-loadType unchanged \-imixTrafficType UNCHANGED 
		
	set trafficSelection [ixNet add $test "trafficSelection"]
	ixNet setMultiAttribute $trafficSelection \-id $trafficItem \-isGenerated false
	ixNet commit
```

## Step 7: Assign ports
To assign ports, run the following:

```python
set vPorts [ixNet getList [ixNet getRoot] vport]
	puts "Step 12 -> Assigning ports to $vPorts"
	::ixTclNet::AssignPorts $py::ports {} $vPorts force
	puts "Done"  
```

## Step 8: Start all Protocols
To start all protocols, run the following code snippet:

```python
puts "Step 13 -> Starting All Protocols"
	ixNet exec startAllProtocols
	puts "Wait 10sec for protocols to start"
	after 10000
```

## Step 9: Apply QuickTest
To apply the QuickTest, run the following code snippet:

```python
puts "Step 14 -> Apply QT"
	ixNet execute apply $test
```

## Step 10: Run QuickTest
To run the QuickTest, run the following code snippet:

```python
puts "Step 15 -> Starting QT"
	if {[catch {ixNet exec start $test} errMsg ]} {
	    error "start QT failed : $errMsg"
	    return $FAILED
	}
	while {true} {
	

		set progress [ixNet getA $test/results -progress]
		if { [string length $progress ] > 3 } {
		    puts $progress  
		}
		after 5000
		if {[ixNet getA $test/results -isRunning] == "false"} {
		    break
		}
	    }
	puts "Test Finished"
``` 
## Step 11: Test Run Status
To test the status of the QuickTest iteration, run the following code snippet:

```python
puts "Step 16 -> Test run status"
	

	if {[ixNet getA $test/results -result] == "fail"} {
	    puts "The test has failed !"
	    return $FAILED
	    } else {
		puts "The test has passed"
		}
```
## Step 12: Cleaning up the client
To clean up the IxNetwork client, run the following code snippet:

```python
puts "Step 17 -> Cleaning up the client: "
	puts " -> Stopping protocols"
	ixNet exec stop $test
	

	after 5000
	

	puts "->Performing New Config"
	ixNet exec newConfig
```



<script src="https://utteranc.es/client.js"	
		repo="Sahana84/IxNetworkAPIDoc"
		branch="master/gh-pages"
		issue-term="url"
		async>
		</script>
