# homelab-docs
Practice lab for CompTIA A+, Network+, and Security+.

## Hardware
- Dell Inspiron 23 AIO (Proxmox VE host)
- TP-Link Archer A7 running OpenWrt

## Architecture

Logical view: the OpenWrt router segments LAN, Guest, and an isolated lab network. Guest and lab egress through a Mullvad WireGuard tunnel.

```mermaid
flowchart TD
  ISP["ISP / CM600 modem"] --> R["OpenWrt router"]
  R --> LAN["LAN 192.168.1.0/24"]
  R --> GUEST["Guest 192.168.10.0/24<br/>client isolation on"]
  R --> LAB["lab 192.168.20.0/24<br/>Proxmox host 192.168.20.10"]
  LAN -->|"router admin access"| R
  LAN -->|"direct, outside tunnel"| NET["Internet"]
  GUEST --> VPN["Mullvad tunnel"]
  LAB --> VPN
  VPN --> NET
```

## What This Shows

- Segmented home network with separate family, guest, and lab networks
- VLAN configuration on a consumer router running OpenWrt
- Firewall zones and inter-zone policy
- Policy routing through a WireGuard VPN
- DNS and DHCP configuration

## Status

Network segmentation is complete. Proxmox VE host setup is next.

## Documentation

- [Router setup](docs/router-setup.md)
- [DNS migration](docs/dns-migration.md)
- [ISP modem replacment](docs/isp-modem-setup.md)



## Build log
- 2026-10-12: Home network segmentation complete. Lab host (Proxmox) setup pending. My home setup now has three separate networks: Family, Guest, and Lab. [Router setup](docs/router-setup.md)
- 2026-10-11: Updated DNS from Mullvad to Quad9. [DNS migration](docs/dns-migration.md)
- 2026-09-21: Replaced ISP modem/router with my own hardware [ISP modem replacment](docs/isp-modem-setup.md)
