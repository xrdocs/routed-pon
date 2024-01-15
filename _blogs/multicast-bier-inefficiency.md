---
published: true
date: '2022-09-08 14:03 +0300'
title: Multicast + Bier = Inefficiency
author: Lampros Gkavogiannis
excerpt: This blog explains the concept of Bier and why it is not recommended
tags:
  - iosxr
  - cisco
  - BIER
  - Multicast
  - Scale
position: hidden
---
## What is Bier

BIER is a new development in multicast transport. It does not require any of the traditional methods of tree building and takes an innovative approach to propagating multicast messages. When a multicast data packet enters the domain, the ingress router determines the set of egress routers to which the packet needs to be sent. The ingress router encapsulates the packet in a BIER header, which contains a bitstring in which each bit represents exactly one egress router in the domain; to forward the packet to a given set of egress routers, the bits corresponding to those routers are set in the BIER header. The multicast forwarding state is created using link-state protocols such as OSPF or IS-IS. Each bit-forwarding router (BFR) is identified as a single bit in the BIER header. When a message is to be sent to multiple BIER nodes, the bits in the bitstring that identifies those particular nodes are set by the ingress BIER node. As the BIER message moves through the network, the packet can be replicated out multiple interfaces by a BIER node to maintain the shortest path to the destination. In an MPLS environment, BIER labels are assigned to MPLS labels; this is called BIER MPLS. The BIER solution eliminates the per-flow state and the explicit tree-building protocols, which results in considerable simplification

BIER address the problems associated with current multicast solutions reviewed earlier in this chapter:
- Explicit tree building protocols are the core of multicast transport. The tree-building process requires memory to build up state in every node in the routing domains. This also impacts multicast convergence in the large routing domain
- Using RPF to build the trees toward the root could result in different paths between the same endpoints in a topology with multiple paths.
- Control plane aggregation of the flows is not possible in the current multicast solutions. Optimal delivery for every flow has two states maintained in the RIB.
- Maintaining and troubleshooting multicast networks can be very difficult. The need to understand the multicast control plane that overlays the unicast control plane increases the operational overhead for solution management.

The BIER concept is as follows:
Step 1. The router assigns a unique bit position mask to each router in the domain.
Step 2. Each edge router floots the bit position to ID mapping, within IGP (OSPF or IS-IS). A new LSA type is used to exchange this new information.
Step 3. ALl the routers in the unicast domain for BIER calculate the best path for each of the BFR indexes. In addition, the bit positions are OR'd (Boolean function) for the bitmask for each BFR neighbor. This consitutes the bit forwarding table.
Step 4. The multicast packets are forwarded based on the bit forwarding table using a logical AND operation of the multicast flow bit.

The bit index table is tied to ECMP for multiple paths from each, where applicable. The ECMP paths taking parallel interfaces to a single neighbor are noted as a single bit-mask. If the ECMP for multiple paths is seen for different neighbors, it is noted as a separate bitmask. BIER also has a concept of sets, whereby you can group a set of routers in a single group (similar to an area). The forwarding behavior is based on the packets' set ID, as illustrated in Figure 3-19.

![BIER_image_1_1.jpg]({{site.baseurl}}/images/BIER_image_1_1.jpg)

Figure 3-19:
The receiver information (egress router for multicast flow) is learned via an overlay mechanism. The current deployment of BIER in IOS-XR is via BGP overlay for receiver learning. Once the receiver is known within the domain, the forwarding is simple, using the bitmask index. This is different from PIM domains, where the forwarding and signaling are maintaned in the multicast state table at the same time. In BIER, multicast forwarding is kept separate from the signaling

![BIER_image_2_1.jpg]({{site.baseurl}}/images/BIER_image_2_1.jpg)

Figure 3-20:
Provides a high-level overview of packet flow in BIER:
- Before the multicast exchange, all the routers in the BIER domain exchange bitmask information by using IGP (OSPF in this case). All the routers in the unicast domain for BIER calculate the best path for each of the BFR indexes.
- The overlay control plane (in this case BGP) transmits the receiver information and the BIER nodes in the domain, as in the step above.
- The multicast data plane from the source connected to R1 is propagated to R4 through the bit index Boolean AND function, as in steps 2 through 4.

BIER, which is relatively new, is available only in IOS-XR at this writing. With this innovative solution, there is no need to run PIM, MLDP or P2MP MPLS traffic engineering in the network to signal multicast forwarding state created in the network is driven by advertisements through the link state protocols OSPF or IS-IS. This forwarding state is not per (*, G) or (S, G) but per egress router. Each egress router for multicast is identified by a bit index. Each BIER router must have on unique bit in a bitstring. This bitstring is also foud in the multicast packets that need to be forwarded through a BIER network.

Summary
MPLS VPN has been around for many years, implemented by service providers and enterprise customers because it provides the ability to logically separate traffic using one physical infrastrcuture. The capability of supporting multicast in an MPLS VPN environment has been available using default MDT. As technology has evolved, so has the capability of supporting multicast in an MPLS network. Today you can still use the traditional default MDT method, but there are now also 26 other profiles to choose from-from IP/GRE encapsulation using PIM, to traffic engineering tunnels, to MLDP, to BIER.

The requiremens for extranet MVPNs, or the capability of sharing multicast messages between VPNs, has also brought about development of new capabilities, such as VRF fallback, VRF select, and VASI. The combinations of solutions are almost endless, and this chapter was specifically written to provide a taste of different solutions and capabilities.

RFC 2365, RFC 6037, RFC 6514, RFC 6516, RFC 7358, RFC 7441


Cisco® Bit Indexed Explicit Replication (BIER) is an architecture used to forward multicast through an IP network. The innovative part of BIER is that it requires no explicit multicast signaling protocol in the network. There is no need to run Protocol Independent Multicast (PIM), multipoint LDP (mLDP), or P2MP MPLS traffic engineering within the network to signal multicast state hop-by-hop. Instead, the multicast forwarding state created in the network is driven by advertisements through the link state protocols OSPF or ISIS. This forwarding state is not per (*,G) or (S,G), but per egress router. Although packet forwarding is new and unique, there is interoperability with existing techniques for overlay signaling. These techniques are widely deployed in mVPN networks today.

How to position Cisco when other vendors promoted BIER? 

How to educate customers why BIER at SP scale SP doesn’t work??

BIER principle: every destination PE is represented by one bit, set to 1 when destination PE is a leaf, set to 0 when destination PE is not a leaf.

Assuming 12 PE, the bitmask will be 12-bit size. 

If initially PE0 send a stream to PE1 and PE11 are leaf, the bit mask will be set like this 1000.000.0010, assuming PE2 and PE10 join the tree, then the bit mask will be set as follow by the sender 1100.0000.0110.

Thus potentially, when architecting BIER, for every potential destination PE we do need one bit. Even if trees are with only few leaves.

Size of current SP network are increasing and not rare to see a forecast of 10,000+ PE’s. Just few examples
-	Docomo : 50,000 PE’s
-	Reliance Jio: 400,000 PE’s
-	DT: 100,000 PE’s
-	Telephonica Germany (3th largest German SP): 10,000 PE’s
-	Altice France (2nd largest French SP): 20,000 PE’s

Even if BIER is limited to the core (compared to core + aggregation + access), we often see at least 1,000 PE and up to 8,000 PEs for the largest IGP’s (largest is Dish).

This mean: BIER will have to support at minimum 2000-bit (80% of the market) and up 8000-bit mask to support SP scale. 

As of today, no HW does support it, all vendors chipping BIER or having experimental BIER implemented did with a 128-bit mask.

As an example: we tested BIER on ASR9K LS+, one of the most optimized line cards for packet replication. After several optimization the conclusion was: BIER switching performance with a 128-bit mask was half performance and scalability compared to label switch multicast (LSM) or IP multicast. 

What about BIER SET as a solution to scale BIER ?

The concept is to scale BIER by dividing the network into multiple BIER domains with smaller bitmask.

In another word for 2000 PE network, we could have 20 BIER domains of 128bits-mask per domain.

 
How it works: 

Every BIER domain will be signal with different label 
-	Label 20001 - BIER domain 1
-	Label 20002 – BIER domain 2
-	Label 200nn– BIER domain nn

The source PE will then replicate packets to all BIER domains having at least one leaf active in the domain. Intermediate PE and P node will then forward packets based on label or BIER domains they belong to.

If only BIER domains 5 and 8 have at least one PE leaf, the source PE will replicate packet twice to the 2 BIER domains over same uplink consuming twice more bandwidth.

Potentially if one leaf is active in every 20 domains, the PE will have to replicate the streams 20 times end send to the next hop. Most PE are dual connected to 2 P nodes, then in best case traffic is replicate 10 times on each uplinks consuming 10 times more bandwidth.

As you can see, BIER SET (or BIER Domain) concept allows to help to reduce forwarding complexity by having smaller bitmask BUT requires replicating traffic from the source and could requires provisioning network with way more bandwidth. 

And lastly, BIER and BIER SET doesn’t address tree disjoiness, this will most likely address by combining Flex-Algo and BIER or combining BIER with TE, putting even more constraint on design, architecture and NPU.

The conclusion I made in 2012 to the BIER author still valid today: BIER at SP scale. (2,000- 8,000 PE’s) is not something even doable on Cisco, Broadcom or even competitor NPUs. BIER SET is not a good option has it is going to consume 10x..20x more bandwidth, and thus no improvement.

What about competition:
-	Huawei do support BIER with 128-bit mask.
o	Wanted to show leadership, no true customer behind
-	Nokia do support BIER with 128-bit mask
o	Made for a single customer.
-	Is Huawei BIER or Nokia BIER a threat to our business?
o	Absolutely not
o	But we must educate customers why…

