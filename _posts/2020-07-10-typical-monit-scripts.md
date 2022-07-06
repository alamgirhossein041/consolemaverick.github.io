---
title: Typical Monit scripts
layout: post
subtitle: Copy and paste
description:
image: /assets/img/uploads/copy_paste.png
date: '2020-07-10 13:28:05 +0900'
optimized_image: /assets/img/uploads/copy_paste.png
category: fundementals
tags:
  - monit
author: Pawel
---

## Typical monit scripts

### MongoDB

```
check process mongodb matching "mongod"
   start program = "/bin/systemctl start mongodb"
   stop program = "/bin/systemctl stop mongodb"
   if 5 restarts within 5 cycles then timeout
```

### InfluxDB

```
check process influxdb with pidfile /var/run/influxdb/influxd.pid
    start program = "/bin/systemctl start influxdb"
    stop  program = "/bin/systemctl stop influxdb"
    if 5 restarts within 5 cycles then timeout
```

### MySQL

```
check process mysql with pidfile /var/run/mysqld/mysqld.pid
   start program = "/bin/systemctl start mysql"
   stop program = "/bin/systemctl stop mysql"
   if failed host 127.0.0.1 port 3306 protocol mysql then restart
   if 5 restarts within 5 cycles then timeout
```

### Nginx

```
check process nginx with pidfile /var/run/nginx.pid
  start program = "/bin/systemctl start nginx"
  stop program = "/bin/systemctl stop nginx"
  if failed host localhost port 80 protocol HTTP request "/nginx_status" with timeout 10 seconds 2 cycles then restart
  if 5 restarts within 5 cycles then timeout
```

### Redis

```
 check process redis with pidfile /var/run/redis/redis-server.pid
   start program = "/etc/init.d/redis-server start"
   stop program = "/etc/init.d/redis-server stop"
   if 5 restarts with 5 cycles then timeout
```

### PostgreSQL

```
check process postgresql with pidfile /var/run/postgresql/10-main.pid
    start program = "/bin/systemctl start postgresql@10-main"
    stop  program = "/bin/systemctl stop postgresql@10-main"
    if failed host localhost port 5432 type TCP 2 cycles then restart
    if failed host localhost port 5432 type TCP 2 cycles then alert
    if 5 restarts within 5 cycles then timeout
```

### ElasticSearch

```
check process elasticsearch with matching "elasticsearch"
  start program = "/bin/systemctl start elasticsearch"
  stop program = "/bin/systemctl stop elasticsearch"
  if 5 restarts within 5 cycles then timeout
```

### Kibana

```
check process kibana with matching "kibana"
  start program = "/bin/systemctl start kibana"
  stop program = "/bin/systemctl stop kibana"
  if 5 restarts within 5 cycles then timeout
```

### Rails Application using Puma

```
check process application_production_puma with pidfile /var/www/staging/application/shared/pids/puma.pid
    start program = "/var/www/staging/application/shared/scripts/puma.sh start" as uid "www-data" and gid "www-data"
    stop program  = "/var/www/staging/application/shared/scripts/puma.sh stop" as uid "www-data" and gid "www-data"
    if cpu > 80% for 5 cycles then restart
    if totalmem > 0.5 GB for 5 cycles then restart
    if 3 restarts within 5 cycles then timeout
    group application_production_ruby
    group application_production_puma
    group application_production
    group all_staging_ruby
```

### Rails Application using Thin

```
check process application_production_thin_1 with pidfile /var/www/production/application/shared/pids/thin.1.pid
  start program = "/var/www/production/application/shared/scripts/thin.sh start 1"
  stop program  = "/var/www/production/application/shared/scripts/thin.sh stop 1"
  if cpu > 60% for 2 cycles then alert
  if cpu > 80% for 5 cycles then restart
  if totalmem > 0.5 GB for 5 cycles then restart
  if 3 restarts within 5 cycles then timeout
  group application_production_ruby
  group application_production_thin
  group application_production
  group all_production_ruby

check process application_production_thin_2 with pidfile /var/www/production/application/shared/pids/thin.2.pid
  start program = "/var/www/production/application/shared/scripts/thin.sh start 2"
  stop program  = "/var/www/production/application/shared/scripts/thin.sh stop 2"
  if cpu > 60% for 2 cycles then alert
  if cpu > 80% for 5 cycles then restart
  if totalmem > 0.5 GB for 5 cycles then restart
  if 3 restarts within 5 cycles then timeout
  group application_production_ruby
  group application_production_thin
  group application_production
  group all_production_ruby
```

