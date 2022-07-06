---
title: Agile Software Development Coles Notes
layout: post
date: '2020-04-20'
subtitle: TLDR
description: "Just give me the executive summary"
image: /assets/img/uploads/coles.jpg
optimized_image: /assets/img/uploads/coles.jpg
category: fundementals
tags:
  - fundementals
  - agile
  - scrum
  - kanban
  - scrumban
author: Pawel
---

Oh man 'Lord of the Flies' I still have nightmares about passing the conch.

Here are some things you may have missed about Agile Development. What follows are some points I learned about Agile from books like [Scrum: The Art of Doing Twice the Work in Half the Time](https://amzn.to/3eW64p1) and doing Agile Web Development for the past few years. It hasn't always been by the book but the general idea has always been there.

## What is Agile

### Deliver value faster 

Don't have to wait for months before something usable can use is deployed

### Welcome change 

As you work on the project you learn things you didn't know in the beginning so don't throw out the new knowledge act on it and make the necessary changes.

### Deliver working software frequently 

It should always be usable, every sprint needs to produce an MVP, this frequency is up to you but 1 week or 2 week sprints are recommended.

### Introspective 

At the end of sprint discuss what went well and what can be improved next time.

Work together everyday, both the business team and the development team working in the same room on the same project resulting in a collaborative discussion between the team and client. The team should be composed of motivated individuals and given the tools to get it done along with enough trust to do it. There is no need to manage highly motivated team. Each player is a problem solver not a 'coder' or 'manager'. You have to do many jobs. Your responsibility goes beyond your title.

Work at a continuous pace, not spurts of overtime and don't put all the pressure on one part of the team. Software can be a large and daunting project so focus on one part you can build out and expand from there. Don't get overwhelmed with the size and complexity of the project. It can be broken down into manageable components. Work on one component at a time.

Client may assume that what they say at the beginning is the scope and cannot be changed so they tell you all their wishes even though most are not providing value. If you allow for change then they can come up with better requirements later on when they have an MVP and have used it for a while. Assure them that this is the case.

![lifecycle](/assets/img/uploads/agile_2.png)

## Sprint

Once you decide what gets done in the sprint it's fixed in stone, it cannot be changed. The team must be autonomous in implementing the sprint.

## Estimating

There are several models to use for estimates:

### BUS

#### Big
- if you have many of these break them down into multiple small ones
- Some will stay big because they cannot provide the necessary value to stake holders if they are altered

#### Uncertain
- Requirements are vague

#### Small
- you want most of the stories to be small effort ie. Can be done in a few days

### T Shirt Size

XS S M L XL

### Fibonacci Sequences

1 2 3 5 8 13 21

## Versus Waterfall

How does Agile compare to the traditional Waterfall model of software development?

![waterfall](/assets/img/uploads/agile_1.png)

A product increment is like a new version of an app with more features. In agile you take vertical slices versus a horizontal flow.

## Cons of Agile

- difficult to apply what you learned in one project to another because knowledge is siloed
- difficult to implement in a company using another methodology
- no one takes ownership of the project, all work together
* there isn't much documentation because you talk things through instead

# Different Types of Agile

## Kanban

- Came from Toyota
* Jira / Trello is Kanban

![kanban](/assets/img/uploads/agile_3.png)

- Don't want multiple 'Work in Progress' tasks (WIP) you want to start and finish tasks 
- In JIRA you can limit the amount of WIP tasks allowed, i.e. 1 per developer
JIRA warns you If something is blocking or if there's too many WIPs

![kanban](/assets/img/uploads/agile_4.png)

- Grouped by department or priority etc
- Red = highest priority

### You can categorize as:

* Todo: not fully specced yet
* Ready: specced, can be moved to WIP
* In progress; WIP
* Done: Completed

## Scrumban 

### Pull/Push Issues
* In Kanban you pull work - you take ownership  
* In scrum you're assigned the work  
* In Scrumban you pull the work  

### Roles
* Scrum has clear roles  
* Kanban has no roles - everyone is on same team  
* Scrumban doesn’t have set roles but may have specialized roles  

### Board
Kanban style
:
Kanban Todo -> Backlog

![scrumban](/assets/img/uploads/agile_5.png)

#### Triage/Feature Freeze 

At the end of project identify what are the priority tickets and freeze the rest of the work. Only high priority items are done in triage stage.

# Scrum

- Find out how fast the team is actually going by monitoring it over a few sprints - burndown chart
- Find out how people actually work not how they say they work
- Periodically inspect what you've done and determine if you should continue or pivot
- Fail fast so you can fix early
- Only work on what produces value
- there is individual performance and team performance - it's usually teams that accomplish something not individuals
- making the worst team mediocre is more beneficial than making an individual 10 times better
- lay out the facts no blame
- each team has all the people necessary to work on the project
- the individual should identify with the team not with their specialty
- if there's more than 9 people on the team velocity will slow
- optimal team size is 7+-2
- it's probably not the person that is performing poorly but rather the process
* 1-4 week sprint

## Teams

* self organizing, self managing, empowered to make necessary decisions  
* vision that transcends the problem   
* feeds on the team members skills  

## Scrum Roles

### Product Owner
- sets priorities
- talks to client
- creates user stories
- if you can't dedicate a person to one project as product owner scrum is not right for you
- "this is a great idea but there are other higher priority features right now"

### Scrum Master
- servant / leader
- not Project Manager
- ensures scrum is being used
- remove impediments
  * is there someone you need to talk to?

## Daily Standup

1. what did you do yesterday to achieve goals
2. what will you do today to achieve goals
3. what are your impediments to doing it


- it's not a status update
* PM does not interrupt can only attend

## Story Points

It's ok to allocate points to customer service story if at this time the client needs a lot of help.

## Retrospective

- 1 hour in length for 2 week sprint
- 2 hours in length for 4 week sprint

### Discuss

- what went well
- what went wrong
- what could be improved

Remember mistakes are not personal, success and failure is created as a team.

![schedule](/assets/img/uploads/agile_11.png)

## User Stories

    As a <end user>   
    I want to <desired action>  
    so that <desired benefit>  

    Who: needs it  
    What: do they want  
    Why: do they need it  

Story must be actionable before you can begin so define acceptance criteria. If you finish the criteria then the story is done.

Go deeper with *INVEST*

**Independent:** does not depend on other stories, can be done alone  
**Negotiable:** if after talking to the team you have a better idea you should be able to change the story  
**Valuable:** has value to users  
**Estimatable:** can determine how much complexity this story has  
**Small:** can finish in a few days  
**Testable:** how do I know it works well  

## Velocity

How many story points can the team complete per sprint

![velocity](/assets/img/uploads/agile_6.png)

## Burndown Chart

Illustrated through an example 

![velocity](/assets/img/uploads/agile_7.png)

* to achieve 120 story points in 10 sprints  
* amount of story points left over at the end of the sprint  
* behind schedule in sprint 1, ahead of schedule on sprint 3  

At the end of the sprints there should be no more points to do because there are no more stories to do.

![velocity](/assets/img/uploads/agile_8.png)

## Burnup Chart

![velocity](/assets/img/uploads/agile_9.png)

**Completed:** what actually got done  
**Total:** what needs to be done

Burnup Chart takes changes/additions/subtractions into account.

![velocity](/assets/img/uploads/agile_10.png)

Although it may look like you were ahead in sprint 3 but actually points were removed which is visible in the Burnup chart unlike the Burndown chart.

## Working Agreement

A statement of what's expected from everyone on the team.  Making everyone aware of what's expected so there are no surprises. Eliminates people being upset about something, it can be very trivial like moving a meeting to a different hour.  One team, one goal. You're trying to all get on the same page. 

Examples  
• You're always late to standup so let's just move the standup  
• Do meetings at the company meeting room instead of your desk  
• If you're going to interrupt me don't just come to my desk, send me a message first  
