---
title: Secure a server
layout: post
subtitle: Fundemental security measures
description: Do it otherwise people will cryptomine on your machine :(
image: /assets/img/uploads/security.jpg
date: '2020-07-30 13:28:05 +0900'
optimized_image: /assets/img/uploads/security.jpg
category: fundementals
tags:
  - security
  - linux
author: Pawel
---

## Disable root ssh access

In file `/etc/ssh/sshd_config`

change

```
PermitRootLogin yes
```

to

```
PermitRootLogin no
```

## Disable password login via ssh

In file `/etc/ssh/sshd_config`

change

```
PasswordAuthentication yes
```

to

```
PasswordAuthentication no
```

## Block all but necessary ports to necessary machines via UFW

```
ufw allow 22
ufw allow 80
ufw allow 443
ufw enable
```

Only open up the necessary ports to the IPs that will be accessing it
```
sudo ufw allow from 192.168.1.1 to any port 5432
```

## Install OSSEC Intrusion Detection System 

### Installation

```
wget -q -O - https://updates.atomicorp.com/installers/atomic | sudo bash
sudo apt-get update
sudo apt-get install ossec-hids-server
```

### Send alert emails to devops@example.com

modify `/var/ossec/etc/ossec.conf`

```
<global>
  <email_notification>yes</email_notification>
  <email_to>__email__</email_to>
  <smtp_server>127.0.0.1</smtp_server>
  <email_from>ossecm@__server__</email_from>
</global>
```

## Install fail2ban

Set bantime to 1h and findtime to 30m

## Hide IPs using Cloudflare

Prevent the server IP from being publicly visible by using CloudFlare.

## Use a gateway server for multiple server configurations

The gateway server only has SSH port exposed. In order to SSH into the rest of the infrastructure you have to use the gateway server.

Example ssh config

```
Host gateway.fitnext.ovh
  User maverick
  IdentityFile ~/.ssh/identity/id_rsa
  Hostname 99.99.99.99

Host app1.example.com
  ProxyJump gateway.example.com
```

so in order to ssh into app1.example.com you go through the gateway server. This prevents exposure of the other servers to the internet.
