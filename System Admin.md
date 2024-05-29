---
title: System Admin
lastmod: 2024-05-24T15:51:32-05:00
---
# System Admin
## Proxmox
Initial setup steps
1. 
## Linux
Initial Debian Setup
```bash
su -
apt install sudo
adduser <username> sudo
reboot
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
## NFS
To autoconnect client to nfs at startup, append this line to the `/etc/fstab` file:
```
hostname.com:/mnt/datastore/share/username/folder /mnt/nfs nfs timeo=500,intr,_netdev 0 0
```
