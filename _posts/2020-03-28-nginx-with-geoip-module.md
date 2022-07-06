---
title: How to compile and configure Nginx to use Geoip module
layout: post
date: '2020-03-28 13:28:05 +0900'
subtitle: Be a minimalist, you only wear 5 outfits anyway.
description: Geoip module lets you identify the location of the visitor using the IP address. I will show you how to integrate that into Nginx.
image: /assets/img/uploads/nginx.jpg
optimized_image: /assets/img/uploads/nginx.jpg
category: devops
tags:
  - nginx
  - geoip
  - geolocation
author: Pawel
---

If you want to know where your users are coming from based on their IP you need Geoip compiled into Nginx. The apt version of Nginx won't help you here you have to compile from source.

In order to do that we grab Nginx source

```
cd /usr/local/src
sudo add-apt-repository ppa:maxmind/ppa
sudo apt-get update
sudo apt install libmaxminddb0 libmaxminddb-dev mmdb-bin
wget https://nginx.org/download/nginx-1.13.12.tar.gz
tar -zxvf nginx-1.13.12.tar.gz
cd
git clone https://github.com/leev/ngx_http_geoip2_module.git --depth=1
```

Build the prerequisites

```
cd ~/nginx-1.13.12
apt build-dep nginx
```

Since you probably already have Nginx installed via apt you can use it's configuration settings as a baseline for the new Nginx by running `nginx -V`

You want to append `--add-dynamic-module=/usr/local/src/ngx_http_geoip2_module` so you get:

```
./configure --sbin-path=/usr/sbin/nginx --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_gzip_static_module --add-dynamic-module=/root/ngx_http_geoip2_module --with-openssl=/usr/local/src/openssl
```

Then you compile Nginx using the above flags

```
sudo make
sudo make install
```

Next you ned the GeoLite database which has the information to determine what IP comes from where

```
mkdir /etc/geolite
cd /etc/geolite
wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz
wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.mmdb.gz
gunzip GeoLite2-City.mmdb.gz
gunzip GeoLite2-Country.mmdb.gz
```

Reconfigure Nginx to use GeoLite databases

```
vim /etc/nginx/nginx.conf
```

```
load_module modules/ngx_http_geoip2_module.so; 

http {
...
  geoip2 /etc/geolite/GeoLite2-Country.mmdb {
    $geoip2_data_country_code default=NA country iso_code;
    $geoip2_data_country_name default=NA country names en;
    $geoip2_data_city_name default=NA city names en;
    $geoip2_data_postal_code default=NA postal code;
    $geoip2_data_latitude default=NA location latitude;
    $geoip2_data_longitude default=NA location longitude;
  }

  geoip2 /etc/geolite/GeoLite2-City.mmdb {
    $geoip2_data_city_name default=NA city names en;
    $geoip2_data_state_name default=NA subdivisions 0 names en;
    $geoip2_data_state_code default=NA subdivisions 0 iso_code;
  }
...
}
```

And finally configure your application to expose the variables in the header

```
location / {
  ...
  proxy_set_header GEOIP_COUNTRY_CODE $geoip2_data_country_code;
  proxy_set_header GEOIP_COUNTRY_NAME $geoip2_data_country_name;
  proxy_set_header GEOIP_CITY_NAME $geoip2_data_city_name;
  proxy_set_header GEOIP_POSTAL_CODE $geoip2_data_postal_code;
  proxy_set_header GEOIP_DATA_LATITUTDE $geoip2_data_latitude;
  proxy_set_header GEOIP_DATA_LONGITUDE $geoip2_data_longitude;
  proxy_set_header GEOIP_STATE_NAME $geoip2_data_state_name;
  proxy_set_header GEOIP_STATE_CODE $geoip2_data_state_code;
  ...
}
```

