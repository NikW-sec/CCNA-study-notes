# Subnetting Quick Reference

## Mask table

| CIDR | Mask | Block size | Usable hosts |
|---|---|---|---|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |
| /31 | 255.255.255.254 | 2 | 2 (point-to-point, RFC 3021) |
| /32 | 255.255.255.255 | 1 | 1 (host route / loopback) |

Powers of two: 2, 4, 8, 16, 32, 64, 128, 256. Octet bit values: 128 64 32 16 8 4 2 1.

## Finding the subnet of an address, in four steps

Given **172.16.77.200/21**:

1. Which octet is interesting? /21 lands in the third octet (bits 17–24). Mask = 255.255.248.0.
2. Block size = 256 − 248 = **8**.
3. Subnets step by 8 in the third octet: 0, 8, 16 … 72, **80**. 77 falls in the 72 block.
4. So: network **172.16.72.0**, broadcast **172.16.79.255**, usable **172.16.72.1 – 172.16.79.254**.

## VLSM worked example

Allocate from **192.168.1.0/24** for: 100 hosts, 50 hosts, 25 hosts, and two /30 WAN links. Always start with the largest.

| Requirement | Bits needed | Prefix | Subnet | Usable range |
|---|---|---|---|---|
| 100 hosts | 7 (126) | /25 | 192.168.1.0/25 | .1 – .126 |
| 50 hosts | 6 (62) | /26 | 192.168.1.128/26 | .129 – .190 |
| 25 hosts | 5 (30) | /27 | 192.168.1.192/27 | .193 – .222 |
| WAN 1 | 2 | /30 | 192.168.1.224/30 | .225 – .226 |
| WAN 2 | 2 | /30 | 192.168.1.228/30 | .229 – .230 |

192.168.1.232 – .255 stays free for growth. Working largest-first is what keeps the blocks from overlapping.

## Summarization

To summarise 10.1.4.0/24, 10.1.5.0/24, 10.1.6.0/24, 10.1.7.0/24, compare the third octet in binary:

```
4 = 00000100
5 = 00000101
6 = 00000110
7 = 00000111
     ^^^^^^ 6 bits in common
```

16 bits (first two octets) + 6 = **/22**, so the summary is **10.1.4.0/22**. Note that the block must start on a multiple of its own size: 10.1.5.0/22 would be invalid.
