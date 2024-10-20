---
aliases: 
publish: 
---

# Topic: OSPF

## Overview

It is a link state routing protocol because each routers originates and shares the states of its links with all other routers.
Each device builds it's own loop free topology for all networks.

Vectors carry pathogens and routes and each router modifies the route before sending it on.

Stub network information advertises a router's directly connected stub networks (networks with no neighbours) with a triple of (Router ID, Network ID, Cost).
# 5 OSPF Network Interface Types
When OSPF enabled the interface types it determines if message sent using unicast or multicast. The 5 network types can also be split into 2 broader categories stub or transit.
1. **Serial interfaces** are **P2P** (Point-to-point) - Uses 224.0.0.5 AllSPFRouters and unicast LSACKs
2. **Ethernet** are **Broadcast** networks. Note **DR** are elected **centralise messaging**. **All Hellos and DR messages** AllSPFRouters **224.0.0.5**. **Non DRs** send messages to DRs using AllDRRouters **224.0.0.6**.
3. Non-broadcast Multi-access (NBMA) networks - Manual neighbours, DR elected, all packets unicast
4. Point-to-multipoint networks - No DR all OSPF packets are multicast.
5. Virtual links - All OSPF packets are unicast

# Packets and Message Types
5 Types
1. **Hello**, Neighbor discovery Adjacency negotiation Adjacency keepalive
2. **DD** Database Description (DD), DB sync
3. **LSRs** Link State Request, DB sync
4. **LSAck** Link State Acknowledgement, DB sync
5. **LSU** Link State Update, DB sync & Flooding
![[Pasted image 20241012135959.png]]
# Neighbours
- 2 neighbours adjacent if they see their RID in Hello
- Fully adjacent until database synchronisation complete.

**Network Mask** is **prefix length of senders interface**
Hello Interval specifies, in seconds, the interval at which the originating router sends Hellos. RFC 2328 does not specify a default Hello interval, but it suggests a value of 10 seconds for LANs and 30 seconds for nonbroadcast networks such as X.25. Both Cisco Systems and Juniper Networks use the suggested default of 10 seconds for broadcast networks. However, the default for nonbroadcast networks varies: Cisco uses 30 seconds, whereas Juniper uses 120 seconds. The reason for the longer intervals on non-broadcast networks is an artifact of the days when it could be assumed that the links associated with such networks had less bandwidth to spare for Hello traffic.

Options specifies any optional capabilities the originating OSPF router might have. The Options field is described in Section 4.3.

Router Priority is used by the Designated Router election process.

RouterDeadInterval specifies the time that the originating router's neighbors should wait for a Hello before declaring it dead. As with the Hello interval, RFC 2328 does not specify a default value but suggests a value of four times the Hello interval. Both Cisco Systems and Juniper Networks use that suggested default.

Designated Router and Backup Designated Router are, like the Router Priority, used in the Designated Router election and maintenance mechanism. This mechanism is described in Section 4.4.

Neighbor lists the RIDs of the OSPF neighbors the originating router has received Hellos from on the subnet. This list is used in the three-way handshaking process before an OSPF adjacency is established.


OSPF neighbours talk multicast on broadcast and unicast on P2P.
OSPF routers only like neighbours and add them to their list when they're on the same:
- segment
- subnet for broadcast with DR enablement flag set
- mtu (setting in DBD field - no fragmentation)
- time
- auth
- area
- network type of which there are 5 and all types are either stub no neighbours or transit 2 or more neighbours .

Routers in area all share the same DB using using DBD, LSRs and LSAs packets.
If multiaccess network router with highest priority or RID becomes DR.
Routers build their own SPT when DB synched
Routers link changes it floods AllDR LSAs out of all interfaces if DB change or unicast LSA.

# Reliable Flooding
LSAs sent with a sequence number, incremented whenever new instance created.
LSAs sent with age of 0, incremented by 1 at each hop and each second when in LSDB.
Non originated LSAs age-out from LSDB at 3600s but originated LSAs are refreshed and flooded every 1800s.
LSAs in LSUs are reliable and confirmed with LSAck. 

Then when we want to create a massive OSPF network how do I choose my areas, setup my router roles, and what LSA types are allowed in my areas
Can I further refine my network and filter or summarise LSAs.
How do I protect OSPF network.
LSDB synchronised, route tables built, OSPF is a quiet protocol. Hellos exchanged, LSAs retransmitted every 30 minutes. No topology change = no SPT recalculation.

# SPF Algorithm
The SPF algorithm is run once to build the branches of the tree, which are the links to each node (router) in the area. The algorithm is then run a second time to add the leaves to the tree—the stub networks attached to each router. Contrary to popular belief, the SPF algorithm itself is not particularly processor intensive. It is the related processes, such as flooding and database maintenance, that burden the CPU. OSPF uses areas to reduce these adverse effects. It is links and stability in an area not the number of routers that defines how OSPF should be broken down into areas.  
### Key Takeaways:

1. **Type 1 and Type 2 LSAs**: The **SPF algorithm** runs primarily based on these LSAs, as they contain the full topology details within an area.
2. **Type 3 to 5 LSAs**:
    - **Type 3** (inter-area summary) and **Type 5** (external routes) don’t cause a full SPF recalculation, but they update the routing table to include external or inter-area routes. These LSAs are **not part of the intra-area SPF path selection**, but they influence the overall routing decisions across areas or outside the OSPF domain.
3. **SPF Path Selection Basis**: The phrase “**Type 1 and 2 LSAs are the basis of the SPF path selection**” means that the SPF algorithm calculates the intra-area routes using the information in Type 1 (Router) and Type 2 (Network) LSAs, as these describe the direct topology of the OSPF area.
    
In summary, while the SPF algorithm runs based on **Type 1 and 2 LSAs** to calculate intra-area paths, LSAs like **Type 3 to 5** are used to calculate routes to other areas or external networks, **without running the full SPF**.

---
# Path Selection
When an OSPF router examines the destination address of a packet, it takes the following steps to select
the best route: 
1. Select the route or routes with the most specific match to the destination address - route with the longest address mask.
2. Prune entries by eliminating less-preferred path types in the following order, with 1 being the most-preferred:
	1. Intra-area paths
	2. Inter-area paths
	3. E1 external paths
	4. E2 external paths

*NOTE*: Cisco's OSPF performs equal-cost load balancing
If multiple equal-cost, equal-path-type routes exist in the final set, OSPF will utilise them. By default,
Cisco's load balance over a maximum of four equal-cost paths but can increase to 6.
```
router ospf 1
maximum-paths 6
```

---

### Important Concepts Summary
Concepts:: **Concept 1**, **Concept 2**

---
# LSAs
The LSDB is built up from the 5 different LSA types.

It is worth noting the All OSPF packets begin with a 24 OSPF header. And LSAs can be seen in OSPF packets 3 types i.e. DBDs, LSAck and LSRs and all LSAs types use the same Link State Header format but the info in them varies depending on the LSA type.

Adding LSAs to DB cost CPU cycles and memory when there's thousands of routes so we need to understand how to read them. If you look below you can construct the network quite easily by reading the LSA in the LSDB.

## LSA Header
All LSAs have a header with the following fields:
**LSA type** there's 7 types, Cisco doesn't use type 6 
**Link State ID** the value varies on type but it describes the router it is connected to
**Advertising Router** Router ID of the router that originated the LSA. This could be router, DR, 
**Sequence**
**Age**

## LSA Type and Link State ID Field Summary
- Type 1 LSAs are *Router LSAs* and there will be one type 1 LSA for each router in teh area. Each router LSA contains a list of all the links a router is connected to. The link state id and advertised id will be the same, the *Router ID* that originated the type 1 LSA. 
- Type 2 LSAs are *Network LSAs* that describe the routers connected to a multiaccess segment. Only DRs send these LSAs. The link state id is the *IP address of the DR's interface.*
- Type 3 LSAs are *Summary LSAs* that describe a network outside of the area, the link state id is the *remote subnet address.* Type 3 LSAs are generated by an ABRs only.
- Type 4 LSAs are *ASBR-summary LSAs* that describes how to get to an ASBR so the link state id is *ASBRs Router ID* and the advertised ID is the ABR
- Type 5 LSAs are *As-External-Link LSAs*, the link state id is an *external network* and the advertising router is the ABRs Router ID 
- Type 7 LSAs are for NSSA areas only. They look similar to Type 5 LSAs.

Then there's what's in the LSA itself 
Link Type
1 Point-to-point connection to another router
2 Connection to a transit network
3 Connection to a stub network
4 Virtual link
## LSA Type 1 Self-Originate
From all routers and describes router links
![[Pasted image 20241008130936.png]]

## LSA Type 1
Lists all routers and the link states. Note all LSA type 1 will be connected to a stub or transit network no neighbours or 2 or more neighbours
`show ip ospf database router`
![[Pasted image 20241008125143.png]]
![[LSA Type 1s excalidraw]]
## LSA Type 2
From DR Routers Links
`show ip ospf database network`
From here the router has all the info to run build the SPT and place the best routes into the route table.
![[Pasted image 20241008125208.png]]
![[Excalidraw/LSA Type 2]]
## LSA Type 3
Generated by ABRs and describes route, metric, mask and cost outside of an area. So as everyone says calling them summaries is confusing 
`show ip ospf database summary`
![[Pasted image 20241008125252.png]]
If ABR generates type 3 from a neighbours type 1 new cost = to get to the neighbour + original cost
If ABR generates type 3 from area 0 type 3 new cost = to get to originating ABR + original cost
Advertising routers are changed to ABR
If routers in a non backbone area receives a type 3 LSA from another non-backbone area they will add them to their LSDB but an ABE will not resend them. This prevents non-backbone areas becoming transit between other non-backbone areas.
Area 23 can see area 34 routes but R2 will not send a type 3 onto area 12 for area 34 type 1 LSAs.
![[Pasted image 20241008155104.png]]
## LSA Type 4
Generated by ASBRs and describes a route to the ASBR
`show ip ospf database asbr-summary`
![[Pasted image 20241008125304.png]]
## LSA Type 5
Generated by ASBRs for routes outside of the OSPF domain and that are in the routers global route table. Note that the router will create a route to null0. This way if I created a summary for a network I didn't have a route for that traffic will get dropped. E.g. 192.168.0.0 255.255.0.0 summary for networks 192.168.0.0/20 and 192.168.128.0/20 then if one of those routes goes missing and the summary is still active the traffic will get dropped.
![[Pasted image 20241008125320.png]]
### Important Concepts Summary
Concepts:: **Concept 1**, **Concept 2**

---
# Area Types
We have different area types to limit the LSA types being advertised within that area. We can never filter type 1 and 2 so all types will share type 1 and type 2 LSAs.

There's really only 3 area types.
- There's 2 normal areas backbone and non-backbone
- There's stub (no-external) and a totally option (+no inter area)
- There's not-so-stubby and a totally option
- The stub and stub option matches the NSSA and stub option except 

| **Area**                         | **Restriction**                                                                                                             |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Normal (Backbone & Non-Backbone) | No Type 7 allowed                                                                                                           |
| Stub                             | No Type 5 AS-external LSA allowed                                                                                           |
| Totally Stub                     | No Type 3, 4 or 5 LSAs allowed except the default summary route                                                             |
| NSSA                             | No Type 5 AS-external LSAs allowed, but Type 7 LSAs that convert to Type 5 at the NSSA ABR can traverse                     |
| NSSA Totally Stub                | No Type 3, 4 or 5 LSAs except the default summary route, but Type 7 LSAs that convert to Type 5 at the NSSA ABR are allowed |
### Configuration

```
!
router ospf 1
! Backbone
network 192.168.0.0 0.0.255.255 area 0
! Non backbone
network 10.1.0 0.0.255.255 area 10
! Convert area type
area <area-id> virtual-link <router-id>
area <area-id> stub
area <area-id> nssa
area <area-id> 

```

---
# Summarisation
- Summarisation can only occur on ABRs and ASBRs and between areas.
- Summarisation is configured in the area where the links that need to summarised exist not in the neighbouring areas.
- This feature is all about creating summary type 3 or 5, for internal or external networks and as long as there is an active network within the summary range the border routers will send the summary LSAs underlying summaries for other areas. If there's multiple ABRs or ASBRs connected to the same upstream networks then create the summaries on all the ABRs, ASBRs. Cost can be used then to select the most preferred or leave then same for ECMP. When you summarise the ABR or ASBR it creates a new type 3 or type 5 LSAs, suppress other type 3 or 5 LSAs only if it has LSAs that fall within that range.

If you've only area 0 and no external networks then summarisation will not help to reduce the LSDB.

On ABR R1 connected to area 0 and 1, that has lots 10.10.0.0/16 subnets create summary LSA type 3 and suppress.
On Area 2 IR R2 you can see the summary route in the global table.

Note summary creates a null 0 route in the global route table so traffic to that network will be dropped and not picked up by a default route. This protects against traffic being sent to a default route.
```
router ospf 1
area 0 range 192.168.0.0 255.255.252.0
!
R1# show ip route
  O 192.168.0.0/24 [110/10] via 10.0.0.1, 00:12:33, FastEthernet0/0
  O 192.168.1.0/24 [110/10] via 10.0.0.1, 00:12:33, FastEthernet0/0
  O 192.168.0.0/22 is a summary, [110/20] via Null0

R2# show ip route ospf
O IA 192.168.0.0/22 [110/10] via R1, ...
!
R1# show ip ospf database summary 192.168.0.0

  OSPF Router with ID (1.1.1.1) (Process ID 1)
  
  Summary Net Link States (Area 1)
  
  LS age: 260
  Options: (No TOS-capability, DC)
  LS Type: Summary Links (Network)
  Link State ID: 192.168.0.0 (Summary Network Address)
  Advertising Router: 1.1.1.1
  LS Seq Number: 8000000A
  Checksum: 0x61FA
  Length: 28
  Network Mask: /22
  TOS 0 Metrics: 20

```

On ASBR for LSA type 5 summaries
```
router ospf 1
summary-address 10.10.0.0 255.255.0.0
!
Router# show ip ospf summary-address

Summary address 192.168.0.0, mask 255.255.252.0
  Advertised metric type: 2, metric: 10
  Null0 discard route installed
  External routes summarized: 3

Summary address 172.16.0.0, mask 255.255.248.0
  Advertised metric type: 2, metric: 20
  Null0 discard route installed
  External routes summarized: 5
```

- To verify summary addresses look at the LSDB of the areas connected to the 
---

## Route Filtering

- Using summaries on an ABR you can filter type 3 LSAs from being generated. This generates a type 3 in area 1 but 
```
router ospf 1
 area 1 range 192.168.0.0 255.255.252.0 not-advertise
```
- Or you can use a filter list and you can filter LSAs coming into or flowing out of an area. Probably best to filter as close to the source as possible.
- Filter lists give you more granular control and the same method can be used on the ABR or ASBRs
- The inbound area filter-list, applied for area X filters prefixes from **all areas sent to X area**.
- The outbound area filter-list, applied to area X will filter prefixes from **X area sent to all other areas**.
```
ip prefix-list FILTER_LSA3 deny 192.168.3.0/24
ip prefix-list FILTER_LSA3 permit 0.0.0.0/0 le 32
!
router ospf 1
 area 1 filter-list prefix FILTER_LSA3 in
```

- Type 5
```
router ospf 1
 summary-address 172.16.2.1 255.255.255.255 not-advertise
```

Here's a table summarizing the filtering options for OSPF Type 3 (inter-area routes) and Type 5 (external routes) LSAs, along with an explanation of the differences between distribute lists and filter lists.
Here's the updated table that includes information on whether each filtering method supports direction (in or out).

| **Filtering Method**          | **Type 3 Filtering** | **Type 5 Filtering** | **Direction** | **Description**                                                                                     |
|-------------------------------|----------------------|----------------------|----------------|-----------------------------------------------------------------------------------------------------|
| **Distribute List**           | Yes                  | Yes                  | In              | Controls the redistribution of routes based on access lists. Typically applied to OSPF route redistribution.                        |
| **Filter List**               | Yes                  | Yes                  | In/Out          | Similar to distribute lists but used within OSPF to control which LSAs are processed or advertised based on prefix lists.              |
| **Summary Address (not-advertise)** | Yes                  | No                   | Out             | Prevents specific summary addresses from being advertised to other areas.                          |
| **Route Filtering**           | Yes                  | Yes                  | In/Out          | Controls which routes are included in OSPF advertisements, can be based on metrics or prefixes.   |
| **Route Map**                 | Yes                  | Yes                  | In/Out          | More granular control over route selection and manipulation during redistribution.                 |

### Summary
- **Direction**:
  - **In**: Indicates that the method is used to filter routes entering OSPF.
  - **Out**: Indicates that the method is used to filter routes exiting OSPF.
  - **In/Out**: Indicates that the method can be applied to both incoming and outgoing routes. 

This table provides a comprehensive overview of the filtering methods for OSPF Type 3 and Type 5 LSAs, including their directional capabilities.
### Differences Between Distribute Lists and Filter Lists

- **Distribute List**:
    - Used to filter routes being redistributed into OSPF from other routing protocols or static routes.
    - Applied to the OSPF process and impacts the routes that enter the OSPF routing domain.
- **Filter List**:
    - Specifically designed for filtering LSAs within OSPF itself, affecting which LSAs are processed or advertised.
    - Applied directly to the OSPF configuration, impacting the internal OSPF LSDB.

### Summary
Both distribute lists and filter lists can be used for filtering routes, but they are used in different contexts. Distribute lists are typically about controlling the ingress of routes from other protocols into OSPF, while filter lists operate within the OSPF routing process to control which LSAs are considered for routing decisions.

Here are the configuration examples for each method, with brief explanations provided in comments:

```bash
! Distribute List Example (Type 3 and Type 5 Filtering)
router ospf 1
  distribute-list prefix PREFIX_FILTER in   # Filter routes entering OSPF from the routing table
  distribute-list prefix PREFIX_FILTER out  # Filter routes leaving OSPF and being advertised
!
ip prefix-list PREFIX_FILTER seq 10 deny 192.168.0.0/16
ip prefix-list PREFIX_FILTER seq 20 permit 0.0.0.0/0 le 32
```

```bash
! Filter List Example (Type 3 and Type 5 Filtering)
router ospf 1
  area 0 filter-list prefix PREFIX_FILTER in   # Filter LSAs entering the area from another area
  area 0 filter-list prefix PREFIX_FILTER out  # Filter LSAs leaving the area and advertised to other areas
!
ip prefix-list PREFIX_FILTER seq 5 deny 10.0.0.0/8
ip prefix-list PREFIX_FILTER seq 10 permit 0.0.0.0/0 le 32
```

```bash
! Summary Address (Not-advertise) Example (Type 3 Filtering)
router ospf 1
  area 1 range 192.168.0.0 255.255.252.0 not-advertise  # Summarize networks and prevent Type 3 LSA from being generated
```

```bash
! Route Filtering Example (Type 3 and Type 5 Filtering)
router ospf 1
  area 1 filter-list prefix PREFIX_FILTER in   # Apply a filter to the specific area
!
ip prefix-list PREFIX_FILTER seq 5 permit 192.168.0.0/16
```

```bash
! Route Map Example (Type 3 and Type 5 Filtering)
route-map OSPF_FILTER deny 10
  match ip address prefix-list PREFIX_FILTER
!
router ospf 1
  redistribute connected route-map OSPF_FILTER  # Apply a route map to the redistribution process to filter Type 5 LSAs
!
ip prefix-list PREFIX_FILTER seq 5 deny 172.16.0.0/16
ip prefix-list PREFIX_FILTER seq 10 permit 0.0.0.0/0 le 32
```

### Brief Explanation of Each Method:
- **Distribute List**: Filters routes entering/exiting OSPF, usually applied to redistribution.
- **Filter List**: Controls LSAs processed/advertised based on prefix lists.
- **Summary Address (Not-advertise)**: Prevents summary addresses from being advertised as Type 3 LSAs.
- **Route Filtering**: Used within areas to filter based on prefix lists.
- **Route Map**: Allows granular filtering during redistribution with match conditions.

These configurations showcase how each method can be applied for OSPF filtering with a focus on Types 3 and 5 LSAs.

---

## Commands and Configuration

### Command 1: {{Insert Command Name or Purpose}}
```bash
{{Insert Command Here}}
```
**Example Output:**
```
{{Insert Example Output Here}}
```
**Explanation:**
- Describe what the command does and how it affects your configuration or troubleshooting process.

Command:: {{Insert Command Here}}
Example:: {{Insert Example Output Here}}
Explanation:: {{Brief Summary of the Command’s Purpose}}

---
### Command 2: {{Insert Command Name or Purpose}}
```bash
{{Insert Command Here}}
```
**Example Output:**
```
{{Insert Example Output Here}}
```
**Explanation:**
- Provide a clear and concise explanation.

Command:: {{Insert Command Here}}
Example:: {{Insert Example Output Here}}
Explanation:: {{Brief Summary of the Command’s Purpose}}

---

## Verification and Testing

### Verification Command 1: {{Insert Verification Command Name}}
```bash
{{Insert Verification Command Here}}
```
**Expected Output:**
```
{{Insert Expected Output Here}}
```

**Explanation:**
- Explain what this output confirms or validates about your configuration.

Verification:: {{Insert Verification Command Here}}
ExpectedOutput:: {{Insert Expected Output Here}}

---

## Troubleshooting Tips
- **Issue 1**: How to recognize and resolve it.
- **Issue 2**: Symptoms and troubleshooting steps.

Troubleshooting:: **Issue 1, Issue 2**

---

## Additional Notes
- Include any extra insights, best practices, or links to further resources.

---

## Tags
#important #networking #ospf #research

# References

[Reading and Understanding the OSPF Database - Cisco Community](https://community.cisco.com/t5/networking-knowledge-base/reading-and-understanding-the-ospf-database/ta-p/3145995)


