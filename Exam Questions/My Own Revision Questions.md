---
aliases: 
publish: 
---
%%
date:: [[2024-10-06]]
parent:: 
%%
# [[My Own Revision Questions]]

# OSPF

## General Basic OSPF Operations
#ospf #neighbours #packets #lsa #timers #states #network-types #spf 
- What problem does OSPF solve, why is it different to other solutions, what layer does the control plane operate at?  
- In a broadcast network what 2 settings need configuring for neighbours to form? What 2 area settings must match
- What are the 8 neighbour states, what happens in each state?
- What are the 5 message types, what are they used for and what do they contain?
- What packet type contains just a summary of the LSDB?
- What packet type carries LSAs?
- What happens at when the dead timer expires?
- How are the 2 OSPF messages?
- How does OSPF enable scale and use hierarchy?
- What algorithm does OSPF use, what is it's objective, how does it work when is it 1st calculated?
- What 2 methods and commands enable OSPF on an interface?
- What identifies an OSPF router, how is it chosen or configured?
- How do you configure and clear the OSPF process on a router?
- How long will a broadcast segment take to converge if there's no existing DR or BDR?
## Broadcast Networks
#priority #election #dr 
- How do you ensure a router is not elected as a DR?
- What are the 3 router roles on a broadcast network?
- What happens if I bring a router online with the highest priority?
- What check is made on a broadcast network that is not required on a P2P network for neighbours to become adjacent? What field is used to make this check in the hello packet.
## Optimisations
- What command is used to advertise a default route and what are the two optional parameters?
- What do the hello, dead and wait interval timers do? and what are the default hello and dead timers for ethernet and p2p networks?
- What is the default link cost for a 1 Gig interface and what 2 methods and commands can I use to alter this?
- What are the default network types for an ethernet, GRE tunnel interface, serial and loopback interfaces and what 2 reasons would you change the type and what is the command?
## OSPF LSA Types
- What is the name of the group of fields that precedes os LSAs?
- Name 2 fields in the LSA header that make LSAs reliable? 
- How long are originated LSAs kept in the LSDB?
## OSPF Multi-area Networks
- What does having multiple areas achieve? 
- Broadly are the 3 types of areas?
- What are the 3 types of area routers?
- One what 2 router types can you summarise LSAs?
- When summarising what special route is placed in the global route table and why?
- What cost is used for the inter-area summary LSA? 

# VTP
1. What VTP mode/role is used to enable VTP domain pruning?
2. What type of links does VTP work on, what are the message
3. What VLAN do VTP messages typically travel in?
# VLANs & Trunks
1. What IEEE and Cisco protocols allowed switches to run multiple simultaneous layer 2 networks?
2. How does adding VLANs alter a MAC table?
3. What and where are the noteworthy changes to the IEEE Ethernet frame header to support this new standard?
4. What proprietary Cisco protocol allows ports to negotiate their trunk settings and what are the 5 possible modes?
5. What 2 trunk modes on either side of a trunk will a trunk port not be negotiated 
6. If the encapsulation is set to dot1q what switchport mode must you use?
7. With the default switchport encapsulation set to negotiate, what are are the 2 possible trunk mode settings and what settings will a trunk never form.
# Etherchannel
- What problem do EtherChannels solve?
- What 2 protocols automatically can negotiate and manage EtherChannels between 2 devices?
- What are the 5 possible EtherChannel modes and possible combinations? 
- How would you add an interface to a port channel?
- What physical interface settings must match on a switch for a port channel to come up?
- What are the default and maximum number of interfaces you can have in a PortChannel?
- What is the default hash used to load balance traffic over port channels?
- 
# STP 802.1D & PVST
- What problem does ST solve?
- What 3 steps describe how ST operates?
- What 4 steps are used for all ST decisions?
- What 5 states can a port be in?
- In what states do ports send BPDUs?
- What are 4 802.1D port roles?
- What 2 timers are used control how long a port stays in a state and is used RB whilst sending BPDUs with the TC flag set?
- How long roughly does an 802.1D network take to converge from boot?
- What Bridge controls the timing of the entire ST operation and how does it do this?
- What 2 events cause bridges to originate rather forward BPDUs?
- How do you make a switch most likely to become the RB and what is the default setting?
- Do BPDUs travel inside a VLAN or over the native VLAN on a trunk?
- If a VLAN is removed from a trunk will ST still consider the trunk link when calculating the ST?
- What process does STP use to ensure traffic does not get black-holed after a topology change?
- When BPDUs are sent for with the TC flag set for VLANs 10 do the downstream switches shorten the age out timer for all VLANs or just for VLAN 10? 
# STP Convergence and Cisco Toolkit
- What type of failure occurs when a switch loses its root port and has a blocking port available?
- What type of failure occurs when a distribution switch root port remains valid but its blocking port gets new inferior or better BPDU? What information in the BPDU changed? What role does the blocking port to move to? What timers are used?
- What role is an access port in if STP is enabled on it? How long will it take to begin forwarding traffic and what problems can this cause for end hosts?
- How Cisco feature speeds up access port forwarding and what are it's 3 advantages?
- Do access ports with the above feature still send BPDUs and what happens when they receive a BPDU?
- What happens when PortFast is enabled and a superior BPDU is received?
- How Cisco enhancement turns of the Portfast feature when a BPDU is received?
- What Cisco enhancement disables a Portfast if a BPDU is received?
- 