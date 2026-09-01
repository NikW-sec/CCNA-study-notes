# 2.0 Network Access (20%)

## 2.1 VLANs

A VLAN is a logical broadcast domain on a switch. Ports in different VLANs cannot talk without a router or L3 switch, even on the same physical switch.

- **Access port** — belongs to exactly one VLAN, carries untagged frames, connects to endpoints. A voice VLAN lets an IP phone tag its own traffic while the attached PC stays untagged.
- **Trunk port** — carries many VLANs between switches using **802.1Q** tagging. The tag is a 4-byte field inserted into the Ethernet header holding the 12-bit VLAN ID (1–4094) and the 3-bit PCP/CoS priority.
- **Native VLAN** — the one VLAN sent untagged over a trunk, VLAN 1 by default. It must match on both ends or the switch logs a native VLAN mismatch (CDP) and traffic leaks between VLANs. Best practice is to change it to an unused VLAN.
- VLANs 1002–1005 are reserved; 1–1005 are the normal range, 1006–4094 the extended range.

```
Switch(config)# vlan 10
Switch(config-vlan)# name SALES
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config)# interface gi0/1
Switch(config-if)# switchport trunk encapsulation dot1q   ! only where ISL also exists
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 99
Switch(config-if)# switchport trunk allowed vlan 10,20,99
```

Verify with `show vlan brief`, `show interfaces trunk`, `show interfaces gi0/1 switchport`.

**DTP** negotiates trunking automatically (`dynamic auto` / `dynamic desirable`). Disable it on user-facing ports with `switchport nonegotiate` and explicit access mode.

## 2.2 Inter-VLAN routing

1. **Router on a stick** — one router interface, one subinterface per VLAN, each with `encapsulation dot1q <vlan>`. The subinterface for the native VLAN uses `encapsulation dot1q <vlan> native`.
2. **L3 switch with SVIs** — `interface vlan 10` with an IP address, plus `ip routing` enabled. Faster, done in hardware, and the usual campus design.
3. **Routed port** — `no switchport` on a physical switch port, then an IP address, for point-to-point links to routers or other L3 switches.

```
Router(config)# interface gi0/0.10
Router(config-subif)# encapsulation dot1q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
```

## 2.3 Spanning Tree Protocol

STP (802.1D, and the default on Cisco switches as the per-VLAN Rapid PVST+) prevents Layer 2 loops, which are far more damaging than Layer 3 loops because the Ethernet header has no TTL: a loop produces a broadcast storm, MAC table instability, and duplicate frames.

**Election order:**
1. **Root bridge** — lowest bridge ID wins. Bridge ID = 4-bit priority + 12-bit extended system ID (the VLAN number) + MAC address. Default priority 32768, so the extended BID for VLAN 10 is 32778. Ties are broken by the lowest MAC address.
2. **Root port** — one per non-root switch, the port with the lowest cumulative root path cost. Tie-breakers in order: lowest neighbour bridge ID, then lowest neighbour port ID (priority then port number).
3. **Designated port** — one per segment, on the switch with the lowest cost to the root. Every port on the root bridge is designated.
4. Everything else becomes a **non-designated / blocking** port.

Default costs: 10 Mbps 100, 100 Mbps 19, 1 Gbps 4, 10 Gbps 2.

**802.1D states**: blocking (20 s max age) → listening (15 s) → learning (15 s) → forwarding, so ~50 s of convergence. **RSTP (802.1w)** replaces this with discarding/learning/forwarding plus proposal-agreement, converging in a few seconds, and adds alternate and backup port roles.

**PortFast** moves an access port straight to forwarding; always pair it with **BPDU guard**, which err-disables the port if a BPDU arrives, since a PortFast port should never see one.

```
Switch(config)# spanning-tree vlan 10 root primary
Switch(config)# spanning-tree vlan 10 priority 4096
Switch(config-if)# spanning-tree portfast
Switch(config-if)# spanning-tree bpduguard enable
Switch# show spanning-tree vlan 10
```

## 2.4 EtherChannel

Bundles up to eight physical links into one logical link, so STP treats it as a single port and no member is blocked. Load balancing is per-flow (by a hash of MAC/IP/port), not per-packet, so a single flow never exceeds one member link's speed.

- **PAgP** (Cisco): `desirable` / `auto`
- **LACP** (802.3ad, open standard): `active` / `passive`
- **Static**: `on` at both ends, no negotiation

Two `auto` or two `passive` ends will never form a channel. All members must match in speed, duplex, mode, allowed VLANs, and native VLAN.

```
Switch(config)# interface range gi0/1-2
Switch(config-if-range)# channel-group 1 mode active
Switch(config)# interface port-channel 1
Switch(config-if)# switchport mode trunk
Switch# show etherchannel summary
```

## 2.5 Wireless architectures and configuration

- **Autonomous** — each AP configured individually; does not scale.
- **Split-MAC / lightweight with a WLC** — the AP handles real-time 802.11 (beacons, ACKs, encryption); the WLC handles association, roaming, RF management, and security policy. Traffic tunnels in CAPWAP.
- **Cloud-managed** and **embedded/Mobility Express** are variations on the same control-plane split.

WLC ports and interfaces: the **management** interface for in-band management and AP joins, the **virtual** interface for client web auth and DHCP relay, **dynamic** interfaces mapped to WLANs/VLANs, and a **service port** for out-of-band access. A **LAG** bundles the WLC's distribution ports.

Wireless security: **WPA2** uses AES-CCMP; **WPA3** adds SAE (replacing the PSK handshake), forward secrecy, and protected management frames. Enterprise mode uses 802.1X/EAP with a RADIUS server instead of a pre-shared key.

Configure the WLC through its GUI: create the dynamic interface and VLAN, then WLANs > Create New, set the SSID and profile name, map it to the interface, set Layer 2 security (WPA2/WPA3 Personal or 802.1X), and enable the WLAN.
