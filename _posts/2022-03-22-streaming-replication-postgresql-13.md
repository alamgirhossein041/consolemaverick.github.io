---
title: "How to setup Streaming Replication in PostgreSQL 13"
layout: post
subtitle: "What has changed in 13"
description: "Learn how to setup Streaming Replication in PostgreSQL 13"
image: /assets/img/uploads/streaming.jpg
date: '2022-03-22 06:28:05 +0900'
optimized_image: /assets/img/uploads/streaming.jpg
category: fundementals
tags:
  - postgresql
  - replication
author: Pawel Stoklosa
---

Things have changed in PostgreSQL 13\. If you’re used to replication PostgreSQL instances prior to 13 you will see that the setup is cleaner now. It’s also easier to start the replication process by creating the base backup. Let’s go through the important bits:

On the primary server you do nothing outside of making it accesible from the secondary server. In order to do that you will have to change the listen\_address

```mysql
listen_addresses = '*'
```

Then on the secondary server’s

```mysql
postgresql.conf
```

file add information about the primary:

```mysql
primary_conninfo = 'host=22.80.27.199 port=5432'
```

by specifying the IP address and the port.

If you’re running a firewall such as UFW on the primary you will have to ensure you can connect to the primary from the secondary.

You can verify the connection using telnet

```mysql
telnet 22.80.27.199 5432
```

If your firewall is blocking access then open it up to the secondary by running the following on the primary

```mysql
sudo ufw allow from 22.80.27.198 to any port 5432
```

If everything connects then you can create the base backup which will prime the secondary for replication

```mysql
su postgres
```

As the postgres user execute

```mysql
touch /var/lib/postgresql/13/main/standby.signal
```

to signal for this machine to be the standby and then

```mysql
pg_basebackup -h 22.80.27.199 -U postgres -p 5432 -D $PGDATA -Fp -Xs -P -R
```

to start the base backup. And that’s it. You should now be able to verify the lag between the primary and the secondary via psql console

```mysql
su postgres
psql
select now()-pg_last_xact_replay_timestamp() as replication_lag;
```
