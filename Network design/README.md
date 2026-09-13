# Trunked Multi-VLAN Campus Network 

Router-on-a-stick + VLSM addressing project built in Cisco Packet Tracer (1841 routers, 2960 switches).



## Overview

Simulated network for a small company with 10 departments across 6 offices, 3 routers. Switch 1's office has 3 departments (Sales, Marketing, Customer Success) and Switch 3's office has 3 departments (Finance, Legal, Procurement), so those two run trunk links with the router doing inter-VLAN routing through sub-interfaces (router-on-a-stick). The other 4 offices just have one department each on a plain access link. Everything runs on one `/24` (`192.168.63.0/24`), split with VLSM so all 256 addresses get used.


## Subnetting (VLSM)

| Net | VLAN | Hosts needed | Block | Mask |
|---|---|---|---|---|
| 1 | 10 | 27 | /27 | 255.255.255.224 |
| 2 | 20 | 25 | /27 | 255.255.255.224 |
| 3 | 40 | 22 | /27 | 255.255.255.224 |
| 4 | 60 | 20 | /27 | 255.255.255.224 |
| 5 | 30 | 19 | /27 | 255.255.255.224 |
| 6 | 50 | 18 | /27 | 255.255.255.224 |
| 7 | 11 | 12 | /28 | 255.255.255.240 |
| 8 | 12 | 10 | /28 | 255.255.255.240 |
| 9 | 31 | 8 | /28 | 255.255.255.240 |
| 10 | 32 | 5 | /29 | 255.255.255.248 |
| 11 | WAN R1-R2 | 2 | /30 | 255.255.255.252 |
| 12 | WAN R2-R3 | 2 | /30 | 255.255.255.252 |

Sorted biggest to smallest before allocating so there's no wasted space — 256/256 addresses used.

| Net | Network | Usable range | Broadcast |
|---|---|---|---|
| 1 | 192.168.63.0/27 | .1–.30 | .31 |
| 2 | 192.168.63.32/27 | .33–.62 | .63 |
| 3 | 192.168.63.64/27 | .65–.94 | .95 |
| 4 | 192.168.63.96/27 | .97–.126 | .127 |
| 5 | 192.168.63.128/27 | .129–.158 | .159 |
| 6 | 192.168.63.160/27 | .161–.190 | .191 |
| 7 | 192.168.63.192/28 | .193–.206 | .207 |
| 8 | 192.168.63.208/28 | .209–.222 | .223 |
| 9 | 192.168.63.224/28 | .225–.238 | .239 |
| 10 | 192.168.63.240/29 | .241–.246 | .247 |
| 11 | 192.168.63.248/30 | .249–.250 | .251 |
| 12 | 192.168.63.252/30 | .253–.254 | .255 |

## Hosts

| Host | VLAN | IP | Mask | Gateway |
|---|---|---|---|---|
| PC1 | 10 | 192.168.63.1 | /27 | 192.168.63.30 |
| PC2 | 20 | 192.168.63.33 | /27 | 192.168.63.62 |
| PC3 | 30 | 192.168.63.129 | /27 | 192.168.63.158 |
| PC4 | 40 | 192.168.63.65 | /27 | 192.168.63.94 |
| PC5 | 50 | 192.168.63.161 | /27 | 192.168.63.190 |
| PC6 | 60 | 192.168.63.97 | /27 | 192.168.63.126 |
| PC7 | 11 | 192.168.63.193 | /28 | 192.168.63.206 |
| PC8 | 12 | 192.168.63.209 | /28 | 192.168.63.222 |
| PC9 | 31 | 192.168.63.225 | /28 | 192.168.63.238 |
| PC10 | 32 | 192.168.63.241 | /29 | 192.168.63.246 |

## Router interfaces

| Router | Interface | IP | Notes |
|---|---|---|---|
| R1 | Fa0/0 | no ip address | trunk carrier, VLANs 10/11/12 |
| R1 | Fa0/0.10 | 192.168.63.30/27 | gateway VLAN 10 |
| R1 | Fa0/0.11 | 192.168.63.206/28 | gateway VLAN 11 |
| R1 | Fa0/0.12 | 192.168.63.222/28 | gateway VLAN 12 |
| R1 | Fa0/1 | 192.168.63.62/27 | gateway VLAN 20 |
| R1 | Se0/1/0 | 192.168.63.249/30 | WAN to R2 |
| R2 | Fa0/0 | no ip address | trunk carrier, VLANs 30/31/32 |
| R2 | Fa0/0.30 | 192.168.63.158/27 | gateway VLAN 30 |
| R2 | Fa0/0.31 | 192.168.63.238/28 | gateway VLAN 31 |
| R2 | Fa0/0.32 | 192.168.63.246/29 | gateway VLAN 32 |
| R2 | Fa0/1 | 192.168.63.94/27 | gateway VLAN 40 |
| R2 | Se0/1/0 | 192.168.63.250/30 | WAN to R1 |
| R2 | Se0/1/1 | 192.168.63.253/30 | WAN to R3 |
| R3 | Fa0/0 | 192.168.63.190/27 | gateway VLAN 50 |
| R3 | Fa0/1 | 192.168.63.126/27 | gateway VLAN 60 |
| R3 | Se0/1/1 | 192.168.63.254/30 | WAN to R2 |

## VLANs

| VLAN | Department | Switch | Host |
|---|---|---|---|
| 10 | Sales | Switch 1 | PC1 |
| 11 | Marketing | Switch 1 | PC7 |
| 12 | Customer Success | Switch 1 | PC8 |
| 20 | Engineering | Switch 2 | PC2 |
| 30 | Finance | Switch 3 | PC3 |
| 31 | Legal | Switch 3 | PC9 |
| 32 | Procurement | Switch 3 | PC10 |
| 40 | HR | Switch 4 | PC4 |
| 50 | Support | Switch 5 | PC5 |
| 60 | IT | Switch 6 | PC6 |

Switch 1 and Switch 3 uplinks are trunk ports carrying all 3 local VLANs. Every host port everywhere else is a plain access port.

## Config

Full configs in [`configs/`](configs/). Basic idea:
- SW1/SW3: create VLANs, set host ports to access mode, set uplink port to trunk mode with allowed VLANs
- R1/R2: leave physical Fa0/0 unnumbered, create sub-interfaces per VLAN with `encapsulation dot1Q`, add static routes for everything not directly connected
- R3: plain access interfaces + one default route back to R2

## Routing

R1 and R2 use static routes since they each connect to multiple subnets. R3 just has one default route since it's the last hop. No route summarization here — the subnets were packed by block size, not by which router owns them, so they don't group into clean summary lines.


## Results

Built and tested in Packet Tracer. Trunks, sub-interfaces, and static routes all came up as planned — VLAN and routing tables match the address plan.

## Skills

- VLSM subnetting with mixed block sizes, 100% address use
- 802.1Q trunking
- Router-on-a-stick inter-VLAN routing
- Static routing
  

## Tools

Cisco Packet Tracer, Cisco IOS (1841 router, 2960 switch)
