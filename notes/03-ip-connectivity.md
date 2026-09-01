# 3.0 IP Connectivity (25%)

## 3.1 Reading the routing table

`show ip route` output, one line per route:

```
O    192.168.20.0/24 [110/2] via 10.0.0.2, 00:04:11, GigabitEthernet0/1
```

- `O` — the source (C connected, L local /32, S static, O OSPF, D EIGRP, B BGP, `*` candidate default).
- `[110/2]` — administrative distance / metric.
- `via` — next hop; then age and exit interface.

**Administrative distance** picks between protocols; the **metric** picks between routes from the same protocol.

| Source | AD |
|---|---|
| Connected | 0 |
| Static | 1 |
| eBGP | 20 |
| EIGRP (internal) | 90 |
| OSPF | 110 |
| RIP | 120 |
| EIGRP (external) | 170 |
| iBGP | 200 |
| Unusable | 255 |

## 3.2 Forwarding decision

The router matches the destination against the table and uses the **longest prefix match**, regardless of AD or metric. A /32 host route beats a /24, which beats the 0.0.0.0/0 default. If nothing matches and there is no default, the packet is dropped and an ICMP destination unreachable is returned.

## 3.3 Static routing

```
Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2
Router(config)# ip route 192.168.20.0 255.255.255.0 gi0/1 10.0.0.2   ! fully specified
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1                 ! default route
Router(config)# ip route 10.10.10.0 255.255.255.0 null0              ! null route
Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.6 5       ! floating static
```

Use the exit interface alone only on point-to-point links; on multi-access media (Ethernet) always give a next-hop IP, otherwise the router must ARP for every destination. A **floating static** carries an AD higher than the dynamic protocol's, so it only installs when the primary route disappears.

IPv6 equivalents use `ipv6 route`, and require `ipv6 unicast-routing` to be enabled globally.

## 3.4 OSPF

Link-state, open standard, AD 110, metric = cumulative cost, where cost = reference bandwidth (100 Mbps by default) / interface bandwidth. Because everything above 100 Mbps costs 1, raise the reference bandwidth consistently on all routers: `auto-cost reference-bandwidth 1000`.

**Router ID** selection order: manual `router-id`, then the highest loopback IP, then the highest active physical interface IP. Set it manually and reload the process (`clear ip ospf process`) to make a change take effect.

**Adjacency states**: Down → Init → 2-Way (DR/BDR elected on multi-access) → ExStart → Exchange → Loading → **Full**. Neighbours stuck in 2-Way between two DROTHERs is normal, not a fault.

**These must match to form an adjacency**: area ID, hello and dead intervals (10/40 on broadcast, 30/120 on non-broadcast), subnet and mask on the link, authentication, stub area flags, and MTU. Router IDs must be unique.

**DR/BDR election** on broadcast networks reduces adjacencies from full mesh to a hub: highest OSPF interface priority wins (default 1; 0 means never a DR), then highest router ID. The election is non-preemptive. Point-to-point links elect nobody, so use `ip ospf network point-to-point` on back-to-back Ethernet links to skip it.

**Areas** limit LSA flooding. All areas must attach to area 0 (the backbone); an ABR sits between area 0 and another area, and an ASBR redistributes external routes in.

```
Router(config)# router ospf 1
Router(config-router)# router-id 1.1.1.1
Router(config-router)# network 192.168.1.0 0.0.0.255 area 0
Router(config-router)# passive-interface gi0/2
Router(config-router)# default-information originate
Router(config-if)# ip ospf 1 area 0        ! alternative per-interface config
Router(config-if)# ip ospf cost 10
Router(config-if)# ip ospf priority 100

Router# show ip ospf neighbor
Router# show ip ospf interface brief
Router# show ip protocols
```

The wildcard mask in `network` is the inverse of the subnet mask: 0 means "must match", 1 means "don't care". Always make LAN interfaces facing endpoints **passive** so no needless hellos are sent, while the subnet is still advertised.

## 3.5 First Hop Redundancy Protocols

Endpoints have one default gateway, so FHRPs let two or more routers share a virtual IP and virtual MAC. If the active router fails, the standby takes over and hosts never change their configuration.

- **HSRP** (Cisco) — active/standby, virtual MAC 0000.0C07.ACxx (v1) or 0000.0C9F.Fxxx (v2). Highest priority wins (default 100), and preemption is off by default so it must be enabled explicitly.
- **VRRP** (open standard) — master/backup, virtual MAC 0000.5E00.01xx, preemption on by default.
- **GLBP** (Cisco) — load balances across all gateways by handing different virtual MACs to different hosts in ARP replies.

```
Router(config-if)# standby version 2
Router(config-if)# standby 1 ip 192.168.1.1
Router(config-if)# standby 1 priority 110
Router(config-if)# standby 1 preempt
Router# show standby brief
```
