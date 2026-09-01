# 5.0 Security Fundamentals (15%)

## 5.1 Key concepts

- **CIA triad** — confidentiality, integrity, availability.
- **Vulnerability** is a weakness, a **threat** is what might exploit it, an **exploit** is the method, and **risk** is the likelihood and impact combined. **Mitigation** reduces one of those.
- Common attacks: DoS and DDoS, spoofing, reflection/amplification, man-in-the-middle, phishing and social engineering, password attacks (brute force, dictionary, credential stuffing), malware (virus, worm, trojan, ransomware).

## 5.2 AAA and passwords

**Authentication** proves who you are, **authorization** decides what you may do, **accounting** records what you did. A centralized server does this over **RADIUS** (UDP, encrypts only the password, combines authn and authz, an open standard, typically used for network access/802.1X) or **TACACS+** (TCP 49, encrypts the whole payload, separates all three functions, Cisco, typically used for device administration).

Password hygiene: length and complexity, MFA, no shared accounts, rotate on staff changes, and store as hashes.

```
Router(config)# enable secret Str0ngPass       ! MD5+ hashed, use instead of enable password
Router(config)# service password-encryption    ! weak type 7, obfuscation only
Router(config)# username admin secret Str0ngPass
Router(config)# security passwords min-length 10
Router(config)# line con 0
Router(config-line)# password ConPass
Router(config-line)# login
Router(config-line)# exec-timeout 5 0
Router(config)# banner motd #Authorized access only#
```

## 5.3 Access control lists

An ACL is an ordered list of permit/deny statements evaluated **top down, first match wins**, with an **implicit deny any** at the end. An ACL that permits nothing therefore blocks everything.

- **Standard** (1–99, 1300–1999) — matches source IP only. Place it **close to the destination**, so you do not block traffic that should have reached other subnets.
- **Extended** (100–199, 2000–2699) — matches protocol, source, destination, and ports. Place it **close to the source**, to drop unwanted traffic before it crosses the network.
- Named ACLs are clearer and let you insert and delete individual lines by sequence number.

```
! Standard
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# interface gi0/1
Router(config-if)# ip access-group 10 out

! Extended named
Router(config)# ip access-list extended WEB_ONLY
Router(config-ext-nacl)# 10 permit tcp 192.168.1.0 0.0.0.255 any eq 443
Router(config-ext-nacl)# 20 permit udp any any eq 53
Router(config-ext-nacl)# deny ip any any log
Router(config)# interface gi0/0
Router(config-if)# ip access-group WEB_ONLY in

! Restrict management access
Router(config)# line vty 0 15
Router(config-line)# access-class 10 in

Router# show access-lists
Router# show ip interface gi0/0
```

Wildcard masks: `0.0.0.0` matches one host (or use the `host` keyword), `255.255.255.255` matches everything (or `any`), `0.0.0.255` matches a /24.

## 5.4 Layer 2 security

**Port security** limits which and how many MAC addresses may use an access port.

```
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 2
Switch(config-if)# switchport port-security mac-address sticky
Switch(config-if)# switchport port-security violation restrict
Switch# show port-security interface fa0/1
```

Violation modes: **protect** drops offending frames silently, **restrict** drops them and logs/increments counters, **shutdown** (the default) err-disables the port, which then needs `shutdown` / `no shutdown` or errdisable recovery.

**DHCP snooping** builds a binding table of legitimate client MAC/IP/port/VLAN pairs and drops server-sourced messages (Offer, Ack) arriving on untrusted ports, stopping rogue DHCP servers.

```
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10
Switch(config-if)# ip dhcp snooping trust
Switch(config-if)# ip dhcp snooping limit rate 10
```

**Dynamic ARP Inspection** uses that same binding table to validate ARP replies on untrusted ports, defeating ARP spoofing and the man-in-the-middle attacks built on it (`ip arp inspection vlan 10`).

Other hardening: shut unused ports and put them in an unused VLAN, change the native VLAN, disable DTP, and enable BPDU guard on access ports.

## 5.5 VPNs

- **Site-to-site IPsec** joins two networks through gateways; users are unaware of it. IPsec provides confidentiality (encryption), integrity (hashing), authentication, and anti-replay, using ESP (encrypts) or AH (authenticates only).
- **Remote-access** VPNs (SSL/TLS or IPsec client) connect an individual user to the corporate network.

## 5.6 Wireless security

Open authentication offers nothing. WEP is broken. **WPA2-Personal** uses a PSK with AES-CCMP, **WPA2-Enterprise** uses 802.1X/EAP against RADIUS so each user has their own credentials. **WPA3** replaces the PSK handshake with SAE, which resists offline dictionary attacks, and mandates protected management frames.
