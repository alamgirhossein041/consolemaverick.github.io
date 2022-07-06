---
title: Project start to finish
layout: post
subtitle: All the little pieces
description: Don't forget all the steps when you take a project from start to goodbye
date: '2020-05-15 13:28:05 +0900'
image: /assets/img/uploads/puzzle.jpeg
optimized_image: /assets/img/uploads/puzzle.jpeg
category: fundementals
tags:
  - project
  - management
author: Pawel
---

Here is a typical project workflow template you can use from project start to the application being retired.

## Project Starts

• create DB diagram  
• create gitlab project  
• setup server via ansible scripts to make sure all basics are installed  
• install CI  
• setup failed build notifications on slack  
• Setup deploy notification going to slack (slackistrano)   
• protect master branch  
• protect staging branch   
• set develop as default branch  
• Setup DNS  
• setup develop server  
• setup auto deploy to dev server  
• install newrelic  
• install rollbar  
• robots.txt for staging site  
• automatic DB diagram update  
• if on another server add to munin  
• configure continuous deployment  
* schedule time with entire team to go over gitflow  

## Project Continues

• check client is paying for maintenance  
• make sure project is getting deployed to production once per week (sprint meeting happens before deploy to agree what is getting deployed)  
• server package updates  
• app Rails and Gems updates: update rails on existing apps as they come out - minor versions at minimum  
• make sure Gems are either issuing pull requests or in Company organization so if an employee were to leave you can still use the modified Gem  
• examine NewRelic for performance issues  
• verify GitFlow is being followed  
• make sure core functionality specced and passing  
• CI passing  
	○   rspec passing  
	○   Rubocop passing  
• logo.png present for GitLab  
• Create staging server  
• switch default branch from develop to master  
• Lower costs of server by committing to Reserved Instances on Amazon  

## Project Goes to Production

* setup production server  
* install pingdom  

## Project Ends

* remove from Pingdom  
* remove client's Gitlab access  
* take down staging site
* take down develop site  
* take down production site  
* remove DNS  
* disable cronjobs  
* backup database on S3 in case you need it in the future  
