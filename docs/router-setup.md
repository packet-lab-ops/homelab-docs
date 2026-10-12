
# Home Router Setup: Family, Guest, and Lab Networks

This documents how my home router is set up so the family network, a guest network, and a practice lab stay separate, and how I got to the current design.

## Hardware and Software

The router is a TP-Link Archer A7 (V5) running OpenWrt. It has one WAN port connected to the ISP and four LAN ports. The lab host is a single-NIC Dell all-in-one that I plan to run Proxmox VE on. I don't have a managed switch, so the router's switch chip handles the VLANs.

## The Three Networks

- **Family** (LAN): Everyday devices. It's the only network that can open the router's admin page (LuCI) or SSH into it.
- **Guest**: Visitors. They can reach the internet through a VPN tunnel but can't see the family network or the router's admin page.
- **Lab**: My practice network on its own VLAN and its own Wi-Fi name. It goes through the same VPN tunnel and is isolated from the family and guest networks.

Guest and lab traffic leaves through a WireGuard tunnel to Mullvad. The router marks that traffic and routes it through the tunnel, so the internet sees the VPN's address, not ours.

## How the Switch Is Set Up

The switch chip has three VLANs:

- VLAN 1 is the family LAN, on ports 2 through 4.
- VLAN 2 is the WAN, on port 1.
- VLAN 3 is the lab, on port 5.

The router's CPU port is tagged on all three VLANs so it can route between them. The lab host plugs into port 5 with an untagged connection, so it doesn't need VLAN settings of its own. Family devices on ports 2 through 4 never see VLAN 3.

## How the Firewall Works

Each network is its own firewall zone with its own rules:

| Zone | Can reach | Can't reach | Router admin |

| Family | Internet, guest and lab (as allowed) | Nothing blocked | Yes |

| Guest  | VPN tunnel only | Family, WAN, lab | No |

| Lab    | VPN tunnel only | Family, WAN, guest | No |

Guest and lab devices can talk to each other only where the zone allows it. Guest blocks traffic between its own devices. The lab allows it, so I can work with the lab host and VMs from my laptop.

Each network also has explicit rules that allow DHCP so devices can get addresses, and that block traffic to the family subnet.

## DNS

Guest and lab devices get their DNS server from the tunnel through DHCP option 6. Without that setting, the router would hand out its own address, and lookups would go to the ISP's resolver. With it, lookups go to Mullvad's DNS address inside the tunnel, so the ISP doesn't see them.

## How Traffic Gets Into the Tunnel

The `pbr` (policy routing) package marks traffic from the guest and lab subnets and sends it to the Mullvad routing table. A matching nftables rule sets the mark, and an `ip rule` sends marked packets to that table. Adding the lab was a copy of the guest policy with a different source subnet.


## How I Got Here

The design came from a series of changes and problems:

1. **DNS change:** Mullvad's public DNS servers were shutting down, so I moved the family LAN to Quad9 (9.9.9.9 and 149.112.112.112).
2. **Guest network:** I first set up Guest with the tunnel as its only route. Then I found Guest still had forwarding to WAN enabled, which would have let traffic leave the home connection if the tunnel dropped. I removed WAN from its forwarding rules.
3. **Lab network:** I created the lab as a separate VLAN, interface, firewall zone, DHCP pool, and Wi-Fi network. I first gave it a bridge that didn't come up, so I rebuilt it as a proper bridge device.
4. **Routing:** Connected laptop to Lab but got only an address, no internet. The lab had no policy routing, so its traffic left through WAN. I added a pbr policy for the lab subnet, matching the guest setup.
5. **IPv6 leak:** The laptop showed my home IPv6 address on the Mullvad check page. The lab had received an ISP IPv6 prefix through delegation. I turned off IPv6 and prefix delegation on the lab network. The leak was also caused by the laptop using Wi-Fi and wired connections at the same time, so I now keep the laptop on one network at a time.
6. **Testing:** I tested isolation from the lab side with `nmap` and `ping`. Early results were invalid because the laptop's VPN client was routing the test traffic through its own tunnel. I reran the tests with the VPN disconnected and with split tunneling set up for the test.

## Current Status

Family, guest, and lab networks are separate. Guest and lab use the VPN tunnel. The lab network has its own address range, VLAN, and firewall rules, and it isn't reachable from the family or guest networks.
