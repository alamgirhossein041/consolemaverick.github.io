---
title: "Logical Replication in PostgreSQL"
layout: post
subtitle: "When your PostgreSQL server versions differ"
description: "Learn how to setup Logical Replication in PostgreSQL"
image: /assets/img/uploads/replication.jpg
date: '2022-03-08 07:28:05 +0900'
optimized_image: /assets/img/uploads/replication.jpg
category: fundementals
tags:
  - postgresql
author: Pawel Stoklosa
---

If you’re running a mix of different PosgreSQL versions then logical replication is the way to go. It allows you to run a publisher that V13 and a subscriber that’s V12 for example. This is useful if you’re upgrading the Publisher. If you have multiple subscribers there is a point in time where the version will be out of sync. In order to keep replication going during this process use logical replication.

Here is how to setup logical replication.

On the Publisher modify

```mysql
postgresql.conf
```

file setting the

```mysql
wal_level=logical
```

Then in pg\_hba.conf file add the IP of the subscriber:

```mysql
host    db_name         db_name         10.128.0.199/32           md5
host    all             postgres        10.128.0.199/32           tust
```

Ensure the subscriber has access to the PostgreSQL instance through the UFW firewall.

```mysql
sudo ufw allow from 10.128.0.199 to any port 5432
```

Then create the publication for all tables:

```mysql
su postgres
psql db_name;
create publication db_name_publication for all tables;
```

On each of the subscribers modify

```mysql
postgresql.conf
```

file setting

```mysql
wal_level=logical
```

Then extract the globals and the schema fom the publisher

```mysql
pg_dumpall -h 10.142.0.199 --globals-only | psql
pg_dump -h 10.142.0.199 --schema-only db_name | psql db_name
```

Since this is 2 way communication the Publisher needs to be able to connect to the Subscriber as well so open up the firewall.

```mysql
sudo ufw allow from 10.142.0.199 to any port 5432
```

and the pg\_hba.conf file

```mysql
host    db_name      db_name      10.142.0.199/32           md5
```

Finally create the subscription

```mysql
CREATE SUBSCRIPTION db_name_subscription CONNECTION 'port=5432 user=db_name password=db_password host=10.142.0.199' PUBLICATION db_name_publication;
```

Measure Lag
===========

After subscription is created you can measure the lag on the **Publisher**

```mysql
SELECT 
    slot_name,
    confirmed_flush_lsn, 
    pg_current_wal_lsn(), 
    (pg_current_wal_lsn() - confirmed_flush_lsn) AS lsn_distance
FROM pg_replication_slots;
```

```mysql
                 slot_name                  | confirmed_flush_lsn | pg_current_wal_lsn | lsn_distance
--------------------------------------------+---------------------+--------------------+--------------
 db_name_subscription                       | 21/B7FDA5C0         | 21/B7FDA5C0        |            0
 db_name_subscription_192578_sync_192140    | 21/B7F7ED50         | 21/B7FDA5C0        |       374896
 db_name_subscription_192578_sync_192132    | 21/B7FC2930         | 21/B7FDA5C0        |        97424
(3 rows)
```

Debugging
=========

If you need to debug you may want to drop the subscription or publication which you can do with these commands:

```mysql
drop subscription db_name_subscription;
drop publication db_name_publication;
```

Modifying Schema
================

If you modify the schema on the Publisher you will have to make the same change on the Subscriber before it can replication. For example if I add a new table called test:

```mysql
create table test(id serial, name varchar(25));
insert into test(name) values('test');
```

I have to recreate the same table on the Subscriber

```mysql
create table test(id serial, name varchar(25));
```

and refresh the subscription.

```mysql
alter subscription db_name_subscription refresh;
```
