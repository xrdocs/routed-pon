---
published: true
date: '2022-07-06 16:40 +0300'
title: Tree-SID Demo
author: Lampros Gkavogiannis
excerpt: This will be a Tree-SID demo using Cisco NCS5500 routers
tags:
  - iosxr
  - cisco
  - NCS5500
  - Tree-SID
---
## Tree-SID Demo

This tutorial describes the process of a Tree-SID demo which was implemented on 6 NCS5500 devices. It consists of the topology, the configurations and the relevant outputs on the routers.

### Overview

IGP Segment Routing is configured to establish unicast connectivity between root, mid and leaf nodes.
The MVPN BGP session is established between root and leaf nodes for:
- PE auto discovery
- P-Tunnel signaling and
- C-multicast route signaling

PCE learns the topology via IGP or BGP-LS.
PCE has PCEP sessions with root, mid and leaf nodes.
Tree-SID labels are allocated from the segment routing local block.

There are two MVPNs configured:
1. MVPN VRF RED - P2MP transport with IGP metric and affinity constraint which corresponds to red links (color 10).
2. MVPN VRF BLUE - P2MP transport with IGP metric and affinity constraint which corresponds to blue links (color 20).

Note: Both VPNs have associated "default" and "data" MDTs.

### Topology

The following drawing shows the connections between the routers, the IPs and the affinity links (red, blue). The source is the R1 and the receivers are R5 and R6.

![]({{site.baseurl}}/images/Tree-SID%20demo%201.1.png)

This drawing includes all the interfaces between the routers.

![]({{site.baseurl}}/images/Tree-SID%20demo%201.2.png)

All the routers are running the following IOS-XR releases:
- R1 - 7.4.2
- R2 - 7.5.1
- R3 - 7.5.1
- R4 - 7.5.2
- R5 - 7.5.1
- R6 - 7.3.2

The base release is 7.3.1 +. Anything above that can support this demo configuration.

### Router Configurations

[ROOT-R1](https://github.com/lambros90/xrdocs/blob/main/tree-sid-demo/root-R1.txt)

It is important to notice the Tree-SID policies, the segment routing configuration and the colour affinities.

```
...
route-policy treeSID
  set core-tree sr-p2mp
end-policy
!
route-policy treesid
  set on-demand-color 20
end-policy
!
route-policy pass-all
  pass
end-policy
!
route-policy treesid-color-10
  set on-demand-color 10
end-policy
!
router isis 1
 net 49.0001.0000.0000.0001.00
 distribute link-state
 address-family ipv4 unicast
  metric-style wide
  mpls traffic-eng level-2-only
  mpls traffic-eng router-id Loopback0
  router-id 1.1.1.1
  segment-routing mpls
 !
 interface Loopback0
  passive
  address-family ipv4 unicast
   prefix-sid absolute 16001
  !
 !
multicast-routing
 address-family ipv4
  mdt source Loopback0
  interface all enable
  mdt static segment-routing
 !
 vrf red
  address-family ipv4
   interface all enable
   bgp auto-discovery segment-routing
   !
   mdt default segment-routing mpls color 10
   mdt data segment-routing mpls 1 color 10 threshold 0
  !
 !
 vrf blue
  address-family ipv4
   interface all enable
   bgp auto-discovery segment-routing
   !
   mdt default segment-routing mpls color 20
   mdt data segment-routing mpls 1 color 20 threshold 0
  !
 !
vrf vpn1
  address-family ipv4
   interface all enable
   mdt static segment-routing
  !
 !
!
segment-routing
 global-block 16000 23999
 local-block 15000 15999
 traffic-eng
  interface TenGigE0/0/0/8
   affinity
    name RED
   !
   metric 11
  !
  interface TenGigE0/0/0/9
   affinity
    name RED
   !
   metric 11
  !
  interface TenGigE0/0/0/12
   affinity
    name BLUE
   !
   metric 11
  !
  interface TenGigE0/0/0/13
   affinity
    name BLUE
   !
   metric 11
  !
  on-demand color 10
   dynamic
    metric
     type igp
    !
    affinity
     include-any
      name RED
     !
    !
   !
  !
  on-demand color 20
   dynamic
    metric
     type igp
    !
    affinity
     include-any
      name BLUE
     !
    !
   !
  !
  affinity-map
   name RED bit-position 23
   name BLUE bit-position 24
  !
  pcc
   pce address ipv4 1.1.1.2
    precedence 100
   !
  !
 !
 ...
!
```

[PCE-R2](https://github.com/lambros90/xrdocs/blob/main/tree-sid-demo/PCE-R2.txt)

```
segment-routing
  traffic-eng
   p2mp
    endpoint-set leaf-R6
     ipv4 1.1.1.6
    !
    endpoint-set leaf-R5-R6
     ipv4 1.1.1.5
     ipv4 1.1.1.6
    !
    label-range min 15400 max 15600
    policy p2mp-te-global
     source ipv4 1.1.1.1
     color 101 endpoint-set leaf-R5-R6
     treesid mpls 15101
     candidate-paths
      preference 100
       dynamic
        metric
         type te
        !
       !
      !
     !
    !
    policy p2mp-igp-global
     source ipv4 1.1.1.1
     color 100 endpoint-set leaf-R5-R6
     treesid mpls 15100
     candidate-paths
      preference 100
       dynamic
        metric
         type igp
        !
       !
      !
     !
    !
    policy p2mp-delay-global
     source ipv4 1.1.1.1
     color 102 endpoint-set leaf-R5-R6
     treesid mpls 15102
     candidate-paths
      preference 100
       dynamic
        metric
         type latency
        !
       !
      !
     !
    !
    policy p2mp-igp-red-vpn1
     source ipv4 1.1.1.1
     color 200 endpoint-set leaf-R5-R6
     treesid mpls 15200
     candidate-paths
      constraints
       affinity
        include-any
         RED
        !
       !
      !
      preference 100
       dynamic
        metric
         type igp
        !
       !
      !
     !
    !
    policy p2mp-igp-blue-vpn1
     source ipv4 1.1.1.1
     color 201 endpoint-set leaf-R5-R6
     treesid mpls 15201
     candidate-paths
      constraints
       affinity
        include-any
         BLUE
        !
       !
      !
      preference 100
       dynamic
        metric
         type igp
        !
       !
      !
     !
    !
   !
   affinity bit-map
    RED 23
    BLUE 24
   !
  !
 !
!
...
```
[MID-node-R3](https://github.com/lambros90/xrdocs/blob/main/tree-sid-demo/MID-node-R3.txt)

[MID-node-R4](https://github.com/lambros90/xrdocs/blob/main/tree-sid-demo/MID-node-R4.txt)

[LEAF-R5](https://github.com/lambros90/xrdocs/blob/main/tree-sid-demo/LEAF-R5.txt)

[LEAF-R6](https://github.com/lambros90/xrdocs/blob/main/tree-sid-demo/LEAF-R6.txt)

### Outputs

### ROOT - R1

The root is the node where everything starts from. The traffic will be starting from here towards the rest of the nodes within the topology.

#### show run multicast-routing vrf red

```
ROOT-R1#show run multicast-routing vrf red
multicast-routing
vrf red
  address-family ipv4
   interface all enable
   bgp auto-discovery segment-routing
   !
   mdt default segment-routing mpls color 10
   mdt data segment-routing mpls 1 color 10 threshold 0
  !
!
!
```

#### show run segment-routing traffic-eng on-demand color 10

```
ROOT-R1#show run segment-routing traffic-eng on-demand color 10
segment-routing
traffic-eng
  on-demand color 10
   dynamic
    metric
     type igp
    !
    affinity
     include-any
      name RED
     !
    !
   !
  !
!
!
```

#### show run multicast-routing vrf blue

```
ROOT-R1#show run multicast-routing vrf blue
multicast-routing
vrf blue
  address-family ipv4
   interface all enable
   bgp auto-discovery segment-routing
   !
   mdt default segment-routing mpls color 20
   mdt data segment-routing mpls 1 color 20 threshold 0
  !
!
!
```

#### show run segment-routing traffic-eng on-demand color 20

```
ROOT-R1#show run segment-routing traffic-eng on-demand color 20
segment-routing
traffic-eng
  on-demand color 20
   dynamic
    metric
     type igp
    !
    affinity
     include-any
      name BLUE
     !
    !
   !
  !
!
!
```

#### show bgp vrf red ipv4 mvpn

```
ROOT-R1#show bgp vrf red ipv4 mvpn
BGP VRF red, state: Active
BGP Route Distinguisher: 2:20
VRF ID: 0x60000001
BGP router identifier 1.1.1.1, local AS number 1
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 187
BGP main routing table version 187
BGP NSR Initial initsync version 3 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
 
Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 2:20 (default for vrf red)
*> [1][1.1.1.1]/40    0.0.0.0                                0 i
*>i[1][1.1.1.5]/40    1.1.1.5                       100      0 i
*>i[1][1.1.1.6]/40    1.1.1.6                       100      0 i
*> [3][32][10.10.9.2][32][232.1.1.2][1.1.1.1]/120
                      0.0.0.0                                0 i
*>i[4][3][2:20][32][10.10.9.2][32][232.1.1.2][1.1.1.1][1.1.1.5]/224
                      1.1.1.5                       100      0 i
*>i[4][3][2:20][32][10.10.9.2][32][232.1.1.2][1.1.1.1][1.1.1.6]/224
                      1.1.1.6                       100      0 i
*>i[7][2:20][1][32][10.10.9.2][32][232.1.1.2]/184
                      1.1.1.5                       100      0 i
* i                   1.1.1.6                       100      0 i
 
Processed 7 prefixes, 8 paths
```

#### show bgp vrf blue ipv4 mvpn

```
ROOT-R1#show bgp vrf blue ipv4 mvpn
BGP VRF blue, state: Active
BGP Route Distinguisher: 1:20
VRF ID: 0x60000007
BGP router identifier 1.1.1.1, local AS number 1
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 187
BGP main routing table version 187
BGP NSR Initial initsync version 3 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
 
Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 1:20 (default for vrf blue)
*> [1][1.1.1.1]/40    0.0.0.0                                0 i
*>i[1][1.1.1.5]/40    1.1.1.5                       100      0 i
*>i[1][1.1.1.6]/40    1.1.1.6                       100      0 i
*> [3][32][10.10.8.2][32][232.1.1.1][1.1.1.1]/120
                      0.0.0.0                                0 i
*>i[4][3][1:20][32][10.10.8.2][32][232.1.1.1][1.1.1.1][1.1.1.5]/224
                      1.1.1.5                       100      0 i
*>i[4][3][1:20][32][10.10.8.2][32][232.1.1.1][1.1.1.1][1.1.1.6]/224
                      1.1.1.6                       100      0 i
*>i[7][1:20][1][32][10.10.8.2][32][232.1.1.1]/184
                      1.1.1.5                       100      0 i
* i                   1.1.1.6                       100      0 i
 
Processed 7 prefixes, 8 paths
```

#### show mvpn vrf red database segment-routing --> default mdt

```
ROOT-R1#show mvpn vrf red database segment-routing
* - LFA protected MDT
Core Type      Core            Tree Core        State  On-demand
              Source           Information             Color
 
Default           1.1.1.1       524291 (0x80003)    Up 10
   I-PMSI Leg:         1.1.1.5
                       1.1.1.6
Part              0.0.0.0            0 (0x00000)  Down 10
Control           0.0.0.0            0 (0x00000)  Down 10
```

#### show pim vrf red mdt sr-p2mp cache --> data mdt

```
ROOT-R1#show pim vrf red mdt sr-p2mp cache
 
Core Source      Cust (Source, Group)                Core Data        Expires
1.1.1.1          (10.10.9.2, 232.1.1.2)              [tree-id 524295]  never
 
   Leaf AD:      1.1.1.6
                 1.1.1.5
```

#### show mvpn vrf blue database segment-routing --> default mdt

```
ROOT-R1#show mvpn vrf blue database segment-routing
* - LFA protected MDT
Core Type      Core            Tree Core        State  On-demand
              Source           Information             Color
 
Default           1.1.1.1       524289 (0x80001)    Up 20
   I-PMSI Leg:         1.1.1.5
                       1.1.1.6
Part              0.0.0.0            0 (0x00000)  Down 20
Control           0.0.0.0            0 (0x00000)  Down 20
```

#### show pim vrf blue mdt sr-p2mp cache --> data mdt

```
ROOT-R1#show pim vrf blue mdt sr-p2mp cache
 
Core Source      Cust (Source, Group)                Core Data        Expires
1.1.1.1          (10.10.8.2, 232.1.1.1)              [tree-id 524293]  never
 
   Leaf AD:      1.1.1.6
                 1.1.1.5
```

### PCE-R2

#### show pce ipv4 peer

```
PCE-R2#show pce ipv4 peer
 
PCE's peer database:
--------------------
Peer address: 1.1.1.1
  State: Up
  Capabilities: Stateful, Segment-Routing, Update, Instantiation
 
Peer address: 1.1.1.3
  State: Up
  Capabilities: Stateful, Segment-Routing, Update, Instantiation
 
Peer address: 1.1.1.4
  State: Up
  Capabilities: Stateful, Segment-Routing, Update, Instantiation
 
Peer address: 1.1.1.5
  State: Up
  Capabilities: Stateful, Segment-Routing, Update, Instantiation
 
Peer address: 1.1.1.6
  State: Up
  Capabilities: Stateful, Segment-Routing, Update, Instantiation

```

#### show pce lsp p2mp root ipv4 1.1.1.1 | include Tree

```
PCE-R2#show pce lsp p2mp root ipv4 1.1.1.1 | include Tree
Tree: sr_p2mp_root_1.1.1.1_tree_id_524289, Root: 1.1.1.1 ID: 524289
Tree: sr_p2mp_root_1.1.1.1_tree_id_524291, Root: 1.1.1.1 ID: 524291
Tree: sr_p2mp_root_1.1.1.1_tree_id_524293, Root: 1.1.1.1 ID: 524293
Tree: sr_p2mp_root_1.1.1.1_tree_id_524295, Root: 1.1.1.1 ID: 524295

```

#### show pce lsp p2mp root ipv4 1.1.1.1 "tree id" --> data mdt for red

```
PCE-R2#show pce lsp p2mp root ipv4 1.1.1.1 524289
 
Tree: sr_p2mp_root_1.1.1.1_tree_id_524289, Root: 1.1.1.1 ID: 524289
PCC: 1.1.1.1
Label:    15576     Operational: up  Admin: up
Local LFA FRR: Disabled
Metric Type: IGP
Affinity: exclude-any 0x0 include-any 0x1000000 include-all 0x0
Transition count: 1
Uptime: 18:04:54 (since Thu Mar 31 23:08:39 UTC 2022)
Destinations: 1.1.1.5, 1.1.1.6
Nodes:
  Node[0]: 1.1.1.4 (MID-NODE-R4)
   Role: Transit
   Hops:
    Incoming: 15576 CC-ID: 1
    Outgoing: 15576 CC-ID: 1 (11.4.6.6) [LEAF-R6]
    Outgoing: 15576 CC-ID: 1 (10.4.5.5) [LEAF-R5]
  Node[1]: 1.1.1.1 (ROOT-R1)
   Role: Ingress
   Hops:
    Incoming: 15576 CC-ID: 2
    Outgoing: 15576 CC-ID: 2 (10.1.4.4) [MID-NODE-R4]
  Node[2]: 1.1.1.6 (LEAF-R6)
   Role: Egress
   Hops:
    Incoming: 15576 CC-ID: 3
  Node[3]: 1.1.1.5 (LEAF-R5)
   Role: Egress
   Hops:
    Incoming: 15576 CC-ID: 4
```
  
#### show pce lsp p2mp root ipv4 1.1.1.1 "tree id" --> data mdt for blue
  
```
PCE-R2#show pce lsp p2mp root ipv4 1.1.1.1 524293
 
Tree: sr_p2mp_root_1.1.1.1_tree_id_524293, Root: 1.1.1.1 ID: 524293
PCC: 1.1.1.1
Label:    15578     Operational: up  Admin: up
Local LFA FRR: Disabled
Metric Type: IGP
Affinity: exclude-any 0x0 include-any 0x1000000 include-all 0x0
Transition count: 1
Uptime: 18:05:22 (since Thu Mar 31 23:08:38 UTC 2022)
Destinations: 1.1.1.5, 1.1.1.6
Nodes:
  Node[0]: 1.1.1.4 (MID-NODE-R4)
   Role: Transit
   Hops:
    Incoming: 15578 CC-ID: 1
    Outgoing: 15578 CC-ID: 1 (11.4.5.5) [LEAF-R5]
    Outgoing: 15578 CC-ID: 1 (11.4.6.6) [LEAF-R6]
  Node[1]: 1.1.1.1 (ROOT-R1)
   Role: Ingress
   Hops:
    Incoming: 15578 CC-ID: 2
    Outgoing: 15578 CC-ID: 2 (11.1.4.4) [MID-NODE-R4]
  Node[2]: 1.1.1.5 (LEAF-R5)
   Role: Egress
   Hops:
    Incoming: 15578 CC-ID: 3
  Node[3]: 1.1.1.6 (LEAF-R6)
   Role: Egress
   Hops:
    Incoming: 15578 CC-ID: 5
```
  
### MID-NODE-R4

#### show segment-routing traffic-eng p2mp policy root ipv4 1.1.1.1
  
```
MID-NODE-R4#show segment-routing traffic-eng p2mp policy root ipv4 1.1.1.1
 
SR-TE P2MP policy database:
----------------------
! - Replications with Fast Re-route, * - Stale dynamic policies/endpoints
 
Policy: sr_p2mp_root_1.1.1.1_tree_id_524289  LSM-ID: 0x40016
Root: 1.1.1.1, ID: 524289
Role: Transit
Replication:
  Incoming label: 15576 CC-ID: 1
  Interface: TenGigE0/0/0/1 [11.4.6.6]  Outgoing label: 15576 CC-ID: 1
  Interface: TenGigE0/0/0/9 [10.4.5.5]  Outgoing label: 15576 CC-ID: 1
 
Policy: sr_p2mp_root_1.1.1.1_tree_id_524293  LSM-ID: 0x40015
Root: 1.1.1.1, ID: 524293
Role: Transit
Replication:
  Incoming label: 15578 CC-ID: 1
  Interface: TenGigE0/0/0/8 [11.4.5.5]  Outgoing label: 15578 CC-ID: 1
  Interface: TenGigE0/0/0/1 [11.4.6.6]  Outgoing label: 15578 CC-ID: 1
```
  
#### show mpls forwarding labels "tree-sid"

```
RP/0/RP0/CPU0:MID-NODE-R4#show mpls forwarding labels 15578
Fri Apr  1 16:10:48.055 UTC
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes
Label  Label       or ID              Interface                    Switched
------ ----------- ------------------ ------------ --------------- ------------
15578  15578       mLDP/IR: 0x00000   Te0/0/0/8    11.4.5.5        0
       15578       mLDP/IR: 0x00000   Te0/0/0/1    11.4.6.6        0
```
  
### LEAF-R6
  
In this node we can test the replication by adding/ removing a Leaf node. To do so, we remove the receiver at R6 from l2vpn by removing BV1 from l2vpn bridge-domain blue by executing the follow config:
  
@ R6 apply the following:
  
```
l2vpn
  bridge group bg
    bridge-domain blue
      no routed interface BVI1
commit
end
```
  
Then we get the following output at R4:
 
```
MID-NODE-R4#show mpls forwarding labels 15578
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes
Label  Label       or ID              Interface                    Switched
------ ----------- ------------------ ------------ --------------- ------------
15578  15578       mLDP/IR: 0x00000   Te0/0/0/8    11.4.5.5        0
```

By comparing this output to the same output 2 steps back we see that the interface 10.4.6.6 is removed from the list.
