---
aliases: 
publish: 
---
%%
date:: [[2024-09-27]]
parent:: 
%%
# [[EIGRP Variance and Forwarding Cache]]

## Overview
EIGRP (Enhanced Interior Gateway Routing Protocol) utilises variance for unequal-cost load balancing and maintains a forwarding cache (CEF) for efficient packet forwarding. This note covers EIGRP variance configuration and its impact on the forwarding cache.

---

## Key Concepts
- **Variance**: A multiplier that allows EIGRP to include multiple routes with unequal costs for load balancing.
- **Feasible Distance (FD)**: The metric for the best route to a destination.
- **Feasible Successor (FS)**: Backup routes that meet the feasibility condition.

## Related Concepts
- **CEF (Cisco Express Forwarding)**: A switching method used by Cisco routers to enhance performance and reduce latency.
- **RIB (Routing Table)**: Shows learned routes and their associated metrics (e.g., FD for EIGRP routes).
- **FIB**: Contains optimised routes for fast packet forwarding, storing only the next-hop details without EIGRP metrics.

### Important Concepts Summary
Concepts:: **Variance**, **Feasible Distance (FD)**, **Feasible Successor (FS)**

---

## Commands and Configuration

### Command 1: Configuring Variance
```bash
router eigrp 1
network 10.0.0.0
 variance 2
```
**Example Output:**
```
Router(config)# router eigrp 1
Router(config-router)# variance 2
```
**Explanation:**
- This command sets the EIGRP variance to `2`, multiplies the FS metric so up to twice the FD to be used for load balancing.

Command:: `variance 2`
Example:: Allows load balancing across paths with metrics up to twice the FD.
Explanation:: Enables unequal-cost load balancing by including feasible successor routes within 2x FD.

---

### Verification Command: Checking EIGRP Topology and Load Balancing
```bash
show ip eigrp topology
```
**Expected Output Before Variance:**
```
P 10.1.1.0/24, 1 successors, FD is 51200
   via 10.1.1.1 (51200/25600), GigabitEthernet0/0
   via 10.1.1.2 (76800/38400), FastEthernet0/1
```
**Explanation:**
- Line 1 shows there's only 1 successors used
**Expected Output After Variance:**
```
P 10.1.1.0/24, 2 successors, FD is 307200
    via 192.168.1.2 (307200/281600), FastEthernet0/1
    via 192.168.2.2 (332800/281600), GigabitEthernet0/0
```
**Explanation:**
- Line 1 shows 2 successors used for load balancing

Explanation of show ip eigrp topology
1. **P**: This indicates that the route is a **primary route** (P). It means that this route is valid and is being used in the routing decision process.
2. **10.1.1.0/24**: This is the **destination network** in CIDR notation. It indicates that the route pertains to the `10.1.1.0` subnet with a subnet mask of `255.255.255.0`.
3. **2 successors**: This indicates the number of **successor routes** available for the destination. A successor is a route that meets the feasibility condition and is used for packet forwarding.
4. **FD is 307200**: This value is the **Feasible Distance (FD)** for the best route to reach the destination network. It represents the total cost of the route from the EIGRP perspective.
5. **via 192.168.1.2**: This shows the **next-hop IP address** to reach the destination. This is the IP address of the router where the packets will be sent next.
6. **(307200/281600)**: The first number (307200) is the **Feasible Distance (FD)** of this route (which is the same as shown before), and the second number (281600) is the **Reported Distance (RD)**, which is the metric reported by the neighbor router (192.168.1.2) for this route.
7. **FastEthernet0/1**: This indicates the **outgoing interface** on the router that will be used to send packets to the next hop (192.168.1.2).
8. **via 192.168.2.2 (332800/281600)**: This shows a second successor route to the same destination. The FD is 332800, which is higher than the previous FD, indicating that this route has a higher cost.

Verification:: `show ip eigrp topology`
ExpectedOutput:: Displays multiple paths for a network that are included in load balancing due to variance.

---

## CEF and EIGRP Forwarding Cache

### Checking the Forwarding Cache
```bash
show ip cef
```
**Example Output:**
```
10.1.1.0/24, version 19, epoch 0, per-destination sharing
  via 10.1.1.1, GigabitEthernet0/0, 0 dependencies
  via 10.1.1.2, FastEthernet0/1, 0 dependencies
```
**Explanation:**
- The output indicates that packets destined for `10.1.1.0/24` can be forwarded using both next-hop IP addresses with their respective outgoing interfaces.

Command:: `show ip cef`
Example:: Displays the CEF forwarding information for destination networks.
Explanation:: CEF maintains an entry for destination networks with their respective next hops.

---



---

## Additional Notes
- EIGRP's variance setting is local to the router; it doesn't propagate to other routers.
- CEF is designed to optimise packet forwarding by maintaining a fast cache and reducing the need for complex route lookups.

---

## Tags
#important #eigrp #loadbalancing #networking #cef
```

### Key Features in This Comprehensive Example
- **Integrated Sections**: Combines EIGRP variance and forwarding cache topics for a holistic view.
- **Inline Fields**: Allows for easy extraction of commands, outputs, and explanations.
- **Tags**: Facilitates easy searching and filtering in Obsidian.

This structured format will help you quickly reference essential information while studying EIGRP configurations and concepts!