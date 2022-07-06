---
title: Typical NGINX scripts
layout: post
subtitle: Copy and paste
description:
image: /assets/img/uploads/server.jpg
date: '2020-07-20 13:28:05 +0900'
optimized_image: /assets/img/uploads/server.jpg
category: fundementals
tags:
  - monit
author: Pawel
---

## Sample NGINX configurations

### Munin

```
server {
  listen   80;
  server_name  munin.{{website_url}};

  access_log  /var/log/nginx/munin.access.log;
  error_log  /var/log/nginx/munin.error.log;

  location /robots.txt { alias /etc/nginx/robots.txt; }

  location ^~ /.well-known/acme-challenge/ {
    root   /var/www/services/munin;
    allow all;
    auth_basic off;
  }

  location ^~ /munin-cgi/munin-cgi-graph/ {
    access_log off;
    fastcgi_split_path_info ^(/munin-cgi/munin-cgi-graph)(.*);
    fastcgi_param PATH_INFO $fastcgi_path_info;
    fastcgi_pass unix:/var/run/munin/fcgi-graph.sock;
    include fastcgi_params;
  }

  location / {
    root   /var/www/services/munin;
    index  index.html index.htm;
    auth_basic "Restricted";
    auth_basic_user_file /etc/nginx/auth/munin.htpasswd;
  }
}
```

### Monit

```
server {
  listen   80;
  server_name  monit.{{website_url}} localhost 127.0.0.1;

  access_log  /var/log/nginx/monit.access.log;
  error_log  /var/log/nginx/monit.error.log;

  location /robots.txt { alias /etc/nginx/robots.txt; }

  location ~ /.well-known/acme-challenge {
    root /var/www/services/monit;
    auth_basic off;
    allow all;
  }

  location / {
    proxy_pass   http://127.0.0.1:2812;
    auth_basic "Restricted";
    auth_basic_user_file /etc/nginx/auth/monit.htpasswd;
  }

  location /nginx_status {
    stub_status on;
    access_log   off;
    allow 127.0.0.1;
    deny all;
  }
}
```

### Grafana

```
server {
  server_name grafana.example.com;
  listen [::]:80;

  location ~ /.well-known/acme-challenge {
    root /var/www/services/grafana;
    allow all;
  }
  location / {
    rewrite ^(.*) https://grafana.example.com$1 permanent;
  }
}

server {
  listen 443 ssl;
  listen [::]:443 ssl;
  server_name grafana.example.com;
  ssl_certificate /etc/letsencrypt/live/grafana.example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/grafana.example.com/privkey.pem;

  location / {
    #auth_basic "Restricted";
    #auth_basic_user_file /etc/nginx/admin.htpasswd;
    proxy_pass http://localhost:3000/;
  }
}
```

