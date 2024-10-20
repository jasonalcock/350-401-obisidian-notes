---
aliases: 
publish: 
---
%%
date:: [[2024-09-24]]
parent:: 
%%
# [[VTP VLAN Pruning]]
- VTP Pruning automatically removes VLANs from trunk links if the switch doesn't have any active ports in the VLAN
- VTP Pruning is disabled by default
- VTP pruning by default auto-prunes therefore all VLANs become prune eligible.
```
SW1# show interfaces trunk

Port       Mode       Encapsulation     Status    Native vlan
Et0/2      desirable  n-802.1q          trunking  56

Port   Vlans allowed on trunk
Et0/2  1-4094

Port   Vlans allowed and active in management domain
Et0/2  1-15,56

Port   Vlans in spanning tree forwarding state and not pruned
Et0/2  1-15,56
SW1#
```

- If a switch doesn't have any active ports for a VLAN X it will request the upstream switch to prune VLAN X from it's trunk to it.
- I think a switch sends VTP join message to an upstream switches  for all VLANs that are active on it.
- Upstream switches act on the join and make sure those VLANs are active on the trunk else the block the braodcast traffic.

If you want to remove a VLAN and stop it from being auto pruned then remove it from the eligible prune list for that interface. 
```
int et0/2
switchport trunk pruning vlan remove 7-9
!
SW1# sh interfaces e0/2 switchport
Name: Et0/2
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 56 (NATIVE)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none 
Administrative private-vlan mapping: none 
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: ALL
Pruning VLANs Enabled: 2-6,10-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Appliance trust: none
SW1#

```

 manual adjusting of Pruned VLANs can be configured on either the Server or Clients Trunk interfaces. On interface remove or add to pruning eligible list
```
switchport trunk pruning vlan 2,3,4,5
switchport trunk pruning vlan remove 30
switchport trunk pruning vlan add 30
!
switchport trunk pruning vlan 2-10
switchport trunk pruning vlan add 42
switchport trunk pruning vlan remove 5
switchport trunk pruning vlan except 100-200
switchport trunk pruning vlan none
```



- Autp VTP pruning not recommended with STP as it can block VLANs on the STP forwarding path.
	- Use manual pruning for STP ```switchport trunk allowed vlan 10,20,30,40```
### References 
[VTP – Pruning VLAN explanation and lab to demonstrate, and example of trick question for Exam day (bottom of post)! – The DEVNET GRIND!](https://loopedback.com/2017/09/26/vtp-pruning-vlan-explanation-and-lab-to-demonstrate-and-example-of-trick-question-for-exam-day-bottom-of-post/)