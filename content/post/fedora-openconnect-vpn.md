---
title: "Fedora Openconnect Vpn"
date: 2022-10-24T11:49:59+03:00
draft: false
toc: false
comments: true
categories:
- devops
tags:
- fedora
- vpn
---
<!--more-->
sudo dnf install ocserv

sudo systemctl enable --now ocserv

IPTABLES:
iptables -A INPUT -p tcp -m tcp --dport 444 -m conntrack --ctstate NEW -j ACCEPT
iptables -A INPUT -p udp -m udp --dport 444 -m conntrack --ctstate NEW -j ACCEPT
iptables -A POSTROUTING -s 10.0.1.0/24 -j MASQUERADE

cd /etc/pki/ocserv/
rm -Rf *.*
umask 077
sudo nano ca.tmpl (No trailing symbols!)

cn = "yabbarov.ru CA"
organization = "GreenPeace"
serial = 1
expiration_days = 3652
ca 
signing_key
cert_signing_key
crl_signing_key

sudo nano server.tmpl (No trailing symbols!)

cn = "yabbarov.ru"
organization = "GreenPeace"
serial = 2
expiration_days = 3652
signing_key
encryption_key
tls_www_server
dns_name = "yabbarov.ru"

sudo certtool --generate-privkey --outfile ca.key
sudo certtool --generate-self-signed --load-privkey ca.key --template ca.tmpl --outfile ca.pem

sudo certtool --generate-privkey --outfile server.key
certtool --generate-certificate --load-privkey server.key --load-ca-certificate ca.pem --load-ca-privkey ca.key --template server.tmpl --outfile server.pem

sudo certtool --generate-dh-params --outfile dh.pem

umask 022

```
nano /etc/ocserv/ocserv.conf
```
```
auth = "plain[passwd=/etc/ocserv/ocserv.passwords]"
server-cert = /etc/pki/ocserv/server.pem
server-key = /etc/pki/ocserv/private/server.key
dh-params = /etc/pki/ocserv/dh.pem
ca-cert = /etc/pki/ocserv/ca.pem
max-clients = 2 
max-same-clients = 4
#pid-file = /run/ocserv.pid
default-domain = yabbarov.ru
ipv4-network=10.0.1.0/24
route = default
```
If something strange happened, try to change isolate-workers option.

sudo ocpasswd -c /etc/ocserv/ocserv.passwords username-fedora

Linux:
sudo dnf search NetworkManager-openconnect-gnome  
CA cert
Gateway yabbarov.ru:444

Enter Login and password in a dialog window

Windows: 
gitlab.com/openconnect/openconnect-gui