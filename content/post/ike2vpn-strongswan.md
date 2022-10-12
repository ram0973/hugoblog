---
title: "IKEv2 VPN in Linux Fedora with StrongSwan"
date: 2022-09-29T10:07:04+03:00
draft: false
categories:
- linux
tags:
- iptables
- strongswan
---
<!--more--> 
```
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT # Established and related connections
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP # Invalid packets
iptables -A INPUT -p icmp -j ACCEPT # Ping
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT # Http
iptables -A INPUT -p tcp --dport 443 -m conntrack --ctstate NEW -j ACCEPT # Https
iptables -A INPUT -p tcp --dport 2345 -m conntrack --ctstate NEW -j ACCEPT # SSH


#iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
#iptables -A INPUT -m state --state NEW -p tcp --dport 22 -j ACCEPT # SSH
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
