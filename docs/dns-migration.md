# Home Lab DNS: Replacing Mullvad Public DNS with Quad9

## Goal
Mullvad's public DNS resolvers shut down on November 2, 2026. Move the LAN's upstream DNS to Quad9 and keep Guest DNS inside the Mullvad WireGuard tunnel so Guest clients don't leak queries to the WAN.

## Architecture
- **Router:** OpenWrt
- **LAN:** dnsmasq forwards upstream to Quad9 (9.9.9.9, 149.112.112.112, 2620:fe::fe, 2620:fe::9)
- **Guest:** DHCP option 6 points clients at <TUNNEL_DNS>, which is reachable only through the Mullvad tunnel
- **WAN/WAN6:** Peer-advertised DNS disabled; custom Quad9 resolvers set

## Configuration

### Router (OpenWrt, over SSH)
``` 
# Set LAN upstream resolvers to Quad9

uci -q delete dhcp.@dnsmasq[0].server
uci add_list dhcp.@dnsmasq[0].server='9.9.9.9'
uci add_list dhcp.@dnsmasq[0].server='149.112.112.112'
uci add_list dhcp.@dnsmasq[0].server='2620:fe::fe'
uci add_list dhcp.@dnsmasq[0].server='2620:fe::9'
uci set dhcp.@dnsmasq[0].noresolv='1'

# Guest DNS through the VPN tunnel
uci set dhcp.guest.dhcp_option='6,<TUNNEL_DNS>'

uci commit dhcp
/etc/init.d/dnsmasq restart
```
WAN settings (LuCI)

Network > Interfaces > wan and wan6 > Edit > Advanced Settings:

Uncheck Use DNS servers advertised by peer

wan: custom DNS 9.9.9.9, 149.112.112.112

wan6: custom DNS 2620:fe::fe, 2620:fe::9

Verification
```
# Linux client

dig example.com @<LAN_IP> +short

# OpenWrt router

uci show dhcp.@dnsmasq[0] | grep server

cat /tmp/resolv.conf.d/resolv.conf.auto
```
Guest device: confirmed am.i.mullvad.net/connected reports connected and the Mullvad DNS leak test passed.

Troubleshooting

1. Dnsmasq still used the old upstream after the GUI change

After changing the forwards in LuCI, uci show dhcp.@dnsmasq[0] | grep server still listed Mullvad's old public resolver alongside Quad9. Resetting the list with uci -q delete followed by uci add_list fixed it. Check the live config with uci show rather than trusting the GUI.

2. Router returned no answer

A query through the router returned ANSWER: 0 for mullvad.net, and a direct dig from the router returned nothing. Cause: dnsmasq was still forwarding to Mullvad's old resolver. Quad9 answered correctly once the upstream list was corrected.

Verified: 

LAN clients resolve through the router to Quad9, Guest DNS stays inside the Mullvad tunnel (no leaks on am.i.mullvad.net), and Mullvad's resolver is removed from dnsmasq.

Tested October 11, 2026.
