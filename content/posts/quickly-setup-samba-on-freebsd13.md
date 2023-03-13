---
title: "Quickly Setup Samba on FreeBSD 13"
date: 2023-03-13
tags:
- FreeBSD
---

Some notes how I've quickly set up Samba share for my macOS machines in
the local network.

## Installation

```shell
# pkg install samba413
```

## Samba Configuration

```shell
# vim /usr/local/etc/smb4.conf
```

```
[global]
workgroup = WORKGROUP
server string = FreeBSD Server Samba
netbios name = freebsd-desktop
wins support = yes
security = user
passdb backend = tdbsam


[archive]
path = /backuppool/archive
valid users = gogo
public = no
writable  = yes
browsable = yes
read only = no
guest ok = no
create mask = 0755
directory mask = 0755
```

```shell
pdbedit -a -u gogo
new password:
retype new password:
```

## Enable Samba Server 

```shell
sysrc samba_server_enable="YES"
```

```shell
service samba_server start
```
