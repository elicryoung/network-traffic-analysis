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
4. Phase 4 - Documentation (this readme)

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
## Suspicious Traffic Patterns

Having gained experience analysing normal network traffic in Phase 2, the next objective was to investigate packet captures containing malicious or suspicious activity.

To find suitable captures, I researched publicly available cybersecurity training resources and came across Malware-Traffic-Analysis.net, a website that provides packet captures and investigation scenarios designed to simulate real-world malware infections and incident response investigations.

The training exercises can be found at:

https://www.malware-traffic-analysis.net/training-exercises.html

Unlike the traffic analysed in previous phases, these captures are specifically designed to contain malicious activity and require the analyst to investigate the network traffic to determine what happened, which systems were affected, and what evidence exists within the packet capture.

---

## Investigation 1 - Easy As 123

For the first investigation, I selected the exercise **"Easy As 123"**.

The scenario begins with a Security Operations Centre (SOC) receiving alerts from a Security Information and Event Management (SIEM) platform indicating possible NetSupport Manager RAT activity (remote admin tool). The alerts identify communications with the external IP address **45.131.214.85** over TCP port **443**, beginning on **28 February 2026 at 19:55 UTC**.

The exercise provides a packet capture containing traffic from the affected network segment and challenges the analyst to identify the infected Windows host and the user associated with the activity.

The network environment consists of an Active Directory domain named **EASYAS123** operating on the **10.2.28.0/24** network.

The objective of this investigation is to analyse the packet capture using Wireshark, identify the infected workstation, trace its communications, and gather enough evidence to determine who was using the system at the time of the infection.

The exercise requires identification of:

- Infected Windows client IP address
- Infected Windows client MAC address
- Host name
- User account name
- Full name of the associated user

Before investigating the malicious traffic, I first gathered some general information about the packet capture to better understand the environment and the types of protocols present.

### Capture Overview

| Metric | Value |
|---|---|
| File Name | 2026-02-28-traffic-analysis-exercise.pcap |
| Capture Duration | 04:21:29 |
| First Packet Time | 2026-02-28 19:55:06 |
| Last Packet Time | 2026-03-01 00:16:35 |
| Total Packets | 15512 |
| Average Packets/sec | 1.0 |
| Average Packet Size | 408B |

### Protocol Hierarchy

| Protocol | Percentage | Packets |
|---|---:|---:|
| TCP | 80.9 | 12544 |
| UDP | 13.6 | 2113|
| DNS | 5.2 | 809 |
| HTTP | 2.3 | 354 |
| ARP | 5.4 | 836 |
| ICMP | 0.1 | 19 |

### Investigation

What are we given?

`LAN segment range: 10.2.28.0/24 (10.2.28.0 through 10.2.28.255)`
- Anything start with 10.2.28.x is likely internal
- Anything outside that range is likey external
- The infected machine should be somewhere inside this range because 

`Domain: easyas123[.]tech`
- This is the organisations Windows/Active Directory domain name
- it helps to idenity traffic that belongs to the victim environment 
- eg "DESKTOP-TEYQ2NR.easyas123.tech" likey means "Windows workstation joined to the easyas123.tech domain"


`AD environment name: EASYAS123`
- This is the short Windows domain / NetBIOS - style name
- Tells us the domain account name 
- eg "EASYAS123\jsmith" means user jsmith from EASYAS123 Domain


`Active Directory (AD) domain controller: 10.2.28[.]2 - EASYAS123-DC`
- A domain controller handles things like:
* Logins
* Kerberos authentication
* LDAP directory lookups
* Group policy
* Domain DN

So traffic to/from 10.2.28.2 may reveal:
* Hostnames
* Usernames
* Authentication activity
* Domain membership

`LAN segment gateway: 10.2.28[.]1`
- This is likely the router/firewall for the network.
- Internal machines use this to reach the internet

`LAN segment broadcast address: 10.2.28[.]255`
- standard broadcast of the network

---

## Step 1 - Identifying the Infected Host

The exercise already provides a strong Indicator of Compromise (IOC):

```text

45.131.214.85

```

Therefore, my first step was to determine which internal host was communicating with this address.

### Filter Used

```wireshark

ip.addr == 45.131.214.85

```

### Findings

This filter returned approximately **550 packets**, representing around **3.5%** of all traffic in the capture.

The majority of this traffic involved the internal host:

```text

10.2.28.88

```

The traffic was primarily:

- TCP

- HTTP

I also noticed that communications began roughly 45 seconds into the capture and then appeared repeatedly at approximately 60-second intervals before stopping.

This pattern immediately stood out because periodic communications are commonly associated with **beaconing behaviour**, where malware regularly checks in with a command-and-control server.

At this point I suspected that **`10.2.28.88`** was the infected workstation.

To verify this assumption, I refined the filter:

```wireshark

ip.addr == 10.2.28.88 && ip.addr == 45.131.214.85

```

The results displayed the same communications, reinforcing the conclusion that **`10.2.28.88`** was the system responsible for the suspicious traffic.

### Following the TCP Stream

Having identified the likely infected host, I wanted to understand the nature of the communications.

To do this I selected one of the packets and used:

```text

Follow → TCP Stream

```

This reconstructs the full conversation between the two hosts.

Within the stream I observed repeated requests such as:

```http

HTTP/1.1 200 OK

Server: NetSupport Gateway/1.92 (Windows NT)

POST http://45.131.214.85/fakeurl.htm HTTP/1.1

User-Agent: NetSupport Manager/1.3

Host: 45.131.214.85

```

(See Figure 3.2)

Several important observations can be made from this:

- The traffic explicitly references **NetSupport Gateway**

- The User-Agent identifies itself as **NetSupport Manager**

- The requests are directed toward the exact IP identified by the SIEM alert

- The communication pattern is repetitive and consistent with beaconing behaviour

This was a significant finding because the traffic was no longer simply suspicious due to the destination IP address. The packet contents themselves directly referenced the software identified in the alert.

At this stage I was confident that:

| Artifact | Value |
|---|---|
| Suspected Infected Host | **10.2.28.88** |
| External RAT Server | **45.131.214.85** |
| Malware Activity | **NetSupport Manager / Gateway** |
| URL Observed | **http://45.131.214.85/fakeurl.htm** |
| Behaviour | **Beaconing / periodic communication** |

---

## Step 2 - Identifying the MAC Address

The next objective was to identify the MAC address of the infected workstation.

I applied the filter:

```wireshark

ip.addr == 10.2.28.88

```

I then selected a packet and expanded the **Ethernet II** header.

The source MAC address was:

```text

00:19:d1:b2:4d:ad

```

This provides a unique identifier for the network adapter associated with the infected workstation.

---

## Step 3 - Identifying the Hostname

With the IP address and MAC address identified, the next objective was to determine the hostname of the infected machine.

### Initial Attempt - DHCP

My first thought was to inspect DHCP traffic because hostnames are often included during DHCP lease requests.

I applied:

```wireshark

ip.addr == 10.2.28.88 && dhcp

```

While this confirmed the IP-to-MAC relationship, it did not reveal a hostname.

Although this approach did not provide the answer, it was still useful because it validated information already discovered.

### Pivoting to Domain Controller Traffic

Since Windows systems communicate regularly with the Domain Controller for authentication and directory services, I decided to focus on traffic between the suspected host and the Domain Controller.

```wireshark

ip.addr == 10.2.28.88 && ip.addr == 10.2.28.2

```

This revealed several protocol types including:

- DNS

- SMB

- Kerberos

The DNS and SMB traffic helped confirm domain activity but did not immediately reveal the workstation name.

The most useful protocol turned out to be **Kerberos**.

Within a Kerberos **AS-REQ** packet I observed:

```text

HostAddress: DESKTOP-TEYQ2NR<20>

```

The packet originated from:

```text

Source IP:      10.2.28.88

Destination IP: 10.2.28.2

```

This indicated that the workstation identifying itself during authentication was:

```text

DESKTOP-TEYQ2NR

```

This allowed me to identify the hostname of the infected workstation.

---

## Step 4 - Identifying the User Account

The next step was to identify the user associated with the workstation.

While examining the same Kerberos authentication traffic, I found the Kerberos **cname** field.

In Kerberos, **cname** stands for **Client Name** and identifies the account requesting authentication.

The value contained:

```text

brolf

```

This indicated that the workstation was authenticating using the account:

```text

EASYAS123\brolf

```

At this point I had identified the username, but not the individual behind the account.

---

## Step 5 - Identifying the User's Full Name

My first attempt was to search for references to **brolf** elsewhere in the capture.

I inspected packet details and reviewed LDAP traffic, expecting to find directory information associated with the account. However, this did not produce any useful results.

At this point I researched the Windows protocols present in the capture and discovered **SAMR (Security Account Manager Remote)** traffic.

SAMR is used by Windows systems to query account information from the Domain Controller.

While reviewing SAMR traffic, I identified packets such as:

```text

339    16.755086    10.2.28.2    10.2.28.88    SAMR    QueryUserInfo Response

```

Opening the packet revealed account information associated with the username.

The response contained:

```text

Account Name: brolf

Full Name: Becka Rolf

```

This positively identified the user associated with the infected workstation.

---

## Evidence Chain

```text

45.131.214.85 (IOC from SIEM)

            ↓

10.2.28.88 (Internal host communicating with IOC)

            ↓

00:19:d1:b2:4d:ad (MAC Address)

            ↓

DESKTOP-TEYQ2NR (Hostname)

            ↓

EASYAS123\brolf (User Account)

            ↓

Becka Rolf (User Identity)

```

---

## Final Findings

| Item | Value |

|---|---|

| Infected IP Address | **10.2.28.88** |

| MAC Address | **00:19:d1:b2:4d:ad** |

| Hostname | **DESKTOP-TEYQ2NR** |

| User Account | **brolf** |

| Full Name | **Becka Rolf** |

The investigation successfully traced the SIEM alert from the malicious external IP address through to the affected workstation, associated user account, and ultimately the individual using the system at the time of the activity.