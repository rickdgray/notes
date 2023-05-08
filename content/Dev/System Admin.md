---
title: System Admin
author: Rick Gray
year: 2023
---
# Set Permissions Recursive
```bash
sudo chown -R www-data:www-data ./*
sudo find . -type d -exec chmod 0755 {} \;
sudo find . -type f -exec chmod 0644 {} \;
sudo systemctl restart apache2.service
```
# Certbot
## Plugin
Does not work with the redirect to github I have in place right now.
```bash
sudo certbot --nginx -d rickdgray.com -d www.rickdgray.com -d dev.rickdgray.com -d coder.rickdgray.com -d guac.rickdgray.com -d jambot.rickdgray.com -d nextcloud.rickdgray.com -d speedtest.rickdgray.com
```
## Manual
```bash
sudo certbot certonly --manual -d rickdgray.com -d www.rickdgray.com -d dev.rickdgray.com -d coder.rickdgray.com -d guac.rickdgray.com -d jambot.rickdgray.com -d nextcloud.rickdgray.com -d speedtest.rickdgray.com --agree-tos --no-bootstrap --manual-public-ip-logging-ok --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory
```
# Ubuntu
## Disable cloud-init
```bash
sudo touch /etc/cloud/cloud-init.disabled
```
## nginx
### Enable site
```bash
sudo ln -s /etc/nginx/sites-available/www.example.org.conf /etc/nginx/sites-enabled/
sudo systemctl restart nginx.service
```
## Open port
Ubuntu uses "uncomplicated fire wall" or UFW
```bash
sudo ufw allow 1701

# for more granularity
ufw allow 11200:11299/tcp
ufw allow 11200:11299/udp

# to do a quick test with netcat
nc -l 1701
# then use telnet from windows and send a message
```