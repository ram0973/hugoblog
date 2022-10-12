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
## On server:
### Install Strongswan - IPsec IKEv1/IKEv2 daemon using swanctl
```
sudo dnf install strongswan

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

/etc/strongswan/swanctl/conf.d/swanctl.conf 
```
connections {
  rw {
    pools = primary-pool
    local {
      auth = pubkey
      certs = cert.pem
      id = certificate_domain_name
    }
    remote {
      auth = eap-mschapv2
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
  eap-username1 {
    id = username1
    secret = MyVeryStrongPassword2022#
  }
  eap-username2 {
    id = username2
    secret = MyVeryStrongPassword2022#
  }
}

pools {
  primary-pool {
    addrs = 10.0.0.0/24
    dns = 8.8.8.8, 8.8.4.4
  }
}
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

iptables -t nat -A POSTROUTING  -j MASQUERADE
#iptables -t nat -A POSTROUTING -m policy --pol ipsec --dir out -j MASQUERADE
```
## On client:
Install StrongSwan from Google Play

[StrongSwan in Google Play logo]!(/img/strongswan.png)

