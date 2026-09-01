# Lab 1 — VLANs, trunking, and router on a stick

## Goal
Two VLANs on one switch, carried to a router over a single trunk, routing between each other.

## Topology
```
PC1 (VLAN 10) --- Fa0/1 \
                          SW1 --- Gi0/1 (trunk) --- Gi0/0 R1
PC2 (VLAN 20) --- Fa0/2 /
```

| Device | VLAN | Address | Gateway |
|---|---|---|---|
| PC1 | 10 | 192.168.10.10/24 | 192.168.10.1 |
| PC2 | 20 | 192.168.20.10/24 | 192.168.20.1 |
| R1 Gi0/0.10 | 10 | 192.168.10.1/24 | |
| R1 Gi0/0.20 | 20 | 192.168.20.1/24 | |

## SW1
```
vlan 10
 name SALES
vlan 20
 name ENG
vlan 99
 name NATIVE-UNUSED
!
interface fa0/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
!
interface fa0/2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 spanning-tree bpduguard enable
!
interface gi0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 switchport nonegotiate
```

## R1
```
interface gi0/0
 no ip address
 no shutdown
!
interface gi0/0.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0
!
interface gi0/0.20
 encapsulation dot1q 20
 ip address 192.168.20.1 255.255.255.0
!
interface gi0/0.99
 encapsulation dot1q 99 native
```

## Verify
1. `show vlan brief` on SW1 — Fa0/1 in VLAN 10, Fa0/2 in VLAN 20.
2. `show interfaces trunk` — Gi0/1 trunking, native 99, allowed 10,20,99.
3. `show ip interface brief` on R1 — both subinterfaces up/up.
4. From PC1: ping 192.168.10.1 (own gateway), then 192.168.20.10 (across VLANs).

## Things to break on purpose
- Set the trunk's native VLAN to 1 on one side only and watch the mismatch message.
- Remove VLAN 20 from `switchport trunk allowed vlan` and confirm PC2 loses its gateway.
- Forget `no shutdown` on the physical Gi0/0 and note that the subinterfaces stay down with it.
