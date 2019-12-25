# Sonarr Build
This recipe is for setting up Sonarr version 3. The following assumes your are familiar with Sonarr and its preference menus and configuration options.

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
- [x] Jackett installed as per [Jackett LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#500-jackett-lxc---ubuntu-1804)
- [x] Sonarr LXC with Sonarr SW installed as per [Sonarr LXC - Ubuntu 18.04](https://github.com/ahuacate/proxmox-lxc-media/blob/master/README.md#80-sonarr-lxc---ubuntu-1804)

Tasks to be performed are:
- [1.00 Set your Sonarr API Key](#100-set-your-sonarr-api-key)
- [2.00 Manually Configure Sonarr Settings](#200-manually-configure-sonarr-settings)
	- [2.01 Configure Media Management](#201-configure-media-management)
	- [2.01a Episode Naming](#201a-episode-naming)
	- [2.01b Root Folders](#201b-root-folders)
	- [2.02 Configure Profiles](#202-configure-profiles)
	- [2.02a Create a new Profile - 4K > HDTV-1080p](#202a-create-a-new-profile---4k--hdtv-1080p)
	- [2.02b Delay Profiles](#202b-delay-profiles)
	- [2.02c Release Profiles](#202c-release-profiles)
	- [2.03 Configure Quality](#203-configure-quality)
	- [2.04 Configure Indexers](#204-configure-indexers)
	- [2.05 Configure Download Clients](#205-configure-download-clients)
	- [2.05a  Deluge Download Client](#205a--deluge-download-client)
	- [2.05b  NZBGet Download Client](#205b--nzbget-download-client)
	- [2.06 Configure Connect](#206-configure-connect)
	- [2.06a  Jellyfin Connection](#206a--jellyfin-connection)
	- [2.06b  sonarr-episode-trimmer](#206b--sonarr-episode-trimmer)
	- [2.07 Configure General](#207-configure-general)
	- [2.08 Configure UI](#208-configure-ui)
- [3.00 Create & Restore Sonarr Backups](#300-create--restore-sonarr-backups)
	- [3.01 Create a Base Settings Backup](#301-create-a-base-settings-backup)
	- [3.03 Restore to Sonarr Base Settings](#303-restore-to-sonarr-base-settings)
	- [3.03 Restore the lastest Sonarr backup](#303-restore-the-lastest-sonarr-backup)
- [00.00 Patches & Fixes](#0000-patches--fixes)


## 1.00 Set your Sonarr API Key
For ease of configuring all my API connections I set Sonarr to a known API ID
```
sudo systemctl stop sonarr.service &&
sleep 5 &&
sed -i 's|<ApiKey>.*|<ApiKey>1a0b9fd2dc144ec28141440f72616c74</ApiKey>|g' /var/lib/sonarr/config.xml &&
sudo systemctl restart sonarr.service
```

## 2.00 Manually Configure Sonarr Settings
Browse to http://192.168.50.115:8989 and login to Sonarr. Click the `Settings Tab` and click `Advanced Settings` to set `Shown` state. Configure all your tabs as follows.

### 2.01 Configure Media Management
Set as shown in the image.
### 2.01a Episode Naming

For the field **Standard Episode Format** edit:
```
{Series Title} - S{season:00}E{episode:00} - {Episode Title} - [{Quality Title} {MediaInfo Simple}]
```
For the field **Daily Episode Format** edit:
```
{Series Title} - {Air-Date} - {Episode Title} - [{Quality Title} {MediaInfo Simple}]
```
For the field **Anime Episode Format** edit:
```
{Series Title} - S{season:00}E{episode:00} - [{Quality Title} {MediaInfo Simple}]
```
### 2.01b Root Folders
Here we point Sonarr to our media libraries. Click `Add Root Folder` and add the following folders:

| Root Folders | Value | Notes
| :---  | :---: | :---
| TV Series Folder | `/mnt/video/tv` | ***Here is your TV series folder***
| Documentary Folder | `/mnt/video/documentary/series` | ***Optional --- for use with Flexget to get documentary videos***

Cross check and set all the fields as follows:
![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/media_management.png)

### 2.02 Configure Profiles
Each box in the Profiles tab shows a list of allowed qualities. For a episode that is assigned the "Any" profile, radarr will search for all qualities in the list, and choose highest match found.

You may create new Profiles with specific qualities in any order you wish to use with higher priority qualities at the top, and lower priority qualities at the bottom.

The following is a custom profile for 4K > HDTV-1080p.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/profiles.png)

### 2.02a Create a new Profile - 4K > HDTV-1080p
This Profile is applicable for a 4K TV.

Create the new profile by clicking the + icon. Complete the form as follows making sure of the order of Qualities.
![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/4K_1080p_profile.png)

### 2.02b Delay Profiles
You can set up Delay Profiles to wait to download preferred releases until after a certain time has elapsed, this will allow extra time for releases with your preferred tags or cutoffs to be released.

For example, my delay profile will wait one day to start the download, and if any releases containing my preferred tags come across it will be preferred over others that do not have the preferred tags.

I recommend to set as follows:

| Delay Profile | Value | Notes
| :---  | :---: | :---
| Protocol | `Prefer Usenet`
| Usenet Delay | `720`
| Torrent Delay | `720`

### 2.02c Release Profiles
I would like for TV shows to be sourced or upgraded to my `4K > HDTV-1080p` profile with preferred conditions (i.e HEVC, x265, h265, hdr, 10bit with surround sound). This is done in `Release Profiles`.

| Release Profiles | Value | Value | Notes
| :---  | :---: | :---: | :---
| **Must Contain**
|| Leave Blank
| **Must Not Contain**
|| Leave Blank
| **Preferred**
| | `hdr` | `40`
| | `10bit` | `40`
| | `10 bit` | `40`
| | `hevc` | `20`
| | `h265` | `20`
| | `h.265` | `20`
| | `x265` | `20`
| | `x.265` | `20`
| | `h264` | `10`
| | `h.264` | `10`
| | `x264` | `10`
| | `x.264` | `10`
| | `atmos` | `10`
| | `7.1` | `10`
| | `5.1` | `5`

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/release_profiles.png)

### 2.03 Configure Quality
By default some quality profiles have a minimum limit of 0 MB/h. To maintain bitrate quality I set a minimum of 1Gb/h (17.7 Mb/m) and above depending on quality type.

I recommened the following value edits:

| Quality Definitions | Minimum MB/m | Maximim MB/m
| :---  | :---: | :---: 
| HDTV-1080p | `17.7` | `170.7`
| WEBDL-1080p | `17.7` | `100`
| WEBRip-1080p | `17.7` | `130`
| Bluray-1080p | `17.7` | `170.7`
| Bluray-1080p Remux | `35` | `400`
| HDTV-2160p | `25` | `400`
| WEBDL-2160p | `25` | `400`
| WEBRip-2160p | `25` | `400`
| Bluray-2160p | `35` | `400`
| Bluray-2160p Remux | `35` | `400`

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/quality.png)

### 2.04 Configure Indexers
This is where you configure Lidarr to use Usenet as your primary search indexer and Torrents (Jackett) as the secondary indexer.  For torrents Sonarr uses Jackett which must be installed as shown [HERE](https://github.com/ahuacate/jackett).

### 2.04a Add Jackett as a Indexer
Create a new torrent indexer using the `Torznab Custom` template and fill out the details as shown below.

| Add Torznab | Value
| :---  | :---:
| Name | `Jackett`
| Enable RSS Sync | `No`
| Enable Search | `Yes`
| URL | `http://192.168.50.120:9117`
| API Path | `/torznab/all/api`
| API Key | `s9tcqkddvjpkmis824pp6ucgpwcd2xnc`
| Categories | `5030,5040`
| Anime Categories | leave blank
| Additional Parameters | leave blank
| Minimum Seeders | `10`
| Seed Ratio | leave blank
| Seed Time | leave blank
| Season-Pack Seed Time | leave blank

And click `Save`. The finished Jackett configuration looks like:

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/torznab.png)

### 2.04b Add Usenet Indexers
Add all your Usenet indexers providers with the `Newsnab` presets (or custom if your provider is not listed).

Finally edit the `Options` Retention to `1500` days.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/indexers.png)

### 2.05 Configure Download Clients
Configure `Download Client` fields as follows:

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/download_client.png)

### 2.05a  Deluge Download Client
Create a new download client using the `Torrent > Deluge` template and fill out the details as shown below.

| Add Deluge | Value | Notes
| :---  | :---: | :---
| Name | `Deluge`
| Enable| `Yes`
| Host | `192.168.30.113`
| Port | `8112`
| URL Base| leave blank
| Password| `insert your deluge password` | *This is your Deluge login password*
| Category | `sonarr-series`
| Post-Import Category | leave blank
| Recent Priority | First
| Older Priority | Last
| Add Paused | `☐`
| Use SSL | `☐`
| Client Priority | `2`

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/deluge.png)

### 2.05b  NZBGet Download Client
Create a new download client using the `Usenet > NZBGet` template and fill out the details as shown below.

| Add NZBGet | Value | Notes
| :---  | :---: | :---
| Name | `NZBGet`
| Enable | `☑`
| Host | `192.168.30.112`
| Port | `6789`
| URL Base | leave blank
| Username | `client`
| Password| `insert your client password` | *This is your NZBGet client password*
| Category | `sonarr-series`
| Recent Priority | `High`
| Older Priority | `Normal`
| Add Paused | `☐`
| Use SSL | `☐`
| Client Priority | `1`

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/nzbget.png)

### 2.06 Configure Connect
Here you need to create two connections: A) Jellyfin; and, B) sonarr-episode-trimmer. 

### 2.06a  Jellyfin Connection
Create a new connection using the `Emby (Media Browser)` template and fill out the details as shown below.

| Add - Emby (Media Browser) | Value | Notes
| :---  | :---: | :---
| Name | `Jellyfin`
| On Grab| `No`
| On Download | `Yes`
| On Upgrade | `Yes`
| On Rename | `Yes`
| Filter Series | leave blank
| Host | `192.168.50.111`
| Port | `8096`
| API Key | Insert your Jellyfin API key | *Note, create one in Jellyfin for Sonarr*
| Send Notifications| `Yes`
| Update Library | `Yes`

And click `Test` to check it works. If successful, click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/jellyfin.png)

### 2.06b  sonarr-episode-trimmer
**Under Development**
Create a new connection using the `Custom Script` template and fill out the details as shown below.

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
The required edits are: 1) `URL Base`; and, 2) setting the security section to enable username and login.

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
| Certificate Validation | `Enabled`

And click `Save`.

![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/general.png)

### 2.08 Configure UI
![alt text](https://raw.githubusercontent.com/ahuacate/sonarr/master/images/ui.png)


## 3.00 Create & Restore Sonarr Backups
Sonarr has a built in backup service. Sonarr will execute a backup every 7 days creating a zip file located in `/home/media/.config/NzbDrone/Backups/scheduled`.

But it's good idea to make a clean backup of your working Sonarr settings, including all passwords etc, before adding any tv series media. The clean backup file MUST be stored outside of the Proxmox Radarr LXC container for safe keeping. Then in the event of you needing to recreate your Sonarr LXC you can use this backup file to quickly restore all your Sonarr settings.

This backup file must be named sonarr_backup_base_settings.zip and be located on your NAS in folder /mnt/backup/sonarr for the following scripts to work.

### 3.01 Create a Base Settings Backup
Browse to http://192.168.50.115:8989 and login to Sonarr. Click the `Systems Tab` > `Logs Files` and click `Clear` on all.

Then click `System Tab` > `Backup Tab` and click `Backup Now` to create a new backup file which will be shown with a name like `nzbdrone_backup_2019.09.24_05.39.55.zip`. Now right click on this newly created file (at the top of list) and save to your NAS share `/proxmox/backup/sonarr` (locally mounted as /mnt/backup/sonarr). Rename your backup file from `nzbdrone_backup_2019.09.24_05.39.55.zip` to `sonarr_backup_base_settings.zip`.

### 3.03 Restore to Sonarr Base Settings
With the Proxmox web interface go to `typhoon-01` > `115 (sonarr)` > `>_ Shell` and type the following:
```
sudo systemctl stop sonarr.service &&
sleep 5 &&
rm -r /var/lib/sonarr/sonarr.db* &&
unzip -o /mnt/backup/sonarr/sonarr_backup_base_settings.zip 'sonarr.db*' -d /var/lib/sonarr &&
chown 1605:65605 /var/lib/sonarr/nzbdrone.db* &&
sudo systemctl restart sonarr.service
```

### 3.03 Restore the lastest Sonarr backup
If you want to restore to your last backup (this backup is a maximum of 7 days of age) use the Proxmox web interface and go to `typhoon-01` > `115 (sonarr)` > `>_ Shell` and type the following: 
```
sudo systemctl stop sonarr.service &&
sleep 5 &&
rm -r /var/lib/sonarr/sonarr.db* &&
newest=$(ls -t /var/lib/sonarr/Backups/scheduled/*.zip | head -1) &&
echo $newest &&
unzip -o "$newest" 'sonarr.db*' 'config.xml' -d /var/lib/sonarr &&
chown 1605:65605 /var/lib/sonarr/sonarr.db* &&
chown 1605:65605 /var/lib/sonarr/config.xml &&
sudo systemctl restart sonarr.service
```

## 00.00 Patches & Fixes
