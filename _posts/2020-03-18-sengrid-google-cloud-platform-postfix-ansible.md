---
title: Integrate Postfix with SendGrid on Google Cloud Platform via Ansible
layout: post
date: '2020-03-18 13:28:05 +0900'
subtitle: Automate your life, buy time. Be filthy rich.
description: Learn to use an Ansible recipe to fix email sending via Postfix on a fresh Google Cloud Platform instance installation.
image: /assets/img/uploads/notime.jpg
optimized_image: /assets/img/uploads/notime.jpg
category: ansible
tags:
  - ansible
  - gcp
  - sendgrid
  - postfix
author: Pawel
---

Google Cloud Platform is pretty swanky but it has one drawback. Email port 25 is blocked by default hence your
postfix/sendmail is broken out of the box. In order to fix this across a large number of servers you will need Dalai Lama level of patience which I don't have. Here I teach you how to automate that using ansible.

Normally you would want to:

Install `libsasl2-modules` via 

{% highlight shell %}
apt-get install libsasl2-modules
{% endhighlight %}

Enter this into `/etc/postfix/main.cf`

{% highlight shell %}
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
smtp_tls_security_level = encrypt
header_size_limit = 4096000
relayhost = [smtp.sendgrid.net]:2525
{% endhighlight %}

create a `/etc/postfix/sasl_passwd` file with


{% highlight shell %}
[smtp.sendgrid.net]:2525 username:password
{% endhighlight %}

Change the permissions

{% highlight shell %}
sudo chmod 600 /etc/postfix/sasl_passwd
{% endhighlight %}

Encrypt the password file

{% highlight shell %}
sudo postmap /etc/postfix/sasl_passwd
{% endhighlight %}

and finally restart everything

{% highlight shell %}
sudo systemctl restart postfix
{% endhighlight %}

That's all great and everything except 

![no-time](/assets/img/uploads/notime.jpg)

That's where ansible comes in

{% highlight ansible %}
- hosts: all
  become: true

  tasks:
    - name: install libsasl2-modules
      apt:
        pkg: libsasl2-modules
        state: present

    - name: append to postfix main.cf
      blockinfile:
        path: /etc/postfix/main.cf
        block: |
          smtp_sasl_auth_enable = yes
          smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
          smtp_sasl_security_options = noanonymous
          smtp_sasl_tls_security_options = noanonymous
          smtp_tls_security_level = encrypt
          header_size_limit = 4096000
          relayhost = [smtp.sendgrid.net]:2525

    - name: create sasl_password file
      template:
        src: "{{inventory_dir}}/files/sasl_passwd"
        dest: /etc/postfix/sasl_passwd

    - name: set permissions on sasl_passwd file
      command: chmod 0600 /etc/postfix/sasl_passwd

    - name: postmap /etc/postfix/sasl_password file
      command: postmap /etc/postfix/sasl_passwd

    - name: restart postfix
      service: name=postfix state=restarted

    - name: remove the plaintext sasl_passwd file
      file:
        state: absent
        path: /etc/postfix/sasl_passwd
{% endhighlight %}

Then in your `files/sasl_passwd` place a sample file with this

{% highlight shell %}
{% raw %}
[smtp.sendgrid.net]:2525 {{ sendgrid_username }}:{{ sendgrid_password }}
{% endraw %}
{% endhighlight %}

In order to get this to run you will need a couple of variables defined

{% highlight shell %}
sendgrid_username
sendgrid_password
{% endhighlight %}

Which you can do in your `group_vars` file.

