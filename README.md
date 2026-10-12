# homelab-docs
Practice lab for CompTIA A+, Network+, and Security+.

## Hardware
- Dell Inspiron 23 AIO (Proxmox VE host)
- TP-Link Archer A7 running OpenWrt

## Architecture

```mermaid
flowchart LR
  ISP[ISP] --> WAN[WAN: port 1, VLAN 2]
  WAN --> R[Archer A7 running OpenWrt]
  R --> FAM[Family LAN: ports 2-4, VLAN 1<br/>192.168.1.0/24]
  R --> GUEST[Guest Wi-Fi<br/>192.168.10.0/24]
  R --> LAB[Lab: port 5, VLAN 3, and LAB Wi-Fi<br/>192.168.20.0/24]
  GUEST --> VPN[Mullvad WireGuard tunnel]
  LAB --> VPN
  VPN --> NET[Internet]
  LAB -. blocked .-> FAM
  GUEST -. blocked .-> FAM
  LAB -. blocked .-> GUEST
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



## Build log
- 2026-10-12: Home network segmentation complete. Lab host (Proxmox) setup pending. My home setup now has three separate networks: Family, Guest, and Lab. [Router setup](docs/router-setup.md)
- 2026-10-11: Updated DNS from Mullvad to Quad9. [DNS migration](docs/dns-migration.md)
