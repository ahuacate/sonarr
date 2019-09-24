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

---
## 1.00 Easy Sonarr Configuration
You have two choices to configure Sonarr - the easy way or manually. If you choose manually proceed to Step 2 [HERE](https://github.com/ahuacate/sonarr/blob/master/README.md#200-manually-configure-sonarr-settings).

The easy is use a pre-built backup file which includes all the required configurated except two issues:
*  Setting your login password which can be done by performing this step [2.07 Configure General](); *and,*
*  Adding your NZB Usenet Index providers which can be done by performing this step [2.05 Configure Download Clients](
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

## 2.00 Manually Configure Sonarr Settings
Browse to http://192.168.50.115:8989 and login to Sonarr. Click the `Settings Tab` and click `Advanced Settings` to the `Shown` state. Configure all your tabs as follows.

### 2.01 Configure Media Management
![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/media_management.png)

### 2.02 Configure Profiles
Edit Delay Profiles. Add 300 minutes to the torrent delay.
![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/profiles.png)

### 2.03 Configure Quality
Edit HDTV Quality and BlueRay-1080p size limit to 10.00GB.
![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/quality.png)

### 2.04 Configure Indexers
This is where you configure Sonarr to use Usenet a your primary search indexer and Torrents as secondary. For torrents Sonarr uses Jackett which must be installed as shown [HERE](https://github.com/ahuacate/jackett).

**A) Add Jackett as a Indexer**

Create a new torrent indexer using the `Torznab Custom` template and fill out the details as shown below.

| Add Torznab | Value
| :---  | :---:
| Name | `Jackett`
| Enable RSS Sync | `No`
| Enable Search | `Yes`
| URL | `http://192.168.30.113:9117`
| API Path | `/torznab/all/api`
| API Key | `s9tcqkddvjpkmis824pp6ucgpwcd2xnc`
| Categories | `5030,5040`
| Anime Categories | leave blank
| Additional Parameters | leave blank
| Minimum Seeders | `1`
| Seed Ratio | leave blank
| Seed Time | leave blank
| Season-Pack Seed Time | leave blank

And click `Save`. The finished Jackett configuration looks like:

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/torznab.png)

**B) Add Usenet Indexers**

Add all your Usenet indexers providers with the `Newsnab` presets (or custom if your provider is not listed).

Finally edit the `Options` Retention to `1500` days.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/indexers.png)

### 2.05 Configure Download Clients
**A)  Deluge Download Client**

First create a new download client using the `Torrent > Deluge` template and fill out the details as shown below.

| Add Deluge | Value | Notes
| :---  | :---: | :---
| Name | `Deluge`
| Enable| `Yes`
| Host | `192.168.30.113`
| Port | `8112`
| URL Base| leave blank
| Password| `insert your deluge password` | This is your Deluge login password.
| Category | `sonarr-series`
| Recent Priority | First
| Older Priority | Last
| Add Paused | No
| Use SSL | No

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/deluge.png)

**B)  NZBGet Download Client**

First create a new download client using the `Usenet > NZBGet` template and fill out the details as shown below.

| Add NZBGet | Value | Notes
| :---  | :---: | :---
| Name | `NZBGet`
| Enable| `Yes`
| Host | `192.168.30.112`
| Port | `6789`
| URL Base| leave blank
| Username | `client`
| Password| `insert your client password` | This is your NZBGet client password.
| Category | `sonarr-series`
| Recent Priority | High
| Older Priority | Normal
| Add Paused | No
| Use SSL | No

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/nzbget.png)

Other `download tab` settings must be set as follows:

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/download_client.png)

### 2.06 Configure Connect
Here you need create two connections: A) Jellyfin; and, B) sonarr-episode-trimmer. 

**A)  Jellyfin Connection**

First create a new connection using the `Emby (Media Browser)` template and fill out the details as shown below.

| Add - Emby (Media Browser) | Value | Notes
| :---  | :---: | :---
| Name | `Jellyfin`
| On Grab| `No`
| On Download | `Yes`
| On Upgrade | `Yes`
| On Rename | `Yes`
| Filter Series | leave blank
| Host | `192.168.50.111` | This is your NZBGet client password.
| Port | `8096`
| API Key | Insert your Jellyfin API key | *Note, create one in Jellyfin specially for Sonarr*
| Send Notifications| `Yes`
| Update Library | `Yes`

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/jellyfin.png)

**A)  sonarr-episode-trimmer**

First create a new connection using the `Custom Script` template and fill out the details as shown below.

| Add - Custom Script | Value | Notes
| :---  | :---: | :---
| Name | `sonarr-episode-trimmer`
| On Grab| `No`
| On Download | `Yes`
| On Upgrade | `No`
| On Rename | `No`
| Filter Series | leave blank
| Path | `/home/media/.config/NzbDrone/custom-scripts/sonarr-episode-trimmer.py`
| Arguments | `--config /home/media/.config/NzbDrone/custom-scripts/config --custom-script`

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/sonarr-episode-trimmer.png)

### 2.07 Configure General
Here are required edits: 1) URL Base; and, 2) setting the security section to enable username and login.

| Start-Up | Value | Notes
| :---  | :---: | :---
| Bind Address | `*`
| Port Number | 8989
| URL Base | `/sonarr`
| Enable SSL | No
| Open Browser on start | Yes
| **Security**
| Authentication | `Basic (Browser Pop-up)`
| Username | `storm` | *Note, or whatever username you choose*
| Password | `insert password here` | *Add a complex password and record it i.e oTL&9qe/9Y&RV*
| API Key | leave default

And click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/general.png)

### 2.08 Configure UI
![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/ui.png)



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



