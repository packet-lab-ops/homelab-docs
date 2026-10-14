# homelab-docs
Practice lab for CompTIA A+, Network+, and Security+.

## Hardware
- Dell Inspiron 23 AIO (Fedora Workstation host / KVM hypervisor)
- TP-Link Archer A7 running OpenWrt

## Network Architecture

Logical view: the OpenWrt router segments LAN, Guest, and an isolated lab network. Guest and lab egress through a Mullvad WireGuard tunnel.

```mermaid
flowchart TD
  ISP["ISP / CM600 modem"] --> R["OpenWrt router"]
  R --> LAN["LAN 192.168.1.0/24"]
  R --> GUEST["Guest 192.168.10.0/24<br/>client isolation on"]
  R --> LAB_NET["Lab VLAN 192.168.20.0/24"]

  subgraph AIO["Dell Inspiron 23 AIO (Fedora Host)"]
    direction TB
    HOST["Host OS: Fedora Workstation<br/>KVM / QEMU / libvirt"]
    VIRBR0["Virtual Bridge: virbr0 (NAT)<br/>192.168.122.0/24"]
    
    subgraph VMS["Guest Virtual Machines"]
      DC["WinServer2022-DC<br/>192.168.122.200 (Static)<br/>AD DS & DNS (homelab.local)"]
      CLIENT["WinClient01<br/>192.168.122.x (DHCP)<br/>Windows 11 Enterprise (Domain Joined)"]
      LINUX["server01<br/>192.168.122.x (DHCP)<br/>Ubuntu Server 24.04 LTS (OpenSSH)"]
    end

    HOST --- VIRBR0
    VIRBR0 --- DC
    VIRBR0 --- CLIENT
    VIRBR0 --- LINUX
  end

  LAB_NET --> HOST
  LAN -->|"router admin access"| R
  LAN -->|"direct, outside tunnel"| NET["Internet"]
  GUEST --> VPN["Mullvad tunnel"]
  LAB_NET --> VPN
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
