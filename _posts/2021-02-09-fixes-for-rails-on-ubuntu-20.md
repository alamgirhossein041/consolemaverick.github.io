---
title: "Fixes for Ruby on Rails on Ubuntu 20.04"
layout: post
subtitle: After you upgrade from Ubuntu 18.04 you will notice something's awry
description: "Learn how to fix everything you broke in your Ruby on Rails application after upgrading the OS"
image: /assets/img/uploads/upgrade.jpg
date: '2021-02-09 07:28:05 +0900'
optimized_image: /assets/img/uploads/upgrade.jpg
category: fundementals
tags:
  - linux
  - ubuntu
  - ror
  - ruby
  - rails
author: Pawel Stoklosa
---

After upgrading to Ubuntu 20.04 you may have noticed a few errors in your Ruby on Rails application. They aren't obvious. At first everything seems to be working fine. Your `monit` is reporting all services up but you're unable to connect to console. When you try to run it it complains about the lack of libreadline7.

You try to install `libreadline7` but it doesn't exist:

```
sudo apt-get install libreadline7-dev
```

You have to get the deb package itself 

```
wget http://mirrors.kernel.org/ubuntu/pool/main/r/readline/libreadline7_7.0-3_amd64.deb
```

and install it via `dpkg`

```
dpkg -i libreadline7_7.0-3_amd64.deb
```

Your `node` command has also disappeared which means that asset compilation will fail. To prevent this install `nodejs` via `apt install nodejs`.

If your application is relying on an older version of node you will have to install it via `n`.

```
apt install npm
npm install -g n
```

Then finally use `n` to install the desired `node` version.

```
n 8
```

This will install `node 8.17.0` for example.

If your application relies on `libssl1.0` you will notice it's not possible to install via `apt`. First you have to add a `ppa` repository:

```
sudo apt-add-repository -y ppa:rael-gc/rvm
```

Then you can install it via `apt`

```
sudo apt install libssl1.0-dev
```

If you get an issue about a missing `libffi.so.6` you can symlink the currently installed version 7 like so:

```
sudo ln -s /usr/lib/x86_64-linux-gnu/libffi.so.7.1.0 /usr/lib/x86_64-linux-gnu/libffi.so.6
```

Finally if your application relies on weak ciphers and cannot be upgraded at this time you may see this error

```
SSL_CTX_use_certificate: ca md too weak
```

in your logs.

In order to fix that modify `/etc/ssl/openssl.cnf` by prepending the following to the start:

```
openssl_conf = default_conf
```

and appending the following to the end of that file:

```
[ default_conf ]

ssl_conf = ssl_sect

[ssl_sect]

system_default = ssl_default_sect

[ssl_default_sect]
MinProtocol = TLSv1.2
CipherString = DEFAULT:@SECLEVEL=0
```

Finally restart your thin/puma/sidekiq/resque processes so they pick up the changes.
