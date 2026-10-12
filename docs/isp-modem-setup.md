# ISP Modem Setup: Replacing ISP-Provided Gear

I replaced the ISP-supplied modem/router combo with my own modem and router so I control routing, firewall rules, and Wi-Fi. This covers the modem part.

## Why

The ISP gateway combines modem, router, and Wi-Fi in one box, with limited settings. Using my own modem lets the Archer A7 (OpenWrt) handle all routing, firewall rules, and wireless networks.

## Hardware

- **Modem:** Netgear CM600, bought used for $5
- **Router:** TP-Link Archer A7 (V5), running OpenWrt

## Steps

1. **Check compatibility.** Confirmed the modem is on the ISP's approved list before buying.
2. **Firmware.** Logged into the modem's admin page and checked that it had the latest firmware.
3. **Change the admin password.** Replaced the default password so the modem can't be managed by anyone with the factory login.
4. **Activate with the ISP.** Called the ISP to link the modem to my account. Activation needed the modem's serial number and MAC address, which the ISP used to register the device.

## Result

The modem connects to the Archer A7's WAN port. The router gets its public address through the modem, and it handles all routing, DHCP, DNS, Wi-Fi, and firewall rules.

