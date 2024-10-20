---
aliases: 
publish: 
---
%%
date:: [[2024-09-27]]
parent:: 
%%
# [[EXAM 27th Sep 350-401]]

VLAN components
- LAB extend VRF over tunnel between two devices then protect the tunnel using the provided config?
- Configure a BGP peer without using address family then advertise to the directly connected ebgp peers. Peers were already configured. Peers was idle until I activated it under address family? Did i need to do this. Also had to set something else to get network to advertise as bgp peers were directly connected.
- OSPF  didn't look at the lab but and forgot what was asked.
- STP didn't look at the lab but wasn't worried.


BGP
- Why did router 1 prefer BGP route y when it got route y from 2 BGP neighbours who looked like the out showed they route had matching attributes. Possible answer was MED or lower router ID.
- 

FHRP
- HSRP question on priority value, which router would be preferred, 1 with higher or lower.

Wireless
- what antenna makes a radiation pattern that looks like two circles side by side 
- when would one deploy an external antenna indoors? 
- what does the cable antenna and device power and something else give you?
- What mode enables connected using to keep using the AP when the AP loses connection to its manager
- What layer 2 method should I configure in WLC when enabled SSO. Is it wpa3?
- how do you measure tells you about the power level after a wireless survey, RSSI?
- When trying to connect 2 external sites by wireless what type of antenna, yang, omnidirectional or?
- What is DBi?
- What access list do I need to put on a wireless controller to allow users to connect to a splash page
- Why does a user temporarily lose connectivity when roaming between WLC is it different group names or something to do with a VIP?

VXLAN
- what is the outer vlan id used for in VXLANs
- what device encapsulated and decapsulates VXLAN
- What do endpoints connect to in vxlan?
- what device can extend vx-lan or sd-wan to external networks or non sdn compnonets
- what format should json have? Can it start with a list, curly braces etc
- SD-WAN and how to deploy wireless without wireless interfacing with the fabric

AAA
- Ig applied aaa authentication default login local group NAME tacacs+ what does this do exactly
- Without aaa default login local  will a user be limited to tacacs+ login only
- How do I ensure VTY lines can only be logged in using clear text protocol from i 1.1.1.1 and use aaa

Security
- What is least privalges model and where does it come from?
- What features of modern next generation firewalls? IPS?
- What is trustsec?

ACLs
- how do I configure a time based ACL to limit telnet access for a specific ip for the weekend only but allow all other traffic?
- How do I configure and apply vlan ACLs on a VLAN interface to deny ip traffic to host x but allow all others using ACLs?

Multicast
- How does PIM work?

NTP:
- What is stratum levels for
- Is stratum 1 and authortive stratum server?

QoS
- Can policing be applied both and and out bound?

SPAN
- How do I configure a remote span?
- How do I filter span traffic?

Debug filters
- Can I debug packet and apply a filter based on source and destination interface or is it just regular IP acls only?

Virtualisation
- What is a type 2 hypervisor
- What are components of a type 2 hyervisor, host os, ram etc?

Characteristic best describes

Automations
- Does YANG use JSON schema and containers?
- What sdwan device can perform stun? vmanace, vcentre, vbond?
- Python how open plain text file, convert it to json file then and print json dumps to screen
- Applet script on how to run a show command and append to flash file
- How do I secure REST? SSL
- What security does REST provide?
- What do data models provide us with?
- How or what interacts with the southbound API?
- There was a question on the Northbound API.
- What are the agentless and asgent based automation tools cmmonly used with Cisco devices?

Assurance
- What IP SLA thing requires it to be confugured at both ends.
- How do I send multiple flows to the same IP? Different UDP ports.
- An extended ping of 1500 byte payload was failing to router and intermediate router responded with icmp unreachable mtu 1476



Here's the revised, condensed version with full configuration examples and smaller headings.

---

## VLAN Components
### Extending VRF over Tunnel
- **Q:** How can you extend VRF over a tunnel between two devices and protect the tunnel using the provided config?
- **A:** Use MPLS L3VPN or GRE/IPSec for extending VRF over tunnels. Example:

```bash
interface Tunnel0
 ip vrf forwarding CUSTOMER_A
 tunnel protection ipsec profile IPSEC_PROFILE
```

### BGP Configuration
- **Q:** Should you configure a BGP peer without using address family, and why was it idle until activated under the address family?
- **A:** You need to activate peers within the address family:
```bash
router bgp 65001
 neighbor 192.168.1.1 remote-as 65002
 address-family ipv4
  neighbor 192.168.1.1 activate
```

- **Q:** How do you advertise directly connected networks to eBGP peers?
- **A:** Use the `network` command or `redistribute connected`. Example:
```bash
router bgp 65001
 network 10.10.10.0 mask 255.255.255.0
```
If peers are directly connected, use `neighbor <ip> next-hop-self`.

## OSPF and STP
- Details were not provided to address specifics.

---

## BGP
### Route Preference
- **Q:** Why did Router 1 prefer BGP route 'y' when received from two neighbors with matching attributes?
- **A:** If all attributes match, BGP prefers:
  - Lower **Router ID**
  - Lower **MED** if available

---

## FHRP (HSRP)
### Priority
- **Q:** Which router is preferred, one with a higher or lower priority value?
- **A:** Router with **higher priority**. Example HSRP config:
```bash
interface GigabitEthernet0/0
 standby 1 priority 150
 standby 1 preempt
```

---

## Wireless
### Antenna and Patterns
- **Q:** What antenna creates a side-by-side circle pattern?
- **A:** **Dipole antenna** (figure-8 pattern).

### FlexConnect Mode
- **Q:** What mode allows clients to use the AP when it loses WLC connection?
- **A:** **FlexConnect** mode.

### Signal Strength
- **Q:** What measurement indicates power levels? 
- **A:** **RSSI (Received Signal Strength Indicator)**.

### Connecting Sites
- **Q:** What antenna type for point-to-point external sites?
- **A:** **Yagi antenna**.

### Access Lists
- **Q:** What ACL is needed on a WLC for splash page access?
- **A:** Permit HTTP/HTTPS from guest VLAN.

---

## VXLAN
### Components
- **Q:** What device encapsulates/decapsulates VXLAN?
- **A:** **VTEP**. Example VXLAN config:
```bash
interface nve1
 member vni 5001
  associate-vrf
```

### JSON Format
- **Q:** Can JSON start with a list or curly braces?
- **A:** Yes, both `[]` and `{}` are valid.

---

## AAA
### Authentication
- **Q:** What does `aaa authentication default login local group NAME tacacs+` do?
- **A:** Uses the local database first, then falls back to TACACS+.

- **Q:** Restrict VTY lines to clear text from IP 1.1.1.1?
- **A:** Example:
```bash
access-list 10 permit 1.1.1.1
line vty 0 4
 access-class 10 in
```

---

## Security
### Least Privilege Model
- **Q:** What is it?
- **A:** Users have minimal access rights necessary for tasks.

---

## ACLs
### Time-Based ACLs
- **Q:** How to limit Telnet access for weekends?
- **A:** Example:
```bash
time-range WEEKEND
 periodic weekend 00:00 to 23:59
access-list 101 permit tcp any host 192.168.1.10 eq telnet time-range WEEKEND
```

### VLAN ACLs
- **Q:** How to deny IP traffic to host X?
- **A:** 
```bash
vlan access-map VLAN_FILTER 10
 match ip address HOST_X
 action drop
vlan filter VLAN_FILTER vlan-list 10
```

---

## Multicast
### PIM
- **Q:** How does PIM work?
- **A:** Uses RPF checks and operates in dense or sparse mode.

---

## NTP
### Stratum Levels
- **Q:** What are they?
- **A:** Indicate distance from the reference clock (stratum 1 = authoritative).

---

## QoS
### Policing
- **Q:** Can it be applied both inbound and outbound?
- **A:** Yes, e.g., `service-policy input POLICE-IN`.

---

## SPAN
### Remote SPAN
- **Q:** How to configure it?
- **A:** 
```bash
monitor session 1 source remote vlan 100
monitor session 1 destination interface Gi1/0/1
```

---

## Debug Filters
### Filtering
- **Q:** Can debug packet filter by interfaces or just ACLs?
- **A:** Can use both interfaces and ACLs.

---

## Virtualisation
### Type 2 Hypervisor
- **Q:** What is it?
- **A:** Runs atop a host OS (e.g., VirtualBox).

---

## Automation
### YANG and JSON
- **Q:** Does YANG use JSON schema?
- **A:** Yes, YANG can be represented in JSON.

### Python Conversion
- **Q:** Convert text to JSON:
  ```python
  import json
  with open('file.txt') as f:
      data = f.read().splitlines()
  json_data = json.dumps(data, indent=2)
  print(json_data)
  ```

### Applet Script Example
- **Q:** How to append a show command output to a flash file?
```bash
event manager applet APPEND_SHOW
 event none
 action 1.0 cli command "enable"
 action 2.0 cli command "show run | append flash:config.txt"
```

---

This condensed format should fit well within your Obsidian notes, with full configurations shown where applicable.
