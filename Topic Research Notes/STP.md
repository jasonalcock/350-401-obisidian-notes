---
aliases: 
publish: 
---

%%
date:: [[2024-09-16]]
parent:: 
%%

# [[STP]]

Spanning tree enables us to have multiple redundant links in a single broadcast domain and to automatically detect and prevent loops. Without STP redundant links would bring down a layer 2 network due to broadcast storms. There are both Cisco and Open standards for STP implementations.

## What Problem Does It Solve

In a layer 2 network switches build their port-MAC forwarding (MAC) table whenever a port receives a frame with a SMAC. Switches only forward traffic when:

1. The frame's DMAC has a matching entry in the MAC address table. It then sends the frame out of the corresponding port.
2. The frame's DMAC is a L2 broadcast/multicast address. The switch forwards these frames out of all ports except the port the frame was received on.
Whenever links are added between layer 2 network switches in the same broadcast domain and a ring or loop is created there is no mechanism like what L3 has (TTL) to prevent switches from continuously forwarding the broadcast traffic around the loop. Switch A sends broadcast to switch B via one link and forwards the broadcast back to switch A via a second redundant link. When this loop happens, broadcast traffic flows indefinitely, it's called a broadcast storm, the switch's CPU not ASICs process broadcast traffic, copies packets out of every port and so it gets overloaded, as port buffers fill MAC learning is affected and links can become saturated.

Protocols that broadcast are DHCP. ARP, NetBios, BOOTP. Also, multicast traffic is broadcast out of all interfaces except for the interface the multicast was received on unless IGMP snooping is enabled.

*Note* As a switch's MAC table cannot store multiple ports for a single MAC it cannot provide redundancy.

## History

STP was 1st defined in a IEEE 802.1D standard using a well-known spanning tree algorithm (STA comes from graph theory) that is applied specifically for Ethernet networks.

## Spanning Tree

- Switches use BPDUs for communication.
- They use the root path cost to establish the topology.
- The switches need to know who's at the root of the tree.
- STP is tasked with finding any loops and blocking those looped paths.

## How STP Builds a Loop Free Topology

When a network of STP switches power on they go about building a loop free topology by selecting a root bridge, assigning ports to roles using a simple decision process, sharing information, using states and timers to ensure stability, adapt to changes and timely convergence. The process for building the loop free topology takes around 50 seconds.

The process starts with 3 simple steps:

*Step 1.* Elect the Root Bridge (*RB*)

*Step 2.* Elect it's Root Ports (*RP*)

*Step 3.* Elect Designated Ports (*DP*), Block others

Spanning Tree uses the same 4 step decision sequence at each step above.

*Step 1.* Lowest Root BID

*Step 2.* Lowest Root Path Cost (RPC) is the lowest cost to reach the RB

*Step 3.* Lowest BID

*Step 4.* Lowest Port ID

*Note:* Manually drawing out a ST topology like the one below is very easy but first you need to know some of the terminology. Also the process that STP goes through when converging and manages link changes requires more detail. When drawing out a diagram follow the rules:
The RB is the RB with the lowest priority.
Mark all its ports as DP
Mark the opposite RB DP as RP
The switches with RPs can mark the others ports as DP
When 2 non root switches are connected use the decision process above to compare which switch has the lowest cost or id then mark the opposit end as a blocking port.

In the diagram below notice how if a host is connected to sw4 and it wants to communicate with a host on sw5 it will go all the way back to the *sw1* the root.

The diagram shows the root bridge, non root bridges and assigned port roles.

![[Pasted image 20240919170431.png]]

## BPDU (Bridge Protocol Data Unit)

Before delving into STP process further lets see how it communicates and what's in it's messages. The messages are sent inside of layer 2 frames, and they use a multicast destination address of `01:80:c2:00:00:00` for IEEE STP BPDUs or `01:00:0c:cc:cc:cd` for Cisco BPDUs. The IEEE and Cisco BPDUs are the same except for a slight variation in the Bridge ID field. The BPDUs look like this:

![[Pasted image 20240919211320.png]]

**Message/BPDU Types** For PVST+ and 802.1d are:
- **Configuration.** Originated by RBs and they propagate throughout the network in the DP to RP direction. Information in this enables ST to build the loop topology.
- **Topology Change.** Originated by non-RBs to notify the RB there's been a topology change. Sent from root ports and only contain protocol ID, version and type fields, the rest of the fields are omitted.
**Flags** In STP there's only 2 options.
- **Topology Change**: Use by root bridges after when they receive a TC BPDU.
- **Topology Change Ack** Used by non root bridges to Ack the receipt of a TC BPDU.
**Root ID** is the Bridge ID of the root switch.
	802.1D Priority + MAC.
	Per VLAN STP and MST uses Priority + System Extension ID + MAC
**Root Path Cost** (RPC) is the total cost of the links traversed to get to the RB. Ports costs are based on port speed and when a switch receives a [[#Superior BPDUs]] it adds the link cost of the port the BPDU was received to get it's RPC. The RB RPC is 0 so its BPDUs are sent with a RPC of 0. 
**Bridge ID (BID)** Uniquely identifies each switch and is 8 bytes long.
- IEEE 802.1D (2 bytes *priority* + 6 byte switches base *mac* address)
	- **Priority** is 2 bytes and the default value is **32768**. The priority range is 0-65535.
- PVST+ (Only 4 bits for *priority* (12 bits taken for the *VLAN ID*) + 6 byte switches base *mac* address)
	- Priority can only be set in increments of 4096, default is still 32768
**Port ID** priority (4 bits) + ID (Interface number) (12 bits); the default port priority is 128, set as multiples of 16 in the range 0-255. Port priority is used to in a tiebreaker and used to influence the preferred root port of a downstream switch. 
**Timers** can be set on non-RB switches but are overwritten with the settings here. The settings are:
- **Max Age Timer** is (20 seconds default) the length of time a switch considers BPDUs to be valid. The timer is reset whenever a new BPDU is received and when it expires the port will change state.
- **Hello time** (default 2s) is the config BPDU send interval
- **Forward Delay** is amount of time a port stays in the listen and learn states.

It is worth noting BPDUs are sent on the native VLAN untagged. If the default native VLAN default is changed it will be tagged. You can specifically force the switch to untag a new native VLAN on a trunk.

## Root Bridge and Port Role Decision Sequence

So now you know some more lingo you can actually work out the spanning tree topology by finding the RB and all the port roles using 4 step sequence to elect and break ties.

1. **Lowest Root ID**: The switch with the lowest bridge ID becomes the root bridge ID and it sends the most [[#Superior BPDUs]]. (*Priority + MAC)*
2. **Lowest Path Cost to root bridge**: When the switch receives multiple BPDUs, it selects the interface with the lowest cost to reach the root bridge as the root port. *(Path Cost value)*
3. **Lowest Bridge ID**: When a switch is connected to two upstream switches that can reach the root bridge and the cost to reach the root bridge is equal, it select the interface of switch with the lowest bridge ID as the root port. (*Priority + MAC)*.
4. **Lowest Port ID**: When the switch has links to the same switch, the cost to reach the root bridge is equal it will set the interface with the neighbours lowest port number to the root port. (*Priority + Port ID*)

## Ports States, Timers and BPDUs

All ports start at the blocking and work their way through states.

- *Blocking* lasts for *20 seconds* and ensures used to provide stability. 
- *Listening* lasts for *15 seconds*
	- Is where *STP works out the topology*
	- Ports that lose the DP election become non-DPs and move to a Blocking state.
	- Ports that remain as DPs or RPs during this time progress to the Learning state.
- *Learning* lasts for *15-seconds*, processes frames to *build its MAC table*.
- *Forwarding* a port is able to send/receive traffic.

| STP States | Send BPDUs | Receive BPDUs | Forward Traffic | Update MAC table | State Time          | Port Status |
| ---------- | ---------- | ------------- | --------------- | ---------------- | ------------------- | ----------- |
| Disabled   |            |               |                 |                  |                     | Down        |
| Blocking   |            | x             |                 |                  | 20s (max age timer) | Up          |
| Listening  | x          | x             |                 |                  | 15s Hold Timer      | Up          |
| Learning   | x          | x             |                 | x                | 15s Hold Timer      | Up          |
| Forwarding |            |               | x               | x                |                     | Up          |
|            |            |               |                 |                  |                     |             |

![[Pasted image 20240923171344.png]]

![[Pasted image 20240920141128.png]]

## PVST+

- You simply get an STP instance for every VLAN including VLAN 1.

- In **PVST+**, when you block a VLAN on a trunk link due to Spanning Tree, you are **blocking the VLAN's traffic** for that trunk, not the entire interface. The port itself remains up and operational for other VLANs that are **not** blocked, allowing traffic for those VLANs to continue to flow.

- This **VLAN-specific blocking** ensures that only the problematic VLAN is affected, while other VLANs using the same trunk can still operate normally. This behaviour is a key advantage of PVST+ compared to a single-instance spanning tree that would block the entire interface.

- For **access ports**, it's the interface that is blocked, whereas on **trunk ports**, PVST+ blocks traffic on the VLAN not the interface.

- All PVST+ BPDUs are sent via the native VLAN.

## Root Bridge Election

- All switches are in the *Listening* state and *sending* Configuration *BPDUs* every 2s.

- They set the *RB-ID* using their their *BID*. 

- If BPDU RB-ID is < their BID, BPDU ignored.

- If BPDU RB-ID is > their BID, accepts BPDU, stops sending it's own BPDUs, updates new RB, RPC, timers and forward betters BPDU with updated RPC. 

- The width of the network determines the speed of conversion and the furthest distance hellos needs to travel. A 10 node ring would be a width of 4.

## Port Role Election

- Once a switch has found it's RB it elected the port roles:

- **Root Port** RP is the port the where the *best BPDU was received on*. Theres 1 per switch/spanning tree instance and always faces towards the direction of the RB.

- **Designated Port** (DP) is sending the best BPDU on the link. There's 1 DP per *link*. All ports on the RB are DPs. RPC, BID and Port Cost may all need to be considered to calculate the DP for a link connecting to non-RBs.

- **Non Designated (ND) or Blocking Port** All ports that are not a RP or DP become blocking ports. 

## Port Cost Values

- All interfaces have a port cost based on the bandwidth. The values are used to make the Root Path Cost. Port costs can be manually altered for an interface but by default the are preset using what is known as short mode and uses (16 bit) number.

- Short mode costs derive from the formula *20Gbps/interface bandwidth*. 

- IEEE Revised Port Costs in IEEE 802.1t and created long mode which uses a 32-bit number with the formula *20Tbps/interface bandwidth*. 

- Port costs can be configured manually on each port but long mostly the default.

| Data Rate               | Long Mode | Short Mode STP Cost |
| ----------------------- | --------- | ------------------- |
| 4 Mbps                  | 5000000   | 250                 |
| 10 Mbps                 | 2000000   | 100                 |
| 16 Mbps                 | 1250000   | 62                  |
| 100 Mbps                | 200000    | 19                  |
| 1 Gbps                  | 20000     | 4                   |
| 2 Gbps                  | 10000     | 3                   |
| 10 Gbps                 | 2000      | 2                   |
| 20 Gbps (SM ref. value) | 1000      | 1                   |
| 100 Gbps                | 200       | 1                   |
| 1 Tbps (LM ref Value)   | 20        | 1                   |
| 20 Tbps                 |           | 1                   |

Using short mode in a high speed network could mean ST converges in an undesired way. i.e. A switch as 2 paths to the root bridge via 1 Tbps and 100Gbps but the cost is equal so ST converges preferring the slower 100Gbps link is used. Use long mode or you manually set port cost. Switches only alter their internal path cost when they receive a superior BPDU.

## STP Timers

The default STP timers are based on the diameter/length (*default 7* and max value) of the switching topology. Recommended way to adjust timers is to set length. 

**Switches ignore their own timer configurations and learn the values from the root bridge** so that is where you would set them.

| Timer        | Purpose                                                           | Default |
| ------------ | ----------------------------------------------------------------- | ------- |
| Hello Time   | Time between sending of Configuration BPDUs by the Root<br>Bridge | 2s      |
| Forward      | Duration of Listening and Learning states                         | 15s     |
| Max Age Time | BPDU stored                                                       | 20s     |

**Max Age** is the time a bridge stores a BPDU before discarding it. Each port saves a copy of the best BPDU it has seen. As long as the bridge receives a continuous stream of BPDUs every 2 seconds, the receiving bridge maintains a continuous copy of the BPDU's values. However, if the device sending this best BPDU fails, some mechanism must exist to allow other bridges to take over.

![[Pasted image 20240920141211.png]]

## Noteworthy Port Role Transitions, State Changes and Timings

![[Pasted image 20240919170431.png]]

Timers need to expire in order for ports to transition through states and this can make STP slow.

**RP to DP** are *immediate* as they are *both forwarding ports*.

**Blocking to RP** (*direct link failure*) if a blocking port gets a better BPDU and doesn't have a RP it immediately ages out the inferior BPDU bypassing the max-age timer. It will transition the port to the listening state. E.g. If sw1 fails, sw2 sends sw3 blocking port better bpdu so it transitions to Listening. *Takes 30 seconds*.

**Blocking to DP** If we stops receiving better BPDUs on a blocked port, still has a RP then there's been an *indirect failure*. E.g. If sw2 fails, sw5 is still getting BPDUs on RP but worse BPDUs from sw4, it transition a port to blocking and the best port to listening after the **Max Age** to ensure the link is stable. It's called an indirect failure and can take up-to *50s* to change.  

## Topology Changes

- As the ST topology updates and forwarding path their's no mechanism to ensure the bridges MAC tables are aligned. Forwarding will simply send traffic out of a port but then STP may have blocked that port so traffic is dropped.
- Spanning Tree does not natively manage the MAC tables.
- The MAC table store entries for 5 minutes by default before they are removed.
- The TCN process resolves issues by temporarily changing the switches MAC address age timers.

TC Process Steps:

1. A bridge that detects a topology change sends a TC Type BPDU out its Root Port. 
2. The Designated Port for this segment acknowledges the BPDU by setting the **TCA** flag in it's next Confg BPDU.
3. The next bridge also propagates the TCN BPDU out its Root Port. 
4. When the TC BPDU reaches the Root Bridge it then sends it's config BPDUs with TC flag on until the Forward Delay + Max Age (25s) expires.
5. As non root bridges receive the config BPDU with TC flag, they shorten their MAC table's aging period from 300s to that of the Forward Delay timer and inactive flows are then expired after 15s.

```js
SW1#show mac address-table dynamic
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    000c.1111.5c6c    DYNAMIC     Fa0/14
   1    000c.2222.03ba    DYNAMIC     Fa0/16
```

```js
SW1#show mac address-table aging-time 
Global Aging Time:  300
```

## Direct Link Failure (Root Port Offline - 30s)

If sw2 or sw3's root port fails so after the BPDUs have been checked the DP or Blocking port becomes the new to RP. The bridge bypasses the max age timer blocking state as it's direct link failure of the root port and the BPDU from it's root port was removed immediately. The new root port will transition through listening, learning then forwarding.

![[Pasted image 20240923161732.png]]

## Indirect Link Failure (stops Getting Superior BPDUs on Blocked Port but Still Has RP - 50s)

- **Indirect Link Failure**. sw3's gi1/0/2 blocking port transitions to forwarding due to an indirect link failure.
- It no longer gets better BPDUs from sw2's DP.
- It waits for its g1/0/2 max age timer for the last time it got the better BPDU then transitions to the listening, learning then forwarding state.
- When sw2 gets the better BPDU from sw3 it immediately transitions to forwarding.
- As Sw3 port transitioned from blocking to forwarding it will send the TC BPDU.
![[Pasted image 20240923134326.png]]

## Root Bridge Failure

Downstream bridges remove their Root Ports and associated stored BPDU from the Spanning Tree topology so we don't need to wait in the blocking state when we get a better BPDU.

All switches flood the BPDUs from all ports (DPs) as they think they're RB. The best bridge is shortly elected and all it's ports are DPs and forwarding. As the downstream switches receive the best BPDU they recognise they are a non-RB switch, calculate their new RPC and switch their port role to RP. They send TC BPDU to the RB to notify the bridge of the role changes. The RB ports are forwarding and the downstream switches RP will transition through listening and learning then forwarding. As they receive the RB BPDUs with the TC flag they change their forward delay timer.

## STP Design

If you have a devices running in active/standby modes like a 2 FHRP routers then mirror the active standby states with STP using primary secondary STP root bridge priorities. 

## STP Priortity Wording Confusion

As with all Spanning Tree parameters, the lowest numeric Bridge ID value represents the highest priority. To avoid the potential confusion of lowest value and highest priority, this text always refers to values (in other words, the lower amount is preferred by STP). To put this another way don't talk about the STP priority just talk about the value!!!!

Superior BPDU 4 step process:  

1. A lower Root BID  
2. A lower Root Path Cost
3. A lower Sending BID  
4. A lower Port ID

## Cisco Inter Vendor BPDU Interoperability

Cisco's PVST+ (per VLAN 802.1d) sends IEEE BPDUs to `01:80:c2:00:00:00` for inter-vendor compatibility but it uses Cisco proprietary BPDUs to `0100.0ccc.cccd`. The IEEE BPDU does not carry any VLAN info. 

IEEE BPDU

```js
Frame 2: 60 bytes on wire (480 bits), 60 bytes captured (480 bits)
IEEE 802.3 Ethernet 
Logical-Link Control
Spanning Tree Protocol
    Protocol Identifier: Spanning Tree Protocol (0x0000)
    Protocol Version Identifier: Spanning Tree (0)
    BPDU Type: Configuration (0x00)
    BPDU flags: 0x00
    Root Identifier: 32768 / 1 / 52:54:00:04:2a:87
    Root Path Cost: 0
    Bridge Identifier: 32768 / 1 / 52:54:00:04:2a:87
    Port identifier: 0x8001
    Message Age: 0
    Max Age: 20
    Hello Time: 2
    Forward Delay: 15
```

Cisco BPDU

```js
Frame 3: 64 bytes on wire (512 bits), 64 bytes captured (512 bits)
IEEE 802.3 Ethernet 
Logical-Link Control
Spanning Tree Protocol
    Protocol Identifier: Spanning Tree Protocol (0x0000)
    Protocol Version Identifier: Spanning Tree (0)
    BPDU Type: Configuration (0x00)
    BPDU flags: 0x00
    Root Identifier: 32768 / 1 / 52:54:00:04:2a:87
    Root Path Cost: 0
    Bridge Identifier: 32768 / 1 / 52:54:00:04:2a:87
    Port identifier: 0x8001
    Message Age: 0
    Max Age: 20
    Hello Time: 2
    Forward Delay: 15
    Originating VLAN (PVID): 1
        Type: Originating VLAN (0x0000)
        Length: 2
        Originating VLAN: 1
```

- Both IEEE and Cisco BPDUs are sent for VLAN 1
- If VLAN 1 is the native VLAN the Cisco BPDU will be sent untagged.
- If the native VLAN has been changed the IEEE and Cisco native BPDU are tagged with the native VLAN ID
- The Cisco BPDU fields are identical to the IEEE BPDU, apart from an additional ‘PVID’ field which identifies the VLAN ID of the source port.
- PVST+ and RPVST+ both sends a standard IEEE BPDU for the default VLAN 1 to the IEEE MAC address and that's what makes Cisco and IEEE switches compatible with each other.
- IEEE switches will not interpret Cisco proprietary BPDUs, but they will pass them on
- If you change the native VLAN ensure the trunk link to the IEEE switch receives untagged IEEE BPDUs otherwise the IEEE switch's STP will not see the Cisco IEEE BPDU.
- If you set a different native VLAN between 2 Cisco switches and you are not tagging the native traffic then the switch will detect the mismatch in the BPDU and put the trunk port into an *err-disabled* state. It checks if the PVID in the BPDU match the ports native VLAN. Below is what spanning tree show command shows when an interface is in *err-disabled* state.

```js
SW2#show span vlan 1

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     5254.0004.2a87
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5254.0004.2a87
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg BKN*4         128.1    P2p *PVID_Inc 

SW2#show span vlan 2

VLAN0002
  Spanning tree enabled protocol ieee
  Root ID    Priority    32770
             Address     5254.0004.2a87
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32770  (priority 32768 sys-id-ext 2)
             Address     5254.0004.2a87
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg BKN*4         128.1    P2p *PVID_Inc
```

## Load Balancing Between PVST+ and IEEE dot1q Switches CST

- The PVST+ switches can load balance VLANs between switches within their region but when they trunk to the IEEE switches they have to send all VLANs down to the IEEE switches as support only 1 CST for all VLANs. 
- We can't split VLANs across trunks as we're moving to a single instance and w're blocking whole interfaces not just the VLAN on an interface. 
![[Pasted image 20240926220619.png]]

Note that every switch:

1. Accepts and retains only the best current root bridge information
2. Port store the most recent STP BPDUs it. Therefore a BPDU is stored with every root or alternate (blocked port).

Designated ports are responsible for sending BPDUs for a segment. When 2 switches are non root bridge i.e. their BID is not RBID STP knows it's a loop link so they elects the DP and the non-designated port is out into a blocking state.

## STP Configuration Set

```js
#### STP
!
! Set STP mode, no 802.1D
!
spanning-tree mode pvst|rapid-pvst|mst
!
! On by default
!
spanning-tree extend system-id
!
! Will only accept increments of 4096
!
spanning-tree vlan 1 priority 32768
!
! Default mode is short
!
spanning-tree pathcost method short
!
! Cisco STP Toolkit
!
spanning-tree portfast edge default
spanning-tree portfast edge bpduguard default
spanning-tree portfast edge bpdufilter default
!
! Port tranistions to fowarding immediatly, is STP active, only safe to use where loops cannot occur
!
spanning-tree portfast network default
!
! Set Timers
!
spanning-tree vlan 10 forward-time 15
spanning-tree vlan 10 hello-time 2
spanning-tree vlan 10 max-age 20
!
! Set bridge priority without needing to add a value
!
spanning-tree vlan 10 root primary
spanning-tree vlan 10 root secondary
!
! Set bridge priority 
!
spanning-tree vlan 1,10,199 priority 24576
spanning-tree vlan 11 priority 28672
!
!
## Interface settings
int g0/0
spanning-tree portfast edge
spanning-tree portfast edge trunk
!
## More int settings - Cisco STP Toolkit
!
! Affects outcoing BPDUs for neighbour
!
spanning-tree port-priority
spanning-tree guard loop/root/non
spanning-tree cost 1
spanning-tree bpduguard enable/disable
spanning-tree bpdufilter enable/disable
!
! Or per vlan settings
!
spanning-tree vlan 1 cost/port-priority
!
! debugging
!
debug spanning-tree events
!
spanning tree mode mst
spanning-tree mst configuration
spanning-tree mst 0 root primary 
spanning-tree mst forward-time/heelo-time/max-age/max-hops
!
show spanning-tree mst 0
!
```

## Cisco STP Toolkit

Cisco developed three proprietary features that improve PVST+ convergence time:

• PortFast - For Access Ports straight to forwarding no topology changes sent for up/down

• UplinkFast - Access Layer Switches Only, increases priority and cost, PVST only

• BackboneFast - Recover faster from indirect link failures. PVST only and for all switches.

Cisco implemented three mechanisms to protect the PVST+ topology:

• Root Guard - For Root Bridges

• Portfast BPDU Guard - For Access Ports errDisable no forwarding

• PortFast BPDU Filtering - For Access Ports When Vital - disables STP

[[#Unidirectional Link Problem]]

- UDLD - Any port
- Loop Guard - For Blocking and Root Ports - errDisable no forwarding

Because this tends to break the network into a large number of loop-free paths, there are no Blocking ports for UplinkFast and BackboneFast to perform their magic

## PortFast

- Transition from a blocking to a forwarding state immediately, eliminating the typical 50 second delay.
- PortFast does not disable STP on a port - it merely accelerates STP convergence. If a PortFast-enabled port receives a BPDU, it will transition through the normal process of STP states.
- Access Ports are DPs
```js
spanning-tree portfast default
!
! Or
!
int gi1/14
 spanning-tree portfast
!
show spanning-tree backbonefast
```

## BPDU Guard ( PortFast into errDisable)

- Is a **PortFast add-on feature** that protects port if BPDU received.
- BPDU Guard puts a **port in an errdisable state** if a **BPDU received** on a PortFast port.
- Disabled by default, and can be enabled on a per-interface basis:
```js
spanning-tree portfast bpduguard default
!
interface gi1/14
 spanning-tree bpduguard enable
!
! Recovered interface from an errdisable state
shutdown
no shutdown
```

## BPDU Filtering (Disable PortFast or ST on Single port)

- Is a **PortFast add-on feature** that protects port if BPDU received.
- If **enabled globally**, a received BPDU will **disables PortFast** if a BPDU is received and port **defaults to regular STP** states.
- If filtering **enabled at interface level**, incoming **BPDUs ignored**, STP **cannot change state and is disabled**
- BPDU Guard will not work with BPDU filtering on.
- If enabled on interface port is susceptible to creating a loop that cannot be blocked by ST.
```js
spanning-tree portfast edge bpdufilter default
!
interface gi1/14
 spanning-tree bpdufilter enable
```

## UplinkFast (Leaf/Access Switches Only)

- This feature is for **leaf/access switches only**.
- When the root port fails it **immediately transitions Non-designated blocking ports to become the new forwarding RP**.
- It **doesn't use TC** mechanism to expire old MAC tables but immediately **sends dummy multicast frames** with the source MAC addresses of active VLAN MACs using failed port so upstream switches can reprogram their MAC address tables.
- When enabled the bridge priority value and all port costs are increased.
	- If the current **bridge priority value is < 49152 it's set to 49,152**.
	- It **adds 3000 to all port costs**.
	- !!BE CAREFUL UplinkFast feature is built into RPVST+ but if enabled it still changes the priority and port costs.
```js
spanning-tree uplinkfast
!
! See the bridge priority, message about uplinkfast and port cost is + 3000
!
Acc3(config)#do sho span

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    24586
             Address     5000.0005.0000
             Cost        3004
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    49162  (priority 49152 sys-id-ext 10)
             Address     5000.0007.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec
  UplinkFast enabled but inactive in rapid-pvst mode

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 3004      128.1    P2p
Gi0/1               Desg FWD 3004      128.2    P2p Edge
```

## BackboneFast (Saves 20s when Indirect Link fails)

- **Immediately expires Max Age timer** in the event of an indirect link failure.
- **Indirect link failure takes 50s by default as ages old BPDUs needs to age out**.
- Switch receives inferior BPDUs its a signal the other bridge lost it's RP.
- The **switch sends an RLQ** via it's RP and out of DPs.
- Response received from RB it ages out inferior BPDUs immediately **saving 20s**.
- Other **switches respond if they have RP**.
- Can be enabled on all switches.
- RSTP has this feature built in.
```js
spanning-tree backbonefast
show spanning-tree backbonefast
```

## Root Guard

- Root guard protects root ports to always be the root port.
- If superior BPDU received on a RootGuard port it is placed in a *root-inconsistent* state, blocks traffic, still listens for BPDUs and automatically recovers as soon as superior BPDUs not received.
- Root Guard is enabled on a per-port basis, and is disabled by default. 
- **You cannot enable root guard and [[#Loop Guard]] on a port at the same time**. 
```js
!
! If Loop Guard is enabled on a port, it disables Root Guard on the port and vice versa.
!
interface gi1/14
 spanning-tree guard root
!
show spanning-tree inconsistentports
Name       Interface           Inconsistency
VLAN100    GigabitEthernet1/14 Root Inconsistent
```

## References

[Radia Perlman - Wikipedia](https://en.wikipedia.org/wiki/Radia_Perlman)

Trill designed to superseded STP. [TRILL - Wikipedia](https://en.wikipedia.org/wiki/TRILL)

[routeralley.com/guides/stp.pdf](https://www.routeralley.com/guides/stp.pdf)

[lucidresource.com/completed/ccnp\_switching\_studyguide.pdf](https://lucidresource.com/completed/ccnp_switching_studyguide.pdf)

[Understand Rapid Spanning Tree Protocol (802.1w) - Cisco](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/24062-146.html#topic5)

[STP Convergence: Understanding Spanning-Tree Protocol Process](https://ine.com/blog/2009-03-07-understanding-stp-convergence-part-i)

[[MST, RSTP and STP Revision|STP Revision Notes - RSTP MST]]

[Best book I've read for STP #Cisco LAN Switching Configuration Handbook, 2nd Edition](https://www.ciscopress.com/store/cisco-lan-switching-configuration-handbook-9781587140624)

[Detecting Mismatched Native VLANs – Daniels Networking Blog](https://lostintransit.se/2024/07/11/detecting-mismatched-native-vlans/)
