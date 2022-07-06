---
title: If you're not using this CLI tools you're losing money
layout: post
date: '2020-04-04'
subtitle: Did you decide what color Aventador you're going to buy?
description: These are the cream of the crop of CLI tools
image: /assets/img/uploads/console.jpg
optimized_image: /assets/img/uploads/console.jpg
category: cli
tags:
  - cli
  - tools
  - linux
author: Pawel
---

I don't know about you but I only became a console cowboy cause of Keanu Reeves.

I've hand picked these. I'm sure you will find something to your liking here. If it's not fast and pretty I don't use it.

## lnav

![lnav](/assets/img/uploads/lnav.png)

It's like tail but can handle multiple files on the same interface and jumping between errors.

Install via

```
sudo apt install lnav
```

[lnav](http://lnav.org/)

## sssh

Do you find yourself waiting for a machine to come up after you rebooted it? Does the machine not respond to ping? Why keep trying to SSH into the box when you can be checking your crypto portfolio instead.

Save this script as sssh and make it executable. It will continue attempting to connect to a server via ssh until it gets in. 

```
#!/bin/bash

if [ -z "$1" ]; then
  echo '' && echo 'Please also provide server name as in config file...' &&
  exit 1
fi

retries=0
repeat=true
today=$(date)

while "$repeat"; do
  ((retries+=1)) &&
  echo "Try number $retries..." &&
  today=$(date) &&
  ssh "$@" &&
  repeat=false
  if "$repeat"; then
    sleep 5
  fi
done

echo "Total number of tries: $retries"
echo "Last connection at: $today"
```

```
sssh 99.99.99.99
```

## icanhazip

![icanhazip](/assets/img/uploads/icanhazip.png)

Get the IP of the server you're on via curl.

## fd

![fd](/assets/img/uploads/fd.png)

It's like find except it's not shit. It's fast too.

Install via

```
sudo apt install fd-find
```

[fd](https://github.com/sharkdp/fd)

## ncdu

![ncdu](/assets/img/uploads/ncdu.png)

Is the client calling you cause the machine is hung cause there's no HD space left? Find out what's eating up all the space.

Install via

```
sudo add-apt-repository ppa:eugenesan/ppa
sudo apt install ncdu
```

[ncdu](https://dev.yorhel.nl/ncdu)

## htop

![htop](/assets/img/uploads/htop.png)

It's like top except it doesn't hurt the artist in you looking at it.

Install via

```
sudo apt install htop
```

[bat](https://hisham.hm/htop/)

## bat

![bat](/assets/img/uploads/bat.png)

It's an esthetically pleasing `cat`

Install via

```
sudo apt install bat
```

[bat](https://github.com/sharkdp/bat)

## fzf

![fzf](/assets/img/uploads/fzf.png)

fzf is a general-purpose command-line fuzzy finder. You can integrate it into your vim and shell. Use it to fuzzy find anything.

Install via

```
sudo apt install fzf
```

[fzf](https://github.com/junegunn/fzf)

## tig

![tig](/assets/img/uploads/tig.png)

Git repo browser. Merge, commit, browse. Very useful for figuring out who broke the build on production.

[tig](https://jonas.github.io/tig/)

## tree

![tree](/assets/img/uploads/tree.jpg)

Recursively `ls` the subdirectories

Install via

```
sudo apt install tree
```

## pgcli

![pgcli](/assets/img/uploads/pgcli.png)

PostgreSQL CLI with autocomplete. Aware of columns, tables and PostgreSQL commands.

Install via

```
sudo apt install libpq-dev python-dev python-pip
sudo pip install pgcli
```

[pgcli](https://www.pgcli.com/)
