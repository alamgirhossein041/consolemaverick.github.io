---
title: Are your cron jobs blowing up your email?
layout: post
date: '2020-04-07'
subtitle: Do you get too many pages? Why do you still have a pager?
description: "Create silent cronjobs that only bother you when the shit hits the fan"
image: /assets/img/uploads/pager.jpg
optimized_image: /assets/img/uploads/pager.jpg
category: devops
tags:
  - cronjob
author: Pawel
---

If your email is filled with noise you have to filter for the signal manually. Why not quiet down your cronjobs so they only notify you when they fail instead of acting like the teacher's pet.

Depending on the kind of cron we're talking about there are multiple approaches. Here are some examples:

## Certbot

```
/usr/bin/certbot renew -q --post-hook "sudo service nginx reload"
```

use the `-q` option to quiet down the output

## Backup gem

```
source /usr/local/rvm/scripts/rvm;rvm ruby-2.3.3@<APP>_production_backup_gem &> /dev/null;backup perform -qt <APP>_production_psql_backup,<APP>_production_directory_backup --tmp_path /mnt/disk1 -c /var/www/production/<APP>/shared/config/backup_config.rb
```

use the `-q` option along with appending `&> /dev/null` to the `rvm` call

## Others

append

```
&> /dev/null
```

to the end in order to silence regular output
