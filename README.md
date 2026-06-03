# Phase 1 - Getting Familiar with Wireshark

Before diving into larger packet captures, I wanted to spend some time getting comfortable with Wireshark itself.

I downloaded a couple of sample PCAP files from the Wireshark sample capture repository and used them to explore the interface, inspect packets, and understand how different protocols appear inside the application.

The two captures I looked at were:

- `dhcp.pcap`
- `ipv4frags.pcap`

### DHCP

The DHCP capture was a great starting point because there wasn't much going on and it was easy to follow.

Using this capture I got familiar with the three main Wireshark panes:

- Packet List
- Packet Details
- Packet Bytes

I could also clearly see the DHCP process taking place as the client attempted to obtain an IP address from the network. Since the client didn't yet have an IP address, the initial traffic was broadcast across the local network.

Opening individual packets also allowed me to inspect fields such as:

- Source and destination ports
- TTL values
- Packet timestamps
- Protocol-specific information

(See Images 1.1 and 1.2)

### IPv4 Fragmentation

Next I looked at `ipv4frags.pcap`.

This capture introduced fragmented IPv4 traffic and was my first time seeing packet reassembly in practice.

One thing that stood out was Wireshark's ability to reconstruct fragmented packets automatically. The capture showed packets being marked as "Reassembled in #2", indicating that Wireshark had combined multiple fragments back into a complete message.

Although the capture was small, it introduced a concept that becomes important when analysing larger network captures.

(See Image 1.3)

### What I Learned

By the end of this phase I was comfortable:

- Navigating the Wireshark interface
- Opening and inspecting individual packets
- Following communication flows
- Understanding DHCP broadcasts
- Identifying fragmented and reassembled traffic

Most importantly, I felt comfortable enough with the tool to start analysing traffic that I captured myself.

---------

# Phase 2 - Real-Time Traffic Analysis

Once I felt comfortable using Wireshark, I wanted to analyse some real traffic from my own network.

The goal wasn't necessarily to find anything suspicious. Instead, I wanted to get used to working with larger captures and learn how analysts use filters to investigate network activity.

I captured approximately two minutes of traffic from my wireless network.

## Capture Overview

| Metric | Value |
|----------|----------|
| Capture Duration | ~134 seconds |
| Total Packets | 3,642 |
| Primary Protocol | TCP (32.5%) |
| IPv6 Traffic | Present (34.9%) |
| DNS Traffic | Present |
| ARP Traffic | Present |
| NTP Traffic | Present |
| Primary Application Traffic | HTTPS (TCP/443) |

Capture statistics were gathered using:

- `Statistics → Capture File Properties`
- `Statistics → Protocol Hierarchy`

(See Images 2.1 & 2.2)

## Initial Observations

One thing that surprised me immediately was how much traffic was generated in such a short period of time. I wasn't doing anything particularly demanding, yet thousands of packets were captured.

The majority of traffic was TCP-based, with a significant amount of IPv6 traffic also present. Most application traffic was encrypted HTTPS traffic, which is exactly what you would expect on a modern network.

It quickly became obvious why filters are so important. Looking at thousands of packets all at once is overwhelming and makes it difficult to identify anything useful. Filtering allows you to focus on specific protocols, behaviours, and conversations.


## Filters I Used

### DNS

```wireshark
dns
```

This was one of the most useful filters because it allowed me to see which domains my machine was communicating with.

I noticed requests related to Microsoft services, VS Code, and GitHub Copilot infrastructure.

Example:

```text
172.20.10.2 → 172.20.10.1
Standard query HTTPS main.vscode-cdn.net
```

Even though much of the traffic was encrypted, DNS still provided useful context about what services were being accessed.


### TCP

```wireshark
tcp
```

This displayed all TCP traffic in the capture.

Since TCP made up the majority of the capture, this filter generated a large amount of output. It was useful for getting a broad overview of communications but generally needed to be combined with more specific filters.


### HTTPS

```wireshark
tcp.port == 443
```

This filter isolated encrypted HTTPS traffic.

Although the contents of the traffic were encrypted, I could still see source and destination addresses, connection timing, and session behaviour.

It quickly became clear why metadata remains valuable even when payload data cannot be inspected.


### TCP Connection Requests

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

This filter displays the initial SYN packets used to establish TCP connections.

It was useful for visualising when new sessions were being created and how frequently devices initiated new connections.


### TCP Retransmissions

```wireshark
tcp.analysis.retransmission
```

This filter displays packets that Wireshark identifies as retransmissions.

Only a small number appeared during the capture, which is normal and can occur due to packet loss or temporary congestion.


### TCP Reset Packets

```wireshark
tcp.flags.reset == 1
```

This filter displays TCP reset (RST) packets.

I found several examples throughout the capture, showing connections being terminated unexpectedly or rejected by the remote host.


### ARP

```wireshark
arp
```

This filter displays Address Resolution Protocol traffic.

ARP requests showed devices resolving MAC addresses before communication could occur. It was a useful reminder that there is a lot happening underneath normal network communication that users never see.


### IPv6

```wireshark
ipv6
```

This filter displays all IPv6 traffic.

I was surprised by how much IPv6 traffic appeared in the capture. Before this exercise I knew IPv6 was widely used, but seeing it account for over a third of the traffic on my own network made that much more obvious.


## What I Learned

The biggest lesson from this phase was that packet analysis isn't really about looking at packets one by one.

The real skill is narrowing a capture down to something manageable and then asking questions about what you're seeing.

Using filters made it possible to quickly focus on specific protocols, identify patterns, and understand how different types of traffic contribute to overall network activity.

This phase gave me much more confidence using Wireshark, working with larger captures, and understanding what normal network traffic looks like before moving on to malware investigation and incident-response style exercises.

---------

# Phase 3
## Suspicious Traffic Patterns

Having spent time analysing normal traffic in Phase 2, I wanted to investigate a packet capture containing genuinely suspicious activity.

For this phase I used one of the training exercises provided by Malware-Traffic-Analysis.net, a site that publishes realistic packet captures designed to simulate incident response investigations.

The training exercises can be found at:

https://www.malware-traffic-analysis.net/training-exercises.html

Unlike the previous phases, the objective was no longer to understand how protocols work, but to identify an infected system and determine who was using it.

## Investigation 1 - Easy As 123

For the first investigation, I selected the exercise **"Easy As 123"**. (see fig 3.1)

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

At over four hours long and containing more than 15,000 packets, this was significantly larger than the captures I had worked with in previous phases.

### Understanding the Environment

`LAN segment range: 10.2.28.0/24 (10.2.28.0 through 10.2.28.255)`
- Anything start with 10.2.28.x is likely internal
- Anything outside that range is likey external
- The infected machine should be somewhere inside this range because the exercise is asking us to identify an infected internal workstation.

`Domain: easyas123[.]tech`
- This is the organisations Windows/Active Directory domain name
- it helps to idenity traffic that belongs to the victim environment 
- eg "DESKTOP-TEYQ2NR.easyas123.tech" likey means "Windows workstation joined to the easyas123.tech domain"


`AD environment name: EASYAS123`
- This is the short Windows domain / NetBIOS - style name
- Tells us the domain account name 
- eg "EASYAS123\jsmith" means user jsmith from EASYAS123 Domain


`Active Directory (AD) domain controller: 10.2.28[.]2 - EASYAS123-DC`
Since the Domain Controller handles authentication and directory services, I suspected traffic to and from 10.2.28.2 might reveal useful information such as:

- Hostnames
- Usernames
- Authentication activity
- Domain membership

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
(see fig 3.3)

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
(see fig 3.4)

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

## Final Findings

| Item | Value |
|---|---|
| Infected IP Address | **10.2.28.88** |
| MAC Address | **00:19:d1:b2:4d:ad** |
| Hostname | **DESKTOP-TEYQ2NR** |
| User Account | **brolf** |
| Full Name | **Becka Rolf** |

The investigation successfully traced the SIEM alert from the malicious external IP address through to the affected workstation, associated user account, and ultimately the individual using the system at the time of the activity.

---

## Lessons Learned

This was my first attempt at working through a structured malware traffic analysis exercise.

The most interesting part of the investigation was seeing how information could be pieced together across multiple protocols. Rather than finding the answer in a single packet, each step revealed a small piece of evidence that helped build the overall picture.

The investigation also reinforced the importance of pivoting when an approach doesn't work. My initial attempt to identify the hostname through DHCP traffic didn't provide the answer, but it helped validate previous findings and led me towards Kerberos traffic, which ultimately revealed the workstation name and user account.

Through this exercise I gained practical experience with:

- IOC-driven investigations
- TCP stream reconstruction
- Kerberos analysis
- Windows authentication traffic
- User and host attribution
- Building an evidence trail from network data

Most importantly, this exercise showed how a SIEM alert can be followed through a packet capture and linked back to a specific device and user.