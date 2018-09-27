---
title: Tutorial 3
keywords: documentation theme, jekyll, technical writers, help authoring tools, hat replacements
last_updated: September 20, 2018
tags: [Overview]
summary: "This is a test page!"
sidebar: mydoc_sidebar
permalink: mydoc_tutorial3.html
folder: mydoc
---

# How to create a custom view for statistics
This tutorial configures 10 IPv4 sessions on each of the two ports, adds a traffic Item that uses IPv4 endpoints, sends traffic, and creates a new Advanced Filtering View for IPv4 statistics with Device Group Level Grouping.

## Step 1: Start Wish Console
To start a Wish Console session, do the following:
1.	Click Start.
2.	Click All Programs and navigate to the IxNetwork build.
3.	Click Wish Console for IxNetwork. 
The Wish Console window appears.
<div><img src="{{ "/images/IxN.png" | absolute_url }}" alt="github octocat" style="width:100%;" ></div>

## Step 2: Define ports
To define the ports, run the following code snippet:

```python
namespace eval ::py {
	     set ixTclServer 10.205.11.28
	     set ixTclPort   8078
	     set ports       {{10.205.11.22 8 9}} {{10.205.11.22 8 10}}
	}
```

## Step 3: Source the IxNet library
To source the IxNet library, type the following command:

```python
package req IxTclNetwork
## Step 4: Connect to IxNet client
To connect to IxNet client, run the following command:
ixNet connect $::py::ixTclServer -port $::py::ixTclPort -version 8.50
```

## Step 5: Clean up IxNetwork
To clean up IxNetwork, run the following code snippet:

```python
puts "Cleaning up IxNetwork..."
	ixNet exec newConfig
```

## Step 6: Adding ports to configuration
To add ports, run the following code snippet:

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

## Step 7: Configure IPv4
To configure IPv4, run the following code snippet:

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
	

	puts "Add ipv4 stacks to Ethernets"
	ixNet add $mac1 ipv4
	ixNet add $mac2 ipv4
	ixNet commit
	

	set ipv4_1 [ixNet getList $mac1 ipv4]
	set ipv4_2 [ixNet getList $mac2 ipv4]
	

	puts "Setting multi values for ipv4 addresses"
	ixNet setMultiAttribute [ixNet getAttribute $ipv4_1 -address]/counter -start 22.1.1.1 -step 0.0.1.0
	ixNet setMultiAttribute [ixNet getAttribute $ipv4_1 -gatewayIp]/counter -start 22.1.1.2 -step 0.0.1.0
	ixNet setMultiAttribute [ixNet getAttribute $ipv4_1 -resolveGateway]/singleValue -value true
	ixNet setMultiAttribute [ixNet getAttribute $ipv4_2 -address]/counter -start 22.1.1.2 -step 0.0.1.0
	ixNet setMultiAttribute [ixNet getAttribute $ipv4_2 -gatewayIp]/counter -start 22.1.1.1 -step 0.0.1.0
	ixNet setMultiAttribute [ixNet getAttribute $ipv4_2 -resolveGateway]/singleValue -value true
	ixNet commit
``` 

## Step 8: Create traffic for IPv4
To create traffic for IPv4, run the following code snippet:

```python
puts ''
	puts "Creating Traffic for IPv4"
	

	ixNet add [ixNet getRoot]/traffic trafficItem
	ixNet commit
	set ti1 [lindex [ixNet getList [ixNet getRoot]/traffic trafficItem] 0]
	ixNet setMultiAttribute $ti1                \
	        -name                 "Traffic IPv4"\
	        -trafficType          ipv4          \
	        -allowSelfDestined    False         \
	        -trafficItemType      l2L3          \
	        -mergeDestinations    True          \
	        -egressEnabled        False         \
	        -srcDestMesh          manyToMany    \
	        -enabled              True          \
	        -routeMesh            fullMesh      \
	        -transmitMode         interleaved   \
	        -biDirectional        True          \
	        -hostsPerNetwork      1
	ixNet commit
	ixNet setAttribute $ti1 -trafficType ipv4
	ixNet commit
	ixNet add $ti1 endpointSet              \
	        -sources             $ipv4_1    \
	        -destinations        $ipv4_2    \
	        -name                "ep-set1"  \
	        -sourceFilter        {}         \
	        -destinationFilter   {}
	ixNet commit
	ixNet setMultiAttribute $ti1/configElement:1/frameSize \
	        -type        fixed                             \
	        -fixedSize   128
	ixNet setMultiAttrs $ti1/configElement:1/frameRate   \
	        -type       percentLineRate                         \
	        -rate       10                                      \
	

	ixNet setMultiAttrs $ti1/configElement:1/transmissionControl \
	    -duration               1                                   \
	    -iterationCount         1                                   \
	    -startDelayUnits        bytes                               \
	    -minGapBytes            12                                  \
	    -frameCount             10000                               \
	    -type                   fixedFrameCount                     \
	    -interBurstGapUnits     nanoseconds                         \
	    -interBurstGap          0                                   \
	    -enableInterBurstGap    False                               \
	    -interStreamGap         0                                   \
	    -repeatBurst            1                                   \
	    -enableInterStreamGap   False                               \
	    -startDelay             0                                   \
	    -burstPacketCount       1
	

	ixNet setMultiAttribute $ti1/tracking -trackBy {{sourceDestValuePair0}}
	ixNet commit
```

## Step 9: Assign ports
To assign ports, run the following code snippet:

```python
  set vPorts [ixNet getList [ixNet getRoot] vport]
	puts "Assigning ports to $vPorts"
	::ixTclNet::AssignPorts $py::ports {} $vPorts force
  ```

## Step 10: Start all protocols
To start all protocols, run the following code snippet:

```python
puts "Starting All Protocols"
	ixNet exec startAllProtocols
	puts "Sleep 30sec for protocols to start"
	after 30000
```

## Step 11: Generate, apply, and start traffic
To generate, apply, and start traffic, run the following code snippet:

```python
ixNet exec generate $ti1
	ixNet exec apply [ixNet getRoot]/traffic
	ixNet exec start [ixNet getRoot]/traffic
	puts "Sleep 30sec to send all traffic"
	after 30000
	

	puts "#########################"
	puts "## Statistics Samples ##"
	puts "#########################"
	puts ""
```

## Step 12: Define function to get the view object using the view name
To define function to get the view object using the view name, run the following code snippet:

```python
proc getViewObject { viewName } {
	    set views [ixNet getList [ixNet getRoot]/statistics view]
	    set viewObj ""
	    set editedViewName "::ixNet::OBJ-/statistics/view:\"$viewName\""
	    if { [lsearch -regexp -inline $views $editedViewName] != "" } {
	        return [lsearch -regexp -inline $views $editedViewName]
	    }
	    return viewObj
	}
```	

## Step 13: Create advanced filter custom view
To create advanced filter custom view, run the following code snippet:

```python
proc createAdvFilCustomView { cvName protocol grLevel sortExpr } {
	    puts "- creating view $cvName, with protocol $protocol, grouping level $grLevel"
	    ixNet add [ixNet getRoot]/statistics view
	    ixNet commit
	

	    set mv [lindex [ixNet getList [ixNet getRoot] statistics] 0]
	    set view [lindex [ixNet getList $mv view] end]
	

	    ixNet setAttribute $view -caption $cvName
	    ixNet setAttribute $view -type layer23NextGenProtocol
	    ixNet setAttribute $view -visible True
	    ixNet commit
	

	    set view [lindex [ixNet getList $mv view] end]
```

## Step 14: Define function to get the view object using the view name
To define function to get the view object using the view name, run the following code snippet:

## Step 15: Add advanced filtering filter
To add advanced filtering filter, run the following code snippet:

```python
puts "\t - add advanced filtering filter ..."
	    set trackingFilter [ixNet add $view advancedCVFilters]
```

## Step 16: Set protocol for the filter
To set protocol for the filter, run the following code snippet:

```python
puts "\t - setting protocol $protocol for the filter."
	    ixNet setAttribute $trackingFilter -protocol $protocol
	    ixNet commit
```

## Step 17: Select the grouping level for the filter
To select the grouping level for the filter, run the following code snippet:

```python
puts "\t - selecting $grLevel for the filter grouping level."
	    ixNet setAttribute $trackingFilter -grouping $grLevel
	    ixNet commit
```

## Step 18: Add filter expression and filter sorting stats
To add filter expression and filter sorting stats, run the following code snippet:

```python
puts "\t - adding filter expression and filter sorting stats."
	    ixNet setAttribute $trackingFilter -sortingStats $sortExpr
	    ixNet commit
```

## Step 19: Set the filter
To set the filter, run the following code snippet:

```python
puts "\t - setting the filter."
	    set fil [lindex [ixNet getList $view layer23NextGenProtocolFilter] 0]
	    ixNet setAttribute $fil -advancedCVFilter $trackingFilter
	    ixNet commit
``` 

## Step 20: Enable the stats columns to be displayed for the view
To enable the stats columns to be displayed for the view, run the following code snippet:

```python
  puts "\t - enable the stats columns to be displayed for the view."
	    set statsList [ixNet getList $view statistic ]
	    foreach stat $statsList {
	        ixNet setAttribute $stat -enabled True
	    }
	    ixNet commit
```

## Step 21: Enable the view going and start retrieving stats
To enable the view going and start retrieving stats, run the following code snippet:

```python
puts "\t - enabling the view going and start retrieving stats."
	    ixNet setAttribute $view -enabled True
	    ixNet commit
	}
	

	set cvName     "Custom View - IPv4"
	set protocol   "IPv4"
	set grLevel    "Per Device Group"
	set sortExpr   "\[Device Group\] = desc"
```

## Step 22: Create the custom view for IPv4 NGPF
To create the custom view for IPv4 NGPF, run the following code snippet:

```python
puts "Create custom view for IPv4 NGPF"
	createAdvFilCustomView $cvName $protocol $grLevel $sortExpr
```

## Step 23: Refresh the created view
To refresh the created view, run the following code snippet:

```python
puts "Refreshing the new view"
	set newview [getViewObject $cvName]
	ixNet execute refresh $newview
```



<script src="https://utteranc.es/client.js"	
		repo="Sahana84/TestAPIDoc"
		branch="master/gh-pages"
		issue-term="url"
		async>
		</script>
