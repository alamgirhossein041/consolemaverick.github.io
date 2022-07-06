---
title: How to create a good Gitlab Issue
layout: post
date: '2020-04-04'
subtitle: How old would you feel if you didn't know how old you were?
description: There is a way to create Gitlab issues which will ensure you get exactly what you want done, let me show you how.
image: /assets/img/uploads/gitlab.png
optimized_image: /assets/img/uploads/gitlab.png
category: fundementals
tags:
  - gitlab
  - fundementals
author: Pawel
---

## First let’s figure out what not to do
* Don’t just put a title and expect the other person to read your mind or fish through their Slack history to dig out the details when they have time to work on the issue  
* Instead of typing bits and pieces into Slack to describe the problem, write down what you want to say, organize it and put it into the issue  
* There is a chance the person you are assigning it to doesn’t have time to work on it right now or ever so if it's reassigned and you wrote everything in Slack, you will have have to explain the issue again to another person

## Some guidelines
* Make the description such that a developer in a different timezone can read it when they get in and start working on it immediately  
* Everyone can keep abreast on what’s going on with the issue by subscribing to it on Gitlab  
* If your work relies on another issue being completed subscribe to it  
* If the issue is missing information assign it back to the author and ask them to fill in the blanks.  
* If at some point the issue is assigned to another developer they have the full run down of everything that’s happened with the issue and can start working immediately  
* Core idea is to have all information in the issue so when you have time to work on it you don’t have to pry the pertinent information from PMs/developers/clients  
* Everything needs an issue for documentation. If you put it in an email or a chat it's ephemeral, if you put it in the issue it's there forever for everyone to see.

### Bad Example

#### Title

Redis isn’t working

#### Description

Application is supposed to send emails through sidekiq/redis.
I finished setting up sidekiq but redis does not seem to respond. Namespace is “application_production"
I ran redis manually.

##### The problem

This description leads to a naive attempt at a solution.  Go on the server and check if redis runs and the namespace exists.  If so, the issue appears to be solved but the real issue isn’t that redis isn’t working it’s that the email doesn’t get sent out.  So how do you check this?

The issue needs to be improved with:
1. The actual error seen  
2. Steps to reproduce  
3. Expected behavior  

### Good Example

#### Title 

Unable to send emails to users

#### Description

Users should receive emails via sidekiq background job UserNotiferJob.  In order to trigger the job manually use 
User.first.notify 
Right now you will receive this error 

[Screenshot]

I suspect it’s because redis is not configured properly.  I set the namespace in configuration to be ‘application_production’.  Right now redis is running manually and it’s not sending emails.

With this information you can fix the actual problem and be confident it’s complete because you followed the steps to reproduce and witnessed the expected behavior.
