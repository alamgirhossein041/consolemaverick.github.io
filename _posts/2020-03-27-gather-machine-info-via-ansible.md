---
title: Gather machine info via Ansible
layout: post
date: '2020-03-27'
subtitle: Buy an orange Lambo and live on the beach
description: Learn to use ansible to gather information about your infrastructure so you can sleep at night.
image: /assets/img/uploads/automation.jpg
optimized_image: /assets/img/uploads/automation.jpg
category: ansible
tags:
  - ansible
  - cpu
  - memory
  - disk
author: Pawel
---

I often find myself unable to sleep at night when I do not have a clearly defined server infrastucture diagram. I need stats, I need to know the CPU. I need to know about the disk and most importantly I need to know about the memory.

Your probably feel the same way and that's why you have to use Ansible to gather information about all your machines. You're not gonna gonna do that by hand are you? That's silly. You need to automate it so you can spend more time trying to figure out what Nietzsche was talking about and less time sshing into servers.

Today I will teach you how to do that. This recipe uses no additional tools, there is nothing to install on your servers.

```
- hosts: all
  become: true

  tasks:
  - name: capture CPU info
    shell: cat /proc/cpuinfo | grep "model name" | uniq
    register: cpu_1

  - name: capture more CPU info
    shell: lscpu | grep -E '^Thread|^Core|^Socket|^CPU\('
    register: cpu_2

  - name: capture MEM info
    shell: free -g
    register: mem

  - name: disk
    shell: df -h
    register: disk

  - debug:
      msg:
        - "{{ cpu_1.stdout_lines }}"
        - "{{ cpu_2.stdout_lines }}"
        - "{{ mem.stdout_lines }}"
        - "{{ disk.stdout_lines }}"
```

The sample output looks something like this:

```
ok: [my.super.awesome.server.com] => {
    "msg": [
        [
            "model name\t: Intel(R) Xeon(R) Silver 4214 CPU @ 2.20GHz"
        ],
        [
            "CPU(s):                4",
            "Thread(s) per core:    1",
            "Core(s) per socket:    1",
            "Socket(s):             4"
        ],
        [
            "              total        used        free      shared  buff/cache   available",
            "Mem:              5           0           4           0           1           5",
            "Swap:             3           0           3"
        ],
        [
            "Filesystem      Size  Used Avail Use% Mounted on",
            "udev            3.0G     0  3.0G   0% /dev",
            "tmpfs           598M   68M  530M  12% /run",
            "/dev/sda1        46G  1.7G   42G   4% /",
            "tmpfs           3.0G     0  3.0G   0% /dev/shm",
            "tmpfs           5.0M     0  5.0M   0% /run/lock",
            "tmpfs           3.0G     0  3.0G   0% /sys/fs/cgroup",
            "tmpfs           598M     0  598M   0% /run/user/1000"
        ]
    ]
}
```
