---
title: "Openvpn Fedora"
date: 2022-10-25T08:15:14+03:00
draft: false
toc: true
comments: true
categories:
- category1
tags:
- tag1
---
<!--more-->
sudo dnf install easy-rsa openvpn
sudo mkdir -p /etc/pki/easy-rsa
sudo ln -s /usr/share/easy-rsa/3.0.8/* /etc/pki/easy-rsa
cd /etc/pki/easy-rsa
sudo ./easyrsa init-pki
sudo nano vars

vars:
set_var EASYRSA_ALGO "ec"
set_var EASYRSA_DIGEST "sha512"
set_var EASYRSA_REQ_COUNTRY "RU"
set_var EASYRSA_REQ_PROVINCE "NC"
set_var EASYRSA_REQ_CITY "NC"
set_var EASYRSA_REQ_ORG "Goodness"
set_var EASYRSA_REQ_EMAIL "ram0973@gmail.com"
set_var EASYRSA_REQ_OU "Community"

BUILD CA:
sudo ./easyrsa build-ca # nopass

COPY CA TO ROOT CA:
sudo cp pki/ca.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust

sudo cp pki/ca.crt /etc/openvpn/server/

REQUEST FOR GENERATION:
sudo ./easyrsa gen-req server nopass

COPY SERVER KEY TO OPENVPN:
sudo cp pki/private/server.key /etc/openvpn/server/ 

sudo ./easyrsa sign-req server server

sudo cp pki/issued/server.crt /etc/openvpn/server
sudo cp pki/ca.crt /etc/openvpn/server

CLIENTS:
sudo ./easyrsa gen-req ramil-fedora nopass
sudo ./easyrsa sign-req client ramil-fedora
sudo ./easyrsa export-p12 ramil-fedora # BAD, no legacy
GOOD:
sudo openssl pkcs12 -export -legacy -inkey /etc/pki/easy-rsa/pki/private/ramil-phone.key -in /etc/pki/easy-rsa/pki/issued/ramil-phone.crt -out /etc/pki/easy-rsa/pki/private/ramil-phone.p12

GEN static HMAC KEY ; NOT NEED THIS
; sudo openvpn --genkey secret /etc/openvpn/server/ta.key

CP SAMPLE CONFIG FILES
sudo mkdir -p /etc/openvpn/samples/
sudo cp /usr/share/doc/openvpn/sample/sample-config-files/* /etc/openvpn/samples/

GEN DH key
sudo openssl dhparam -dsaparam -out /etc/openvpn/server/dh2048.pem 2048

/etc/openvpn/server.conf

;https://openvpn.net/community-resources/reference-manual-for-openvpn-2-4/

;local a.b.c.d
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key  # This file should be kept secret
dh dh2048.pem
topology subnet
server 10.0.2.0 255.255.255.0
ifconfig-pool-persist ipp.txt
keepalive 5 20
connect-retry 3
;tls-auth ta.key 0 # This file is secret
cipher AES-256-GCM
user nobody
group nobody
persist-key
persist-tun
status openvpn-status.log
log-append  /var/log/openvpn.log
verb 3
;mute 20
explicit-exit-notify 1% 

sudo systemctl enable --now openvpn-server@server.service

sudo iptables -A INPUT -p udp -m tcp --dport 1194 -m conntrack --ctstate NEW -j ACCEPT
iptables -t nat -A POSTROUTING -s 10.0.2.0/24 -j MASQUERADE # NAT for vpn clients

client.ovpn:

client
dev tun
proto udp
remote yabbarov.ru 1194
resolv-retry infinite
keepalive 5 20
connect-timeout 5
connect-retry 3
nobind
;user nobody
;group nobody

; Without this traffic will bypass tunnel
push "redirect-gateway def1 bypass-dhcp" 

persist-key
persist-tun
mute-replay-warnings
cipher AES-256-GCM
#comp-lzo
verb 3
;mute 20
; for Windows clients
cryptoapicert "SUBJ:ramil-phone"
<ca>
-----BEGIN CERTIFICATE-----
CLIENT CERT HERE
-----END CERTIFICATE-----
</ca>

By default all traffic goes on mobile