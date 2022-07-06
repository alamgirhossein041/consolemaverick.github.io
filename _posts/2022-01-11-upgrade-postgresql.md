---
title: "3 ways to Upgrade your PostgreSQL Database Instance"
layout: post
subtitle: "One of these is bound to work"
description: "Learn different ways in which you can upgrade PostgreSQL"
image: /assets/img/uploads/postgresql.jpg
date: '2021-12-28 07:28:05 +0900'
optimized_image: /assets/img/uploads/postgresql.jpg
category: fundementals
tags:
  - linux
  - postgresql
  - database
author: Pawel Stoklosa
---

Are you still running applications on PostgreSQL 10? I know me too. All the kids on TikTok are using PostgreSQL 13 now so get with the times dad.

Here are 3 ways you can do it:

1\. Using pg\_upradecluster
===========================

After you install the new version with

```mysql
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo add-apt-repository "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main"
```

```mysql
apt install postgresql-13
```

You will notice you now have 2 clusters: 10 and 13\. If you try to directly upgrade using

```mysql
pg_upgradecluster
```

it will complain that the cluster already exists so first you have to blow away your new cluster

```mysql
pg_dropcluster 13 main --stop
```

Now make sure your applications are stopped so they are not creating new content in the database. At this point you can upgrade the cluster in place with

```mysql
pg_upgradecluster 10 main /var/lib/postgresql/13/main
```

This method is the simplest and the one I recommend.

2\. Using pg\_dumpall
=====================

If for some reason the above is unsuitable for you you can use this method. The benefit of this method is you can keep your new cluster so if there’s some settings you only want to appear in this cluster you can set that up ahead of the dump. You can also use customized ports and exclude certain databases from the migration process.

Assuming your existing cluster is running on port 5432 and your newly created v 13 cluster is running on port 5433 which is the default on a new install of multiple PostgreSQL versions you will pipe the output of the pg\_dumpall command from port 5432 to the new instance on port 5433.

```mysql
su postgres
pg_dumpall -p 5432 | psql -d postgres -p 5433
```

3\. Using pg\_upgrade
=====================

This is the approch you use if all the other approaches didn’t work. The benefit of this approach is it lets you upgrade instances that were highly customized. If for example your existing instance is in some exotic location on the server you can overwrite that for each component:

1\. data directory 

2\. bin directory 

3\. configuration file

```mysql
su postgres
/usr/lib/postgresql/13/bin/pg_upgrade \
  --old-datadir=/var/lib/postgresql/10/main \
  --new-datadir=/var/lib/postgresql/13/main \
  --old-bindir=/usr/lib/postgresql/10/bin \
  --new-bindir=/usr/lib/postgresql/13/bin \
  --old-options '-c config_file=/etc/postgresql/10/main/postgresql.conf' \
  --new-options '-c config_file=/etc/postgresql/13/main/postgresql.conf' \
  --socketdir=/var/run/postgresql --link --check
```

the —check ensures this is just a test. Once you’re committed you can remove it and it will execute on the actual data. Since this approach is more manual you will have to edit the settings file

```mysql
/etc/postgresql/13/main/postgresql.conf
```

Changing the port of the new instance to be the default one

```mysql
port = 5432
```

Then restarting the new instance using systemctl

```mysql
systemctl start postgresql@13-main
```

If you are using monit on the machines it’s start/stop commands will have to be modified to specify the new cluster version otherwise it will continue to use your old version.

You can then verify and delete the cluster using built in scripts.

```mysql
./analyze_new_cluster.sh
./delete_old_cluster.sh
```

You can also trash the old instance manually with

```mysql
pg_dropcluster 10 main --stop
```

And remove the packages

```mysql
dpkg -l | grep postgresql | grep 10 | awk '{print $2}' | xargs apt-get -y remove
```
