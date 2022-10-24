---
title: "IKEv2 VPN in Linux Fedora Server with StrongSwan and mobile clients"
date: 2022-09-29T10:07:04+03:00
draft: false
categories:
- linux
tags:
- iptables
- strongswan
---
<!--more-->

Команда ipsec в руководствах - это libreswan а не strongswan!

11: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.0.0.1/32 scope global noprefixroute tun0
       valid_lft forever preferred_lft forever
    inet6 fe80::289c:8b38:c8b4:1cc3/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever

На линуксе надо ставить запрашивать inner address ! иначе не подключается

## On server:

### Generate certificates
mkdir -p ~/pki/{cacerts,certs,private,pkcs12,dh}

chmod 700 ~/pki

===================
# DH
openssl dhparam -dsaparam -out ~/pki/dh/dhparams.pem 4096


# CA
openssl req -x509 -newkey rsa:4096 -keyout ~/pki/ca-key.pem -out ~/pki/ca-cert.pem -days 3652 -nodes -subj "/C=RU/O=CA/CN=CA yabbarov.ru"

CHECK: strongswan pki --print --in ~/pki/ca-cert.pem 

# SERVER

WARNING: Never store private key of Certification Authority (CA) on VPN gateway since a theft of this master signing key will compromise your PKI.

openssl req -x509 -newkey rsa:4096 -keyout ~/pki/server-key.pem -out ~/pki/server-cert.pem -days 3652 -nodes -subj /CN=yabbarov.ru -CAkey ~/pki/ca-key.pem -CA ~/pki/ca-cert.pem -addext "subjectAltName=DNS:yabbarov.ru"

CHECK: strongswan pki --print --in ~/pki/server-cert.pem 

# CLIENT
openssl req -x509 -newkey rsa:4096 -keyout ~/pki/ramil-key.pem -out ~/pki/ramil-cert.pem -days 3652 -nodes -subj /CN=yabbarov.ru -CAkey ~/pki/ca-key.pem -CA ~/pki/ca-cert.pem -addext "subjectAltName=email:ramil@yabbarov.ru"

CHECK: strongswan pki --print --in ~/pki/ramil-cert.pem 

# ANDROID PKCS12
openssl pkcs12 -export -legacy -inkey ~/pki/ramil-key.pem -in ~/pki/ramil-cert.pem -out ~/pki/ramil.p12

CHECK:
openssl pkcs12 -legacy -info -in ~/pki/ramil.p12

sudo cp ~/pki/server-key.pem /etc/strongswan/swanctl/private
sudo cp ~/pki/server-cert.pem /etc/strongswan/swanctl/x509/
sudo cp ~/pki/ca-cert.pem /etc/strongswan/swanctl/x509ca/
===================

cp ~/pki/private/server-key.pem /etc/strongswan/swanctl/private
cp ~/pki/certs/server-cert.pem /etc/strongswan/swanctl/x509/
cp ~/pki/cacerts/ca-cert.pem /etc/strongswan/swanctl/x509ca/

### Install Strongswan - IPsec IKEv1/IKEv2 daemon using swanctl
```
sudo dnf install strongswan tpm2-abrmd

# Check service status
sudo systemctl status strongswan
○ strongswan.service - strongSwan IPsec IKEv1/IKEv2 daemon using swanctl
     Loaded: loaded (/usr/lib/systemd/system/strongswan.service; disabled; vendor preset: disabled)
     Active: inactive (dead)

# Enable and start service
sudo systemctl enable strongswan
sudo systemctl start strongswan
```
### Create StrongSwan config 

sudo nano /etc/strongswan/swanctl/conf.d/swanctl.conf 
```
  connections {
    rw {
      pools = primary-pool
      local {
        auth = pubkey
        certs = server-cert.pem
        id = yabbarov.ru # subjectAltName=DNS:yabbarov.ru
      }
      remote {
        auth = eap-mschapv2
        eap_id = %any        
      }
      children {
        rw {
           local_ts = 0.0.0.0/0
        }
      }
      send_certreq = no
    }
  }

  secrets {
    eap-john {
      id = john@connor.us
      secret = MyPassword
    }
}
pools {
    primary-pool {
        addrs = 10.0.0.0/24
        dns = 8.8.8.8, 8.8.4.4
    }
}
```
```
swanctl --reload-settings
```

### Add iptables rules
```
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT # Established and related connections
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP # Invalid packets
iptables -A INPUT -p icmp -j ACCEPT # Ping
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT # Http
iptables -A INPUT -p tcp --dport 443 -m conntrack --ctstate NEW -j ACCEPT # Https
iptables -A INPUT -p tcp --dport 2345 -m conntrack --ctstate NEW -j ACCEPT # SSH

iptables -A INPUT -p udp --dport 500 -j ACCEPT # for ISAKMP (handling of security associations)
iptables -A INPUT -p udp --dport 4500 -j ACCEPT # for NAT-T (handling of IPsec between natted devices)
iptables -A INPUT -p 50 -j ACCEPT # ESP - IP port 50 for ESP payload (the encrypted data packets)
iptables -A INPUT -j DROP

# Protect from trafic routing outside (WAN). Use outer interface here if any
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
#iptables -A FORWARD -j REJECT

iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -j MASQUERADE # NAT for vpn clients
#iptables -t nat -A POSTROUTING -m policy --pol ipsec --dir out -j MASQUERADE
```
## On client:
Install StrongSwan from Google Play

Пароли и безопасность - Конфиденциальность - Шифрование и учётные данные
Установить ramil-cert.p12
В клиенте загрузить и выбрать сертификат CA: ca-cert.pem
Тип подключения IKEv2 EAP (Логин/пароль)
Указать почту и пароль из /etc/strongswan/swanctl/conf.d/swanctl.conf 

![StrongSwan in Google Play logo](/img/strongswan.webp "StrongSwan logo on Android")

Посмотреть логи в момент подключения: 

sudo swanctl --logs
Также подробные логи в самом клиенте.

Если ошибка 

Если ошибка в логине или пароле:
Так и пишет - mschapv2 auth error

## Linux client
sudo dnf install NetworkManager-strongswan NetworkManager-strongswan-gnome