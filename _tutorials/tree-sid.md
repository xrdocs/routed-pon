---
published: true
date: '2022-06-23 10:32 +0300'
title: Tree-SID
author: Lampros Gkavogiannis
excerpt: A quick look on Tree-SID technology
tags:
  - iosxr
  - cisco
  - multicast
  - innovation
  - segment routing
  - Tree-SID
---
# Tree-SID

This tutorial will be dedicated to Tree-SID which is a fresh Multicast technology. It is an innovation based on a controller-based approach to building a tree.

In order to establish the appropriate Tree-SID knowledge we first need to understand two technologies, Multicast and Segment Routing.

There is also a [video](https://www.youtube.com/watch?v=q3VNOnw-bIE&ab_channel=xrdocs) explaining the following concepts.

## Segment Routing

Segment Routing is a technology that uses source routing to forward traffic to the destination. Packet is forwarded from segment to segment based on the information carried in it. Segment Routing achieves a stateless IP fabric and delivers a unified end to end policy aware network infrastructure.

Segment Routing takes information from the control plane and inserts it inside the packet to simplify the network. The source chooses a path, and it gets encoded within the packet header as an ordered list of segments. There are two ways to encode that into the packet, MPLS (a stack of labels) and IPv6 header which is called SRv6.

The segment can be an identifier for any type of instruction forwarding or service. We take this identifier, and we stick it to the packet as a list of segments. That makes sense in Unicast since we have a single destination but in Multicast we do not, we have multiple destinations.

In the following example, we have the source, the destination, and 3 routers. We have a Unicast packet that we want to send from source to destination R1. The source sends the packet to router A, the router A determines the path and encodes it into the packet header as an ordered list of segments. The packet moves along this path by sequentially following the header. The packet eventually reaches R1. However, this would not work on Multicast because at the replication node if you have two receivers, the segment is ordered, and cannot choose one of them.

![treesid_1_1.png]({{site.baseurl}}/images/treesid_1_1.png)

## Multicast

Segment Routing for Unicast is orthogonal to Multicast. Multicast protocols and technologies can still work such as ingress replication, PIM, mLDP, RSV-TE without any issues. Nothing needs to be changed regarding Unicast deployment. However, Multicast can be improved and simplified. To simplify Multicast delivery, we can leverage existing Segment Routing solutions and implement new such as SR P2MP Policy Tree (Tree-SID) which is a centralized solution that can be both static and dynamic.

## Tree-SID

Tree-SID is a software-defined network controller-based approach to build P2MP trees in an SR domain. The SR Path Computation Element (SR-PCE) acts as the controller with the central knowledge. The tree can be built using constraints with the central knowledge at the SR-PCE. A Tree-SID identifier can be a:

- IPv4 / IPv6 Source and Group (S, G)
- A name string
- A numeric value

## SR-PCE

The next step is to understand what it takes to build a tree using a controller. SR-PCE is responsible for the following:
1.	Learn the topology
2.	Know the root and leaves of the tree
3.	Compute the tree
4.	Know the MPLS labels it can use
5.	Have a mechanism to program the forwarding state to all the nodes

## Learn the Topology

To learn the topology the controller requires a link to the network and a way to calculate the path. The link can be provided by a common mechanism such as the BGP Link-State which connects the controller with the rest of the network. This provides the database to the controller and gives him the ability to calculate the path by using any sort of algorithm (e.g. Djikstra).

![treesid_1_2.png]({{site.baseurl}}/images/treesid_1_2.png)

## Learning the Tree

To learn the tree the SR-PCE needs to discover the tree root and endpoints. There are two ways to achieve this. It can either be defined statically by an operator or dynamically through a protocol, like BGP Auto-Discovery (AD). The latter approach can automatically let the controller know who the receiver for a particular multicast stream is.

![treesid_1_3.png]({{site.baseurl}}/images/treesid_1_3.png)

## Computing the Tree

With the central knowledge at the controller, the tree can be computed according to different metrics and constraints. These can be optimization metric constraints such as IGP, TE, Delay and affinity constraints.

![treesid_1_4.png]({{site.baseurl}}/images/treesid_1_4.png)

## MPLS Label Allocation

In an MPLS architecture labels are platform specific. Each router has its own database of labels that advertise to somebody else, but the allocation of labels takes place on the router itself. The controller should allocate labels that are not used by routers for a different purpose. Proper label management is needed to avoid collisions and can be either global or dynamic. With global labels, every branch of the tree has the same label while with dynamic labels, each one can be different.

![treesid_1_5.png]({{site.baseurl}}/images/treesid_1_5.png)

## Programming the Tree

The controller so far has gathered all the required information. Now, it can program the forwarding state on all the routers in the path of the tree which is done by the PCEP (Path Computation Element Protocol) and it communicates with the nodes to push that state to them.

![treesid_1_6.png]({{site.baseurl}}/images/treesid_1_6.png)

## Tree-SID Types

The Tree-SID types are defined based on the way the tree root and leaves are learnt either static or dynamic. The former trees are created by the operator and the latter are dynamically created by the controller.

## IETF Standards

Tree-SID technology supports the IETF standards as it stated below.
It shall be possible for a node to advertise Tree-SID capability via IGP and/or BGP-LS. Similarly, a PCE can also advertise its TreeSID capability via IGP and/or BGP-LS. Capability advertisement allows a network node to dynamically choose one or more PCE(s) to obtain services pertraining to SR Replication policies, as well a PCE to dynamically identify TreeSID capable nodes

[IETF draft](https://datatracker.ietf.org/doc/draft-ietf-pce-sr-p2mp-policy/)
