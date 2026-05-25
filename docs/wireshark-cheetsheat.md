# Wireshark Quick Reference Notes

---

# Packet List Pane Columns

| Column | Meaning |
|---|---|
| No. | Packet number in chronological order |
| Time | Timestamp since capture started |
| Source | Device/IP sending the packet |
| Destination | Device/IP receiving the packet |
| Protocol | Protocol identified in the packet |
| Length | Packet size in bytes |
| Info | Quick summary of packet contents |

---

# Common Protocols Seen in Wireshark

| Protocol | Purpose | Common Ports |
|---|---|---|
| TCP | Reliable connection-based communication | 80, 443, 22 |
| UDP | Fast connectionless communication | 53, 67, 68 |
| HTTP | Web traffic | 80 |
| HTTPS | Encrypted web traffic | 443 |
| DNS | Domain name resolution | 53 |
| DHCP | Automatic IP address assignment | 67 / 68 |
| ARP | MAC address resolution on local network | N/A |
| ICMP | Network diagnostics / ping traffic | N/A |

---

# Wireshark Colour Rules (Default)

| Colour | Common Meaning |
|---|---|
| Light Purple | TCP traffic |
| Light Blue | DNS traffic |
| Dark Blue | UDP traffic |
| Green | HTTP traffic |
| Black / Red | Bad TCP packets / errors / resets |
| Yellow | ARP traffic |
| Grey | General system or miscellaneous traffic |

> Note: Colours can vary slightly depending on Wireshark theme and profile settings.

---

# Common TCP Flags

| Flag | Meaning |
|---|---|
| SYN | Starts a TCP connection |
| ACK | Acknowledges received data |
| FIN | Gracefully closes a connection |
| RST | Immediately resets a connection |
| PSH | Pushes buffered data immediately |
| URG | Urgent data present |

---

# Useful Wireshark Filters

## Basic Filters

```wireshark
ip.addr == 192.168.1.1
```

Traffic to or from a specific IP.

```wireshark
tcp.port == 80
```

Traffic using TCP port 80.

```wireshark
http
```

Only HTTP traffic.

```wireshark
dns
```

Only DNS traffic.

```wireshark
dhcp
```

Only DHCP traffic.

---

# Useful Filter Combinations

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Possible SYN scan / TCP connection attempts.

```wireshark
http.request.method == "POST"
```

HTTP POST requests only.

```wireshark
ip.addr == 10.0.0.1 && ip.addr == 10.0.0.2
```

Conversation between two hosts.

---

# Packet Structure Layers

| Layer | Example |
|---|---|
| Layer 2 | Ethernet |
| Layer 3 | IPv4 / IPv6 |
| Layer 4 | TCP / UDP |
| Layer 7 | HTTP / DNS / DHCP |

---

# Common Addresses

| Address | Meaning |
|---|---|
| 0.0.0.0 | Device has no assigned IP yet |
| 255.255.255.255 | IPv4 broadcast address |
| 127.0.0.1 | Loopback / localhost |

---

# Notes

- Packet captures become much easier to read once filtered.
- The packet list pane usually tells the high-level story very quickly.
- Timing and repetition are often more important than individual packets.
- Following TCP streams is useful for reconstructing conversations.
- Protocols can often be identified visually before even opening packet details.