# 1.0 Network Fundamentals (20%)

## 1.1 Network components

| Device | Layer | Role |
|---|---|---|
| Router | 3 | Forwards packets between different networks/subnets. Breaks up broadcast domains. |
| Switch (LAN) | 2 | Forwards frames within a LAN using MAC addresses. Each port is its own collision domain. |
| Firewall | 3–7 | Policy enforcement between security zones; stateful inspection of flows. |
| WLC | 2/3 | Central control plane for lightweight APs (CAPWAP): RF, roaming, config, security. |
| Access point | 1/2 | Bridges the 802.11 wireless medium to the wired Ethernet LAN. |
| Endpoint / server | — | Consumers and providers of the services the network carries. |

**Collision vs broadcast domain.** A collision domain is a set of interfaces that share the medium; with modern full-duplex switching each switch port is a separate collision domain and CSMA/CD is effectively disabled. A broadcast domain is the set of ports a broadcast frame reaches, which is one VLAN. Only a router (or an SVI / routed port) separates broadcast domains.

## 1.2 Network topology architectures

- **Two-tier (collapsed core)** — access + distribution/core combined. Cheaper, common in single-building campuses.
- **Three-tier** — access, distribution, core. Scales to many buildings; core is kept fast and simple (routing only, no policy).
- **Spine-leaf** — every leaf connects to every spine, never to another leaf. Predictable, equal-cost latency for east-west data-centre traffic.
- **WAN** — leased line, MPLS L3VPN, Metro Ethernet, broadband/DSL/cable, cellular, and Internet VPN as backup or primary.
- **SOHO** — one integrated device usually performs routing, switching, wireless, NAT, DHCP, and firewalling.

## 1.3 Physical interfaces and cabling

- **Copper**: Cat5e (1 Gbps), Cat6 (1 Gbps at 100 m, 10 Gbps to ~55 m), Cat6a (10 Gbps at 100 m). 100 m is the Ethernet copper limit.
- **Fibre**: single-mode (small 9 µm core, laser, tens of kilometres) vs multimode (50/62.5 µm core, LED/VCSEL, a few hundred metres). Fibre is immune to EMI and has no crosstalk.
- **Straight-through vs crossover**: straight-through connects unlike devices (PC↔switch, switch↔router), crossover connects like devices (switch↔switch, PC↔PC, PC↔router). Auto-MDIX makes this mostly moot on modern gear but is still examinable.
- **Duplex/speed mismatch**: mismatched duplex produces late collisions, CRC errors, and terrible throughput without the link going down. Check `show interfaces` counters.

## 1.4 TCP/IP and OSI models

| OSI | PDU | TCP/IP | Examples |
|---|---|---|---|
| 7 Application | Data | Application | HTTP, DNS, FTP, SNMP |
| 6 Presentation | Data | Application | TLS, encoding, compression |
| 5 Session | Data | Application | session setup/teardown |
| 4 Transport | Segment/Datagram | Transport | TCP, UDP |
| 3 Network | Packet | Internet | IPv4, IPv6, ICMP, OSPF |
| 2 Data link | Frame | Network access | Ethernet, 802.11, ARP |
| 1 Physical | Bit | Network access | cables, connectors, signalling |

Encapsulation adds a header at each layer downward; de-encapsulation strips it upward. The frame is rewritten hop by hop (new source/destination MAC) while the IP header's source and destination addresses stay the same end to end (absent NAT).

## 1.5 TCP vs UDP

- **TCP** — connection-oriented, three-way handshake (SYN, SYN-ACK, ACK), sequence/acknowledgement numbers, retransmission, windowing/flow control, ordered delivery. 20-byte minimum header.
- **UDP** — connectionless, 8-byte header (source port, destination port, length, checksum), no retransmission or ordering. Preferred for voice, video, DNS, DHCP, TFTP, SNMP.

Well-known ports: FTP 20/21, SSH 22, Telnet 23, SMTP 25, DNS 53 (UDP and TCP), DHCP 67/68, TFTP 69, HTTP 80, POP3 110, NTP 123, SNMP 161/162, HTTPS 443, syslog 514.

## 1.6 IPv4 addressing and subnetting

A 32-bit address written as four dotted decimal octets, paired with a mask that splits it into network and host bits. Hosts per subnet = 2^h − 2 (network address and broadcast address are not assignable).

Classful ranges (still tested as background): A 1–126 (/8), B 128–191 (/16), C 192–223 (/24), D 224–239 multicast, E 240–255 experimental. 127.0.0.0/8 is loopback.

RFC 1918 private ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16. 169.254.0.0/16 is APIPA/link-local, assigned when DHCP fails.

**Subnetting method.** Work in the "interesting octet" (the one the mask ends in). Block size = 256 − mask octet value. Subnets increment by the block size; the network address is the multiple of the block size at or below the host address.

Example: 192.168.10.100/26. Mask 255.255.255.192, block size 64. Subnets: .0, .64, .128, .192. The host lives in 192.168.10.64/26 — usable .65–.126, broadcast .127.

**VLSM** applies different masks to different subnets from one block. Always allocate the largest requirement first, then the next largest, so the smaller allocations fit into what remains without overlap.

**Summarization**: find the longest common prefix of the routes being combined; that prefix length is the summary mask. 172.16.0.0/24 through 172.16.3.0/24 share the first 22 bits, so the summary is 172.16.0.0/22.

## 1.7 IPv6 addressing

128 bits, eight hextets. Compression rules: drop leading zeros in each hextet, and replace one (and only one) run of all-zero hextets with `::`.

| Type | Prefix | Notes |
|---|---|---|
| Global unicast | 2000::/3 | Internet-routable |
| Unique local | FC00::/7 (in practice FD00::/8) | The IPv6 analogue of RFC 1918 |
| Link-local | FE80::/10 | Mandatory on every interface; used by ND and as the next hop for routing protocols |
| Multicast | FF00::/8 | FF02::1 all nodes, FF02::2 all routers, FF02::5/::6 OSPFv3, FF02::1:FFxx:xxxx solicited-node |
| Loopback | ::1/128 | |
| Unspecified | ::/128 | |

There is no broadcast in IPv6; its jobs are done by multicast. **EUI-64** builds the interface ID from the MAC: split it, insert FFFE in the middle, and flip the 7th bit (the U/L bit) of the first byte. **SLAAC** lets a host build its own address from the RA's prefix.

Neighbor Discovery replaces ARP: NS/NA for address resolution and duplicate address detection, RS/RA for router and prefix discovery.

## 1.8 Wireless principles

- **2.4 GHz** — better range and wall penetration, only three non-overlapping 20 MHz channels (1, 6, 11), crowded.
- **5 GHz** — many more non-overlapping channels, higher throughput, shorter range. **6 GHz** (Wi-Fi 6E) adds far more spectrum again.
- **SSID** is the network name; **BSSID** is the AP radio's MAC identifying one basic service set. An ESS is multiple APs sharing one SSID to allow roaming.
- Modes: infrastructure (via AP), ad hoc/IBSS, mesh, repeater. **Autonomous APs** hold their own config; **lightweight APs** join a WLC over CAPWAP (control tunnel encrypted on UDP 5246, data on UDP 5247).
- 802.11 is half-duplex and uses **CSMA/CA** with acknowledgements, because stations cannot detect collisions while transmitting.

## 1.9 Virtualization and cloud

A **hypervisor** runs multiple VMs on one physical server. Type 1 (bare metal, e.g. ESXi) is the data-centre norm; type 2 runs on a host OS. **Containers** share the host kernel and package only the application and its dependencies, so they are far lighter than VMs.

Cloud service models: **IaaS** (you manage OS and up), **PaaS** (you manage the app), **SaaS** (you manage only data and users). Deployment models: public, private, hybrid, community.

## 1.10 Switching concepts

- **MAC learning**: the switch records the source MAC and the port it arrived on in the MAC address table (default ageing 300 seconds).
- **Forwarding**: known unicast is forwarded out the single mapped port.
- **Flooding**: unknown unicast, broadcast, and multicast are sent out every port in the VLAN except the one it came in on.
- **Filtering**: a frame is not sent back out the port it was received on.
