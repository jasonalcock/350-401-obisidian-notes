---
aliases: 
publish: 
---
# Topic: SD-WAN

## SD-WAN Overview
SD-WAN, or Software-Defined Wide Area Network, is a technology that allows organisations to manage and optimise their wide area networks through software. It provides enhanced agility, centralised control over network traffic, and the ability to steer data across complex network paths like the Internet, LTE, or MPLS more efficiently. 

SD-WAN solutions offer several benefits, including improved network speed, reduced costs, increased flexibility, and enhanced security features.

Cisco has 2 SD-WAN solutions:  Viptela based and Meraki. The Maraki solution bundles in UTM (Unified Threat Management)  which is an all in one security solution whereas the Viptela offers a more complex and custom, feature rich solution for manageing security, routing and complex topologies. 

SD-WAN sits at the edge of a network and the fabric is overlays the WAN and internet circuits

![[Pasted image 20241001184222.png]]



![[Pasted image 20241001154800.png]]
![[Pasted image 20241001155135.png]]
---
## SD-WAN Overview Key Concepts
- **SD-WAN Management**: SD-WAN is the management of WANs through software, providing centralised control and traffic management.
- **Cisco SD-WAN Solutions**: Cisco offers two SD-WAN solutions: Viptela (complex/customizable) and Meraki (bundled with UTM).
- **SD-WAN Secure Fabric** or overlay network is the network of secure tunnels built by the SD-WAN to forward data traffic.
- **SD-WAN Underlay** is network the SD-WAN is built on. It includes all the devices and protocols used to enable connectivity within, sites their WAN transports and internet links.
### Important Overview Concepts Summary
Concepts:: **SD-WAN Management**, **Cisco SD-WAN Solutions**

---
## SD-WAN Key Components
The primary components for the Cisco SD-WAN solution consist of the vManage network management system (management plane), the vSmart controller (control plane), the vBond orchestrator (orchestration plane), and the vEdge router (data plane). They each operate in a plane and also use Datagram Transport Layer Security (DTLS) for secure connectivity.

![[Pasted image 20241001121931.png]]
## Key Concepts
- **1. vManage**: Is the NMS GUI used for onboarding, provisioning, policy, software management and it sits in the management plane. You can also interact with vManage by REST for automation. It maintains all SD-WAN devices and their connected links in the underlay and overlay network. The device can be deployed in the cloud or on prem.
- **2. vBond Orchestrator** (Now called the validator): Is a virtual vEdge device running the dedicated vBond persona. It can be located via IP or FQDN.
	- *Authentication load balancing*: As edge devices authenticate, they are directed to the appropriate vSmart and vManage device. It will also load balance connections to the vSmart and vManage from vEdge devices.
	- *NAT detection*: If vEdges are behind a NAT device it facilitates IPsec NAT traversal and allows Authentication Header security to be applied to our data plane tunnels.
- **3. vSmart Controllers** (now called the Catalyst Controller):
	- Software-based component is responsible for the centralised control plane of the SD-WAN network.
	- Establishes a *secure DTLS connections to vEdge routers* to establish Overlay Management Protocol (*OMP*) neighbourships. OMP is a BGP like protocol and the vSmart acts as a route reflector advertising prefixes learnt to devices in the fabric.
	- It also orchestrates the *secure data plane connectivity between the vEdge routers* by distributing crypto key information, allowing for a very scalable, *IKE-less architecture*. Traditional IPSec peers would calculate their own key material but the controller now managers this and the distribution.
- **4. vEdge**: software or hardware, located at site perimeter, support standard router features such as OSPF, BGP ACLs, QoS, policies + SD-WAN overlay and data plane functions.
	- If you choose a cloud hosted control/management plane deployment this is the only component of the architecture that you will need to deploy.
	- Responsible for the data plane of the SD-WAN fabric as they bring up *IPsec or GRE tunnels between your sites edge devices*.
	- Form *control plane connections with vSmart controllers*, not between each other.
	- IOS XE devices known as cEdge devices not vEdge.
- **vAnalytics**: Optional component for analytics and assurance.
- **OMP**: OMP is used to manage and control the overlay beyond just routing (key management, configuration updates, and so on). It runs between vSmart and the WAN Edges inside of a secured tunnel. When a policy is built via the management plane, this policy is distributed to vSmart via NETCONF, and the vSmart will distribute this policy via an OMP update to the WAN Edges.
- **DTLS** (Datagram Transport Layer Security): Open standard for secure communications over UDP. Secures OMP, SNMP and NETCONF.
- **NETCONF**: Is used between the manager and the vSmart and edge devices for managing the fabric and the underlay:![[Pasted image 20241001132216.png]]
### Important Concepts Summary
Concepts:: **vManage is the GUI**, **vBond is edge auth, NAT & load balancing**, **vSmart pushes policies and routes to vEdge via OMP**, **vEdge data plane IPSec GRE via one another**, **OMP is more than BGP + keys + policy**, **DTLS secures OMP + all IP comms in SD-WAN**, **NETCONF is used to manage the fabric vSmart and underlay vEdge**

---
## SD-WAN Policy Concepts

You can push a unified policy across the fabric to many devices at once. The policy can modify the topology, influence traffic forwarding decisions, or be used to filter traffic. The SD-WAN policy is much like traditional IOS routing policies of ACLs, Route-Maps and applying the route-map to an interface.

Policies are further classified as:
 - **Local Policy** — Per edge device configuration from vManage. e.g. ACLs, QoS policies, and routing policies. Centralised policies can include device security configuration to.
 - **Centralised Policy** — Centralised configuration from vManage, activated by vSmarts and enforced on vEdge devices. where control plane functions are processed before OMP routes are advertised to edge devices. Data plane functionality is saved to edge devices local memory for enforcement.
	 - Centralised policies can contain the following component policies:
		 - *Control Plane*:
			 - *Topology* — A control plane policy to drop or modify routing behaviours by changing a path metrics, or even the next-hop.
			 - *VPN Membership* — A control plane policy to control the advertisement of specific VPNs prefixes to a specific site.
		 - *Data Plane:*
			 - *Application Aware Routing (AAR)* — Ensure a class of traffic is always transported across a WAN links that meet the apps minimum SLA. It uses BFD (Bidirectional Forwarding Detection) to monitor the WAN link and an SLA is set with latency, loss and jitter requirements.
			 - *Traffic Data* — A data plane policy that can filter traffic (on an application by application basis (or more general characteristics), modify traffic flows (change the next-hop, service-chaining, or even NAT), QoS functions, and/or implement packet loss protection mechanisms.

![[Pasted image 20241001144936.png]]
![[Pasted image 20241001132819.png]]
### Important SD-WAN Concepts Summary
Concepts:: 
- **Local Policy: Affects single vEdge in control or data plane**
- **Centralised Policy: Affect whole overlay fabric**
- **Central Control Plane Policy: affects for routing, TLOC and VPN building via OMP**
- **Central Data Plane Policy: Affects the forwarding of traffic in the data plane**
- **Application Aware Routing: Data Plane Policy, monitors transports (e.g. BFD) and routes apps over paths that meet SLA.**
- **Cloud OnRamp: monitor SaaS apps using probes and routes SaaS via edge that meets SLA.**
- **Viptela Quality of Experience (vQoE) score quantifies SaaS probes on a scale of 0 to 10, 10 best.**

---

## Additional Notes
- SD-WAN Cloud OnRamp for IaaS means you can extend fabric into cloud and provide security.

---

## Tags
#important #sd-wan 
## References
- [Design Zone for Branch/WAN - Cisco Catalyst SD-WAN Design Guide - Cisco](https://www.cisco.com/c/en/us/td/docs/solutions/CVD/SDWAN/cisco-sdwan-design-guide.html)
- [Underlay vs Overlay Routing | NetworkAcademy.io](https://www.networkacademy.io/ccie-enterprise/sdwan/underlay-vs-overlay-routing)

