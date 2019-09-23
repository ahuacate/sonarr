# Sonarr Build
This recipe is for setting up Sonarr.

Network Prerequisites are:
- [x] Layer 2 Network Switches
- [x] Network Gateway is `192.168.1.5`
- [x] Network DNS server is `192.168.1.5` (Note: your Gateway hardware should enable you to a configure DNS server(s), like a UniFi USG Gateway, so set the following: primary DNS `192.168.1.254` which will be your PiHole server IP address; and, secondary DNS `1.1.1.1` which is a backup Cloudfare DNS server in the event your PiHole server 192.168.1.254 fails or os down)
- [x] Network DHCP server is `192.168.1.5`

Other Prerequisites are:
- [x] Synology NAS, or linux variant of a NAS, is fully configured as per [SYNOBUILD](https://github.com/ahuacate/synobuild#synobuild).
- [x] Proxmox node fully configured as per [PROXMOX-NODE BUILDING](https://github.com/ahuacate/proxmox-node/blob/master/README.md#proxmox-node-building).
- [x] pfSense is fully configured as per [HAProxy in pfSense](https://github.com/ahuacate/proxmox-reverseproxy/blob/master/README.md#haproxy-in-pfsense)
- [x] Deluge LXC with Deluge SW installed as per [Deluge LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#40-deluge-lxc---ubuntu-1804).
- [x] NZBGet LXC with NZBGet SW installed as per [NZBget LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#300-nzbget-lxc---ubuntu-1804)
- [x] Sonarr LXC with Sonarr SW installed as per [Sonarr LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#80-sonarr-lxc---ubuntu-1804)

Tasks to be performed are:
- [ ] 1.00 Setting Up Sonarr
- [ ] 2.00 Setup your ShowRSS, MVGroup, TheTVDb and Documentary Torrent Accounts
- [ ] 3.00 Download the FileBot deluge-postprocess.sh script for Deluge
- [ ] 00.00 Patches & Fixes

## 1.00 Setup the Basics
Q
