---
title: Crypto Currency Notifier
layout: post
date: '2020-04-04'
subtitle: Bitcoin Trillionaire
description: Build your own crypto currency notifier because you're a nerd and would rather build software than make money.
image: /assets/img/uploads/3commas.jpg
optimized_image: /assets/img/uploads/3commas.jpg
category: crypto
tags:
  - crypto
  - coincheck
  - trillionaire
  - bitcoin
author: Pawel
---

If you want to be in the 3 commas club you have to dabble in crypto. If you want to be a bitcoin trillionaire you have to know how your portfolio is standing on a daily basis at the minimum. You could use a service for this but because you're paranoid and stored all your crypto in paper wallets you have to find another way. 

What we're trying to build is a daily email which lets you know if you can buy the orange Lambo yet or not. Something like this:

```
total: 10,340,477¥ paid:321,530¥ diff:416%
btc: total:4,183,480¥ trading:100,871¥ paid:350,913¥ diff:54%
eth: total:3,183,480¥ trading:100,871¥ paid:350,913¥ diff:47%
```

What you will need

1. free sendgrid account with an API key for sending email to yourself
1. access to a linux based machine to run the cronjob

Let's break the code up into pieces:

## Get rate from coincheck

```
def ticker
  JSON.parse(get(URI.parse("https://coincheck.com/api/rate/all"), nil))
end
```

## Assemble your data

```
btc = ticker["jpy"]["btc"].to_f
eth = ticker["jpy"]["eth"].to_f

balances = {"btc":"<HOW MUCH BITCOIN YOU HOLD>",
            "eth":"<HOW MUCH ETHEREUM YOU HOLD>"}

btc_balance = balances[:btc].to_f
eth_balance = balances[:eth].to_f

btc_total = (btc_balance * btc)
eth_total = (eth_balance * eth)

total = btc_total +
        eth_total
```

This wallet comprises BTC and ETH only but you can add whatever other coins you hold as long as Coincheck API supports it.

## Create the message

```
message = <<-EOS
   total: #{cleanup(total)}
   btc:   #{cleanup(btc_total)} @ #{cleanup(btc)}
   eth:   #{cleanup(eth_total)} @ #{cleanup(eth)}
EOS
```

## Parse the output from the API

```
def get(uri, headers)
  https = Net::HTTP.new(uri.host, uri.port)
  https.use_ssl = true
  response = https.start {
    https.get(uri.request_uri, headers)
  }

  response.body
end
```

## Cleanup the output from the API

```
def cleanup(num)
  num.truncate(0).to_s.reverse.gsub(/(\d{3})(?=\d)/, '\\1,').reverse
end
```

## Send email

```
def email(message)
  sendgrid_key = "<YOUR SENDGRID API TOKEN>"
  from = Email.new(email: '<YOU>@gmail.com')
  to = Email.new(email: '<YOU>@gmail.com')
  subject = 'Coincheck update'
  content = Content.new(type: 'text/plain', value: message)
  mail = Mail.new(from, subject, to, content)

  sg = SendGrid::API.new(api_key: sendgrid_key)
  response = sg.client.mail._('send').post(request_body: mail.to_json)
  puts response.status_code
  puts response.body
  puts response.headers
end
```

and finally let's put it together:

```
require 'net/http'
require 'uri'
require 'json'
require 'sendgrid-ruby'

include SendGrid

def email(message)
  sendgrid_key = "<YOUR SENDGRID API TOKEN>"
  from = Email.new(email: '<YOU>@gmail.com')
  to = Email.new(email: '<YOU>@gmail.com')
  subject = 'Coincheck update'
  content = Content.new(type: 'text/plain', value: message)
  mail = Mail.new(from, subject, to, content)

  sg = SendGrid::API.new(api_key: sendgrid_key)
  response = sg.client.mail._('send').post(request_body: mail.to_json)
  puts response.status_code
  puts response.body
  puts response.headers
end

def get(uri, headers)
  https = Net::HTTP.new(uri.host, uri.port)
  https.use_ssl = true
  response = https.start {
    https.get(uri.request_uri, headers)
  }

  response.body
end

def ticker
  JSON.parse(get(URI.parse("https://coincheck.com/api/rate/all"), nil))
end

def cleanup(num)
  num.truncate(0).to_s.reverse.gsub(/(\d{3})(?=\d)/, '\\1,').reverse
end

btc = ticker["jpy"]["btc"].to_f
eth = ticker["jpy"]["eth"].to_f

balances = {"btc":"<HOW MUCH BITCOIN YOU HOLD>",
            "eth":"<HOW MUCH ETHEREUM YOU HOLD>"}

btc_balance = balances[:btc].to_f
eth_balance = balances[:eth].to_f

btc_total = (btc_balance * btc)
eth_total = (eth_balance * eth)

total = btc_total +
        eth_total

message = <<-EOS
   total: #{cleanup(total)}
   btc:   #{cleanup(btc_total)} @ #{cleanup(btc)}
   eth:   #{cleanup(eth_total)} @ #{cleanup(eth)}
EOS

email(message)
```

Now all you have to do is log onto your Linux box and enter a cronjob to run this task:

```
crontab -e
```

assuming you named your script daemon.rb in /home/<YOU>/crypto folder

```
0 21 * * * bash -c "cd /home/<YOU>/crypto;source /usr/local/lib/rvm;rvm 2.5.1@crypto;bundle exec ruby ./daemon.rb"
```
