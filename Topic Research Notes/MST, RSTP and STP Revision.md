---
aliases: 
publish: 
---
%%
date:: [[2024-09-20]]
parent:: 
%%
# [[MST, RSTP and STP Revision]]

# STP Versions

IEEE Standards bodies and the timeline for STP and VLAN standards.
- 802 (1990) Main IEEE standard for local area networks (LANs) and metropolitan area networks (MANs) (re-issued as IEEE 802–2001) 802.1 (1986) Higher layers and management interface standards for LANs (re-designated 802–1990)
- 802.1d (1990) Spanning tree algorithm for preventing circular loops when bridging LANs (MAC bridges)
- 802.1q (1999) Virtual bridged LANs (VLANs)

- **IEEE 802.1d STP** was before VLANs and there it ran one instance of STP.
- **802.1q** came almost 10 years after 1st STP defined and worked with VLANs but still 1 STP instance for all VLANs called the **CST**.
- **PVST+** (Per VLAN Spanning Tree Plus) is Cisco's enhancement of *802.1d and q*. It creates a Spanning Tree instance per VLAN, supported over .1q and ISL trunks and is the default on most Cisco switches. Load Balancing possible.
- **RPVST+** Rapid Per VLAN Spanning Tree Plus. Cisco enhancement to *802.1w* RSTP. Significantly improved convergence time over PVST+. Uses a separate Spanning Tree per VLAN. Load Balancing possible.
- **MST**, based on the IEEE *802.1s* MST standard. Load balancing possible due to VLANs being mapped to RSTP instances

*RPVST+/MST* have built in root and loop guard protection features that do not need enabling?

All versions of ST operate in the same 3 way, in 3 steps, to build their initial loop free topology and 4 step sequence when making any decisions. All Cisco versions are compatible with CST.
## STP Version Comparison Table
- PVST+, RSTP, and MST backwards compatible with 802.1D

| STP Version | Standard | Instances | STP Calculation | Load Balancing <br>VLANs | LB ST Instances | Convergence Time |
| ----------- | -------- | --------- | --------------- | ------------------------ | --------------- | ---------------- |
| STP         | 802.1D   | 1         | 1               | No                       | No              | 50+ seconds      |
| PVST+       | Cisco    | /VLAN     | /VLAN           | Yes                      | No              | 50+ seconds      |
| RSTP        | 802.1w   | 1         | 1               | No                       | No              | 2s               |
| Rapid PVST+ | Cisco    | /VLAN     | /VLAN           | Yes                      | No              | 2s               |
| MSTP        | 802.1s   | MST0 +    | /Instance + IST | Yes                      | Yes             | 2s               |
### Port Role Comparison

| 802.1D     | PVST+      | RPVST+, MSTP     |
| ---------- | ---------- | ---------------- |
| Root       | Root       | Root             |
| Designated | Designated | Designated       |
| Blocked    | Alternate  | Alternate/Backup |
# RPVST+
- IEEE replaced 802.1D with RSTP 802.1w and convergence is much faster.
- 30 - 50 seconds port transitions down to 6 seconds or less
- RSTP can revert back to STP so is back compatible.
- RSTP switches sends it's own BPDU every 2 seconds
- PVST+ uses timers to transition through states but RPVST+ uses synchronisation.
- Topology changes are faster.
- Supports Load balancing
## RPVST+ Topology
### Initial Topology
As always with ST the initial topology is created using the same 3 steps and it's also done per VLAN.
*Step 1.* Elect one Root Bridge
*Step 2.* Elect Root Ports
*Step 3.* Elect Designated Port 

RPVST+ uses the same 4 step sequence for all decisions:
1. A lower Root BID  
2. A lower Root Path Cost
3. A lower Sending BID  
4. A lower Port ID
### New Port Roles
The only change to roles was the alternate role is split, a new backup role is created if a a switch sees it's own BPDUs on another port.
- **Root Port** – Port on each switch that has the best path cost to the Root Bridge. A switch can only have one root port.
- **Designated Port**  - Non-root port that represents the best path cost for each network segment to the Root Bridge.
- **Alternate Port** – Backup root port, same as non-Designated port and the less desirable root port. 802.1D called this role Blocked, PVST+ called it Alternate RSTP split this role into Alternate and Backup.
- **Backup** - This is a backup to designated port and will only appears when switch sees its own BPDU because it's connected to a hub via 2 of it's ports or it was connected back to itself directly.
*Role diagram*
![[Pasted image 20240920151838.png]]
### New RPVST+ Port Types
New port types have been added and determine if the port can use the fast transition sync feature. Port types are auto set based on the interface duplex but they can be set manually.
*Point-to-point* ports: Automatically assigned to *full duplex* links, connects to another switch, 
*Shared* ports: Assigned to *half duplex* links and cannot use the fast transition sync feature.
*Edge* ports connect to end hosts/servers, transition immediately to forwarding state and are enabled manually with *PortFast*. If it receives a BPDU loses its Edge status. It does not generate TCN when its link flaps

Note PVST+ does not use these port types but RSTP Edge Ports are equivalent to PVST+ PortFast ports using the same `spanning-tree portfast` command.

```
Sw# show spanning-tree [vlan VLAN-ID]
Fa0/1               Desg FWD 19        128.1    P2p
Fa0/2               Desg FWD 19        128.2    Shr
Fa0/3               Desg FWD 19        128.3    P2p Edge
```
### RPVST+ Port States
There's only 3 states and the new *discarding state* is the same as the *learning state* - can send/receive BPDUs but discards traffic and doesn't learn MACs.
![[Pasted image 20240920144406.png]]

**NOTE**: In Cisco implementation you'll see blocking state and that's the same as discarding.**
### RSTP BPDUs
An RSTP BPDU uses the same format as an STP BPDU except that a Version1 length field is added to the payload of RSTP BPDUs. The differences between an RSTP BPDU and an STP BPDU are as follows:

- **Protocol version ID** — The value is 0x02 for RSTP.
- **BPDU type**—The value is 0x02 for RSTP BPDUs.
- **Flags**—All 8 bits are used.
- **Version1 length**—The value is 0x00, which means no version 1 protocol information is present.

RSTP does not use TCN BPDUs to advertise topology changes. RSTP floods BPDUs with the TC flag to advertise topology changes.
#### Flags

![[Pasted image 20240922114608.png]]
### RSTP Synchronisation/Handshake Process
- **Faster Convergence**: RSTP does not use timers, unlike 802.1d STP, it uses synchronisation to rapidly transition through a its port states and converge. Synchronisation takes less than 2 seconds to transition ports on a link.
- **P2P Links**: The handshake mechanism only works on **Point-to-Point (P2P) links**.
	- All full duplex ports are P2P, half duplex shared but interface can be set `#spanning-tree link-type point-to-point`.
- **Shared Links**: If the link is shared or if no agreement is received, RSTP falls back to using **802.1d timer mechanisms**.

**Proposal:**
Bridges request to be the Designated Port (DP) for a link and send BPDUs with the proposal flag set when:
- **It first comes online:** The bridge initially considers itself as the Root Bridge and floods BPDUs with the proposal flag out of all designated blocking ports. This is different from 802.1D as it starts all ports in forwarding whereas RSTP starts all ports discarding to begin the proposal-agreement process.
- **A new point-to-point (P2P) port/link becomes active:** The bridges are already active the link is in blocking so it sends a BPDU with the proposal flag to see if will be the DP.
- **A port’s role changes from Root Port (RP) to Designated Port (DP):** This often happens when topology changes occur, and an intermediate switch takes on a new role.
**Additional Notes:**
- While a bridge is sending BPDU proposals, it keeps the corresponding port in the **Designated Discarding** state until it receives an agreement BPDU from the downstream switch.
- If the bridge receives an inferior BPDU (indicating that the other bridge is not the best candidate), it disregards that BPDU with the proposal, maintains its role and doesn't start to sync.
### RSTP Sync Process in 4 Simple Steps
1. **Proposal**: Bridges send BPDUs with the proposal flag set, indicating they want to be the Designated Port (DP) for that link. While sending proposals, the bridge remains in the **Blocking** state.
2. **Sync**: The less-preferred bridge receiving the proposal transitions its port to an RP and may start forwarding immediately if it's in sync.
   - It then initiates the sync process 1 with all its downstream switches over P2P links, ensuring no loops.
   - A bridge is considered "in sync" when all its **non-Edge Designated Ports** are in a **Discarding** state.
3. **Agreement**: 
   - Once the inferior bridge completes synchronisation of its ports, it sends an **Agreement BPDU** back to the superior bridge, indicating it’s "in sync."
4. **Forwarding**: 
   - The superior bridge, upon receiving the Agreement BPDU, transitions its Designated Port into the **Forwarding (FWD) State**. As the sync process flows downstream switches begin forwarding over their DP after they receive their agreement.
### Topology Changes
#### TC and MAC Address Table Updates
- Only non-edge ports generate Topology Changes (TCs) on the network when they transition to forwarding state only.
- TCs are flooded quickly out of non-edge ports and RPs, and neighbouring switches flush their CAM entries, resulting in faster convergence time.
- When a port transition occurs the switch starts its TC while timer equal to 4 seconds (2 * hello interval) for all its non-edge ports.
- Whilst the timer is active, it sends Configuration BPDUs with the Topology Change (TC) flag bit set out of its Designated and Root ports.
- This process continues until the timers expire and the BPDUs have flooded throughout the ST topology.
- The downside to this process is some flooding does take place in the network. 
- When a switch receives a config BPDU on it's root port with the TCN bit set immediately clears all MAC addresses learned on all ports except the port the BPDU was received on.
2. If a BPDU with the TC flag set is received on an alternate blocking port it is ignored.
3. When the root port on an RSTP-enabled device misses three consecutive BPDUs, it begins the convergence process. If the hello timer is 2 then it starts to converge after 6 seconds. 
4. Edge ports are immediately placed into the forwarding state (PortFast) and do not generate BPDUs with a TC flag set.
5. MAC addresses learned on Edge ports ARE NOT flushed when a Topology Change BPDU is received (PortFast).

RSTP 802.1D Topology Change Differences:
- Only the Root Bridge Configuration BPDUs with the TC Topology Change flag bit. In RSTP any switch can initiate them.
- TC type BPDUs are not used in RSTP
- Topology Change Acknowledgements are not used in RSTP

```
SW1#show mac address-table dynamic
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    000c.1111.5c6c    DYNAMIC     Fa0/14
   1    000c.2222.03ba    DYNAMIC     Fa0/16
```

```
SW1#show mac address-table aging-time 
Global Aging Time:  300
```

![[Pasted image 20240925142016.png]]

#### Direct Link To Root Bridge Failure Scenarios
- If any switch above loses its directly connected *RP*, it immediately transitions it's *Alternate port* to become the *new RP* and places the port into *forwarding*
- The switch will automatically removes any MAC address entries for the old RP interface from its table.
- The TC process is started.
If a directly connected *Alternate or DP* port goes offline the interface is simply removed from the ST topology and MAC address entries for the interface are removed.
#### Upstream Link to Root Bridge Failure
- If sw2 loses its 0/0 RP link 
- sw3 receives an inferior BPDU (worse RPC) from sw2 with the proposal bit set so synchronisation begins
- sw2 0/1 DP transitions to RP
- sw3 0/1 Alt Blocking transitions to DP and triggers the TC process.
### Timers 
- RSTP doesn't use timers to transition through states but it will age out any stored BPDUs received on its port if they are not seen for three consecutive times. It takes 3x Hello timer (6 seconds total by default) for a stored BPDU to be aged out.
 - The timer fields are the same in all STP versions to ensure compatibility e.g. RSTP bridge can fallback to STP topology change process.
- In RSTP the Message Age field and **Max Age** timer determine the width or maximum distance a BPDY can travel before it is dropped. The process also acts as a loop prevention mechanism similar to TTL hop count and is also present in 802.1D.
- The Root Bridge sends its BPDUs with the Message Age set to 0 it's then incremented  by 1 in each BPDU. If the Message Age value reaches the **Max Age timer** (20 ‘seconds’ by default) the BPDU will be dropped.
## Bridge ID for RSTP & RPVST+ 
**Bridge ID** Uniquely identifies each switch and is 8 bytes long. 
2 bytes (4 bits for **priority** value + 12 bits for **Extended System ID**) + 6 byte base **MAC** address.
- **Priority** default 32768, range 0-65535 but increments of 4096 only.
- **Extended System ID** is the VLAN ID for RSTP & RPVST+
# Unidirectional Link Problem
Acc4 g0/0 port is blocking but uni failure means occurred so it stops getting BPDUs from CD2, ages out that BPDU and  transitions to a DP. Now Acc4 has created a unidirectional loop CD1 --> Acc4 --> CD2.
![[Pasted image 20240920170805.png]]
Two methods available to prevent the unidirectional link problem. 
STP Loop Guard is native to STP and also prevents if IOS stopped sending BPDUs.
UDLD which is a separate L2 protocol and specific for detecting layer 2 loops.
### Unidirectional Link Detection (UDLD) 
A layer 2 protocol not part of STP. A hardware fault can cause traffic to be transmitted in only one direction e.g. Fibre ports.
- STP requires switches to exchange BPDUs bidirectionally
- Cisco's UDLD ensure that bidirectional communication is maintained, on both sides of a link. UDLD has 2 modes:
- Normal – the port remains up, syslog message is flagged. Undetermined state no response, unknown state no UDLD neighbour detected.
- Aggressive Mode – the port is placed in an *errdisable* state
- UDLD frames 15 or 7 seconds. If response with own ID seen UDLD assumes unidirectional fault.
- UDLD can be enabled globally, though it will only apply for fibre ports:
```
!
! aggressive parameter is for aggressive mode set hello interval
!
udld enable message time 20
udld aggressive message time 20
!
udld enable
udld disable
!
Switch(config-if)# udld port aggressive
Switch(config-if)# no udld port aggressive
!
! View UDLD status on ports, and reset any ports disabled by UDLD: 
!
show udld
udld reset
```
### Loop Guard
- We always expect to receive BPDUs on root or blocking ports.
- If thats stops, when the max age timer expires the port is placed into an inconsistent state and all traffic is blocked.
- Loop Guard does not disable DPs because BPDUs are not expected to be received
- Auto re-enabled when BPDUs received again.
-  Loop guard is effective only if the port is a root port or an alternate port. **You cannot enable loop guard and [[#Root Guard]] on a port at the same time**. 
```
!
! If Loop Guard is enabled on a port, it disables Root Guard on the port.
!
spanning-tree loopguard default
!
! Or
!
Switch(config-if)#spanning-tree loopguard default
!
! Test loopguard by enabling 
!
CD1 (config-if)#spanning-tree bpdufilter enable on upstream switch
!
show spanning-tree summary
Acc4#show spanning-tree interface g0/0 detail
	Port 1 (GigabitEthernet0/0) of VLAN0001 is alternate blockingPort 
		path cost 4, Port priority 128, Port Identifier 128.1.
		Designated root has priority 24577, address 5254.0003.8b86
		Designated bridge has priority 28673, address 5254.0003.c6e3
		Designated port id is 128.1, designated path cost 4
		Timers: message age 3, forward delay 0, hold 0
		Number of transitions to forwarding state: 4
		Link type is point-to-  point by default
		!
		Loop guard is enabled on the port
		BPDU: sent 311, received 2104
```

![[Pasted image 20240920172801.png]]

## Mixing PVST+ and RPVST
RSTP sends PVST+ BPDUs and uses STP timers for convergence, classic TC notifications, classic STP states.   RTSP bridge falls back to STP when it receives a STP BPDU. When a RSTP BPDU is received by a STP bridge it discards the frame.

```
SW1#show spanning-tree | begin Interface
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Fa0/14              Desg FWD 19        128.16   P2p Peer(STP) 
```

# MSTP (Multiple Spanning Tree Protocol)
802.1D used 1 instance for all VLANs and that instance is called CST as it was common to all VLANs.
802.1s IEEE created MSTP that groups VLANs into an **MST Instances**.
Each instance gets its own ST topology, RB and loop free path.

There's a special MST instance called the **Internal Spanning Tree (IST) or MST0.**
- It's the only instance that processes BPDUs, it maintains it's own topology and carries in its BPDUs information about each instance inside an MST record (M-record).
- The MST0 topology works on every port regardless of VLAN.
- The IST is used to interface an MST region with an external spanning tree, such as a CST on a non-MST switch.

For switches to talk MST then they all need to belong to the same region. The MST region is defined with the following *identical* MST parameters:
- 32-byte configuration name
- 16-bit revision number
- VLAN-to-instance mapping database

 instances together so they appear as a single bridge with a single CST to any bridge outside the region. An MST region is hidden from non-MST switches such as 802.1D, RSTP or from other MST regions and they see the region as a single CST. 

```
spanning-tree mode mst
!
spanning-tree mst configuration
name REGION1
revision 2
instance 2 vlan 11-100
instance 3 vlan 101-200
instance 0 vlan 10
exit ! only applies after you exit
!
name REGION2
revision 3
instance 4 vlan 201-300
instance 5 vlan 301-400
exit
!
spanning-tree mst 2 root primary
spanning-tree mst 3 root primary
!
```

For most Cisco platforms, a region can contain a maximum of 16 MST instances, numbered 0 through 15. By default, all VLANs belong to instance 0.



The IST is always mapped to instance 0 (MST0). 

MST is compatible with all other implementations of STP.

Cisco recommends mapping all VLANs to MST instances other than the IST unless those VLANs are used to connect to switches that are not MST-capable or outside the MST region.

- VTP VLAN pruning ensures switches remove VLAN broadcast traffic on their trunk ports if they do not have an active interface in that VLAN.
- You cannot prune the management vlan 1 and should not prune the management vlan if changed.
- If you prune a VLAN from a trunk your preventing that vlan from being used on a trunk. If the  ST topology is actively blocking the instance but VTP is pruning you've effectively disabled that VLAN from working.
[Understand the Multiple Spanning Tree Protocol (802.1s) - Cisco](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/24248-147.html#two_vlans_same) Switches remove 

The IST instances operates across all links regardless of the VLAN assigned to a port. If vlan 10 is assigned to instance 0 but the vlan is not active across all links IST may well block a link that does not carry vlan 10. To resolve this:
- Move the VLAN to it's own MST instance and the ST forwarding path will be based on the links that VLAN is active on.
- Or Allow VLAN 10 on all links.

If a VTP prune is actived for VLAN 10 but the MST instance is blocking the other it will VLAN 10 out of action. 



[[VTP VLAN Pruning]]

```
show spanning-tree mst
```

# MST Overview: IST, CIST, and CST

## Key Concepts

When IEEE created 802.1s MST they combined the best aspects from both the PVST+, 802.1q and RSTP 802.1w. The idea is that several VLANs can be mapped to a reduced number of spanning tree instances because most networks do not need more than a few logical topologies.
In the topology below diagram, there are only two different final logical topologies, so only two spanning tree instances are really necessary. There is no need to run 1000 instances. If you map half of the 1000 VLANs to a different spanning tree instance, as shown in this diagram, these statements are true:
- The desired load balancing scheme can still be achieved because half of the VLANs adhere to one separate instance.
- The CPU is spared because only two instances are computed.
[![Map Half of the 1000 VLANs to a Different Spanning Tree Instance](https://www.cisco.com/c/dam/en/us/support/docs/lan-switching/spanning-tree-protocol/24248-147-02.gif)](https://www.cisco.com/c/dam/en/us/support/docs/lan-switching/spanning-tree-protocol/24248-147-02.gif "Map Half of the 1000 VLANs to a Different Spanning Tree Instance")Simple MST Instances

From a technical standpoint, MST is the best solution but from an end-user perspective, the main drawbacks associated with a migration to MST are:
- The protocol is more complex than the usual spanning tree and requires additional training of the staff.
- Interaction with legacy bridges can be a challenge. For more information refer, to the [Interaction Between MST Regions and the Outside World](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/24248-147.html#mst_region_world) section of this document.
### Terminology
From the IEEE 802.1s specification, an MST bridge must be able to handle at least these two instances:
- One Internal Spanning Tree (IST)
- One or more Multiple Spanning Tree Instance(s) (MSTIs)
The terminology continues to evolve, as 802.1s is actually in a pre-standard phase. It is likely these names can change in the final release of 802.1s. The Cisco implementation supports 16 instances: one IST (instance 0) and 15 MSTIs.
### Standard 802.1q Case
The original IEEE 802.1q standard defines much more than simply trunking. This standard also defined a Common Spanning Tree (CST) which is simply 1 spanning tree for the entire bridged network, regardless of the number of VLANs.

If the CST is applied to the topology of this next diagram, the result resembles the diagram shown here:
### 1. **IST (Internal Spanning Tree)**
- The **IST** is the spanning tree that runs inside an MST region.
- **Instance 0** is reserved as the IST for a region, while other MST instances (MSTIs) are numbered from 1 to 15.
- The IST is the only instance that sends and receives BPDUs within the MST region, encapsulating all other MST instance information (M-records).
- All VLANs are assigned to the IST by default.
- MST instances are local to a region; MSTI 1 in region A is independent of MSTI 1 in region B.
### 2. **CIST (Common and Internal Spanning Tree)**
- The CIST inside an MST region is the same as the IST (Instance 0).
- The CIST outside MST regions is equivalent to the CST (Common Spanning Tree).
### 3. **CST (Common Spanning Tree)**
- In 802.1D 
- The **CST** interconnects all MST regions and any 802.1D STP switches in the network.
- The CST ensures that all MST regions and non-MST spanning tree instances form a single unified spanning tree.
# STP Timer Optimsation
STP timers can be optimised by considering four key IEEE-defined delay parameters:
- BPDU transmission delay: Max value of 1 second. Time between sending/receiving BPDU frames.
- Bridge transit delay: Max value of 1 second. Time between sending/receiving frames.
- Medium access delay: Max value of 0.5 seconds. CPU to make forwarding decision.
- Transit halt delay: Max value of 1 second. Transition port to blocked state.
Description: These delays represent the time taken for data transmission, switching, and port state transitions in a network, ensuring efficient communication and network stability.

# STP Configurations

## All spanning tree config commands


```
! Global
!
spanning-tree <mode pvst|rapid-pvst|mst>
udld <enable|aggressive>
spanning-tree pathcost method <long|short>
spanning-tree extend system-id
spanning-tree loopguard
spanning-tree backbonefast (not needed for RSTP)
spanning-tree uplinkfast (not needed for RSTP)
spanning-tree portfast edge
spanning-tree portfast edge bpdufilter default
spanning-tree portfast edge bpduguard default
spanning-tree vlan 10 forward-time 15
spanning-tree vlan 10 hello-time 2
spanning-tree vlan 10 max-age 20
spanning-tree link-type point-to-point
!
interface gi1/14
 spanning-tree guard <loop|root|none>
 spanning-tree bpdufilter enable
 spanning-tree cost <1-200000000>
 spanning-tree vlan 10 root <incremenets of 4096>
!
```
## STP Versions
```
!
spanning-tree <mode pvst|rapid-pvst|mst>
!
! Spanning tree referring shows ieee, pvst or mstp. ieee refers to the BPDU type
!
show spanning-tree

VLAN0001
  Spanning tree enabled protocol [iee|pvst|mstp]
!
```
## Set Bridge Priority Per Vlan
```
! Auto config
!
spanning-tree vlan 10 root [primary|secondary]
!
! PVST and RPVST use extended system-id as command enabled by default
! You cannot turn off extend system-id
!
spanning-tree extend system-id
!
! Or manually
!
spanning-tree vlan 10 root <incremenets of 4096>
```
## STP Portfast Edge
```
!
! Gloabal default
!
spanning-tree portfast edge default
!
! Interface opverides
!
int g0/1
spanning-tree portfast <edge|network|disable>
spanning-tree portfast edge bpdufilter default
spanning-tree portfast edge bpduguard default
!
! portfast on trunk without trunk keyword won't work so add keword trunk
!
spanning-tree portfast edge [trunk|disable]
```
## Set Port Costs and Long/Short Mode
```
!
spanning-tree pathcost method <long|short>
!
! Per interface costs (default is 128)
!
int g0/1
spanning-tree cost <1-200000000>
```
## Set BPDU Filter
```
spanning-tree portfast bpdufilter default
!
! Enable BPDU Filtering on a per-interface basis - disabled STP!
!
Switch(config-if)# spanning-tree bpdufilter enable ## Verification Commands
```
## Show Commands
```
SwitchA#show spanning-tree

VLAN0001
  Spanning tree enabled protocol pvst
  Root ID    Priority    24576
             Address     32769 000c.d3ad.b33f
             Cost        19
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32768
             Address     32770 000c.d3ad.b33f
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 15

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- ----
Fa0/1               Root LRN 19        128.1    P2p
Fa0/3               Desg LRN 19        128.3    P2p
Fa0/8               Desg LRN 19        128.8    P2p
```

```
SwitchA#show spanning-tree root

                                        Root    Hello Max Fwd
Vlan                   Root ID          Cost    Time  Age Dly  Root Port
---------------- -------------------- --------- ----- --- ---  ------------
VLAN0001         32769 000c.d3ad.b33f         0    2   20  15
VLAN0002         32770 000c.d3ad.b33f         0    2   20  15
VLAN0010         32778 000c.d3ad.b33f         0    2   20  15
VLAN0020         32788 000c.d3ad.b33f         0    2   20  15
VLAN0030         32798 000c.d3ad.b33f         0    2   20  15
VLAN0040         32808 000c.d3ad.b33f         0    2   20  15
```

```
SwitchA#show spanning-tree vlan 1

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     000c.d3ad.b33f
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000c.d3ad.b33f
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- -----------------------------
Fa0/22              Desg FWD 19        128.22   P2p
Fa0/23              Desg FWD 19        128.23   P2p
Fa0/24              Desg FWD 19        128.24   P2p
Fa0/46              Desg FWD 19        128.46   P2p
Fa0/47              Desg FWD 19        128.47   P2p
Fa0/48              Desg FWD 19        128.48   P2p
Gi0/1               Desg FWD 4         128.49   P2p
Gi0/2               Desg FWD 4         128.50   P2p
```

```
SwitchA#show spanning-tree interface fastethernet 0/1

VLAN                Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- -----------------------------
VLAN0001            Root FWD 19        126.1    P2p
VLAN0002            Altn BLK 19        126.2    P2p
```
## Debug Commands

```
SwitchA#debug spanning-tree events
```

```
SW2#debug spanning-tree bpdu
```

You can use the `debug spanning-tree bpdu` command to view BPDUs that are sent or received.

```
SW2#
STP: VLAN0001 rx BPDU: config protocol = rstp, packet from FastEthernet0/14  , linktype IEEE_SPANNING , enctype 2, encsize 17 
STP: enc 01 80 C2 00 00 00 00 11 BB 0B 36 10 00 27 42 42 03 
STP: Data     000002023C10010011BB0B36000000000010010011BB0B360080100000140002000F00
STP: VLAN0001 Fa0/14:0000 02 02 3C 10010011BB0B3600 00000000 10010011BB0B3600 8010 0000 1400 0200 0F00
RSTP(1): Fa0/14 repeated msg
RSTP(1): Fa0/14 rcvd info remaining 6
RSTP(1): sending BPDU out Fa0/16
RSTP(1): sending BPDU out Fa0/17
STP: VLAN0001 rx BPDU: config protocol = rstp, packet f
```


### References

[MST and affect of VTP pruning single VLANs (802.1s) - Cisco](https://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/24248-147.html#two_vlans_same)
[LAN switching first-step :: Networking :: eTutorials.org](https://etutorials.org/Networking/lan+switching/)
[Catalyst 6500 Release 12.2SX Software Configuration Guide - STP and MST \[Cisco Catalyst 6500 Series Switches\] - Cisco](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst6500/ios/12-2SX/configuration/guide/book/spantree.html#wp1098444)
[[STP]]
