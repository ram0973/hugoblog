---
title: "Generate Letsencrypt Certificate for Subdomains"
date: 2022-10-10T12:36:13+03:00
draft: false
categories:
- devops
tags:
- letsencrypt
---
<!--more--> 
So, let's generate Let's Encrypt certificates

First, we need to install requisites:
```
sudo dnf install certbot python3-certbot-dns-digitalocean
```
Next, create Digital Ocean token file
```
touch ~/certbot-do-creds.ini
```

Next, restrict the permissions on the file

```
chmod go-rwx ~/certbot-do-creds.ini
```
Open file:
```
nano ~/certbot-do-creds.ini
```
Insert Digital Ocean API credentials used by Certbot:

`dns_digitalocean_token = your_do_token`

Generate certificates:
```
sudo certbot certonly -n --dns-digitalocean --dns-digitalocean-credentials ~/certbot-do-creds.ini -d your.domain -d *.your.domain
```
Check certificates renew job:

```
sudo systemctl status certbot-renew.timer  

● certbot-renew.timer - This is the timer to set the schedule for automated renewals
     Loaded: loaded (/usr/lib/systemd/system/certbot-renew.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Tue 2022-10-04 09:57:19 MSK; 6 days ago
      Until: Tue 2022-10-04 09:57:19 MSK; 6 days ago
    Trigger: Mon 2022-10-10 17:09:13 MSK; 4h 27min left
   Triggers: ● certbot-renew.service
```
