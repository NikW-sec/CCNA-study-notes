# Lab 2 — OSPF single area

## Goal
Three routers in a triangle, all LANs reachable, with the LAN interfaces passive and deterministic router IDs.

## Topology
```
        R1
      /    \
 .1 /        \ .1
   /          \
  R2 ---------- R3
       .2  .3
```

| Link | Subnet |
|---|---|
| R1–R2 | 10.0.12.0/30 |
| R1–R3 | 10.0.13.0/30 |
| R2–R3 | 10.0.23.0/30 |
| R1 LAN | 192.168.1.0/24 |
| R2 LAN | 192.168.2.0/24 |
| R3 LAN | 192.168.3.0/24 |

## R1 (repeat the pattern on R2 and R3)
```
interface loopback 0
 ip address 1.1.1.1 255.255.255.255
!
interface gi0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
interface gi0/1
 ip address 10.0.12.1 255.255.255.252
 ip ospf network point-to-point
 no shutdown
!
interface gi0/2
 ip address 10.0.13.1 255.255.255.252
 ip ospf network point-to-point
 no shutdown
!
router ospf 1
 router-id 1.1.1.1
 auto-cost reference-bandwidth 1000
 passive-interface gi0/0
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
```

R2 uses router-id 2.2.2.2 and networks 192.168.2.0, 10.0.12.0, 10.0.23.0. R3 uses 3.3.3.3 and 192.168.3.0, 10.0.13.0, 10.0.23.0.

## Verify
1. `show ip ospf neighbor` — two neighbours per router, state **FULL**, and no DR/BDR because the links are point-to-point.
2. `show ip route ospf` — each router learns the two remote LANs with AD 110.
3. `show ip protocols` — confirms the router ID, the passive interface, and the networks.
4. Ping from R2's LAN interface to R3's LAN: `ping 192.168.3.1 source 192.168.2.1`.

## Things to break on purpose
- Change the hello interval on one side (`ip ospf hello-interval 5`) and watch the adjacency drop.
- Set mismatched areas on the R2–R3 link and confirm no neighbour forms.
- Shut the R1–R2 link and watch the route to 192.168.2.0/24 reconverge via R3, with a higher cost.
- Remove `auto-cost reference-bandwidth 1000` from one router only, which is a classic silent asymmetric-routing bug.
