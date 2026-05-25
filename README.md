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
2. Phase 2 —  Protocol Deep Dives
3. Phase 3 - Suspicious Traffic Patterns
4. Phase 4 - Analysis Challenges and Documentation

---

# Phase 1  
## Environment Setup & Wireshark Basics

Firstly, I updated wireshark  on my computer as I had already used the application during my masters degree. For the first phase I initialised the github repo and created my folder structure for my analysis. 

A structured project directory named `network=traffic-analysis` was then created, separating:
- investigation write-ups
- documentation
- pcap-files
- iamges (screenshots)

Because this project is a little bit less like a real world investigation and more about getting to grips with network traffic and packet analysis I won't be mirroring any type of organisation expected in a workplace. 

I went online and just searched for 'sample pcap files for analysis' adn the first thing that came up was `https://wiki.wireshark.org/SampleCaptures#tcp`.
It is a directory with a ton of free to download wireshark sample captures for people to analyse. There are so many to choose from, I decided for phase 1 that two simple and easy to understand captures would be more than enough to learn about the interface of and how to navigate Wireshark. The two .pcap files I went with are :

- `dhcp.pcap`
- `ipv4frags.pcap`



