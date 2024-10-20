---
aliases: 
publish: 
---
%%
date:: [[2024-09-13]]
parent:: 
%%


```table-of-contents
```
# What Are FHRPs
- FHRPs enable a group of routers to share a VIP & VMAC for L3 gateway redundancy.
- HSRP is Cisco proprietary and iIve never seen GLBP or VRRP used on any Cisco production networks.
- FHRP routers respond to ARP requests with their VMAC and the exact operation depends on the protocol in use.
- All FHRPs support object tracking.
# HSRP (Hot Standby Routing Protocol)
- Is Cisco proprietary.
- Both versions support ipv4 and ipv6.
- HSRP version 2 supports advertising millisecond timers in its packets.
## Control Plane
- HSRPs control plane comms is via UDP multicast. For:
	- v1 `224.0.0.2:1985` all routers link local address for HSRP packets.
	- v2 `224.0.0.102:2029` for HSRP packets.
- The VMACs can be defined manually or auto generated. For:
	- v1 format `000.0c07.acXX` where `XX` is the group number 8 bits therefore max 256 groups.
	- v2 format `000.0c9f.fXXX` where `XXX` is the group number 12 bits therefore max 4096 groups.
- A HSRP group of 3 or more L3 devices share a VIP and VMAC and will elect 1 active and standby.
## 6 HSRP Router States
When a HSRP is enabled the router will transition through states 1 and 2 then onto **active** or **listen** via **speak** or end up in the **listen** state.
1. Initial - HSRP is initially inactive when just enabled, interface bounced or booting.
2. Learn - Waits to hear from active router to determine VIP (can configure routers without VIP therefore learn VIP via HSRP packet).
3. **Listen** - Listens for hellos ONLY from active & standby routers. 
4. Speak - Participating in an **election** for active/standby, sends hellos. Cannot enter this state without VIP.
5. **Active** - Device with highest priority (highest value) or if priority the same the device with the highest IP becomes active, responds to ARPs for VIP and sends and listens hellos.
6. **Standby** - Device with second highest priority (2nd highest value) or if priority is the same the device with the highest IP becomes standby, sends and listens to hellos.

One doesn't need anymore detail beyond the 6 possible HSRP states so don't worry about looking into the HSRP state machine. 
## UDP Hello Packet Format
| Version             | Op Code<br>0 = hello | State | Hellotime |
| ------------------- | -------------------- | ----- | --------- |
| Holdtime            | Priority             | Group | Reserved  |
| Authentication Data |                      |       |           |
| Authentication Data |                      |       |           |
| Virtual IP Address  |                      |       |           |
 - The Op Code can be hello, coup or resign. The coup message is sent by the standby router when it detects it has the higher priority. A resign message is sent by the active router
## HSRP Preempt
- Disabled by default
- **Preempt**: The act of doing something before others become aware. Routers can preempt other active/standby routers when they come online.
- If a HSRP router has preemption enabled with a higher priority or equal priority with a higher IP than the current active or standby HSRP routers will relinquish their role. 
- When higher priority HSRP router preempts it will send out a coup message type to the active router and it responds with a resign message.  
## Timers
Timers are per group and do not have to be the same on each device in a group.
### Active Timer
Monitors active router and starts any time a router receives a hello packet from the active router. Expires per the hold time value in the HSRP hello message. 
### Standby timer	
This timer is used in order to monitor the standby router. The timer starts any time the standby router receives a hello packet. This timer expires in accordance with the hold time value that is set in the respective hello packet.
### Hello timer
This timer is used to clock hello packets. All HSRP routers in any HSRP state generate a hello packet when this hello timer expires.
### Slow Down Preemption
It's a good idea to delay a device with preemption in certain conditions. Let's say there was a failover to a standby device and the previously active router with preempt has just come back online. Delaying the failover will allow the rebooted device to fully converge before taking back over.

If there are intermittent interface flaps on your neighbouring device with preempt then allowing it to take over immediately but for it to then fail shortly thereafter will cause instability and packet loss. 

I read somewhere the a reload time should be set to the measured reboot time + 50%
### Configurable Timers
- Default hello and dead timers are 3s and 10s.
- Dead timer should be x3 the hello interval.
- Never lower below 1 and 4.
- You lower the intervals to speed up detection of a neighbour going down and increase the speed of the failover.
- See the Cisco High Availability Campus Recovery Analysis guide for lowering hello and dead interval timers to 250ms and 800ms.
- There is no default delay values set for preempt or preempt reload. 
## Configuration Commands
- Note version 1 is used by default
### Example

```
! HSRP
!
interface g0/0
 standby 1 ip 10.10.1.1
 standby 1 preempt priority 110
 !--Vary hello and deadtime intervals to speed up down detection
 standby 1 timers 1 4
 !--Timers
 standby 1 preempt delay minimum 7 reload 90
 !--Authentication
 standby 1 authentication md5 key-string cisco
 !--Switch version
 standby version 2
 !--Native tracking
 
```
To become active router with highest priority or if a tie highest IP.
HSRP commands begin with **standby** which is handy to remember as the word is in the protocol title.
### Verification Commands
```
R1(config-if)#do show standby
GigabitEthernet0/1 - Group 1
  State is Listen
  Virtual IP address is 10.10.10.254
  Active virtual MAC address is unknown
    Local virtual MAC address is 0000.0c07.ac01 (v1 default)
  Hello time 3 sec, hold time 10 sec
  Preemption disabled
  Active router is unknown
  Standby router is unknown
  Priority 100 (default 100)
  Group name is "hsrp-Gi0/1-1" (default)
```

```
R1(config-if)#do show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Gi0/1       1    100 P Standby 10.10.10.2      local           10.10.10.254
```

```
R1#debug standby ?
  errors   HSRP errors
  events   HSRP events
  packets  HSRP packets
  terse    Display limited range of HSRP errors, events and packets
  <cr>     <cr>
```

---
# VRRP (Virtual Router Redundancy Protocol)
- One **master** router per group and the rest are **backup** routers.
- Preemption is enabled by default.
- VRRP version 1 supports ipv4 VIPs ONLY
- VRRPv3
	- supports v4 and v6 VIPs
	- Config requires global enabledment
	- Is backwards compatible via a switch
	- Cannot do authentication
- Timers can be set but there's no requirement to learn these for Encore exam
## Control Plane
- HSRPs control plane comms is via UDP multicast. 
	- Address `224.0.0.18` for all VRRP routerss link local address.
- The VMACs can be defined manually or auto generated.
	- format `000.5e00.01XX` where `XX` is the group number 8 bits therefore max 256 groups.
- A VRRP group of 3 or more L3 devices share a VIP and VMAC and will elect 1 master and backup.

## Configuration Commands
- Note version 1 is used by default
### V1 Example

```
ip sla 1
 icmp-echo 8.8.8.8
 frequency 10
!
ip sla schedule 1 start-time now life forever
!
Loopback0  
 ip address 1.1.1.1 255.255.255.0  
!  
track 1 interface Loopback0 line-protocol
no track 1
track 1 ip sla 1
!  
interface g0/0
 ip address 10.10.10.1 255.255.255.0
 vrrp 1 ip 10.10.10.100
 vrrp 1 priority 110
 !--Object tracking
 vrrp 1 track 1 decrement 15
 !--Authentication
 vrrp 1 authentication md5 key-string cisco
```

### V2 Example

```
!--enable vrrp version globally
fhrp version vrrp v3
!
interface g0/0
 ip address 10.10.10.1 255.255.255.0
 vrrp 3 address-family ipv4
  address 10.10.10.100
  priority 110
  track 1 decrement 15
 ```
### Verification Commands Are Same for Both v1 and v2

#### v1 show vrrp
```
R1(config-if)#do show vrrp
GigabitEthernet0/1 - Group 1
  State is Master
  Virtual IP address is 10.10.10.100
  Virtual MAC address is 0000.5e00.0101
  Advertisement interval is 1.000 sec
  Preemption enabled
  Priority is 110
    Track object 1 state Up decrement 15
  Authentication text, string "test"
  Master Router is 10.10.10.1 (local), priority is 110
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.570 sec
```
#### v2 Show VRRP
```
R1(config-if)#do sh vrrp

GigabitEthernet0/1 - Group 1 - Address-Family IPv4
  State is MASTER
  State duration 8 mins 24.888 secs
  Virtual IP address is 10.10.10.50
  Virtual MAC address is 0000.5E00.0101
  Advertisement interval is 1000 msec
  Preemption enabled
  Priority is 110
    Track object 1 state UNDEFINED decrement 20
  Master Router is 10.10.10.1 (local), priority is 110
  Master Advertisement interval is 1000 msec (expires in 579 msec)
  Master Down interval is unknown>)
  ```

#### v1 Show Brief
```
R1(config-if)#do show vrrp brief
Interface          Grp Pri Time  Own Pre State   Master addr     Group addr
Gi0/1              1   110 3570       Y  Master  10.10.10.1      10.10.10.100
```
#### v2 Show Brief
```
R1(config-if)#do sh vrrp bri
  Interface          Grp  A-F Pri  Time Own Pre State   Master addr/Group addr
  Gi0/1                1 IPv4 110     0  N   Y  MASTER  10.10.10.1(local) 10.10.10.50
  ```

#### Debug
```
R1#debug vrrp ?
  all      Debug all VRRP information
  auth     VRRP authentication reporting
  errors   VRRP error reporting
  events   Protocol and Interface events
  packets  VRRP packet details
  state    VRRP state reporting
  track    Monitor tracking
```

---
# GLBP (Global Load Balancing Protocol)
- It's Cisco proprietary and provides gateway **Redundancy** as well as **Load Balancing** across gateways.
- Uses UDP multicast of 224.0.0.102:3222
- There can be up to 1024 AVFs per group.
- GLBP routers have 2 roles
	- **AVFs** (Active Virtual Forwarders) do the actual routing of traffic
	- **AVGs** (Active Virtual Gateways) manage AVFs and how traffic is LB across the AVFs
- AVGs responds to broadcast ARP requests with unicast response with a VMAC of an AVF.
## AVG Redundancy
The AVG is essential and manages redundancy and load balancing. It uses priority like VRRP and HSRP. The default priority value is 100 and the AVGs IP is used in tie break situations. AVG preemption is disabled by default. The AVG states are x1 active, x1 standby and any other router will  go into a listen state.
## AVF Operation
- There can only be 4 AVFs per group and the remaining routers become SVFs (standby). 
- Each AVFs is assigned a number and a VMAC. If there's < AVFs than VMACs then VMACs are distributed to the remaining AVFs. 
- The VMAC is only **active** on one AVF but can float between AVFs during failover so other AVFs go into a **listening** state for that VMAC.  
- The VMAC or AVF doesn't determine the load balancing it's the AVG that does by responding to ARP requests but the AVFs are the ones doing the actual forwarding.
- If an AVF fails and is active for a VMAC the AVG will move the VMAC to another AVF.
- The AVFs use **weight** (default 100) to determine if they can become a virtual forwarder. If AVFs have an equal weight their IP is used as a tie breaker.
- If their weight is below the lower threshold it's disabled and if it's above it is enabled. Object tracking can be used to alter weights. The default thresholds are 1 to 100. 
- Preemption is enabled by default and can be configured with a delayed start.
## Load Balancing
-  Is controlled by the AVG so it only needs configuring on the AVG.
- The AVG controls how the clients then get load balanced across the Active Virtual Forwarders (AVFs) so the load balancing options need only to be configured on the AVG.
- Load balancing algorithms are:
	- **Round Robin**
		- AVG gives out next VMAC of next AVF for each ARP request
		- This is GLBPs default algorithm
	- **Host**
		- Sticky AVG responds to same SMAC ARPs from same VMAC
	- **Weighting**
		- Hands out AVFs MAC for an ARP based on it's weight amongst the group
		- 
### VMACs
The format `0007.b4xx:xxyy` where:
	- xx.xx is the GLBP group
	- yy is the AVF number.
  
Example:
```
0007.b400.0101
  |    |    |
  |    |    01 = AVF number
  |    00.01 = GLBP groupnumber
```


## Configuration Commands
### V1 GLBP Using Weighted Load Balancing

```
!--R1
interface GigabitEthernet0/0
 glbp 1 weighting 200
!
!--R2
interface GigabitEthernet0/0
 glbp 1 weighting 100
!
!--R3
interface GigabitEthernet0/0
 glbp 1 weighting 80
!
!--R4
interface GigabitEthernet0/0
 glbp 1 weighting 20
!
!--R5
interface Loopback0  
 ip address 1.1.1.1 255.255.255.0  
!  
track 1 interface Loopback0 line-protocol  
!  
interface GigabitEthernet0/1
 mac-address 0000.0000.0005
 ip address 10.10.10.5 255.255.255.0
 glbp 1 ip 10.10.10.254
 glbp 1 priority 105
 glbp 1 load-balancing weighted
 glbp 1 weighting 100 lower 90 upper 95  
 glbp 1 weighting track 1 decrement 20
!
```
Each AVF has it’s vMAC used in a percentage of the ARP replies equal to it’s weight divided by the sum of all AVF weights.  In this case, that means each AVF receives:
**R1:**    200 / (200 + 100 + 80 + 20)    = 50%, or 10 clients
**R2:**    100 / (200 + 100 + 80 + 20)    = 25%, or 5 clients
**R3:**    80 / (200 + 100 + 80 + 20)    = 20%, or 4 clients
**R4:**    20 / (200 + 100 + 80 + 20)    = 5%, or 1 client
### Verification and Troubleshooting
```
# show glbp
  Shows who’s active.
  Shows who’s standby.
  HSRP/VRRP/GLBP statuses.
  Group, status, version.
  
# show standby + mac + auth.
  Shows delays, hold state changes, version.
  
# debug standby events/packets.
  HSRP/VRRP/GLBP debugging.
  
# debug standby state changes.
  Clearly shows priority/timers/standby/HSRP/auth/MAC.
```


### GLBP Specifics:


```
GLBP weighted track
1 per AVG, 1 per AVF
```

```
# glbp 1 weighting 100 (lower)
```

```
# glbp 1 track 1 decrement 10
```

- Track 2 for VIP in subnet.
- Take active/standby with higher priority and track for GLBP.
---
# Object Tracking
- A FHRP device can use object tracking to track if an interface is in the up state or a route exists in the route table then reduce the FHRP priority so another router can become active. 
- In a dual homed site with a primary and secondary WAN link connected to each FHRP router the router with the primary WAN interface would be the active router. If the primary interface goes down the secondary router can become the active device.
- Tracking works with all FHRPs
## Object Tracking Configuration

### Track SLA
```
ip sla 1
 icmp-echo 1.1.1.1
 frequency 10
 exit
!
track 1 ip sla 1 reachability
!
int g01
 standby 1 track 1 decrement 15
```
### Track Interface
```
track 8 interface g0/0 line-protocol
!
int g01
 standby 1 track 1 decrement 15
```
### Track Route
```
(config)# track 1 ip route x.x.x.x /30
(config)#int g01
(config-if)#standby 1 track 1 decrement 15
```
### Track Lists
```
track 1 ip route 3.3.3.3/32 reachability
track 2 ip route 1.2.3.4/32 reachability
track 3 ip route 10.0.20.2/32 reachability
track 4 ip route 10.0.20.0/24 reachability
!
track 5 list boolean and
 object 1
 object 2
!
track 6 list boolean or
 object 3
 object 4
```
### Track Verification:
```
do sh run | sec track
show standby track
```

---
# FHRP Comparison

| FHRP | Cisco Protocol | Preempt By Default  | Load Balancing | Roles                 | Default Priority           | Groups Per Interface          | Auth.                  | VIPs                   | Timers   | Native Interface Tracking            |
| ---- | -------------- | ------------------- | -------------- | --------------------- | -------------------------- | ----------------------------- | ---------------------- | ---------------------- | -------- | ------------------------------------ |
| HSRP | Yes            | No                  | No             | active,standby        | 100                        | v1 256, v2 4096               | Yes                    | Not int. IP            | 3 & 10   | Only for older IOSs. Now object only |
| VRRP | No             | Yes                 | No             | master,backups        | 100                        | 256                           | VRRP v1 Yes VRRP v2 No | Can be IP on interface | 1 & 6    | object only                          |
| GLBP | Yes            | AVGs No<br>AVFs Yes | Yes            | AVG (+standby) & AVFs | AVG is 100<br>AVFs are 100 | 1024 but may depend on device | Yes                    | Not int. IP            | AVG 3 10 | object for weight tracking only      |
|      |                |                     |                |                       |                            |                               |                        |                        |          |                                      |

- HSRP and GLBP hello and hold timers are communicated by the active router's hello messages, and timers learned from the active router override manually configured timers. Actual and configured timers are reflected in the show <glbp|hsrp> output. 
---
#  Links
- [Troubleshoot HSRP Problems in Catalyst Switch Networks](https://www.cisco.com/c/en/us/support/docs/ip/hot-standby-router-protocol-hsrp/10583-62.html#:~:text=Possible%20values%20are%3A%200%20%2D%20hello,to%20be%20the%20active%20router.)
- [GLBP Weights, Load Balancing, and Redirection « Cisconinja’s Blog](https://cisconinja.wordpress.com/2009/02/11/glbp-weights-load-balancing-and-redirection/)
- [HSRP VRRP and GLBP « Cisconinja’s Blog](https://cisconinja.wordpress.com/category/hsrp-vrrp-and-glbp/)
- [[GLBP in depth guide - Cisco Community](https://community.cisco.com/t5/networking-knowledge-base/glbp-in-depth-guide/ta-p/3987525)](https://cisconinja.wordpress.com/category/hsrp-vrrp-and-glbp/)[Fetching Title#a76s](https://cisconinja.wordpress.com/category/hsrp-vrrp-and-glbp/)
- [[ARP]]
- [ENCOR Training » Gateway Load Balancing Protocol GLBP Tutorial](https://www.digitaltut.com/gateway-load-balancing-protocol-glbp-tutorial#:~:text=A%20GLBP%20group%20only%20has,address%20in%20GLBP%20is%200007.)

