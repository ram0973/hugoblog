---
title: "Prepare Ssh Environment"
date: 2022-10-12T16:11:59+03:00
draft: true
toc: false
comments: false
categories:
- devops
tags:
- ssh
---
<!--more-->
Check existed keys and create if needed

```
# check existing ssh keys
ls -al ~/.ssh

# create key if needed or skip and copy your key to ~/.ssh/id_rsa, if existed
ssh-keygen -t rsa -b 4096 -C "your_mail@your_domain"
```

Public key permissions (.pub file): 644 (-rw-r--r--)
Private key permissions (id_rsa): 600 (-rw-------)

```
sudo chown $USER ~/.ssh/config
chmod 700 ~/.ssh/
chmod 644 ~/.ssh/id_rsa.pub
chmod 400 ~/.ssh/config
chmod 400 ~/.ssh/id_rsa
```

SSH agent:

```
# Ssh-agent: add next two lines to ~/.bashrc or ~/.zshrc
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
source ~/.bashrc
source ~/.zshrc
```

Go to github and paste contents of ~/.id_rsa.pub there https://github.com/settings/ssh/new

```
# Test key on github
ssh -T git@github.com
# change passphrase if desired: 
ssh-keygen -p
```

Write ~/.ssh/config just like in this example:

```
Host me
  Hostname domain.tld
  Port 1234
  User boss
  IdentityFile ~/.ssh/id_rsa
Host dev
  Hostname localhost
  Port 22
  User user
  IdentityFile ~/.ssh/id_rsa
  StrictHostKeyChecking no
Host staging
  Hostname localhost
  Port 2223
  User vagrant
  IdentityFile ~/.ssh/id_rsa
  StrictHostKeyChecking no
Host prod
  Hostname domain_name
  Port XXXX # ssh port on production server
  User user_name
  IdentityFile ~/.ssh/id_rsa
```
