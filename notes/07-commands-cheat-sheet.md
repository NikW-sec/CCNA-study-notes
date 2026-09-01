# Commands Cheat Sheet

## Modes and navigation
```
enable                       ! user EXEC -> privileged EXEC
configure terminal           ! -> global config
exit                         ! back one mode
end  (or Ctrl-Z)             ! straight to privileged EXEC
do <command>                 ! run an EXEC command from config mode
```

## Basic device setup
```
hostname R1
no ip domain-lookup                     ! stop DNS lookups on typos
enable secret <pw>
service password-encryption
banner motd #text#
line console 0
 password <pw>
 login
 logging synchronous                    ! stop logs interrupting typing
 exec-timeout 5 0
copy running-config startup-config      ! (write memory)
```

## Interfaces
```
interface gi0/0
 description LINK-TO-SW1
 ip address 192.168.1.1 255.255.255.0
 ipv6 address 2001:db8::1/64
 no shutdown
 speed 1000 / duplex full
interface range fa0/1-24
interface loopback 0
```

## Verification (the ones that earn marks)
```
show running-config                 show startup-config
show ip interface brief             show interfaces
show interfaces status              show interfaces trunk
show ip route                       show ipv6 route
show ip protocols                   show ip ospf neighbor
show vlan brief                     show mac address-table
show spanning-tree                  show etherchannel summary
show cdp neighbors [detail]         show lldp neighbors [detail]
show ip nat translations            show ip dhcp binding
show version                        show standby brief
show access-lists                   show port-security interface fa0/1
```

## Troubleshooting
```
ping 8.8.8.8
ping 8.8.8.8 source lo0
traceroute 8.8.8.8
telnet / ssh -l admin 10.0.0.1
debug ip ospf adj
undebug all                          ! always turn debugs off
terminal monitor                     ! see console logs over an SSH session
```

## Discovery protocols
```
cdp run / no cdp run                 ! global, Cisco proprietary
no cdp enable                        ! per interface
lldp run                             ! global, open standard, off by default
```

## Routing
```
ip route <net> <mask> <next-hop|interface> [AD]
ipv6 unicast-routing
router ospf 1
 router-id 1.1.1.1
 network <addr> <wildcard> area 0
 passive-interface default
 no passive-interface gi0/1
 default-information originate
 auto-cost reference-bandwidth 1000
```

## Switching
```
vlan 10 / name SALES
switchport mode access / switchport access vlan 10
switchport mode trunk / switchport trunk native vlan 99
switchport trunk allowed vlan 10,20
switchport nonegotiate
channel-group 1 mode active
spanning-tree vlan 10 root primary
spanning-tree portfast / spanning-tree bpduguard enable
```

## Services and security
```
ip nat inside source list 1 interface gi0/1 overload
ip dhcp pool LAN / network / default-router / dns-server
ip helper-address 10.0.0.10
ntp server 216.239.35.0
logging host 10.0.0.5 / logging trap 4
snmp-server community CCNAro ro
crypto key generate rsa modulus 2048
ip ssh version 2
access-list 10 permit 192.168.1.0 0.0.0.255
ip access-group WEB_ONLY in
switchport port-security [maximum 2 | mac-address sticky | violation restrict]
ip dhcp snooping / ip arp inspection vlan 10
```

## Erasing a device for a clean lab
```
write erase
delete vlan.dat
reload
```
