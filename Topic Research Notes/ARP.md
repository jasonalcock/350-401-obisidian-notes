---
aliases: 
publish: 
---
%%
date:: [[2024-09-18]]
parent:: 
%%
# [[ARP]]

An ARP frame sent without being solicited by a request is “gratuitous” But technically, they are not exactly the same as a Gratuitous ARP. ARP probes are sent to see if IP already in use using opcode 1. The SMAC, DMAC & IP are zeroed but destination IP is that of the one it wants to use. ARP announcement sent to officially claim IP with senders SMAC, DMAC is zeroed and SIP and DIP set to the claimed IP.

Proxy ARP is done when a node sends an ARP response on behalf of another.

Gratuitous ARP (GARP) is an ARP response without being requested. A node can announce an IPs MAC update.  SIP and DIP are set to the VIP and the SMAC is set to new MAC and typically the DMC is broadcast but in the case GLBP will use the client cache to unicast and set the ARPs DMAC. HSRP and VRRP will simply broadcast as they are not load balancing. If there's a L2 switch between the router and clients then the switch can use the GARP to updates it's MAC address table.

NAT and Proxy ARP. Proxy ARP allows a L3 device to send an ARP response for IPs that it is directly connected to clients that are one of it's other LAN interfaces. If the NAT VIP is not in the same subnet as the outside interface then the upstream router will need to have a route for the public NAT addresses but if the NAT addresses are on the same LAN as the outside interface the L3 device will need to respond to ARP requests with a proxy ARP with it's own SMAC for the IP. Proxy ARP essentially allows a device to connect to remote networks through a device with using routing. 