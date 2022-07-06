---
title: Create a VPN on Google Cloud Platform via Ansible
layout: post
date: '2020-04-05'
subtitle: 
description: Automate creating your own VPN on Google Cloud. Change IPs as they get blocked.
image: /assets/img/uploads/stop.jpg
optimized_image: /assets/img/uploads/stop.jpg
category: vpn
tags:
  - firewall
  - vpn
  - google cloud
  - ansible
author: Pawel
---

Does your VPN keep getting blocked? Do you want to have your own VPN on which you can change the IP at anytime? Keep reading.

You will need to create an account on Google Cloud Platform. Create a project to house your VPN. From there you will need to allow API access and download `credentials.json` file.

It should look something like this

```
{
  "type": "service_account",
  "project_id": "<YOUR PROJECT ID>",
  "private_key_id": "<LONG ASS STRING>",
  "private_key": "-----BEGIN PRIVATE KEY-----....--------END PRIVATE KEY------",
  "client_email": "<GCP USER NAME>@<YOUR PROJECT ID>.iam.gserviceaccount.com",
  "client_id": "<YOUR CLIENT ID>",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://accounts.google.com/o/oauth2/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/<YOUR CLIENT ID><YOUR PROJECT ID>.iam.gserviceaccount.com"
}

```
I will automate the configuration using Ansible because doing things manually is for peasants. 

{% highlight ansible %}
{% raw %}
- hosts: localhost
  gather_facts: false
  become: false

  vars_prompt:
  - name: "vpn_user"
    prompt: "User Name"
    default: "vpnuser"
    private: false

  - name: "instance_disk_size"
    prompt: "Size of Disk (GB)"
    private: false
    default: 10

  - name: "instance_type"
    prompt: "Type of instance"
    default: "f1-micro"
    private: false

  - name: "instance_region"
    prompt: "Instance region"
    default: "asia-east1-a"
    private: false

  vars:
    service_account_email: "<YOUR GOOGLE ACCOUNT EMAIL>"
    credentials_file: "<LOCATION OF CREDENTIALS FILE>"
    project_id: "<YOUR GOOGLE PROJECT ID>"
    instance_name: "vpn-{{vpn_user}}"

  tasks:
    - set_fact: vpn_user="{{vpn_user}}"

    - name: create instance
      gce:
        instance_names: "{{ instance_name }}"
        zone: "{{ instance_region }}"
        machine_type: "{{ instance_type }}"
        image: debian-9
        state: present
        service_account_email: "{{ service_account_email }}"
        credentials_file: "credentials.json"
        project_id: "{{ project_id }}"
        metadata : '{ "startup-script" : "apt-get update" }'
      register: gce

    - name: Save host data
      add_host:
        hostname: "{{ item.public_ip }}"
        groupname: gce_instances_ips
      loop: "{{ gce.instance_data }}"

  post_tasks:
    - name: Wait for SSH for instances
      wait_for:
        delay: 1
        host: "{{ item.public_ip }}"
        port: 22
        state: started
        timeout: 30
      loop: "{{ gce.instance_data }}"

- hosts: gce_instances_ips
  become: true

  vars:
    vpn_psk: "{{ lookup('password', '~/.ansible/credentials/google_cloud/vpn_psk@' + ansible_host + ' length={{password_length}}') }}"
    vpn_password: "{{ lookup('password', '~/.ansible/credentials/google_cloud/' + hostvars['localhost']['vpn_user'] + '@' + ansible_host + ' length={{password_length}}') }}"
    root_password: "{{ lookup('password', '~/.ansible/credentials/google_cloud/root@' + ansible_host + ' length={{password_length}}') }}"

  tasks:
    - set_fact: vpn_user="{{ hostvars['localhost']['vpn_user'] }}"

    - name: copy vpnsetup.sh
      template: src=../../files/scripts/vpnsetup.sh dest=/root/vpnsetup.sh mode=0775

    - name: run vpnsetup.sh
      shell: /root/./vpnsetup.sh

    - name: change root password
      user: name=root update_password=always password={{ root_password | password_hash('sha512') }}

    - debug:
        msg: "Server Passwords root:{{root_password}}"

    - debug:
        msg: "VPN Server:{{ansible_host}} Username:{{vpn_user}} Password:{{vpn_password}} Secret:{{vpn_psk}} Type:Cisco IPSec"

{% endraw %}
{% endhighlight %}

Now comes the key piece, `vpnsetup.sh`.  This is the script that gets run on the server and it comes from [github](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/vpnsetup.sh)

Save as `plays/vpn.yml` and run via 

```
ansible-playbook -l <YOUR ANSIBLE GROUP NAME> plays/vpn.yml
```

When you run the script you will be asked to provide the username for the vpn, this is what you will use to login. You need to provide the region which defaults to Taiwan at this point. An instance size which defaults to the smallest one. If you don't know just accept the defaults but change the username.

At the end of all this you will see a debug message with the credentials

{% highlight ansible %}
{% raw %}
VPN Server:{{ansible_host}} Username:{{vpn_user}} Password:{{vpn_password}} Secret:{{vpn_psk}} Type:Cisco IPSec
{% endraw %}
{% endhighlight %}
