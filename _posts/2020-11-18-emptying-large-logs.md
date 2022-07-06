---
title: Emptying Large Logs
layout: post
subtitle: Don't just rm them
description: Otherwise you'll be in a world of hurt
image: /assets/img/uploads/large_files.jpg
date: '2020-11-18 13:28:05 +0900'
optimized_image: /assets/img/uploads/large_files.jpg
category: fundementals
tags:
  - linux
author: Pawel Stoklosa
---

Say you're on your Ubuntu instance and you do

```
df -h
```

and it tells you you're all out space cause some processes developed a gigantic log in

```
/var/log/some_process.log
```

Don't

```
rm /var/log/some_process.log
```

Do a

```
echo '' > /var/log/some_process.log
```

instead. 

This helps you in 3 ways.

First the file never actually disappears so you won't get 'open file deletion' problem. 

Second if you were to instead delete the file the space won't be freed up because the memory continues to be allocated to the process that's writing to the file. If you examine the total disk space using:

```
df -h
```

it will look like you didn't delete anything. Whereas checking with

```
du -h
```

will reveal it's actually gone. 

Thirdly the process that expects the file to be there won't have to be restarted 🙃
