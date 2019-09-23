# Sonarr Build
This recipe is for setting up Sonarr. The following assumes your are familiar with Sonarr and its preference menus and configuration options.

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

## 1.00 Restore a Sonarr Backup
Sonarr has a built in backup service. Sonarr will execute a backup every 7 days creating a zip file located in `/home/media/.config/NzbDrone/Backups/manual`.

But it's good idea to make a raw backup of your working base settings configuration, including all settings, before adding any series media (TV shows). This backup file must be stored outside of the Sonarr CT. In the event of needing to recreate a Sonarr CT you can use this backup file to quickly restore your Sonarr settings. This backup file must be named `nzbdrone_backup_base_settings.zip` and be located on your NAS at `/mnt/backup/sonarr`.

If you have a backup of your Sonarr settings you can restore them.
*  restore the Sonarr installation Base Settings;
*  restore the lastest dated Sonarr backup you've made.

### 1.01 Restore to Sonarr Base Settings
With the Proxmox web interface go to `typhoon-01` > `115 (sonarr)` > `>_ Shell` and type the following:
```
sudo systemctl stop sonarr.service &&
sleep 5 &&
rm -r /home/media/.config/NzbDrone/nzbdrone.db* &&
unzip -o /mnt/backup/sonarr/nzbdrone_backup_base_settings.zip 'nzbdrone.db*' -d /home/media/.config/NzbDrone &&
chown 1005:1005 /home/media/.config/NzbDrone/nzbdrone.db* &&
sudo systemctl restart sonarr.service
```

### 1.02 Restore the lastest Sonarr backup
If you want to restore to your last backup (this backup is a maximum of 7 days of age) use the Proxmox web interface and go to `typhoon-01` > `115 (sonarr)` > `>_ Shell` and type the following: 
```
sudo systemctl stop sonarr.service &&
sleep 5 &&
rm -r /home/media/.config/NzbDrone/nzbdrone.db* &&
newest=$(ls -t /home/media/.config/NzbDrone/Backups/manual/*.zip | head -1) &&
echo $newest &&
unzip -o "$newest" 'nzbdrone.db*' -d /home/media/.config/NzbDrone &&
chown 1005:1005 /home/media/.config/NzbDrone/nzbdrone.db* &&
sudo systemctl restart sonarr.service
```

## 2.00 Configure Sonarr Settings
Browse to http://192.168.50.115:8989 and login to Sonarr. Click the `Settings Tab` and click `Advanced Settings` to the `Shown` state. Configure all your tabs as follows.

### 2.01 Configure Media Management
![Screenshot_2019-09-23 Settings - Sonarr](https://user-images.githubusercontent.com/43461046/65425639-9f150300-de38-11e9-9429-68dbd0d70901.png)

