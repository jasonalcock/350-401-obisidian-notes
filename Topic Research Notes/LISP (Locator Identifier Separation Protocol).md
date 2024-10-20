---
aliases: 
publish: 
---
%%
date:: [[2024-10-04]]
parent:: 
%%
# [[LISP (Locator Identifier Separation Protocol)]]

Uses [[LISP (Locator Identifier Separation Protocol)]] which was originally designed by Cisco to reduce the burden of ASs having to have the entire route tables on their edge routers. LISP still needs an underlying routing protocol to work and it uses *IP-in-UDP tunnelling* for forwarding LISP traffic. Routes behind a LISP router are registered in a global LISP DB. LISP routers check with a DB lookup to determine which LISP router has the route. If the route is in the DB and LISP domain it gets tunnelled to the destination LISP router's IP else it forwarded to a border LISP router IP which checks it's external routes. LISP routers can support multihoming, failover and load balancing. One of the biggest benefits of LISP is routes can move easily between sites there for L3 mobility.
##### LISP
- LISP is an IETF standard protocol defined in RFC 6830 that is based on a simple endpoint ID (*EID*) to routing locator (*RLOC*) mapping system to separate the identity (*endpoint IP address*) from its current location (*network edge/border router IP address*).
- All fabric devices that play a role in Cisco SD-Access (edge, control plane, and border nodes) are configured as RLOCs by DNAC, and their Loopback0 interface address = RLOC address, while hosts/endpoint IP addresses in a fabric are tracked as EIDs.
- Underlay network needs to know only where the RLOCs are located and to route LISP encapsulated traffic between them.
-  Each router maintains its local routes and queries the map system to locate destination EIDs.
-  Centralised mapping database is called the LISP map server (*MS*).
###### LISP roles
![[Pasted image 20241003174856.png]]
In Cisco SD-Access, the fabric roles mapped to corresponding LISP roles. See table:

| Cisco SD-Access Fabric Role | LISP Role                                          | Function(s)                                                                                                                                                              |
| --------------------------- | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Edge                        | Ingress Tunnel Router (ITR)<br>Tunnel Router (xTR) | Encap packets in LISP forwards to RLOC<br>Endpoint connectivity<br>Resolves mapping to EID<br>Sends Map-Requests                                                         |
|                             | Egress Tunnel Router (ETR)                         | Decap packets from LISP and forward to EID<br>Endpoint connectivity<br>Sends Map-Register with EID prefixes to MS<br>Sends Map-Replies to Map-Requests                   |
|                             | Tunnel Router (xTR)                                | Does ITR and ETR roles                                                                                                                                                   |
| Control plane               | Map Server (MS)                                    | Process Map-Register entries and sends Map-notify<br>Learns EID-to-prefix mapping entries from ETRs<br>Stores EID-to-RLOC DB<br>Answers or Forwards Map-Requests to ETRs |
|                             | Map Resolver                                       | Responds to EID lookup requests from fabric edge and borders<br>Sends Map-Responses<br>Forwards Map-Responses to MS                                                      |
|                             | MS/MR                                              | Does both MS and MR roles                                                                                                                                                |
| Border                      | Proxy Ingress Tunnel Router (PITR)                 | Routes traffic from fabric to non-fabric networks<br>Non fabric endpoint connectivity<br>Resolves mapping to EID<br>Sends Map-Requests<br>Encap packets in LISP          |
|                             | Proxy Egress Tunnel Router (PETR)                  | Routes traffic from fabric to non-fabric<br>Decap LISP packets                                                                                                           |
|                             | (PETR/PITR)                                        |                                                                                                                                                                          |

###### LISP Packets
![[Pasted image 20241003133242.png]]
![[Pasted image 20241003133411.png]]
- **LISP Header:** The instance ID is a 24-bit value that has a similar function as the [Route Distinguisher (RD) in MPLS VPN](https://networklessons.com/cisco/ccnp-encor-350-401/mpls-layer-3-vpn-explained). The instance ID is a unique identifier, which keeps prefixes apart when you have overlapping (private) EID addresses in your LISP sites. Virtual networks (VNs) or virtual routing and forwarding instances (VRFs) in Cisco SD-Access are represented by LISP instance IDs, with each instance ID housing its own table of EIDs. This allows LISP to maintain macro-segmentation between its tables, which also makes troubleshooting easier. In the control plane, LISP instance IDs are automatically used to maintain separate VRF instances. Instance ID, which is used to identify which table the EID is in and maintains segmentation in the fabric.
- **Outer LISP UDP header**: The source port changes to prevent polarisation (taking same path) and allows ECMP. The UDP ports for:
	- Map-register/notify is 4341
	- Map-request/reply is 4342
	- Data forwarding is 4341
- **Outer LISP IP header**: RLOC source and destination IP for ITR to ETR.
###### LISP MAP-Register/Reply and Map-Request/Reply
All LISP messages are LISP encapsulated.
![[Pasted image 20241003141821.png]]
ETR sends **Map-Register** to all control planes modes/**MS** and looks like:
- **EID-prefix**: 192.168.2.0/24.
- **RLOC address**: 192.168.123.2.
- **UDP source port**: Chosen by the ETR.
- **UDP destination port**: 4342.
- At registration ETR can request MS to answer Map-Responses on its behalf by setting (P-bit).

**Map-Notify** is sent from MS back to the **ETR**:

ITRs send **Map-Request** goes to **MRs**:
- If entry in DB MR responds with a Map-Reply
- Else *MR* forwards request *to MS* which forwards *to ETR*, that answers with *Map-Reply* directly.

ITR gets **Map-Reply** from the MR, MS (proxy) or ETR and puts EID-to-RLOC in its local map-cache and programs FIB for LISP forwarding.
###### ITR LISP Forwarding & Encapsulation
ITR receives the IP packet, does a lookup in its FIB table, if entry forward using FIB. If theirs no route, a default route or a route to null 0 it checks if the source IP is a registered EID-prefix in the local map-cache. If source registered ITR sends Map-Request to the MR for destination. If ITR receives **negative MAP-Reply** packet from MR, it caches that prefix and maps it to the **PETR** and programs FIB for LISP forwarding. If **positive Map-Reply** ETR puts EID-to-RLOC in its local map-cache and programs FIB for LISP forwarding.

- To route packets externally LISP needs a MR and a PETR

The ITR has FIB entry for destination EID-RLOC, LISP encapsulated sent using:
- **Outer Source IP**: the ITRs RLOC address.
- **Outer Destination IP**: the RLOC address from the ETR-RLOC FIB entry or PTER if external.
- **Outer Source UDP port**: selected by ITR.
- **Outer Destination UDP port**: 4341.
- LISP Header + Payload
###### Sending/Receiving External Traffic PITR/PETR
- PETRs never register EIDs with the MS and they also never check check if the source is registered in its map-cache before sending Map-requests. 
- When EIDs are not in the DB the MS sends a -VE response with the PETRs RLOC.
- When MS gets a map request for a non-LISP destination, it calculates the shortest prefix (most general) for the destination IP that does not match any LISP EIDs. This entry is stored in the DB. The calculated non-LISP prefix is included in the negative map reply so that the ITR can add this prefix to its map cache and FIB. From that point forward, the ITR can send traffic that matches that non-LISP prefix directly to the PETR.
- For non-LISP destinations, the shorter prefix might be used in a **negative map reply** to summarise larger networks, preventing multiple cache entries for each host and improving scalability.
- PITRs never check the incoming traffic source IP against their local-map cache, they simply send and forward traffic into the fabric using destination EID and a local-map-cache lookup or using Map-Requests.
![[Pasted image 20241003170023.png]]
###### LISP Load Balancing
In the LISP context, for each RLOC mapped to an EID, the mapping system provides a priority and a weight. When several RLOCs have the same priority, the LISP traffic is split among the different RLOCs in proportion to their weight.

What is the purpose of the weight attribute in an EID-to-RLOC mapping? It indicates the load-balancing ratio between ETRs of the same priority. 
###### Lisp Notes
- **EID** and **ROLCS** can be IPv4 or IPv6
- ITR checks if the source EID is registered in its local map cache before sending a map request message to the MR. If the source isn’t registered as an EID, the traffic is not eligible for LISP encapsulation, and traditional forwarding rules apply. Prefixes that reside behind an EID site are setup so it can register them with the MS. No prefix no LISP routing.
###### Important LISP Concepts
Concepts::
- **LISP separates IPs into endpoint identifiers and routing locators so endpoints can roam between sites, their RLOC changes but the EIDs don't**
- **LISP has control plane messaging request/reply to/from Map Resolver (MR) and register/notify messaging to/from Map Server (MS) and it also data path forwarding and encapsulates all packets into LISP IP-in-UDP**
- **LISP has PITRs/ITRs that encapsulate into LISP and PETRs/ETRs that decapsulate LISP and can also combine roles xPTRs/xTRs**
- **LISP has a MS that stores all the EID-to-RLOC mappings, built from ETRs register messages and shared via Map-Response**
- **LISP has an Instance 24-bit ID field in its shim which provides 16 million possible device-and path-level network virtualisations i.e. VRF virtualisation and VPN segmentation**