---
published: true
date: '2021-03-22 06:48 -1000'
title: Multicast 101 Introduction
author: Lampros Gkavogiannis
excerpt: Multicast 101 Introduction
position: top
tags:
  - iosxr
  - cisco
  - multicast
---

## Multicast 101

Multicast is a technology that allows us to send IP packets to a group of receivers in a single transmission. It has been developed in order to save network resources such as bandwidth by sending information to a specific group of devices and avoid duplication examples.

It is designed to be used for applications that have multiple receivers also known as one one-to-many or many-to-many. Such applications can be:

- Audio and video communication (live video distribution)
- Financial applications (stock exchange market) 
- Television 
- Data distribution and caching 

![multicast_101_introduction_1_1.png]({{site.baseurl}}/images/multicast_101_introduction_1_1.png)

The picture above illustrates a basic multicast topology. There is the server, the routers interconnected with each other and the receivers.

![multicast_101_introduction_1_2.png]({{site.baseurl}}/images/multicast_101_introduction_1_2.png)

In a typical Unicast scenario, the traffic would be broadcasted by the server to all the connected devices while in Multicast, traffic streams will be replicated on each node to be delivered to the next hop. The receivers consist the Multicast “Group”, which is a group of IPs willing to receive the same traffic stream. The server creates the stream, sends it to the first node and the packet is replicated. 

![multicast_101_introduction_1_3.png]({{site.baseurl}}/images/multicast_101_introduction_1_3.png)

Once the packet is replicated, it moves to the next hop/ node and keeps on flowing until the packet reaches the receivers.

![multicast_101_introducction_1_4.png]({{site.baseurl}}/images/multicast_101_introducction_1_4.png)

The receivers belong to the “group” 224.1.1.1 and this is the destination of the packet. Whenever there is a branch in the network connected to a receiver the packet is replicated and sent to it. The packet has no information about the receivers but only for the “group”.
