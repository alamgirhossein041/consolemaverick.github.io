---
title: Get munin node slave to report to a munin master
layout: post
date: '2020-03-20 13:28:05 +0900'
subtitle: Escape the rate race, retire by 30.
description: Learn how to setup 2 servers communicating in a munin master/slave pattern.
image: /assets/img/uploads/munin.png
optimized_image: /assets/img/uploads/munin.png
category: devops
tags:
  - munin
  - monitoring
  - coronavirus
  - lockdown
author: Pawel
---

Munin is old school but it works so I keep installing it. Or maybe I'm old and I think of Prometheus the same way old people think of rap music 🙊  Regardless let's install munin in a master / slave configuration.

Some assumptions:
* The master server will have IP 99.123.123.123
* The slave server will have IP 88.123.123.123
* Master server has UFW firewall installed
* you can type at least 45 words a minute

## Slave

```
sudo apt install munin-node
```

Open up the firewall so the master can get in

```
sudo ufw allow from 99.123.123.123 to any port 4949
sudo ufw allow from 99.123.123.123 to any port 4948
```

Setup your plugins. Let me show you with nginx

```
sudo apt install libwww-perl
sudo ln -s /usr/share/munin/plugins/nginx_request /etc/munin/plugins/nginx_request
sudo ln -s /usr/share/munin/plugins/nginx_status /etc/munin/plugins/nginx_status
```

In order to get this to work nginx iteslf must be configured to respond on localhost to `nginx_request` and `nginx_status` via a block like so in `/etc/nginx/sites-enabled/monit`

```
server {
  listen   80;
  ...
    location /nginx_status {
    stub_status on;
    access_log   off;
    allow 127.0.0.1;
    deny all;
  }
}
```

Here is another popular plugin config: PostgreSQL

```
sudo ln -s /usr/share/munin/plugins/postgres_bgwriter /etc/munin/plugins/postgres_bgwriter
sudo ln -s /usr/share/munin/plugins/postgres_cache_ /etc/munin/plugins/postgres_cache_ALL
sudo ln -s /usr/share/munin/plugins/postgres_checkpoints /etc/munin/plugins/postgres_connections_ALL
sudo ln -s /usr/share/munin/plugins/postgres_connections_db /etc/munin/plugins/postgres_connections_db
sudo ln -s /usr/share/munin/plugins/postgres_locks_ /etc/munin/plugins/postgres_locks_ALL
sudo ln -s /usr/share/munin/plugins/postgres_querylength_ /etc/munin/plugins/postgres_querylength_ALL
sudo ln -s /usr/share/munin/plugins/postgres_size_ /etc/munin/plugins/postgres_size_ALL
sudo ln -s /usr/share/munin/plugins/postgres_transactions_ /etc/munin/plugins/postgres_transactions_ALL
sudo ln -s /usr/share/munin/plugins/postgres_users /etc/munin/plugins/postgres_users
sudo ln -s /usr/share/munin/plugins/postgres_xlog /etc/munin/plugins/postgres_xlog
```

Restart `munin-node`

`sudo systemctl restart munin-node`

## Master

Install munin

```
sudo apt install munin
```

Notify munin master of the new node

```
[slave;slave.com]
  address 88.123.123.123
  use_node_name yes
```

Restart munin

`sudo systemctl restart munin`

Check that the link to the slave is working:

`telnet 88.123.123.123 4949`

You can even verify that munin is responding back through telnet i.e.

`fetch df`

should give you back disk information. And then you wait because waiting for munin to do it's thing is like waiting at the DMV.

You probably want to force an update cause time is money

`su - munin --shell=/bin/bash -c "/usr/share/munin/munin-update -debug"`
