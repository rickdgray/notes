---
title: System Admin
lastmod: 2025-02-10T13:56:12-06:00
---
# System Admin
## Proxmox
Initial setup steps
1. [Disable license nag](https://github.com/rickycodes/pve-no-subscription)
2. Switch to free update repository
3. Install sudo
4. [Redirect to port 80/443 using nginx](https://pve.proxmox.com/wiki/Web_Interface_Via_Nginx_Proxy)
5. Setup notifications
## Linux
Initial Debian Setup
```bash
su -
apt install sudo
adduser <username> sudo
reboot
```
Release and Request new IP
```bash
# get device for internet
ip addr
# delete old ip
sudo ip addr del 192.168.1.10/24 dev ens10
# release for eth0
sudo dhclient -r ens10
# renew for eth0
sudo dhclient ens10
```
Set hostname
```bash
sudo vim /etc/hostname
```
Change username of current user
```bash
# you need a temp user to do the change
sudo adduser temp
# temp needs sudo perms
sudo adduser temp sudo

# now logout and login as temp user

# change username and set new home folder
sudo usermod -l new-username -m -d /home/new-username old-username
# change user's groupname
sudo groupmod -n new-username old-username
# change user's home directory (not needed because you change in first command)
sudo usermod -d /home/newHomeDir -m newUsername

# now logout and login as your new username

# delete the temp
sudo deluser temp
sudo rm -r /home/temp
```
## Docker
Docker engine install script
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
## NGINX
Good default for security headers
```
# Security headers
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header Content-Security-Policy 'frame-ancestors https://mywebapp.mywebsite.example';
add_header X-Content-Type-Options nosniff;
add_header Content-Security-Policy "default-src 'self' www.google-analytics.com ajax.googleapis.com www.google.com google.com gstatic.com www.gstatic.com connect.facebook.net facebook.com;";
add_header X-XSS-Protection "1; mode=block";
add_header Referrer-Policy "origin";
```
[413 Request Entity Too Large](https://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)
Allow large uploads; default is only 1MB
```
location /uploads {
    ...
    client_max_body_size 100M;
}
```
Reverse proxy to GitHub Pages as your host
```
server {
	location / {
		proxy_pass              http://taddevries.github.io;
		proxy_redirect          default;
		proxy_buffering         off;
		proxy_set_header        Host                    $host;
		proxy_set_header        X-Real-IP               $remote_addr;
		proxy_set_header        X-Forwarded-For         $proxy_add_x_forwarded_for;
		proxy_set_header        X-Forwarded-Protocol    $scheme;
	}
}
```
## Set Permissions Recursive
```bash
sudo chown -R www-data:www-data ./*
sudo find . -type d -exec chmod 0755 {} \;
sudo find . -type f -exec chmod 0644 {} \;
sudo systemctl restart apache2.service
```
## Certbot
### Plugin
Does not work with the redirect to github I have in place right now.
```bash
sudo certbot --nginx -d rickdgray.com -d www.rickdgray.com -d dev.rickdgray.com -d code.rickdgray.com -d guac.rickdgray.com -d jambot.rickdgray.com -d nextcloud.rickdgray.com -d speedtest.rickdgray.com
```
### Manual
```bash
sudo certbot certonly --manual -d rickdgray.com -d www.rickdgray.com -d dev.rickdgray.com -d code.rickdgray.com -d guac.rickdgray.com -d jambot.rickdgray.com -d nextcloud.rickdgray.com -d speedtest.rickdgray.com --agree-tos --no-bootstrap --manual-public-ip-logging-ok --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory
```
### Issues with multiple certs
If certbot creates a new set of certificates such as `example.com-0001.conf`, then it is getting confused and the easiest solution is to just delete all certificates and generate new.
```bash
sudo certbot certificates
sudo certbot delete
# delete all
./certs.sh
# create all new certs
sudo systemctl restart nginx.service
```
## Ubuntu
### Disable cloud-init
```bash
sudo touch /etc/cloud/cloud-init.disabled
```
### nginx
#### Enable site
```bash
sudo ln -s /etc/nginx/sites-available/www.example.org.conf /etc/nginx/sites-enabled/
sudo systemctl restart nginx.service
```
### Open port
Ubuntu uses "uncomplicated fire wall" or UFW
```bash
sudo ufw allow 1701

# for more granularity
ufw allow 11200:11299/tcp
ufw allow 11200:11299/udp

# to do a quick test with netcat
nc -l 1701
# then use telnet from windows and send a message

# to undo a change
ufw delete allow 80
```
## NAS
To autoconnect client to nfs at startup, append this line to the `/etc/fstab` file:
```
hostname.com:/mnt/datastore/share/username/folder /mnt/nfs nfs timeo=500,intr,_netdev 0 0
```
To migrate a ZFS dataset to a new pool:
```bash
zfs snapshot -r PoolName/DatasetName@SnapshotName
zfs send -R PoolName/DatasetName@SnapshotName | zfs receive -v PoolName/DatasetName
```

## Proxmox

### Import Disk for HAOS VM

1. Navigate to the installation page on the [HA website](https://www.home-assistant.io/installation/alternative).
2. Right-click the KVM/Proxmox link and copy the address.
3. In your Proxmox console, use `wget` to download the file, then `unxz` to decompress it.
```bash
wget https://github.com/home-assistant/operating-system/releases/download/14.2/haos_ova-14.2.qcow2.xz
unxz ./haos_ova-14.2.qcow2.xz
```
4. Create the VM.
    1. General:
        - Select your VM name and ID.
        - Select "start at boot."
    2. OS:
        - Select "Do not use any media."
    3. System:
        - Change "machine" to "q35."
        - Change "BIOS" to "OVMF (UEFI)."
        - Select the "EFI storage" (typically `local-lvm`).
        - Uncheck "Pre-Enroll keys."
    4. Disks:
        - Delete the SCSI drive and any other disks.
    5. CPU:
        - Set minimum 2 cores.
    6. Memory:
        - Set minimum 4096 MB.
    7. Network:
        - Leave default unless you have special requirements (static, VLAN, etc).
    8. Confirm and finish. Do not start the VM yet.
5. Add the image to the VM; In your node's console, use the following command to import the image from the host to the VM specified by it's ID.
```bash
qm importdisk 101 ./haos_ova-14.2.qcow2.xz local-lvm
```
6. Select your HA VM.
7. Go to the "Hardware" tab.
8. Select the "Unused Disk" and click the "Edit" button.
9. Check the "Discard" box if you're using an SSD then click "Add."
10. Select the "Options" tab.
11. Select "Boot Order" and hit "Edit."
12. Check the newly created drive (likely `scsi0`) and move to first in priority order.
13. Finish Up:
    1. Start the VM.
    2. Check the shell of the VM. If it booted up correctly, you should be greeted with the link to access the Web UI.
    3. Navigate to http://homeassistant.local:8123 to access Web UI.
14. Done. Everything should be up and running now.
