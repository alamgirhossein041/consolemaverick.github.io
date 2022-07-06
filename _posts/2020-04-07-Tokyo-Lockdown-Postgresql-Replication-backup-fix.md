---
title: Tokyo is on Lockdown so I'm gonna teach you how to dump a replicated PostgreSQL Database
layout: post
date: '2020-04-07'
subtitle: I run in place on my balcony for exercise
description: "Is your PostgreSQL throwing up  ERROR:  canceling statement due to conflict with recovery?"
image: /assets/img/uploads/lockdown.jpg
optimized_image: /assets/img/uploads/lockdown.jpg
category: postgresql
tags:
  - postgresql
  - backup
author: Pawel
---

Do you have a replicated PostgreSQL instance being used solely for the purpose of backing up your main database server? Did you think all your problems would be solved with the additional server but when sitting down to have your whiskey and morning cigarette you discovered:

```
 ERROR:  canceling statement due to conflict with recovery
```

in your mailbox?

It's bullshit I know but I have a solution for you. You need to pause replication before you start the `pg_dump` 

```
/usr/bin/sudo -u postgres /usr/bin/psql -c 'select pg_wal_replay_pause();'
```

and resume it after

```
/usr/bin/sudo -u postgres /usr/bin/psql -c 'select pg_wal_replay_resume();'
```

A hypothetical backup script would look like this:

```
/usr/bin/sudo -u postgres /usr/bin/psql -c 'select pg_wal_replay_pause();'
/usr/bin/pg_dump -U $USER $DB > $FILE.sql
/usr/bin/sudo -u postgres /usr/bin/psql -c 'select pg_wal_replay_resume();'

# compress the file
/bin/tar -cvzf $FILE.tar.gz $FILE.sql

# upload to google cloud
/root/google-cloud-sdk/bin/gsutil cp $FILE.tar.gz $BUCKET

# remove temp file
rm -f $FILE.sql $FILE.tar.gz
```
