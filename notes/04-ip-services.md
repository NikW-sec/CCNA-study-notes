# 4.0 IP Services (10%)

## 4.1 NAT

NAT translates addresses as packets cross from inside to outside, letting many private RFC 1918 hosts share public addresses.

- **Static NAT** — one fixed inside local to one inside global. Used to publish an internal server.
- **Dynamic NAT** — inside locals draw from a pool on demand. Runs out when the pool is exhausted.
- **PAT (NAT overload)** — many inside locals share one global address, distinguished by unique source port numbers. This is what almost every home and office edge does.

Terminology: **inside local** is the private address as seen inside, **inside global** is what it is translated to, **outside global** is the real public address of the remote host, **outside local** is how that host appears inside.

```
Router(config)# interface gi0/0
Router(config-if)# ip nat inside
Router(config)# interface gi0/1
Router(config-if)# ip nat outside

! Static
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.10

! Dynamic
Router(config)# ip nat pool MYPOOL 203.0.113.20 203.0.113.30 netmask 255.255.255.0
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255
Router(config)# ip nat inside source list 1 pool MYPOOL

! PAT on the outside interface
Router(config)# ip nat inside source list 1 interface gi0/1 overload

Router# show ip nat translations
Router# show ip nat statistics
Router# clear ip nat translation *
```

## 4.2 NTP

Keeps clocks synchronised so logs, certificates, and time-based ACLs are meaningful. **Stratum** is the distance from the authoritative source: stratum 0 is the reference clock itself, stratum 1 a directly attached server, and each hop adds one. Stratum 16 means unsynchronised.

```
Router(config)# ntp server 216.239.35.0
Router(config)# ntp master 3          ! act as a reference clock
Router(config)# clock timezone GMT 0
Router# show ntp status
Router# show ntp associations
```

## 4.3 DHCP and DNS

**DHCP** uses the DORA exchange: Discover (broadcast), Offer, Request (broadcast, so declined offers are withdrawn), Acknowledge. It supplies address, mask, default gateway, DNS servers, and lease time.

Because Discover is a broadcast, a router between client and server needs a **DHCP relay**:

```
Router(config-if)# ip helper-address 192.168.50.10
```

Router as a DHCP server:
```
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
Router(config)# ip dhcp pool LAN
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8
Router(dhcp-config)# lease 7
Router# show ip dhcp binding
```

Router as a DHCP client: `ip address dhcp` on the interface.

**DNS** resolves names to addresses over UDP/TCP 53. Record types worth knowing: A (IPv4), AAAA (IPv6), CNAME (alias), MX (mail), NS, PTR (reverse). On IOS, `ip name-server`, `ip domain-lookup`, and `ip host NAME 10.0.0.1` for static entries.

## 4.4 SNMP and Syslog

**SNMP** — the manager polls agents with Get, sets values with Set, and agents send unsolicited **traps** (or informs, which are acknowledged) on UDP 161/162. v1 and v2c authenticate with a plaintext community string; **v3** adds authentication and encryption and is the only version to use in production.

**Syslog** — messages to a collector on UDP 514, formatted `%FACILITY-SEVERITY-MNEMONIC: description`.

| Level | Name | Mnemonic |
|---|---|---|
| 0 | Emergency | **E**veryone |
| 1 | Alert | **A**pparently |
| 2 | Critical | **C**an |
| 3 | Error | **E**at |
| 4 | Warning | **W**et |
| 5 | Notification | **N**oodles |
| 6 | Informational | **I**n |
| 7 | Debugging | **D**aylight |

Configuring a level logs that severity **and everything more severe**.

```
Router(config)# logging host 192.168.1.100
Router(config)# logging trap 4
Router(config)# service timestamps log datetime msec
Router(config)# snmp-server community CCNAro ro
Router(config)# snmp-server host 192.168.1.100 version 2c CCNAro
```

## 4.5 QoS

Congestion is handled by classification, marking, queuing, shaping, and policing.

- **Marking**: Layer 2 CoS (3 bits in the 802.1Q tag) and Layer 3 **DSCP** (6 bits in the IP ToS byte). Common values: EF (46) for voice, AF41 (34) for video, CS6 (48) for routing protocols, default/BE (0).
- **Queuing**: CBWFQ gives classes a bandwidth guarantee; **LLQ** adds a strict-priority queue for voice.
- **Shaping** buffers excess traffic to smooth it out (used on the sending side, typical for a slower carrier link). **Policing** drops or re-marks excess immediately (typical on the receiving/provider side).
- Voice targets: one-way latency under 150 ms, jitter under 30 ms, loss under 1%.

## 4.6 Management access: SSH, TFTP/FTP

```
Router(config)# hostname R1
Router(config)# ip domain-name example.com
Router(config)# crypto key generate rsa modulus 2048
Router(config)# username admin privilege 15 secret S3cret
Router(config)# ip ssh version 2
Router(config)# line vty 0 15
Router(config-line)# transport input ssh
Router(config-line)# login local
```

A hostname, a domain name, and an RSA key of at least 768 bits are all required before SSH will start. Copy images and configs with `copy running-config tftp:`, `copy tftp: startup-config`, or the FTP equivalents.
