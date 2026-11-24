# Networking-fundamentals-lab
A hands-on exploration of networking fundamentals covering TCP/IP, DNS resolution, network ports, firewall concepts, and Wireshark packet analysis, with practical exercises and documented findings using Kali Linux.
# Networking Fundamentals: Practical Network Analysis

**Focus:** TCP/IP · DNS · Network Ports · Firewalls · Wireshark · TLS

## Project Overview

Understanding how devices communicate across networks is an important foundation in cybersecurity. Before investigating suspicious traffic or identifying network-based threats, it is necessary to understand what normal network communication looks like.

This project explores fundamental networking concepts through practical exercises conducted in a Kali Linux environment. I used Linux networking utilities and Wireshark to examine network configuration, test connectivity, investigate DNS resolution, identify listening ports, and analyse TCP and TLS communication.

Rather than focusing solely on theoretical definitions, the project involved generating network traffic, observing how communication takes place, and interpreting the results.

## Objectives

The main objectives were to:

- Understand IP addressing and basic network communication.
- Explore the roles of TCP, UDP and IP.
- Investigate how DNS translates domain names into IP addresses.
- Identify listening network ports and their common purposes.
- Understand the role of firewalls in controlling network traffic.
- Capture and analyse DNS queries, TCP handshakes and HTTPS/TLS traffic.
- Develop practical experience interpreting network behaviour using Wireshark.

## Tools and Technologies

| Tool | Purpose |
|---|---|
| Kali Linux | Practical networking environment |
| Wireshark | Network traffic capture and analysis |
| `ip addr` | Inspect network interfaces and IP addresses |
| `ping` | Test network connectivity using ICMP |
| `nslookup` | Perform DNS lookups |
| `ss` | Inspect listening network sockets |
| `curl` | Generate HTTP/HTTPS requests |

---

## 1. Network Configuration and Connectivity

The first stage involved examining the network interfaces and IP configuration of the Kali Linux machine.

**Command:**

```bash
ip addr
```

The active network interface was `eth0`, with the following configuration:

- IPv4 address: `192.168.43.66`
- Network prefix: `/24`
- Interface state: UP

The `/24` prefix corresponds to the subnet mask `255.255.255.0`.

After identifying the network configuration, I tested external connectivity.

**Command:**

```bash
ping -c 4 8.8.8.8
```

**Results:**

| Metric | Result |
|---|---:|
| Packets transmitted | 4 |
| Packets received | 4 |
| Packet loss | 0% |
| Average response time | 95.831 ms |

All four packets received replies, indicating successful connectivity to the external IP address during the test.

This exercise also demonstrated the use of ICMP for network connectivity testing.

**Key takeaway:** Network interface inspection and connectivity testing provide useful starting points when investigating network communication.

## 2. TCP/IP Fundamentals

TCP/IP provides the underlying protocols that allow devices to communicate across interconnected networks.

During this project, I examined the different responsibilities of the protocols involved in network communication.

**Internet Protocol (IP)** handles logical addressing and routing, allowing packets to travel between networks.

**Transmission Control Protocol (TCP)** provides connection-oriented communication, including sequencing, acknowledgements and reliable data delivery.

**User Datagram Protocol (UDP)** provides connectionless communication with lower overhead and without TCP's delivery guarantees.

Understanding the differences between these protocols is important when interpreting network traffic.

For example, the connectivity test used ICMP, DNS queries commonly use UDP, and the HTTPS connection examined in this project used TCP.

## 3. DNS Resolution and Analysis

The Domain Name System (DNS) translates human-readable domain names into IP addresses that devices can use to communicate.

To investigate DNS resolution, I performed a lookup for `example.com`.

**Command:**

```bash
nslookup example.com
```

The configured DNS resolver was:

`192.168.43.1`

The lookup returned the following addresses:

**IPv4 (A records):**
- `104.20.23.154`
- `172.66.147.243`

**IPv6 (AAAA records):**
- `2606:4700:10::ac42:93f3`
- `2606:4700:10::6814:179a`

### Wireshark investigation

I captured the DNS traffic generated during the lookup and applied the following display filter:

```text
dns
```

Four relevant packets were identified:

1. An A record query for `example.com`.
2. A response containing two IPv4 addresses.
3. An AAAA record query for `example.com`.
4. A response containing two IPv6 addresses.

The traffic showed communication between my Kali machine (`192.168.43.66`) and the DNS resolver (`192.168.43.1`) over destination port `53`.

**Key finding:** DNS resolution can return multiple addresses for one domain, including both IPv4 and IPv6 records. Wireshark allowed me to observe the queries and their corresponding responses rather than relying only on the output of the lookup command.

## 4. Network Ports and Listening Services

Network ports provide endpoints through which applications and services communicate.

I used the Linux socket statistics utility to inspect listening network sockets.

**Command:**

```bash
ss -tuln
```

The output revealed several listening TCP ports:

| Port | Common service association |
|---|---|
| 80 | HTTP |
| 443 | HTTPS |
| 1514 | Wazuh agent communication |
| 1515 | Wazuh agent registration |
| 9200 | OpenSearch/Elasticsearch API |
| 9300 | OpenSearch/Elasticsearch cluster communication |
| 3333 | Application-specific service |
| 55000 | Wazuh API |

These are common service associations, not verified process identifications. The exercise focused on identifying listening ports rather than investigating the processes behind each one.

**Key finding:** Examining listening ports provides visibility into the network services waiting for connections on a system. This is useful when assessing a machine's network exposure.

## 5. Firewalls and Network Traffic Control

Firewalls help protect systems and networks by controlling traffic according to defined security rules.

Some important firewall concepts include:

- **Inbound traffic:** Traffic entering a system or network.
- **Outbound traffic:** Traffic leaving a system or network.
- **Allow rules:** Permit specified network traffic.
- **Deny rules:** Block specified network traffic.
- **Stateful filtering:** Considers the state of active network connections.
- **Stateless filtering:** Evaluates individual packets against predefined rules.

Common Linux firewall technologies include UFW, nftables and iptables.

### Environment observation

I attempted to inspect the firewall environment using:

```bash
sudo ufw status
```

UFW was not available as a command in the tested environment.

I also attempted:

```bash
sudo nft list ruleset
```

The command returned no visible rules in the terminal.

No firewall configuration was changed during this exercise. This section therefore documents the firewall concepts and the limitations encountered during inspection rather than claiming a completed firewall configuration or filtering test.

## 6. Practical Packet Analysis with Wireshark

One of the main practical components of this project was examining actual network traffic using Wireshark.

Three investigations were conducted.

### 6.1 TCP Three-Way Handshake

A connection to `example.com` was initiated using:

```bash
curl https://example.com
```

I then examined the TCP traffic using the Wireshark display filter:

```text
tcp.flags.syn == 1
```

The following three packets were identified:

| Packet | TCP flag | Description |
|---|---|---|
| 42 | SYN | Client initiates a connection |
| 43 | SYN, ACK | Server acknowledges and responds |
| 44 | ACK | Client acknowledges the server |

**Connection details:**

- Client IP: `192.168.43.66`
- Server IP: `104.20.23.154`
- Client source port: `36416`
- Destination port: `443`

The three packets demonstrated the TCP three-way handshake:

1. The client sends SYN to request a connection.
2. The server responds with SYN-ACK.
3. The client sends ACK to complete the handshake.

**Key finding:** TCP establishes a connection through a handshake before normal application data is exchanged.

### 6.2 HTTPS and TLS 1.3 Analysis

After establishing the TCP connection, I investigated the TLS traffic generated by the HTTPS request.

The request was made using:

```bash
curl https://example.com
```

The command successfully returned the HTML content of the Example Domain webpage.

The Wireshark capture revealed the following TLS 1.3 packets:

| Packet | Observation |
|---|---|
| 45 | Client Hello with SNI identifying `example.com` |
| 48 | Server Hello and Change Cipher Spec |
| 54 | Application Data; Wireshark also indicated a TCP retransmission |
| 56 | Change Cipher Spec and Application Data |
| 57 | Application Data |
| 58 | Application Data |

These observations showed the beginning of TLS negotiation followed by application-data traffic.

The retransmission label on packet 54 was recorded as an additional network observation. It does not, by itself, indicate an attack or security incident.

**Key finding:** HTTPS communication involves TLS negotiation and encrypted application-data exchange, helping protect information transmitted between the client and server.

### 6.3 DNS Packet Analysis

The DNS investigation provided another opportunity to examine network communication at packet level.

Rather than simply observing the addresses returned by `nslookup`, I used Wireshark to identify the DNS query and response packets.

This helped connect the practical command-line output to the underlying network traffic.

---

## 7. Skills Demonstrated

Through the exercises, I gained practical experience with:

- IPv4 network configuration and interface inspection.
- ICMP-based connectivity testing.
- TCP, UDP and IP fundamentals.
- DNS resolution and interpretation of A and AAAA records.
- Identifying listening network ports.
- Understanding firewall concepts and inspection limitations.
- Wireshark traffic capture and display filters.
- TCP three-way handshake analysis.
- TLS 1.3 handshake and application-data observations.
- Interpreting network traffic and documenting findings.

## 8. Project Evidence

The project includes a separate document titled **Networking Practical Evidence & Results Report**.

It contains the recorded command outputs, packet observations, screenshots and explanations from the exercises.

The supporting evidence includes:

- Network interface configuration.
- External connectivity test.
- Listening socket output.
- DNS lookup results.
- DNS packet analysis screenshot.
- TCP handshake screenshot and packet details.
- TLS 1.3 packet observations.


## 9. Conclusion

This project provided practical exposure to the fundamentals of network communication and traffic analysis.

By using Linux networking utilities and Wireshark, I was able to move beyond simply learning protocol definitions and observe how network communication occurs in practice.

The exercises demonstrated how devices use IP addressing, how DNS resolves domain names, how TCP establishes connections, and how TLS protects application traffic.

These skills provide a foundation for further learning in network security, security operations, traffic investigation and threat detection.

---


