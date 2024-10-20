---
aliases: 
publish: 
---
# Topic: SD-Access

## What Is It?
Cisco Software-Defined Access (SD-Access) is an intent based campus LAN solution.  Intent-based networking (IBN) and the Cisco Digital Network Architecture means SD-Access network is defined by a policy (intent-based networking) that is automatically programmed down to the network infrastructure (automation).

The SD-Access solution is enabled by an application package that runs as part of the Cisco DNA Centre software and provides automated end-to-end segmentation to separate user, device, and application traffic.  These user access policies are automated so that organisations can ensure that the right policies are established for any user or device with any application anywhere in the network.

SD-Access uses logic blocks called fabrics which leverage virtual network overlays that are driven through programmability and automation to create mobility, segmentation, and visibility.  Network virtualisation becomes easy to deploy through software-defined segmentation and policy for wired and wireless campus networks.  Single physical networks are abstracted and can host one or more logical networks which are orchestrated through software.  Error-prone manual operations in these dynamic environments are circumvented altogether, providing consistent policy for users as they move around the wired and wireless network.

![[Pasted image 20241001184222.png]]
## Components and Concepts
- **Cisco DNA Centre**: The central management and automation platform (GUI is in Management Layer).
- **Fabric Underlay**: Traditional network infrastructure (e.g., OSPF, BGP) providing basic connectivity and we expect this to be running on physical equipment.
- **Cisco Campus Fabric Overlay**: Is the virtual network that uses VXLAN for tunnelling (Data Plane) and LISP (Locator/ID Separation Protocol) for segmentation and identity mapping (Control Plane) and Cisco TrustSec (Policy Plane).
- **Identity Services Engine (ISE)**: Provides identity-based policy enforcement such 802.1x and segmentation.
## Functions
- **Network Automation**: Uses Cisco DNA Centre for centralised automation and orchestration, simplifying network design, provisioning, and deployment with best-practice configurations.
- **Network Assurance & Analytics**: Uses telemetry for proactive risk prediction and improved performance of the network, endpoints, and applications, even for encrypted traffic.
- **Host Mobility**: Provides seamless mobility for both wired and wireless clients.
- **Identity Services**: Cisco ISE identifies users/devices and provides context for security policies and network segmentation.
- **Policy Enforcement**: Implements scalable, identity-based policies using Security Group Access Control Lists (SGACLs) instead of traditional IP-based ACLs.
- **Secure Segmentation**: Offers secure, end-to-end segmentation across the network.
## Benefits
- **Simplified Management**: Centralised configuration and policy control via DNA Center.
- **Enhanced Security**: Identity-based segmentation and policy enforcement with ISE.
- **Improved User Experience**: Ensures consistent wired/wireless network performance.
- **Faster Troubleshooting**: Real-time analytics help quickly identify and resolve issues.
## Important Concepts Summary
Concepts:: 
- **SD-Access is a campus LAN solution, multi-site possible for private IP underlay with SLA**
- **SD-Access fabric is a virtual network, defined in policy and is configured and pushed automatically to fabric devices**
- **SD-Access is a DNAC application and features segmentation, device access control and user authentication, integrates seamlessly with ISE + much more**

---
# Architecture Layers
![[Pasted image 20241001190755.png]]
- **SD-Access Layered Architecture** describes the SD-Access building blocks in layers so one can understand and get an overview of all the devices, components, protocols, workflows that goes into designing, building, deploying and managing the SD-Access solution.
- **Management Layer**: Consists only of the Catalyst Centre (formerly DNA Centre) which is the SPOG GUI app that can be used to deploy and manage the entire SD-Access solution lifecycle. Cat Centre has 4 main sections and workflows Design, Policy, Provision and Assurance. The intent and policy is passed down and up to the controller layer. The GUI simplifies everything and no configuration commands are required to be input here.
- **Control Layer**: Cat Centre sub process and ISE are the gateway between the DNAC GUI and the physical layer.  The controller layer is where policies are translated into configurations that are pushed to devices to build the overlay and can also manage the underlay. Device configuration will be deployed using protocols such NETCONF, SNMP or SSH. This layer will also collect logs, NetfFlow etc, This layer is also where the ISE sits and it provides AAA, RADIUS and EAPoL.
- **Network Layer** is where the logical underlay and overlay network topologies are described. The underlay network is a traditional network, typically routed access. The overlay network describes the fabric and its operational planes. The planes further details the protocols, fabric device roles, fabric WLCs and key SD-Access fabric concepts such as host pools, SGT, TrustSec and VNs.
- **Physical Layer** these are the routers, switches, wireless devices that provide the overlay network, access to the fabric as well as the essential SD-Access components such as DNAC and ISE appliances. 
## Important Concepts Summary
Concepts:: **SD-Access architecture is made of 4 layers**, **SD-Access MGMT layer is the DNAC GUI with 4 main workflows of Design, Policy, Provision and Assurance**, **SD-Access Control Layer is where policies are translated into configuration and pushed to end devices**, **SD-Access Network Layer is where the underlay and overlay logical topologies are described and the protocols used**, **SD-Access Physical Layer is where all the physical devices that are used for SD-Access are defined.**

---
## Physical Layer
![[Pasted image 20241002165908.png]]
All devices participating in the fabric have SD-Access specific ASICs, FPGAs and software hardware. 
- **Access switches** provide wired access to the fabric.
- **Routers** provide WAN and branch access to the fabric.
- **WLCs and APs** provide wireless clients access to the fabric.
- **DNA centre and ISE** are the 2 required components for SD-Access.
- **SD-Access extension nodes** are models of Cisco access layer switches that do not actively participate in the SD-Access fabric as they do not have the hardware but so participate in SD-Access automation. They connect into fabric Access switches and allow IoT devices to connect to the Fabric.
#### Important Concepts Summary
Concepts:: **Switches, routers, APs, WLC is where fabric access is provided**, **DNA Centre and ISE are essential SD-access components that can be physical or virtual**, **SD-Access extension nodes do actively participate in the fabric but automation does and they permit access for IoT device**

---
## Network Layer
![[Pasted image 20241002165919.png]]
### Underlay Network
Is the traditional IP network, connecting all the fabric nodes and user or network devices together. It's only purpose is to transport SD-Access traffic. The performance, scalability, and high availability of the underlay can affect the operation of the fabric overlay and routed access is the preferred design with no STP. ISIS is also the preferred IGP as it is IPv4, v6 and non-IP agnostic and can establish peering without IP.
	- **Manual**: Built using traditional methods and not using DNAC and highly customisable with any IGP.
	- **Automated**: DNAC managed devices underlay that uses a PnP (Plug-and-Play) feature to deploy an IS-IS routed network. Supports unicast and multicast and reduces complexity but no manual configuration is possible.

IS-IS
- Uses fewer resources
- Supports IPv4, IPv6 and other non IP traffic
- Forms neighbourships without IP
- Peering using loopbacks
### Overlay Network (Fabric)
The SD-Access fabric provides policy-based network segmentation, host mobility for wired and wireless hosts, and enhanced security beyond the normal switching and routing capabilities of a traditional network. It is fully automated includes all necessary overlay control plane protocols and addressing, as well as all global configurations associated with operation of the SD-Access fabric. The **3 basic network planes of operation** in the SD-Access fabric is:
#### Control Plane
Uses LISP simplifies routing, routers only process local routes, host mobility + Cisco added enhancements Anycast gateway, Fabric Wireless, VN Extranet provide access to shared services.


#### Data Plane
Uses a Cisco customised version of the open VXLAN (Virtual Extensible LAN) standard. VXLAN was designed to address the 4096 VLAN limitation of 802.1q and to be able to extend Ethernet traffic over existing IP networks. This meant a SP with 1000s of customers can provide up-to 16 million virtual networks. VXLAN stretches and tunnels the layer 2 network between sites over existing IP networks and systems like storage or VMs can easily be moved between sites or be placed centrally without changing IP or MAC address. The protocol supports layer 2 MAC learning from remote sites to facilitate host L2 communication. Customers can use the same IP in VLAN 10 as site a and b and customers A and B can also use the same VLAN IDs, Subnets and IPs. VXLAN is control plane agnostic but in SD-WAN uses LISP to accommodate IP traffic routing and mobility.
#### Policy Plane
Cisco TrustSec decouples access that is based strictly on IP addresses and VLANs by using logical groupings in a method known as Group-Based Access Control (GBAC).  The goal of Cisco TrustSec technology is to assign an SGT value to the packet at its ingress point into the network.  An access policy elsewhere in the network is then enforced based on this tag information.
#### 5 Basic Fabric Device Roles 
![[Pasted image 20241001191003.png]]
1. **Control plane node**: This node contains the settings, protocols, and mapping tables to provide the endpoint-to-location (EID-to-RLOC) mapping system for the fabric overlay.
2. **Fabric border node**: This fabric device (for example, core layer device) connects external Layer 3 networks to the SDA fabric. There's 3 Fabric Border PxTR Node Types: **Internal** connects internal org such as WLC, firewall and data centre, **default** border external via default route and **internal + default**.
3. **Fabric edge node**: This fabric device (for example, access or distribution layer device) connects wired endpoints to the SDA fabric.
4. **Fabric WLAN controller (WLC)**: This fabric device connects APs and wireless endpoints to the SDA fabric.
	- **Fabric WLC** is external to fabric via internal border node.
	- It performs PxTR Map-registers to MS on behalf of the edges devices.
	- The MS maps the EIDs to the fabric access point and edge node location the AP it is attached to. CAPWAP tunnel only used for control plane and AP client forwarding done through site-to-site VXLAN tunnels.
	- SGT and VRF-based policies for wireless users on fabric SSIDs are applied at the fabric edge in the same way as for wired users.
	- Wireless clients (SSIDs) use regular host pools for traffic and policy enforcement same as wired clients, and the fabric WLC registers client EIDs with the control plane node. 
1. **Intermediate nodes**: These are intermediate routers or extended switches that do not provide any sort of SD-Access fabric role other than underlay services

![[Pasted image 20241003180031.png]]

#### Important Fabric Concepts
Concepts::  **Control Plane LISP for messaging and communication between infrastructure devices in the fabric.**, **Data Plane VxLAN encapsulation data packets, Policy Plane for security and segmentation.** 

---

## Controller Layer
- **Concept 1**: Description
- **Concept 2**: Description
### Important Concepts Summary
Concepts:: **Concept 1**, **Concept 2

---
## Management Layer
- **Concept 1**: Description
- **Concept 2**: Description
### Important Concepts Summary
Concepts:: **Concept 1**, **Concept 2

---
# SD-Access and Wireless

Wireless LAN Controller (WLC): The wireless LAN controller integrates with the control plane node to manage wireless users and access points (APs).

Access Points  
+ AP is directly connected to FE (or to an extended node switch)  
+ AP is considered a part of Fabric (overlay).
+ AP is in local mode an must connected directly to the fabric edge switch
+ Fabric mode APs continue to support the same wireless media services that traditional APs support; apply AVC, quality of service (QoS), and other wireless policies; and establish the CAPWAP control plane to the fabric WLC. 
+ **Fabric APs join as local-mode APs and must be directly connected to the fabric edge node switch** to enable fabric registration events, including RLOC assignment via the fabric WLC.
+ The fabric edge nodes use CDP to recognise APs as special wired hosts, applying special port configurations and assigning the APs to a unique overlay network within a common EID space across a fabric.
+ The assignment allows management simplification by using a single subnet to cover the AP infrastructure at a fabric site.


---
# Other SD-Access Info
- SD-Access multisite distributed campus solutions exists.
![[Pasted image 20241002091444.png]]
- Cisco Digital Network Architecture (Cisco DNA) ![[Pasted image 20241002093121.png]]
- If 16 million unique VxLAN IDs is not enough then fill your boots with Q-in-VNI over VxLAN
- When VXLAN was first implemented in Linux 3.7, the UDP port to use was not defined. Several vendors were using 8472 and Linux took the same value. To avoid breaking existing deployments, this is still the default value. Therefore, if you want to use the IANA-assigned port, you need to explicitly set it with `dstport 4789`.
- A **fusion router** perform VRF route leaking between user VRFs and Shared-Services, which may be in the Global routing table (GRT) or another VRF. Shared Services may consist of DHCP, Domain Name System (DNS), Network Time Protocol (NTP), Wireless LAN Controller (WLC), Identity Services Engine (ISE), DNAC components which must be made available to other virtual networks (VN’s) in the Campus. What is the role of a fusion router in an SD-Access solution? Performs route leaking between user-defined virtual networks and shared services
# Tags
#study #networking #encore #sd-access

# References
- [SD-Access - Networking Learning Maps - Cisco Live On-Demand - Cisco](https://www.ciscolive.com/on-demand/learning-maps/networking/sd-access.html)
- [Design Zone - Cisco SD-Access Solution Design Guide (CVD) - Cisco](https://www.cisco.com/c/en/us/td/docs/solutions/CVD/Campus/cisco-sda-design-guide.html#Introduction)
- [Cisco Software-Defined Access Compatibility Matrix](https://www.cisco.com/c/dam/en/us/td/docs/Website/enterprise/sda_compatibility_matrix/index.html)
- [Solutions - Cisco SD-Access Ordering Guide - Cisco](https://www.cisco.com/c/en/us/solutions/collateral/enterprise-networks/software-defined-access/guide-c07-739242.html)
- [Software-Defined Access for Distributed Campus Deployment Guide - Cisco](https://www.cisco.com/c/en/us/td/docs/solutions/CVD/Campus/SD-Access-Distributed-Campus-Deployment-Guide-2019JUL.html)
- [Q-in-VNI over VxLAN Fabric Deployment Guide - Cisco](https://www.cisco.com/c/en/us/td/docs/dcn/whitepapers/q-in-vni-over-vxlan-fabric-deployment-guide.html)
- [VXLAN & Linux](https://vincent.bernat.ch/en/blog/2017-vxlan-linux)
- [VXLAN, OTV and LISP « ipSpace.net blog](https://blog.ipspace.net/2011/09/vxlan-otv-and-lisp/)
- [ENCOR Study Group by KishSquared](https://www.youtube.com/playlist?list=PLOpoM7HdItaLzI-ikzNRbeE-6JJJV2tSu)
- [Design Zone - Cisco SD-Access Solution Design Guide (CVD) - L3 Routed Access](https://www.cisco.com/c/en/us/td/docs/solutions/CVD/Campus/cisco-sda-design-guide.html#L3_Routed_Access)
- [SD-Access Segmentation Design Guide - Cisco Community](https://community.cisco.com/t5/networking-knowledge-base/sd-access-segmentation-design-guide/ta-p/4935734)
- 