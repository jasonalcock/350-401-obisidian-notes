---
aliases: 
publish: 
---

%%
date:: [[2024-09-26]]
parent:: 
%%

# [[VLANs]]

IEEE Introduced VLANs with 802.1q but the standard also defined a Common Spanning Tree (CST) which is simply 1 spanning tree for the entire bridged network of 802.1 switches, regardless of the number of VLANs. Cisco had their own version for VLAN standard called ISL which was supported by PVST then PVST+ came out to support per VLAN spanning trunk on 802.1q switches. All ports on a Cisco switch are by default assigned to VLAN1.

```js
vlan 199
 name DEFAULT
 exit
!
int g0/0
switchport access vlan 199
```

## 802.1q Tag

![[Pasted image 20240926184104.png]]

- Under 802.1Q, the maximum ethernet frame size is extended by 4 bytes from 1,518 bytes to 1,522 bytes.
- Two bytes are used for the tag protocol identifier (TPID)
- Two bytes for tag control information (TCI). The TCI field is divided into PCP, DEI, and VID.
	- Tag protocol identifier (TPID): A 16-bit field set to a value of 0x8100
	- Tag control information (TCI) A 16-bit field containing the following sub-fields:
		- Priority code point (PCP) CoS 3-bits to prioritise different classes of traffic.
		- Drop eligible indicator (DEI) 1-bits
		- VLAN identifier (VID): 12-bits specifying VLANs 0 and 4095 reserved. 1-4,094 VLANs OK.

## VLAN 1 and Native

- You can never delete VLAN 1
- By default switch control plane traffic (CDP, STP) sent via VLAN 1
- By default VLAN 1 traffic is sent untagged on trunk ports
- If the native VLANs are not the same then the VLANs are bridged. 
- By default all VLAN traffic is tagged except for traffic from VLAN 1 on trunk ports. 
- By default trunks carry all VLANs.

```js
vlan 199
 name NEW-DEFAULT
 exit
! 
!- Always tag native traffic trunk links
vlan dot1q tag native
!
!- Set native vlan on physical or portchannel interface with trunk
!
int g0/0
switchport trunk native vlan 199
```

## Trunkport Encapsulation 

```js
(config-if)#switchport trunk encapsulation ?
 dot1q      Only 802.1q trunking encapsulation when trunking
 isl        Only ISL trunking encapsulation when trunking
 negotiate  Negotiate trunking encapsulation with peer
```

## Manually Prune VLANs from Trunk

```js
int g0/0
 switchport trunk allowed vlan 3,9,11-15
 switchport trunk allowed vlan remove 12
 switchport trunk allowed vlan add 25
 switchport trunk allowed vlan except 50-99 
 switchport trunk allowed vlan all 
```

## Dynamically Configure Port Trunking with DTP

- Best used for initial deployment then switched off and used to negotiate port type is access or trunk and trunking protocol.
- It is unauthenticated and packets can be spoofed but only useful if attacker can get onto a non-access port. Trunk ports by default allow all VLANs.
- It is there to dynamically set the mode of the interface access or trunk and also to choose a trunking protocol.
- DTP is enabled by default in “dynamic auto” or “dynamic desirable” mode so they can become trunk ports automatically.
- DTP has 3 options:
	- trunk - trunk is switched on for port but it still sends DTP packets to get neighbour to become trunk
		- trunk mode will not auto select encapsulation protocol, it's manually set.
	- dynamic auto - doesn't send DTP packets, negotiates mode and prefers to be an access port
	- dynamic desirable - sends DTP packets, negotiates to become a trunk and prefers to be a trunk
- Cisco recommends Desirable-Desirable mode for all trunk ports as I think they both become trunks rather than start as trunks.
- Switchport encapsulation is ISL by default when using dynamic
- If dynamic options are used and you want dot1q then that has to be set as encap is auto neg by default
- If 2 sides of a link are set to use mode dynamic auto they will not form a trunk and become access ports
- DTP sends frames every 30s

### DTP Configuration

```js
!
switchport trunk encapsulation <dot1q|isl|negotiate>
switchport mode <trunk|dynamic auto|dynamic desirable>
!
! Turn off DTP
!
int g0/0 0 
!
! if using dot1 I don't see the advtange of having DTP so you could trun it off
!
switchport nonegotiate
!
! Cisco recommended way to setup a dot1q trunk
!
switchport trunk encapsulation dot1q 
switchport mode trunk
```

### DTP Verification

```js
SW1#show interfaces gi0/1 switchport        
Name: gi0/1
Switchport: Enabled
	Administrative Mode: dynamic auto
	Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
	Negotiation of Trunking: On
```

### DTP Port Mode Combination Outcomes

|            |                       | sw2 port 1 | sw2 port 1       | sw2 port 1            |
| ---------- | --------------------- | ---------- | ---------------- | --------------------- |
|            | SwitchPort Modes      | **Trunk**  | **Dynamic Auto** | **Dynamic Desirable** |
| sw1 port 1 | **Trunk**             | Trunk      | Trunk            | Trunk                 |
| sw1 port 1 | **Dynamic Auto**      | Trunk      | Access           | Trunk                 |
| sw1 port 1 | **Dynamic Desirable** | Trunk      | Trunk            | Trunk                 |
|            |                       |            |                  |                       |

To disable DTP negotiation:

- Configure the interface for access mode as access ports do not send or process DTP packets.
- Hard setting an interface to mode trunk does not turn off DTP.
- Use the `switchport nonegotiate` interface command to disable DTP.

## Share VLAN Config amongst Switches with VTP

```js
#### VTP Virtual Trunking protocol - multicast packets
!
! Maintains a consistant VLAN DB across switches
! VTP packets sent on trunk ports only
! If in transparent it passes on VTP packets only and maintains its own table
! v3 4096 vlans
!
vtp mode [client|server|transparent|off]
!
vtp version <1|2|3>
!
! v1/v2 always reset revision number by changing name or switch modes
!
vtp domain NAME
vtp mode transparent then 
vtp mode client
!
! v3 no need to change revision - set server to primary - 1 per domain
!
vtp primary
!
vtp domain MYDOMAIN
vtp password P@SSWORD!
!
! VTP stretches vlans across a domain via trunks so broadcast/multicast traffic everywhere
! VTP pruning allows a switch to learn which VLANs are active on its neighbors.
! Thus, broadcasts are only sent out the necessary trunk ports 
! With mst if you prune a vlan from an active forwarding path you'll bring it down
!
vtp pruning
!
int gi0/0
switchport trunk pruning vlan <vlan number, range or list|add|remove|except|none>
!

```

## VLAN Hopping

Older switches if an access port receives a packet with a VLAN tag containing a VLAN ID **which is the same as that assigned to the access port**, the frame will be accepted, and the tag will be stripped. If the VLAN ID is not the same as that assigned to the access port, the packet is dropped without learning its source MAC.

## MAC Address Tables and VLANs

VLAN aware switches associate MAC address with their vlan and port 

`show mac address-table [address mac-address | dynamic | vlan vlan-id]`

## References

[Solved: Why DTP is used? - Cisco Community](https://community.cisco.com/t5/switching/why-dtp-is-used/td-p/1377495#)