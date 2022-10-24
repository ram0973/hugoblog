---
title: "Fedora Server Setup"
date: 2022-10-13T14:36:36+03:00
draft: false
toc: false
comments: true
categories:
- devops
tags:
- linux
- fedora
---
Fedora Server Setup

<!--more-->
```
sudo dnf update

sudo nano /etc/systemd/timesyncd.conf
```
```
[Time]
NTP=0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org 3.pool.ntp.org
FallbackNTP=0.fedora.pool.ntp.org 1.fedora.pool.ntp.org 2.fedora.pool.ntp.org 3.fedora.pool.ntp.org
```
```
systemctl enable --now systemd-timesyncd.service
```

CHECK:
```
sudo systemctl status systemd-timesyncd.service
sudo timedatectl
```

systemd-resolved is needed by systemd. Unless you're installing an alternative DNS resolver, you should keep it.

It's important to note that it is actually listening for UDP packets on 127.0.0.53:53 to do DNS resolution for you:

The port 5355 sockets are to implement Link-Local Multicast Name Resolution (LLMNR) which is a feature only useful in LANs.

To disable it, edit /etc/systemd/resolved.conf and change the line

#LLMNR=yes
to

LLMNR=no
and then restart the service with service systemd-resolved restart and check again:

$ sudo ss -tupln | grep systemd-resolve
udp   UNCONN 0      0         127.0.0.54:53        0.0.0.0:*    users:(("systemd-resolve",pid=44426,fd=14))              
udp   UNCONN 0      0      127.0.0.53%lo:53        0.0.0.0:*    users:(("systemd-resolve",pid=44426,fd=12))              
tcp   LISTEN 0      4096   127.0.0.53%lo:53        0.0.0.0:*    users:(("systemd-resolve",pid=44426,fd=13))              
tcp   LISTEN 0      4096      127.0.0.54:53        0.0.0.0:*    users:(("systemd-resolve",pid=44426,fd=15))  
