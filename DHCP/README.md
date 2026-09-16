# DHCP Relay Across a Multi-VLAN Network

DHCP server setup with relay (`ip helper-address`) across trunked VLANs, built in Cisco Packet Tracer (1841 router, 2960 switches).

![Topology](topology-diagram.png)

## What this is

1 router, 2 switches, 4 VLANs. Switch1 trunks 3 VLANs (PC0, PC1, PC5) up to the router, which does inter-VLAN routing through sub-interfaces (router-on-a-stick). Switch2 has one VLAN (PC2) on a plain access link. A DHCP server sits on Switch2's VLAN and hands out addresses to all 4 VLANs — including the 3 that aren't local to it — using DHCP relay.

## Subnetting

Base network: `192.168.63.0/24`, split into 4 equal `/27` blocks.

| VLAN | Host | Network | Usable range | Gateway (last usable) |
|---|---|---|---|---|
| 10 | PC0 | 192.168.63.0/27 | .1–.30 | 192.168.63.30 |
| 20 | PC1 | 192.168.63.32/27 | .33–.62 | 192.168.63.62 |
| 30 | PC5 | 192.168.63.64/27 | .65–.94 | 192.168.63.94 |
| 40 | PC2 | 192.168.63.96/27 | .97–.126 | 192.168.63.126 |

DHCP server is static at `192.168.63.97`, gateway `192.168.63.126`.

## Router interfaces

| Interface | IP | Role |
|---|---|---|
| Fa0/0 | no ip address | trunk carrier |
| Fa0/0.10 | 192.168.63.30/27 | gateway VLAN 10 |
| Fa0/0.20 | 192.168.63.62/27 | gateway VLAN 20 |
| Fa0/0.30 | 192.168.63.94/27 | gateway VLAN 30 |
| Fa0/1 | 192.168.63.126/27 | gateway VLAN 40 |

## DHCP pools

| Pool | Default Gateway | DNS | Start IP | Mask | Max Users |
|---|---|---|---|---|---|
| Poolvlan10 | 192.168.63.30 | 8.8.8.8 | 192.168.63.1 | 255.255.255.224 | 25 |
| Poolvlan20 | 192.168.63.62 | 8.8.8.8 | 192.168.63.33 | 255.255.255.224 | 25 |
| Poolvlan30 | 192.168.63.94 | 8.8.8.8 | 192.168.63.65 | 255.255.255.224 | 25 |
| Poolvlan40 | 192.168.63.126 | 8.8.8.8 | 192.168.63.98 | 255.255.255.224 | 25 |

Pool 4 starts at `.98`, not `.97`, since `.97` is the server's own static IP and can't overlap with its own leasable range.

## Why DHCP relay is needed

DHCP requests are broadcasts, and broadcasts don't cross VLAN boundaries. The server sits in VLAN 40, so a broadcast from VLAN 10/20/30 would never reach it on its own. `ip helper-address` on the router's gateway interfaces converts the broadcast into a normal routed unicast packet addressed straight to the server, then relays the reply back the same way. Without it, only PC2 (same VLAN as the server) could get a DHCP lease.

## Config

Full configs in [`configs/`](configs/). Core pieces:
- **S1**: VLANs 10/20/30, host ports set to access mode, uplink port set to trunk mode with those 3 VLANs allowed
- **S2**: one VLAN (40), plain access ports for PC2 and the server, plain access uplink to the router
- **R1**: Fa0/0 left unnumbered as trunk carrier, sub-interfaces per VLAN with `encapsulation dot1Q`, `ip helper-address 192.168.63.97` added on every gateway interface (Fa0/0.10, .20, .30, and Fa0/1)
- All devices: enable secret, console/vty passwords, password encryption, login banner

## Testing

- Static IPs tested first on all 4 PCs to confirm VLAN/trunk/routing worked before adding DHCP
- Switched each PC to DHCP and confirmed lease assignment
- `show vlan brief` / `show interfaces trunk` on S1
- `show ip interface brief` / `show ip route` on R1
- Cross-VLAN ping using DHCP-assigned addresses

Screenshots in [`screenshots/`](screenshots/).

## Results

Built and tested in Packet Tracer. Static addressing was verified working end to end first, then all 4 PCs were switched to DHCP.

Hit one real bug along the way: the router's `ip helper-address` was pointed at the wrong IP (one address off from the server's actual static IP), so cross-VLAN requests were silently relaying to the wrong device instead of the DHCP server — while same-VLAN requests still worked fine, since those never needed relay at all. Traced it using Packet Tracer's Simulation Mode by watching where the DHCPDISCOVER packet actually went, found the mismatch, corrected the helper-address to match the server's real IP, and all 4 VLANs pulled valid leases after that.

## Skills

- DHCP server configuration with per-VLAN pools
- DHCP relay (`ip helper-address`) across trunked VLANs
- Router-on-a-stick inter-VLAN routing
- Packet-level troubleshooting using Simulation Mode
- Isolating a fault by testing static addressing before layering DHCP on top

## Tools

Cisco Packet Tracer, Cisco IOS (1841 router, 2960 switch)
