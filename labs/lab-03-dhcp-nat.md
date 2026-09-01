# Lab 3 — DHCP, NAT/PAT, and a default route

## Goal
A LAN whose hosts get their addresses from the router by DHCP, reach the Internet through PAT, and have one internal server published with static NAT.

## Topology
```
PC1, PC2 --- SW1 --- Gi0/0 R1 Gi0/1 --- ISP (203.0.113.1)
SRV 192.168.1.50
```

R1 Gi0/0: 192.168.1.1/24 (inside). R1 Gi0/1: 203.0.113.2/30 (outside).

## R1
```
interface gi0/0
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown
!
interface gi0/1
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 203.0.113.1
!
ip dhcp excluded-address 192.168.1.1 192.168.1.99
ip dhcp pool LAN
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
 lease 0 8
!
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface gi0/1 overload
ip nat inside source static 192.168.1.50 203.0.113.3
```

## Verify
1. Set PC1 and PC2 to DHCP; they should land in .100+ because .1–.99 is excluded.
2. `show ip dhcp binding` on R1 lists the leases.
3. Ping 8.8.8.8 from PC1, then `show ip nat translations` on R1 — you should see several inside local addresses sharing 203.0.113.2 on different ports.
4. From the ISP side, ping 203.0.113.3 and confirm it reaches the server.

## Things to break on purpose
- Remove `ip nat inside` from Gi0/0. Pings fail and no translations are created, even though the routing table is fine.
- Remove the default route and watch pings to 8.8.8.8 fail with "Destination host unreachable" from R1 itself.
- Move the LAN to a second router and fix DHCP with `ip helper-address` instead of a local pool.
