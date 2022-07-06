---
title: PostgreSQL Database Replication
layout: post
date: '2020-03-29'
subtitle: Be a digital nomad, live out of a backpack
description: Learn to use PostgreSQL Database replication between your master and slave servers
image: /assets/img/uploads/elephant.jpg
optimized_image: /assets/img/uploads/elephant.jpg
category: postgresql
tags:
  - postgresql
  - database
  - replication
author: Pawel
---

You created a project and it's marginally successful. Now you have so much traffic that your database can't handle the load. You need to scale. 

One way out of this mess is to divide up the work. Put the database server on it's own instance. Put the background jobs server on another instance. Put the website on yet another. But how do you make them all share the same database? Making reads remotely is too slow, you need a copy of the database locally for reads and you can afford to do writes remotely. That's where replication comes in handy. You have a single master database on it's own instance and the slaves mirror the database. The processes on each slave use their own copy of the database and they all write back to the master.

But how do you actually do that in PostgreSQL?

Easy, follow me:

Some assumptions
* Your postgresql version is 11, replace this with your version
* Your slave IP is 54.123.123.123, replace this with your slave IP
* Your master IP is 94.123.123.123, replace this with your master IP
* Your database name is production_database, replace this with the name of the database you want to replicate
* Your database is running on port 5432, replace with your own port number

## Master

On the Master server you edit `/etc/postgresql/11/main/pg_hba.conf`

```
hostssl replication           postgres              54.123.123.123/32     trust
hostssl production_database   production_database   54.123.123.123/32     md5
```

This will allow your slave to connect to the PostgreSQL instance on your master.

Now create the SSL certificate because you will be replicating over SSL.

```
cd /var/lib/postgresql/11/main
openssl req -new -text -days 36500 -out server.req openssl rsa -in privkey.pem -days 36500 -out server.key
rm privkey.pem 
openssl req -x509 -in server.req -text -key server.key -days 36500 -out server.crt
chmod og-rwx server.key
```

You can verify the end date of the certificate like so `openssl x509 -in server.crt -noout -enddate`

Setup PostgreSQL on master for replication by editing `/etc/postgresql/11/main/postgresql.conf`

```
listen = ‘*'
ssl = true
wal_level = replica 
fsync = off # turns forced synchronization on or off
synchronous_commit = off
max_wal_senders = 12 # max number of walsender processes
wal_keep_segments = 32 # in logfile segments, 16MB each; 0 disables
log_min_duration_statement = 75 # -1 is disabled, 0 logs all statements
```

## Slave

Create `/var/lib/postgresql/11/main/recovery.conf` file and add the following

```
standby_mode = on
primary_conninfo = 'host=94.123.123.123 port=5432 sslmode=verify-ca’
```

Set PostgreSQL to replication slave mode by modifying `/etc/postgresql/11/main/postgresql.conf`

```
hot_standby = on
```

copy the end of server.crt from master to (the part that starts with BEGINNING CERTIFICATE…) and paste it into `/var/lib/postgresql/.postgresql/root.crt`

### Allow postgres user to connect to master without a password

```
sudo su
su postgres
cd ~
ssh-keygen
```

Copy this id_rsa.pub over to `/var/lib/postgresql/.ssh/authorized_keys` on the master server.

Try to ssh into the master server and accept the key

```
sudo su
su postgres
ssh 54.123.123.123
```

Execute `boot_slave.sh 94.123.123.123` script on the slave to bootstrap replication

```
#!/bin/bash
psql_version=11
psql_port=5432

datadir=/var/lib/postgresql/$psql_version/main
archivedir=/var/lib/postgresql/$psql_version/archive
archivedirdest=/var/lib/postgresql/$psql_version/archive

#Usage
if [ "$1" = "" ] || [ "$1" = "-h" ] || [ "$1" = "-help" ] || [ "$1" = "--help" ] ;
then
    echo "Usage: $0 masters ip address"
    echo
exit 0
fi

PrepareLocalServer () {

if [[ -f "/tmp/trigger_file" ]]
then
    rm /tmp/trigger_file
fi

echo "Stopping local postgresql.."
/bin/systemctl stop postgresql

if [[ -f "$datadir/recovery.done" ]];
then
    mv "$datadir"/recovery.done "$datadir"/recovery.conf
fi
}

CheckForRecoveryConfig () {
if [[ -f "$datadir/recovery.conf" ]];
    then
    echo "Slave Config File Found, Continuing"
    else
    echo "Recovery.conf not found Postgres Cannot Become a Slave, Exiting"
    exit 1
fi
}

#put master into backup mode
PutMasterIntoBackupMode () {
ssh -i /var/lib/postgresql/.ssh/id_rsa postgres@"$1" "/usr/lib/postgresql/$psql_version/bin/psql -p $psql_port -c \"SELECT pg_start_backup('Streaming Replication', true)\" postgres"
}

#rsync masters data to local postgres dir
RsyncWhileLive () {
rsync -C -av --delete --progress -e 'ssh -i /var/lib/postgresql/.ssh/id_rsa' --exclude recovery.conf --exclude recovery.done --exclude postmaster.pid  --exclude pg_xlog/ postgres@"$1":"$datadir"/ "$datadir"/
}

#this archives the the WAL log (ends writing to it and moves it to the $archive dir
StopBackupModeAndArchiveIntoWallLog () {
ssh -i /var/lib/postgresql/.ssh/id_rsa postgres@"$1" "/usr/lib/postgresql/$psql_version/bin/psql -p $psql_port -c \"SELECT pg_stop_backup()\" postgres"
}

#Execute above operations
PrepareLocalServer "$datadir"
CheckForRecoveryConfig "$datadir"
PutMasterIntoBackupMode "$1"
RsyncWhileLive "$1"
StopBackupModeAndArchiveIntoWallLog "$1" "$archivedir" "$archivedirdest"

echo 'Start postgresql'
/bin/systemctl start postgresql
```

## Verify it's working

Execute these 2 commands and compare the results

### Master

```
SELECT pg_current_wal_lsn();
```

### Slave

```
SELECT pg_last_wal_receive_lsn();
```
