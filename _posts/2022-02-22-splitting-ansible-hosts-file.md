---
title: "Splitting your Ansible Hosts file"
layout: post
subtitle: "Split hosts file by client"
description: "Learn how to split your Ansible hosts file by client instead of a single monolithic file."
image: /assets/img/uploads/ansible.png
date: '2022-02-22 07:28:05 +0900'
optimized_image: /assets/img/uploads/ansible.png
category: fundementals
tags:
  - ansible
author: Pawel Stoklosa
---

Instead of having a single monolithic ansible hosts file you can split them up by client.

Previously you would have:

```mysql
ansible/hosts
ansible/host_vars
ansible/host_vars/client_1.com.yml
ansible/host_vars/client_2.com.yml
ansible/group_vars
ansible/group_vars/client_1_production.yml
ansible/group_vars/client_1_staging.yml
ansible/group_vars/client_2_production.yml
```

Where the hosts file would contain

```mysql
[client_1_production]
client_1.com

[client_1_staging]
staging.client_1.com

[client_2_production]
client_2.com
```

Which gets difficult to manage when you have many clients and their infrastructure starts to bloat. A nice way to handle this is to split the ansible hosts using an inventories directory.

This allows you to create a directory structure like this:

```mysql
ansible/inventories/client_1
ansible/inventories/client_1/hosts.ini
ansible/inventories/client_1/group_vars/client_1_production
ansible/inventories/client_1/group_vars/client_1_staging
ansible/inventories/client_1/host_vars/client_1_com.yml

ansible/inventories/client_2
ansible/inventories/client_2/hosts.ini
ansible/inventories/client_2/group_vars/client_1_production
ansible/inventories/client_2/host_vars/client_2_com.yml
```

With the hosts files split

```mysql
> ansible/inventories/client_1/hosts.ini
[client_1_production]
client_1.com

[client_1_staging]
staging.client_1.com
```

```mysql
> ansible/inventories/client_2/hosts.ini
[client_2_production]
client_2.com
```
