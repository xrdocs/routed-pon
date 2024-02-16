---
published: true
date: '2024-02-14 16:13 -0800'
title: Cisco Routed PON Whitepaper
author: Tejas Lad
excerpt: This document introduces the Cisco Routed PON solution
tags:
  - iosxr
  - cisco
  - PON
  - Routed-PON
  - Routed PON
  - OLT
position: hidden
---
{% include toc icon="table" title="Cisco Routed PON" %}

## Introduction

Since its development in mid 1990's, [Passive Optical Network (PON)](https://www.cisco.com/c/en/us/products/switches/what-is-passive-optical-networking.html) has evolved a long way. In the initial deployments, Fiber to the home - FTTH was the dominating use case. Recently new business cases like video (8k, 12k), AR/VR, Enhanced Gaming, and Enterprise Services are driving the needs for broadband network transformation. To prepare for the future growth, industry is moving from 1G GPON to 10G XGS-PON. XGS-PON as defined by ITU-T G.9807.1, supports high speed 10G symmetrical data rates in both upstream and downstream directions. XGS-PON is becoming the popular choice of deployment for last mile and middle mile operators. Apart from subscriber needs, there are various [broadband funding initiatives](https://www-author.cisco.com/content/en/us/solutions/service-provider/rural-broadband.html?wcmmode=disabled) taken by the governments worldwide to provide broadband access to the underserved communities. This brings in new opportunities for the Broadband Service Providers - BSPs to establish a new network foundation to transform the rural communities and make reliable connectivity more accessible with Cisco solutions and expertise.

## Challenges of the current PON deployments

![Screenshot 2024-02-15 at 12.45.27 PM.png]({{site.baseurl}}/images/Screenshot 2024-02-15 at 12.45.27 PM.png)

Traditional deployments have huge OLT (Optical line terminal) chassis terminating on the BSPs access or aggregation network. They typically have flat Layer2 domains using native ethernet technologies. Though many operators feel that deploying layer2 switching is simpler but [moving to IP](https://xrdocs.io/design/blogs/2023-11-15-routed-access-for-rural-broadband/) brings in more benefits to access networks. BSPs are looking for faster, simpler and economical ways of providing the subscriber service and there are various challenges faced by them, when it comes to deploying single purpose OLT chassis:

- **Vendor Lockin**: Traditional OLT chassis tend to have a closed and proprietary software ecosystem which causes the product and support available only from a particular vendor. These causes the BSPs to depend on a fixed vendor.
- **Increase in OPEX**: The OLT chassis need dedicated power and space requirements causing resulting in higher operational cost.
- **Lack of Solution Modularity**: The OLT chassis lack pay-as-you grow model and BSPs end up purchasing full line cards. This sometimes ends up having the chassis under utilized.  
- **Difficulties to upgrade to higher speeds**: Upgrading from 1G to 10G/25G/50G becomes a heavy investment for the BSPs. The older chassis are not capable of supporting the higher speeds and they have to be forklifted to be replaced by newer chassis.
- **Software upgrades**: Operators have to maintain two different life cycles. One of the network equipments and other for the OLT chassis. This sometimes increases the operational overhead.
- **Lack of MEF compliance**: The purpose built OLT chassis lack or have minimal support for MEF compliance. This poses a problem for the operators to deliver services and SLA compliance as per MEF standards.


## Cisco's Approach to the problem

Cisco's PON solution aims at solving the above problems by collapsing the OLT chassis to a pluggable form factor. PON is now considered just another ethernet port.

![Screenshot 2024-02-15 at 2.48.38 PM.png]({{site.baseurl}}/images/Screenshot 2024-02-15 at 2.48.38 PM.png)

This will make PON network a direct part of the [Routed Access Layer](https://xrdocs.io/design/blogs/2023-11-15-routed-access-for-rural-broadband/). Operators can now directly plug the Cisco PON SFP+ into 10G ports of [NCS540](https://www.cisco.com/c/en/us/products/routers/network-convergence-system-540-series-routers/index.html) and [NCS5500/NCS5700](https://www.cisco.com/c/en/us/products/routers/network-convergence-system-5500-series/index.html) series routers. Operators can now leverage port based scalable model and decide whether to have all the ports with PON or Ethernet optics. This also allows form factor for future growth to higher speeds like 25G or 50G PON. Operators can choose the native ports that support the 25G and 50G form factors and use the existing infrastructure to deliver the new services. This results in the elimination of the dedicated PON chassis built for single purpose. The Cisco Routed PON solution allows the PON services to be delivered seamlessly in similar manner like direct internet access, mobile backhaul and WAN services.

## Solution Architecture and Components 

Below are the main components of the solution:


### Cisco PON OLT

![Screenshot 2024-02-15 at 4.42.23 PM.png]({{site.baseurl}}/images/Screenshot 2024-02-15 at 4.42.23 PM.png)


Cisco PON OLT is a feature rich device contained in a hot pluggable SFP+ form factor facilitating a software defined broadband network deployment. It contains built-in 10G Ethernet to XGS PON MAC Bridge IC and L1 optical transceiver. The integrated chip allows the module to connect to a PON network to a point-to-point Ethernet SFP+ port on the routers. It supports port by port expansion on NCS540 and NCS5500/5700 router ports. It is suitable for high bandwidth business PON connectivity as well as high density PON aggregation. It supports symmetric rates of 10G upstream and downstream. It is compliant with [ITU-T G.9807.1 XGS PON](https://www.itu.int/rec/T-REC-G.9807.1) standard. It can interop with various non proprietary ONTs/ONUs available in the market as long as it supports OMCI protocol. Below are some of the quick facts of the OLT.

| Dimension(H x W x D)      | 8.55mm x 13.4mm x 80.65mm                        |
|---------------------------|--------------------------------------------------|
| PID                       | SFP-10G-OLT20-X                                  |
| Data rate                 | Symmetric rates: 9.95G upstream/9.95G downstream |
| Connector Type            | SC/UPC                                           |
| Maximum Distance          | 20 km                                            |
| Operating Temperature     | -20°C to 75°C                                    |
| Typical Power Consumption | 2.475W                                           |
| Average Launch Power      | 4 dbm min 7 dbm max                              |
| ODN Class                 | N2                                               |
| Cable Type                | Single Mode Fiber                                |


### Cisco PON Controller

### Cisco PON Manager
