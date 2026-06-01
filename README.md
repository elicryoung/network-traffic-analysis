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

Having completed the COMPTIA Network+ Exam, this project builds on the theoretical knowledge gained and puts it into practice. Focusing on developing foundational packet analysis and network traffic investigation skills using Wireshark and PCAP analysis. 

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
3. Phase 3 - Suspicious Traffic Patterns & Analysis Challenges
4. Phase 4 - Documentation

---

# Phase 1  
## Environment Setup
Firstly, I updated wireshark  on my computer as I had already used the application during my masters degree. For the first phase I initialised the github repo and created my folder structure for my analysis. 

A structured project directory named `network=traffic-analysis` was then created, separating:
- investigation write-ups
- documentation
- pcap-files
- images (screenshots)

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
## Real-Time Traffic
For this phase I wanted to capture a portion of realtime traffic from my own network and analysis it using filters and other wireshark features to get a sense of what a real network security anlysist would do on a day-to-day basis.

First thing to do was choose the network to record, I chose my wireless access point closest to my work station. I used wiresharks record feature the "Start capturing packets" button which looks like a shark fin. The section of traffic I captured was as follows

| Metric | Value |
|----------|----------|
| Capture Duration | ~134 seconds |
| Total Packets | 3,642 |
| Primary Protocol | TCP (32.5%)|
| IPv6 Traffic | Present (34.9%)|
| DNS Traffic | Present |
| ARP Traffic | Present |
| NTP Traffic | Present |
| Primary Application Traffic | HTTPS (TCP/443) |

How I got this information was ny using the statistics page on wireshark, specifically:

`Statistics → Capture File Properties` (img 2.1)
`Statistics → Protocol Hierarchy` (img 2.2)

### Initial Observations

One thing that stood out immediately was just how much traffic was being generated in a relatively short period of time. In just over two minutes, more than 3,600 packets were captured, without doing anything specific.

The majority of traffic was TCP-based, with a large amount of IPv6 traffic also present. Most application traffic was encrypted HTTPS traffic over port 443, which is expected on a modern network.

Another thing I noticed was how difficult it would be to investigate a packet capture without using filters. With thousands of packets visible at once, it quickly becomes overwhelming to identify useful information manually.

This reinforced why packet filtering is such an important skill for network and security analysts.

## Using Filters

To make the capture easier to analyse, I used a number of Wireshark display filters to isolate specific protocols and types of traffic.

### DNS

```wireshark
dns
```

This filter displayed only DNS requests and responses. DNS traffic is particularly useful because it reveals which domains a system is communicating with, even when the actual application traffic is encrypted.

Using this filter I was able to observe requests for a number of Microsoft, VS Code and GitHub Copilot related services.

`487	29.165834	172.20.10.2	172.20.10.1	DNS	79	Standard query 0x696b HTTPS main.vscode-cdn.net`

Was the initial DNS query from my machine to the DNS server

### TCP

```wireshark

tcp

```

This filter displayed all TCP traffic. TCP was responsible for the majority of communication within the capture and included most HTTPS connections. Because it is so broad on its own, there is a huge amount of information on the interface and definitely needs to use additional filters. 

### HTTPS

```wireshark

tcp.port == 443

```

This filter isolated encrypted HTTPS traffic. Although the packet contents could not be viewed directly due to encryption, source and destination information was still visible. Again, displays a lot of traffic due to most traffic being forced to use port 443 on modern web browsers.



### TCP Connection Establishment

```wireshark

tcp.flags.syn == 1 && tcp.flags.ack == 0

```

This filter displayed the initial SYN packets used to begin TCP connections. These packets represent the first step of the TCP three-way handshake and allowed me to observe new connections being established between hosts.

---

### TCP Retransmissions

```wireshark

tcp.analysis.retransmission

```

This filter displayed packets that Wireshark identified as retransmissions. Retransmissions occur when a packet is not acknowledged and must be sent again. A small number of retransmissions is normal on most networks and can occur due to packet loss or temporary congestion.

---

### TCP Reset Packets

```wireshark

tcp.flags.reset == 1

```

This filter displayed TCP reset (RST) packets. Reset packets are used to immediately terminate a connection and can indicate that a service is unavailable, a connection was rejected, or communication was interrupted unexpectedly.

---

### IPv6 Traffic From a Specific Host

```wireshark

ipv6.addr == <address>

```

This filter isolated traffic associated with a specific IPv6 address. It was useful for examining communication involving a particular device and understanding how IPv6 was being used within the capture.

### ARP

```wireshark

arp

```

This filter displayed Address Resolution Protocol traffic. ARP traffic showed devices resolving MAC addresses on the local network before communication could occur.

### IPv6

```wireshark

ipv6

```

This filter displayed IPv6 traffic. I was surprised by how much IPv6 communication was present, highlighting how commonly modern systems use IPv6 alongside IPv4.

## Summary

This phase provided my first experience analysing a live packet capture generated from my own network traffic. Using Wireshark, I captured over 3,600 packets in just over two minutes, demonstrating how active a modern system is even when relatively idle.

By examining protocol statistics and applying display filters, I was able to isolate and investigate different types of traffic including TCP, HTTPS, DNS, ARP, and IPv6. This made it significantly easier to identify patterns within the capture and understand the role each protocol plays in normal network communication.

One of the most valuable lessons from this exercise was learning how much information can be obtained from network metadata alone, so much information comes from such basic things that we take for granted when using the internet. Although the majority of application traffic was encrypted using HTTPS, DNS requests, connection information, and protocol behaviour still provided useful insight into what services were being accessed and how communication was occurring.

Overall, this phase improved my confidence using Wireshark and reinforced the importance of packet filtering when analysing network traffic. It also provided a practical introduction to the type of traffic investigation and protocol analysis that security analysts perform during day-to-day monitoring and incident response activities.

# Phase 3
## Suspicious traffic patterns

Having gained experience analysing normal network traffic in Phase 2, the next objective was to investigate network captures containing realistic attack activity. To achieve this, a suitable source of publicly available packet captures was needed.

After researching datasets commonly used for network security training and incident response exercises, I discovered the Netresec MACCDC packet capture archive. The archive contains traffic captured during the Mid-Atlantic Collegiate Cyber Defense Competition (MACCDC), a cybersecurity competition where teams defend enterprise networks while being subjected to realistic attacks and intrusion attempts. Found at `https://www.netresec.com/?page=MACCDC` (see img 3.1)

The MACCDC datasets are widely used within the cybersecurity community because they contain genuine network activity, including both legitimate business traffic and malicious behaviour such as reconnaissance, exploitation attempts, and other forms of suspicious communication. This makes them valuable resources for developing packet analysis and threat hunting skills.

For this phase, I selected two packet capture files for investigation/analysis:

- maccdc2010_00000_20100310205651.pcap
- maccdc2010_00003_20100311184520.pcap

These captures were imported into Wireshark and analysed using a combination of protocol statistics, endpoint analysis, conversation tracking, and packet filtering techniques. The objective was to identify indicators of suspicious activity and apply the investigative process that a Security Operations Centre (SOC) analyst might follow when examining network traffic during a security incident.

## maccdc2010_00000_20100310205651.pcap

### Initial Capture Overview

| Metric | Value |
|---|---|
| File name | maccdc2010_00000_20100310205651.pcap |
| Capture duration | 21:17:05 |
| First packet time | 2010-03-10 19:56:51 |
| Last packet time | 20-03-11 17:13:56 |
| Total packets | 10000000 |
| Average packets/sec | 130.5 |
| Average packet size | 77B |

