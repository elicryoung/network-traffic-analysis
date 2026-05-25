---
title: Network Traffic Analysis
subtitle: Wireshark Packet & Protocol Investigation
author: Eli Young
project: 2 of 4
---

# Network Traffic Analysis

## Practical Wireshark & Packet Investigation Project

---

# Project Overview

Having completed the COMPTIA Network+ Exam, this project builds on the theoretical knoweldge gained and puts it into practice. Focusing on developing foundational packet analysis and network traffic investigation skills using Wireshark and PCAP analysis. 

The goal is to understand how common network protocols operate in practice, how analysts inspect packet captures, and how suspicious traffic patterns can be identified during security investigations.

This project also reinforces networking concepts studied as part of Network+ preparation, including:

- TCP/IP communication
- protocol layering
- packet Structure
- broadcast communication
- common network services
- 
The project will later expand into:
- protocol deep dives
- suspicious traffic analysis
- command-and-control detection
- DNS tunneling analysis
- PCAP investigation write-ups

---

# Contents

1. Phase 1 — Environment Setup & Wireshark Basics
2. Phase 2 — Real-Time Traffic & Protocol Deep Dives
3. Phase 3 - Suspicious Traffic Patterns
4. Phase 4 - Analysis Challenges and Documentation

---

# Phase 1  
## Environment Setup
Firstly, I updated wireshark  on my computer as I had already used the application during my masters degree. For the first phase I initialised the github repo and created my folder structure for my analysis. 

A structured project directory named `network=traffic-analysis` was then created, separating:
- investigation write-ups
- documentation
- pcap-files
- iamges (screenshots)

Because this project is a little bit less like a real world investigation and more about getting to grips with network traffic and packet analysis I won't be mirroring any type of organisation expected in a workplace. 

## Wireshark basics

I went online and just searched for 'sample pcap files for analysis' adn the first thing that came up was `https://wiki.wireshark.org/SampleCaptures#tcp`.
It is a directory with a ton of free to download wireshark sample captures for people to analyse. There are so many to choose from, I decided for phase 1 that two simple and easy to understand captures would be more than enough to learn about the interface of and how to navigate Wireshark. The two .pcap files I went with are:

- `dhcp.pcap`
- `ipv4frags.pcap`

### DHCP

I started with `dhcp.pcap`, and I'm glad I did because it was extremely basic and perfect to learn about the types of headers and fields within the application. see img 1.1.

The packet list pane at the top of Wireshark displays a summary of each captured packet, including:
- packet number #
- timestamp
- source and destination addresses
- protocol
- packet size
- packet summary information


This was a good practical reminder of how broadcast communication works during DHCP assignment. The client initially has no configured IP address, so traffic is broadcast across the local network until the DHCP server responds.

The capture also reinforced how useful Wireshark's layout is for quickly understanding network conversations. Even without opening the individual packet details, the packet list pane alone already tells a very clear story about what is happening.

At this stage, the main focus wasn't really about protocol analysis and more about becoming comfortable navigating:
- packet summaries
- protocol identification
- communication flows
- timestamps
- source and destination relationships
before moving into larger and more complex captures later in the project.

A cool feature of Wireshark is opening individual network packets, for example when I opened the initial Discover message it displays much more in depth information about it (see img 1.2). For example:
- packet size
- UTC arrival time
- UDP SRC & DST Port numbers (67/68 for DHCP)
- TTL in the IP field.
This could be really useful in packet analysis to get deeper than shown in the home interface which kind of summarises the packets information.

One thing I've noticed is that all the colours behind the DHCP messages are light blue, wondering if there are other colours depending on the different types of messages being thrown around (e.g. arp, dns etc)


### ipv4frags
As suspected, the colour changes does change depending on the message, as seen in img 1.3. I have made a `wireshark-cheatsheet.md`, to keep track of the tools/features of wireshark for future use.

The second PCAP I looked at was `ipv4frags.pcap`, which introduced fragmented IPv4 traffic and ICMP (`ping`) communication. Even though the capture only contained three packets, it was already noticeably more technical than the DHCP example. See img 1.3.

What's really interesting about this capture is in the first message, the ipv4 message, in the info section, it says that the message was originally framgent and has been reassembled.
It says "Reassembled in #2" meaning that it was two messages and the second message was the end of it. Basically “packet #1 is part of the full ICMP message that Wireshark reconstructed and displayed in packet #2”, which is incredibly handy when trying to analyis network traffic. 
 
 The following messages were just simple `ping` messages, a request and reply from 2.1.1.2 to 2.1.1.1.

## outcome
By the end of Phase 1 I have:
- updated and configured Wireshark
- downloaded and analysed sample PCAP files
- learned the basic Wireshark interface layout
- investigated DHCP and ICMP traffic
- explored packet headers and protocol fields
- started building a personal Wireshark cheat sheet

# Phase 2
## Real-Time Traffic.



