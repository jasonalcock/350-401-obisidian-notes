---
aliases: 
publish: 
---
%%
date:: [[2024-09-26]]
parent:: 
%%
# [[EtherChannels]]
- Groups multiple interfaces/ports together to form 1 virtual interface/PortChannel.
- LACP is a protocol to build LAGs automatically, versus static. You can usually build an 802.1AX (old working group was 802.1ad) LAGs without using LACP. Many devices support static and dynamic LAGs.
- It increases the available bandwidth or provides redundancy.
- The virtual interface bandwidth combines all it's member interfaces together and load balances traffic across them.
- EtherChannel is Cisco proprietary and IEEE 802.3ad is open and are both "Link aggregation"standards, are very similar and accomplish the same goal.
- EtherChannel supports dynamic link aggregation protocols. These protocols:
	- Dynamically establish and maintain the port channel interface.
	- Maintains the EtherChannel by check the member links L2 health and and can pass traffic.
![[Pasted image 20240926180813.png]]
	- If aggregation protocol traffic is not seen passing it will remove the member link and automatically failover.
	- **Best to use one of the dynamic link aggregation protocols**.
- The two LAG protocols available are PAgP which only works with Cisco kit and IEEE 802.3ad's LACP which will work with Cisco and other vendor kit.
- For catalyst switches we call bundling EtherChannel but on other Cisco platforms such as Nexus or servers we usually say port channels. 
- STP works only over the virtual interface not on the members ports.
- The Port Channel interface will only go down when all interfaces are down.
- I think a lot of the underlying mechanisms for PAgP and LACP are hidden and not needed for the CCNP exam.
- EtherChannels virtual interfaces can be access or trunk ports and with switched L2 or IP L3 interfaces.
## LACP (Part of the IEEE 802.3ad standard)
- Dynamically Builds the 802.3ad LAG group
- LACP advertises to the multicast address `0180.c200.0002`. 
- Has 2 modes - **active/passive** - LACP on and talking/LACP on but listening only
- Supports up-to 8 active and 8 standby members
- The default System/Link priority value is 32768
## Port Aggregation Protocol PAgP (Cisco proprietary)
- Uses multicast address `0100.0ccc.cccc` with the protocol code `0x0104`.  
- Has 2 modes **desirable/auto** - on and talking/on and listening only
- Use non-silent connecting 2 PAgP-compliant switches together; link established faster
- Think is is very similar to LACP
## Manual (802.3ad LAG but no LACP)
- This enables EtherChannel bundling without any link aggregation protocol.
- The EtherChannel PortChannel interface stays up at all times.
- If a member interface is not passing traffic the port-channel interface will remain up and traffic gets black-holed.
- **on** - Builds EtherChannel LAG without LACP. Don't do this unless you have to. e.g. some devices don't support LACP like WLC and you want to push data up.
# Configuration
## Creating The EtherChannel
```
!
channel-group auto (LACP EtherChannel but No LACP Aggregation)
channel-group 1 mode <active|passive|desirable|auto|on>
!
```
- Physical port properties, such as speed and duplex configured on physical not port-channel interface.
## Port Channel Interfaces
- Can have up to 48 and can be configured before interfaces are assigned to the port-channel
```
interface Port-channel 1
 switchport mode <trunk|access>
```
## Port Settings That Must Match For Port Channel
```
interface range gi2/23 – 24
mtu 1500
speed 100
duplex full
switchport mode access
switchport mode trunk
switch trunk native vlan 1

```

interface range GigabitEthernet 0/1 - 2
no switchport

## Advanced LACP Configuration Options
### LACP Error Detection Fast/Slow
- LACP Slow - LACP Packets sent every 30 seconds. A link unusable after three intervals (90 seconds). Int removed from group.
- LACP Fast - packets sent every second. 3 seconds detection and link is removed from group.
- Has to be same on all switches otherwise it will not come up.
```
interface range gi1/0/1-2”
lacp rate <fast|slow>
```
### Limit Max. No. of PortChannel Ports 
- Default is 4, max value is 8.
- If you have 3 active links, the hashing may not distribute the load as evenly since the hashing algorithm is optimised for powers of two.
- However, if you limit it to 2 or increase to 4, the load-balancing efficiency improves significantly.
- Also if you have 2 links and want the 2nd to be a Hot-standby then set this to 1”
```
int port-channel1
 lacp max-bundle 2
```
### LACP System Priority
- Primary switch gets to choose which interfaces/links become active if the available no. of interfaces in the port-channel > maximum interfaces setting. Like STP lower values are preferred. 0 means it will always become active 1st.
- Default is 32768 and in a tie break it uses the lowest base MAC address.
- Only needs to be set on primary.
```
int port-channel1
 lacp system-priority 1
```
### LACP Port Priority
- The Primary switch gets to choose the ports/links become active, when the no. of ports > maximum port setting, using port priority. Like STP lower values are preferred. 0 means it will always become active 1st.
- Set on the interface

```
interface gi 0/1
lacp port priority 0
```

# Show Commands
## Show EtherChannel Load Balancing 
```
sw1#show etherchannel load-balance
EtherChannel Load-Balancing Configuration:
        src-dst-ip

EtherChannel Load-Balancing Addresses Used Per-Protocol:
Non-IP: Source XOR Destination MAC address
  IPv4: Source XOR Destination IP address
  IPv6: Source XOR Destination IP address
!
```
## Show EtherChannel Summary 
```
sw1(config)#do show EtherChannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

        M - not in use, minimum links not met
        m - not in use, port not aggregated due to minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

        A - formed by Auto LAG


Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)          -        Gi0/0(P)    Gi0/3(P)

```
## Show Int PortChannel for Port L2 info
- BW is for 2 1 gig ports combined 2000000 Kbps which is used by QoS, STP RPC and for routing protocol calculations.
```
sw2#show int po1
Port-channel1 is up, line protocol is up (connected)
  Hardware is EtherChannel, address is 5000.0002.0002 (bia 5000.0002.0002)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:00:01, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     2156 packets input, 117204 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     520 packets output, 70564 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```
## EtherChannel Neighbour Info + Port Priority
```
sw2# show etherchannel port
		Channel-group listing:
		----------------------

Group: 1
----------
		Ports in the group:
		-------------------
Port: Gi0/2
------------

Port state    = Up Mstr Assoc In-Bndl
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/2     SA      bndl      32768         0x1       0x1     0x3         0x3D

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/2     SA      32768     5000.0001.8000  12s    0x0    0x1    0x1     0x3D

Age of the port in the current state: 0d:01h:12m:52s

Port: Gi0/3
------------

Port state    = Up Mstr Assoc In-Bndl
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/3     SA      bndl      32768         0x1       0x1     0x4         0x3D

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/3     SA      32768     5000.0001.8000  12s    0x0    0x1    0x4     0x3D

Age of the port in the current state: 0d:00h:05m:50s
```
## Show LACP System ID
```
sw1#show lacp sys-id
32768, 5000.0001.8000
```
## Show LACP Counters
```
sw1#show lacp counters
             LACPDUs         Marker      Marker Response    LACPDUs
Port       Sent   Recv     Sent   Recv     Sent   Recv      Pkts Err
---------------------------------------------------------------------
Channel group: 1
Gi0/0       195    176      0      0        0      0         0
Gi0/3       222    45       0      0        0      0         0

sw1#
```
## Show LACP Neighbour
```
sw1#show lacp neighbor
Flags:  S - Device is requesting Slow LACPDUs
        F - Device is requesting Fast LACPDUs
        A - Device is in Active mode       P - Device is in Passive mode

Channel group 1 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SA      32768     5000.0002.8000  24s    0x0    0x1    0x3     0x3D
Gi0/3     SA      32768     5000.0002.8000  18s    0x0    0x1    0x4     0x3D
```


## Show PAgP Neighbour
```
show pagp neighbor
Flags:  S - Device is sending Slow hello. C - Device is in Consistent state.
        A - Device is in Auto mode. P - Device learns on physical port.

Channel group 2 neighbors
        Partner              Partner          Partner         Partner Group
Port     Name                Device ID        Port       Age  Flags   Cap.
Gi1/0/3   SW2                0081.c4ff.8b00    Gi1/0/3     11s  SC     20001
Gi1/0/4   SW2                0081.c4ff.8b00    Gi1/0/4      5s  SC     20001
```
## Debug
```
debug etherchannel event
```
# Load Balancing
- Port-channel traffic uses a hash, not round-robin per packet.
- Each hash value is assigned to a specific link in a "sticky" manner.
- Packets are consistently forwarded over the same link based on their hash result.
- Hashes don’t change ports unless a link goes down or a change occurs.
- Load-balancing hash set using: port-channel load-balance hash.
- The default load-balancing mode for Layer 3 interfaces is the source and destination IP L4 ports, and the default load-balancing mode for non-IPtraffic is the source and destination MAC address.
- The default method for Layer 2 packets is src-dst-mac.
- The default method for Layer 3 packets is src-dst ip-l4
- Common hash options:
  - dst-ip: Destination IP address
  - dst-mac: Destination MAC address
  - dst-port: Destination TCP/UDP port
  - src-dst-ip: Source and destination IP addresses
  - src-dst-mac: Source and destination MAC addresses (default for L2 interfaces)
  - src-dst-port: Source and destination TCP/UDP ports (default for L3 interfaces)
  - src-ip: Source IP address
  - src-mac: Source MAC address
  - src-port: Source TCP/UDP port
- Only the following hashes support symmetric flows src-dst ip and src-dst ip-l4 port

## Config
### Change Load Balancing Hash
```
sw1(config)#port-channel load-balance ?
  dst-ip       Dst IP Addr
  dst-mac      Dst Mac Addr
  src-dst-ip   Src XOR Dst IP Addr
  src-dst-mac  Src XOR Dst Mac Addr
  src-ip       Src IP Addr
  src-mac      Src Mac Addr
```
### View Load Balancing Config & Hash Table
```
sw1#show etherchannel load-balance
EtherChannel Load-Balancing Configuration:
        src-dst-ip

EtherChannel Load-Balancing Addresses Used Per-Protocol:
Non-IP: Source XOR Destination MAC address
  IPv4: Source XOR Destination IP address
  IPv6: Source XOR Destination IP address
!
```


# References
- [1.1d EtherChannel - CCIE Study Blog](https://ccie.jd-networks.co.uk/2020/04/01/1-1d-etherchannel/)
- [LAG Nexus Config Guide](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/7-x/interfaces/configuration/guide/b_Cisco_Nexus_9000_Series_NX-OS_Interfaces_Configuration_Guide_7x/b_Cisco_Nexus_9000_Series_NX-OS_Interfaces_Configuration_Guide_7x_chapter_0111.pdf)
- [Link Aggregation Confusion | The Data Center Overlords](https://datacenteroverlords.com/2013/09/02/link-aggregation/)

