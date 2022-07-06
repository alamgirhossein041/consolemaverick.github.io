---
title: Moving PostgreSQL to an additional disk
layout: post
subtitle:
description: Moving PostgreSQL to an additional disk
image: /assets/img/uploads/floppy.jpg
date: '2020-07-01 13:28:05 +0900'
optimized_image: /assets/img/uploads/floppy.jpg
category: fundementals
tags:
  - postgresql
  - hardware
  - disk
author: Pawel
---

## How to add an additional disk to an instance

If it's a baremetal server you will have to order the disk and have it installed. If it's a machine provided by Amazon or Google you can provision the disk on the fly and add it to the machine.

Once the disk is added you can see it using 

```
sudo fdisk -l
```

Assuming the disk is `/dev/sdb` you first have to create a partition on the disk using

```
sudo fdisk /dev/sdb
```

*  n (create partition)
*  p (primary)
*  1 (partition number)
*  default (256 - first sector)
*  default (98303999 - last sector)
*  w (save and quit)

```
sudo mkfs.ext4 /dev/sdb1
```

create a place to mount the disk 

```
sudo mkdir /mnt/disk
```

mount the disk

```
sudo mount /dev/sdb1 /mnt/disk
```

make the disk come up when the machine is rebooted by modifying `/etc/fstab`

```
/dev/sdb1                /mnt/disk              ext4    defaults,nofail,x-systemd.device-timeout=1         0 0
```

## How to move PostgreSQL to the additional disk

stop postgresql

```
sudo monit stop postgresql
```

move the postgresql data directory to the mounted disk

```
mv /var/lib/postgresql /mnt/disk/postgresl
```

symlink the mounted directory back to it's original place

```
sudo ln -s /mnt/disk/postgresql /var/lib/postgresql
```

