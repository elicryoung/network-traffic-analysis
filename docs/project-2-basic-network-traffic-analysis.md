# PROJECT 2: Basic Network Traffic Analysis

---

## Project Overview

| Attribute | Details |
|-----------|---------|
| **Difficulty** | Beginner |
| **Time Estimate** | 1 week (2-4 hours/day, ~15-20 hours total) |
| **When to Build** | Week 2 - After completing Project 1 |
| **Prerequisites** | Project 1 complete, basic understanding of logs |

**What You'll Build:** A collection of analysed network packet captures (PCAPs) with documented findings, protocol analysis notes, and a reference guide for identifying suspicious network traffic patterns.

**Why This Matters:** SOC analysts frequently analyse network traffic to identify threats that don't appear in endpoint logs. Understanding what's happening on the wire is essential for detecting command-and-control traffic, data exfiltration, and lateral movement. This directly complements your Network+ study.

---

## Tech Stack (All Free, Mac-Compatible)

| Tool | Purpose | Installation |
|------|---------|--------------|
| **Wireshark** | GUI packet analysis | brew install wireshark (or download from wireshark.org) |
| **tshark** | Command-line packet analysis | Included with Wireshark |
| **tcpdump** | Lightweight packet capture | Already installed on Mac |
| **Sample PCAPs** | Practice material | Downloaded from public repositories |
| **NetworkMiner** | Flow analysis (optional) | Free version at netresec.com |

---

## Learning Objectives

### Technical Skills
- Capture live network traffic using tcpdump
- Navigate Wireshark interface efficiently
- Apply display filters to isolate relevant traffic
- Follow TCP streams and reconstruct conversations
- Export objects from packet captures (files, images)
- Use command-line tshark for scripted analysis

### Security Concepts
- OSI model in practice (seeing layers in real packets)
- TCP/IP fundamentals (handshakes, flags, ports)
- Common protocols: HTTP, HTTPS, DNS, DHCP, ARP
- What "normal" network traffic looks like
- Indicators of malicious traffic (beaconing, unusual ports, encoding)
- How attackers use legitimate protocols to hide

### Interview Ammunition
- "I can analyse packet captures to identify command-and-control traffic"
- "I understand how to follow TCP streams and extract transferred files"
- "I've investigated DNS tunneling and HTTP-based malware communication"
- Demonstrate understanding of the Network+ concepts in practice

---

## Build Phases

### Phase 1 — Environment Setup & Wireshark Basics (Day 1, ~3 hours)
- Install Wireshark on Mac
- Learn the interface layout (packet list, details, bytes)
- Understand colour coding and what it means
- Practice basic navigation on a sample PCAP
- Learn essential display filter syntax

### Phase 2 — Real-Time Traffic & Protocol Deep Dives (Days 2-3, ~6 hours)
- Capture live traffic from your own machine using tcpdump
- Load live captures into Wireshark and explore what you generated
- Analyse TCP handshakes and connection teardowns
- Examine HTTP traffic and extract useful information
- Understand DNS queries and responses
- Look at DHCP and ARP in network context
- Identify which protocols are normal vs concerning on different networks

### Phase 3 — Suspicious Traffic Patterns & Analysis Challenges (Days 4-6, ~8 hours)
- Analyse PCAPs containing port scans
- Identify brute force attack patterns in network traffic
- Examine command-and-control (C2) beaconing
- Look for data exfiltration indicators
- Analyse DNS tunneling examples
- Complete 3-4 CTF-style PCAP challenges
- Create analysis write-ups for each challenge as you go

### Phase 4 — Documentation (Day 7, ~3 hours)
- Build your Wireshark filter cheat sheet
- Compile your suspicious traffic patterns guide
- Finalise and polish all portfolio write-ups
- Answer the self-assessment questions as a review pass

---

## Key Challenges & Learning Moments

### What Will Be Hard
- **Information overload**: A 1-minute capture can contain thousands of packets
- **Knowing what to look for**: It's easy to open a PCAP and feel lost
- **Filter syntax**: Wireshark filters have specific syntax that takes practice
- **Context switching**: Moving between packet details and big-picture patterns

### What's Important to Understand
- **Packets are just data**: Once you understand the structure, it's like reading a language
- **Normal varies by network**: A home network looks different from an enterprise
- **Timing matters**: Beaconing, slow exfiltration, and timing-based attacks require time analysis
- **Protocols can be abused**: DNS, HTTP, and ICMP are often used by attackers because they're allowed through firewalls

---

## Deliverables (For Your Portfolio)

1. **Wireshark Quick Reference Guide** (Markdown or PDF)
   - Essential display filters
   - Common analysis workflows
   - How to follow streams and export objects
   - Time analysis techniques

2. **Protocol Analysis Notes** (Markdown)
   - TCP/UDP key characteristics
   - HTTP headers that reveal interesting information
   - DNS query types and what they mean
   - What normal vs abnormal looks like for each protocol

3. **Suspicious Traffic Patterns Guide** (Markdown or PDF)
   - Port scan indicators
   - C2 beaconing characteristics
   - Data exfiltration red flags
   - DNS tunneling detection

4. **3-4 PCAP Analysis Write-ups**
   - Each should be 1-2 pages
   - Include: timeline, protocols involved, findings, IOCs extracted
   - Format like a real SOC analyst report

5. **Display Filter Cheat Sheet**
   - Filters you've tested and use regularly
   - Organised by use case

---

## Learning Resources

### Read BEFORE Starting
- Wireshark User's Guide: Chapter 1-3 (just the basics)
- Your Network+ study material on TCP/IP and OSI model
- Understand ports 0-1024 (well-known ports)

### Use DURING the Project
- Wireshark Display Filter Reference: wiki.wireshark.org/DisplayFilters
- Wireshark Sample Captures: wiki.wireshark.org/SampleCaptures
- Chris Sanders' Packets Guide (free resources at chrissanders.org)

### Sample PCAP Sources
- Malware Traffic Analysis: malware-traffic-analysis.net (excellent, realistic samples)
- NETRESEC: netresec.com/index.ashx?page=PCAPs
- Wireshark Samples: wiki.wireshark.org/SampleCaptures
- PacketTotal: packettotal.com (PCAP analysis and samples)

---

## Common Pitfalls

| Pitfall | Why It Happens | How to Avoid |
|---------|---------------|--------------|
| Capturing on wrong interface | Mac has multiple interfaces | Use tcpdump -D to list interfaces first |
| Opening huge PCAPs | Wireshark struggles with large files | Filter while capturing, or use tshark for big files |
| Not using filters | Trying to look at everything | Always start with a filter hypothesis |
| Ignoring timestamps | Focusing only on content | Time patterns reveal beaconing and automation |
| Only using GUI | Wireshark is comfortable | Practice tshark for scripting and efficiency |

---

## Essential Wireshark Filters to Learn

```
# Basic filters
ip.addr == 192.168.1.1          # Traffic to/from IP
tcp.port == 80                   # Traffic on port 80
http                             # HTTP traffic only
dns                              # DNS traffic only

# Useful combinations
http.request.method == "POST"    # HTTP POST requests
dns.qry.name contains "evil"     # DNS queries containing string
tcp.flags.syn == 1 && tcp.flags.ack == 0  # SYN packets (scan detection)

# Following conversations
tcp.stream eq 5                  # Follow specific TCP stream
ip.addr == 10.0.0.1 && ip.addr == 10.0.0.2  # Conversation between two IPs
```

---

## Self-Assessment Questions

Before moving to Project 3, you should be able to answer these:

### Protocol Knowledge
1. What are the three packets in a TCP handshake? What flags are set in each?
2. What's the difference between TCP and UDP? When would an attacker prefer one over the other?
3. What information can you extract from an HTTP GET request?
4. What's the difference between an A record and a CNAME in DNS?
5. What port does DNS typically use? What about HTTPS?

### Wireshark Skills
1. How do you filter for all traffic to a specific IP address?
2. How do you follow a TCP stream to see the full conversation?
3. How do you export a file that was transferred over HTTP?
4. How do you find all DNS queries in a capture?
5. How do you identify which packet a conversation started with?

### Security Analysis
1. What would a port scan look like in a packet capture?
2. How would you identify C2 beaconing in network traffic?
3. What are signs of DNS tunneling?
4. How might an attacker exfiltrate data using ICMP?
5. Why might an attacker use HTTPS for C2 traffic?

---

## Claude Context Instructions

**Copy everything below this line into a new Claude chat when starting this project:**

---

CONTEXT FOR CLAUDE - PROJECT 2: BASIC NETWORK TRAFFIC ANALYSIS

I'm working on Project 2 of my SOC Analyst preparation - Basic Network Traffic Analysis. I completed Project 1 (Security Event Log Analysis) and now have foundational log reading skills.

MY SITUATION:
- Completed Project 1, can read and analyse security logs
- Currently studying for Network+ (this project reinforces that)
- Working on a Mac
- Have 2-4 hours per day for this project
- Goal: Complete in 1 week

WHAT I'M BUILDING:
- Wireshark quick reference guide
- Protocol analysis notes
- Suspicious traffic patterns guide
- 3-4 PCAP analysis write-ups
- Display filter cheat sheet

MY LEARNING PHILOSOPHY:
- I want to understand WHY, not just click buttons
- If I'm stuck, point me to documentation and ask diagnostic questions
- Connect this to Network+ concepts when relevant
- Challenge me with questions to test understanding
- Build on what I learned in Project 1 (correlating network and log evidence)

TOOLS I HAVE:
- Mac Terminal (tcpdump available)
- Wireshark (installed or ready to install)
- Sample PCAPs (I'll download as needed)

WHERE I AM IN THE PROJECT:
[UPDATE THIS AS YOU PROGRESS]
- [ ] Phase 1: Environment Setup & Wireshark Basics
- [ ] Phase 2: Real-Time Traffic & Protocol Deep Dives
- [ ] Phase 3: Suspicious Traffic Patterns & Analysis Challenges
- [ ] Phase 4: Documentation

CURRENT PHASE: [Tell Claude which phase you're on]

Please help me work through this project step by step. Start by confirming you understand my context, then guide me through my current phase.

---

## Progress Checklist

### Phase 1 — Environment Setup & Wireshark Basics
- [ ] Wireshark installed and working
- [ ] Understand packet list, details, and bytes panes
- [ ] Understand colour coding
- [ ] Can navigate and select packets
- [ ] Know basic display filter syntax
- [ ] Downloaded at least 2 sample PCAPs

### Phase 2 — Real-Time Traffic & Protocol Deep Dives
- [ ] Captured live traffic from your own machine using tcpdump
- [ ] Loaded live capture into Wireshark and explored it
- [ ] Can identify and explain TCP handshake
- [ ] Understand HTTP request/response structure
- [ ] Know DNS query/response format
- [ ] Understand DHCP and ARP basics
- [ ] Created protocol analysis notes

### Phase 3 — Suspicious Traffic Patterns & Analysis Challenges
- [ ] Analysed a port scan PCAP
- [ ] Identified beaconing in sample traffic
- [ ] Examined C2 communication patterns
- [ ] Looked at a DNS tunneling example
- [ ] Completed at least 3 CTF-style PCAP challenges
- [ ] Created analysis write-ups for each challenge

### Phase 4 — Documentation
- [ ] Built display filter cheat sheet
- [ ] Compiled suspicious traffic patterns guide
- [ ] Finalised all portfolio write-ups
- [ ] Answered all self-assessment questions correctly

---

**Project Complete When:** All checklist items done, all self-assessment questions answered correctly, and 3-4 portfolio-ready PCAP analysis write-ups created.

**Next Project:** Project 3 - Home SIEM Lab
