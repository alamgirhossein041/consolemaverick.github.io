---
title: '97 Things Every Programmer Should Know'
layout: post
date: '2020-04-28'
subtitle: Solid book with some insights
description: "Hightlights from the book by Kevlin Henney"
image: /assets/img/uploads/97.jpg
optimized_image: /assets/img/uploads/97.jpg
category: fundementals
tags:
  - fundementals
  - book
author: Pawel
---

Here are some things that stuck out reading [97 Things Every Programmer Should Know](https://amzn.to/3eYTLrS)
 

## Ask, “What Would the User Do?” (You Are Not the User) - Giles Colborne

* WE all tend to assume that other people THiNK LiKE US. But they don’t. Psychologists call this the false consensus bias
* best way to find out how a user thinks is to watch one.
* “Add up a column of numbers” is OK; “Calculate your expenses for the last month” is better.
* Don’t interrupt. Don’t try to help.
* When you get stuck, you look around. When users get stuck, they narrow their focus.
* becomes harder for them to see solutions elsewhere on the screen.
* If you must have instructions or help text, make sure to locate it right next to your problem areas
* They’ll find a way that works and stick with it, no matter how convoluted
* better to provide one really obvious way of doing things than two or three shortcut


## Before You Refactor - Rajith Attapattu

* Avoid the temptation to rewrite everything. It is best to reuse as much code as possible. No matter how ugly the code is, it has already been tested, reviewed, etc. Throwing away the old code—especially if it was in production—means that you are throwing away months (or years) of tested, battle-hardened code that may have had certain workarounds and bug fixes you aren’t aware of. If you don’t take this into account, the new code you write may end up showing the same mysterious bugs that were fixed in the old code. T
* Many incremental changes are better than one massive change. Incremental changes allows you to gauge the impact on the system more easily through feedback, such as from tests. It is no fun to see a hundred test failures after you make a change. This can lead to frustration and pressure that can in turn result in bad decisions. A couple of test failures at a time is easier to deal with, leading to a more manageable approach.
* Do not throw away the tests from the old code without due consideration. On the surface, some of these tests may not appear to be applicable to your new design, but it would be well worth the effort to dig deep down into the reasons why this particular test was added.
* code does not meet your personal preference is not a valid reason for restructuring. Thinking you could do a better job than the previous programmer is not a valid reason, either.
* One of the worst reasons to refactor is because the current code is way behind all the cool technology we have today, and we believe that a new language or framework can do things a lot more elegantly. Unless a cost-benefit analysis shows that a new language or framework will result in significant improvements in functionality, maintainability, or productivity, it is best to leave it as it is.


## Code Reviews - Mattias Karlsson

* There may be some organizations that need such a rigid and formal process, but most do not. In most organizations, such an approach is counterproductive. Reviewees can feel like they are being judged by a parole board. Reviewers need both the time to read the code and the time to keep up to date with all the details of the system; they can rapidly become the bottleneck in this process, and the process soon degenerates.
* Instead of simply correcting mistakes in code, the purpose of code reviews should be to share knowledge and establish common coding guidelines.
* Sharing your code with other programmers enables collective code ownership
* xt * Wow! eBook dot Com
* Ensure that comments are constructive, not caustic.
* roles could include having one reviewer focus on documentation, another on exceptions, and a third to look at the functionality.
* spread the review burden across the team members.
* Make it an informal code review whose principal purpose is to share knowledge among team members
* Leave sarcastic comments outside
 

## Comment Only What the Code Cannot Say - Kevlin Henney

* Comments should say something code does not and cannot say.
* Instead of compensating for poor method or class names, rename them. Instead of commenting sections in long functions, extract smaller functions whose names capture the former sections’ intent. Try to express as much as possible through code
 

## Continuous Learning - Clint Shank

* Always try to work with a mentor, as being the top guy can hinder your education. Although you can learn something from anybody, you can learn a whole lot more from someone smarter or more experienced than you. If you can’t find a mentor, consider moving on.
* Use virtual mentors. Find authors and developers on the Web who you really like and read everything they write. Subscribe to their blogs.
* Get to know the frameworks and libraries you use. Knowing how something works makes you know how to use it better. If they’re open source, you’re really in luck. Use the debugger to step through the code to see what’s going on under the hood. You’ll get to see code written and reviewed by some really smart people.
* Whenever you make a mistake, fix a bug, or run into a problem, try to really understand what happened.
* A good way to learn something is to teach or speak about it. When people are going to listen to you and ask you questions, you’ll be highly motivated to learn


## Convenience Is Not an -ility - Gregor Hohpe

* A diverse vocabulary allows us to express subtleties in meaning. For example, we prefer to say run instead of walk(true), even though it could be viewed as essentially the same operation, just executed at different speeds.
 

## Do Lots of Deliberate Practice - Jon Jagger

* If you ask yourself, “Why am I performing this task?” and your answer is, “To complete the task,” then you’re not doing deliberate practice.
* do deliberate practice to master the task, not to complete the task.
* principal aim of paid development is to finish a product, whereas the principal aim of deliberate practice is to improve your performance. They are not the same. Ask yourself, how much of your time do you spend developing someone else’s product? How much developing yourself?
* 10,000 hours is a lot: about 20 hours per week for 10 years.
* little point to deliberately practicing something you are already an expert at
* Deliberate practice means practicing something you are not good at
* challenging yourself, doing what you are not good at. So it’s not necessarily fun.
 

## Don’t Be Afraid to Break Things - Mike Lewis

* A skilled surgeon knows that cuts have to be made in order to operate, but she also knows that the cuts are temporary and will heal.
* Don’t be afraid of your code. Who cares if something gets temporarily broken while you move things around?
* Be the surgeon who isn’t afraid to cut out the sick parts to make room for healing. The attitude is contagious and will inspire others to start working on those cleanup projects they’ve been putting off.
 


## Don’t Just Learn the Language, Understand Its Culture - Anders Norås

* Once you’ve learned the ropes of a new language, you’ll be surprised how you’ll start using languages you already know in new ways.
 

## Don’t Repeat Yourself - Steve Smith

* Every line of code that goes into an application must be maintained, and is a potential source of future bugs
* “every piece of knowledge must have a single, unambiguous, authoritative representation within a system.”
 

## Don’t Touch That Code! - Cal Evans

* Once checked into SCC, whether automatically or manually, it should be rolled over to the development server, where it can be tested and tweaked if necessary to make sure everything works together. From this point on, though, the developer is a spectator to the process.
* Under no circumstances—ever, at all—should a developer have access to a production server. If there is a problem, your support staff should either fix it or request that you fix it.
* If it’s broke, production is not the place to fix it.

 
## The Guru Myth - Ryan Brush

* Similar questions arise in program design, such as “I’m building inventory management. Should I use optimistic locking?” Ironically, people asking the question are often better equipped to answer it than the question’s recipient. T
* It’s time for the software industry to dispel this guru myth. “Gurus” are human. They apply logic and systematically analyze problems like the rest of us.
* Consider the best programmer you’ve ever met: at one point, that person knew less about software than you do now
* A “guru” is simply a smart person with relentless curiosity.
* when working with someone smarter than me, I am sure to do the legwork, to provide enough context so that person can efficiently apply his or her skills.
* Removing the guru myth also means removing a perceived barrier to improvement. Instead of a magical barrier, I see a continuum along which I can advance.
* We don’t need gurus. We need experts willing to develop other experts in their field. There is room for all of us.

 
## Hard Work Does Not Pay Off - Olve Maudal

* AS a PROGRAMMER, YOU’LL FiND that working hard often does not pay off. You might fool yourself and a few colleagues into believing that you are contributing a lot to a project by spending long hours at the office. But the truth is that by working less, you might achieve more—sometimes much more. If you are trying to be focused and “productive” for more than 30 hours a week, you are probably working too hard. You should consider reducing your workload to become more effective and get more done.
* direct consequence of the fact that programming and software development as a whole involve a continuous learning process. As you work on a project, you will understand more of the problem domain and, hopefully, find more effective ways of reaching the goal.
* To avoid wasted work, you must allow time to observe the effects of what you are doing, reflect on the things that you see, and change your behavior accordingly.
* Professional programming is usually not like running hard for a few kilometers, where the goal can be seen at the end of a paved road. Most software projects are more like a long orienteering marathon. In the dark. With only a sketchy map as guidance. If you just set off in one direction, running as fast as you can, you might impress some, but you are not likely to succeed. You need to keep a sustainable pace, and you need to adjust the course when you learn more about where you are and where you are heading.
* you always need to learn more about software development in general and programming techniques in particular. You probably need to read books, go to conferences, communicate with other professionals, experiment with new implementation techniques, and learn about powerful tools that simplify your job. As a professional programmer, you must keep yourself updated in your field of expertise—just as brain surgeons and pilots are expected to keep themselves up to date in their own fields of expertise. You need to spend evenings, weekends, and holidays educating yourself; therefore, you cannot spend your evenings, weekends, and holidays working overtime on your current project.
* Do you really expect brain surgeons to perform surgery 60 hours a week, or pilots to fly 60 hours a week? Of course not: preparation and education are an essential part of their profession.
* Avoid embarrassing yourself, and our profession, by behaving like a hamster in a cage spinning the wheel. As a professional programmer, you should know that trying to be focused and “productive” 60 hours a week is not a sensible thing to do. Act like a professional: prepare, effect, observe, reflect, and change.

 
## Know Well More Than Two Programming Languages - Russel Winder

* paradigms of computation: procedural, object-oriented, functional, logic, dataflow, etc.
* Why are these challenges good? That has to do with the way we think about the implementation of algorithms and the idioms and patterns of implementation that apply. In particular, cross-fertilization is at the core of expertise. Idioms for problem solutions that apply in one language may not be possible in another language. Trying to port the idioms from one language to another teaches us about both languages and about the problem being solved.
* Programmers should always be interested in learning new languages, preferably from an unfamiliar paradigm.
* Although it’s a start, a one-week training course is not sufficient to learn a new language: it generally takes a good few months of use, even if part-time, to gain a proper working knowledge of a language.


## Know Your IDE - Heinz Kabutz

* My favorite is automated refactoring, particularly Extract Method, where I can select and convert a chunk of code into a method. The refactoring tool will pick up all the parameters that need to be passed into the method, which makes it extremely easy to modify code. My IDE will even detect other chunks of code that could also be replaced by this method and ask me whether I would like to replace them, too.
* if during a code review, I noticed that the programmers had named lots of classes the same, I could find these very easily using the tools find, sed, sort, uniq, and grep


## Know Your Limits - Greg Colvin

* YOUR resources are LiMiTED. You only have so much time and money to do your work, including the time and money needed to keep your knowledge, skills, and tools up to date. You can only work so hard, so fast, so smart, and so long. Your tools are only so powerful. Your target machines are only so powerful. So you have to respect the limits of your resources.



## Learn to Estimate - Giovanni Asproni

* The following exchange between a project manager and a programmer is not atypical: Project Manager: Can you give me an estimate of the time necessary to develop feature xyz? Programmer: One month. Project Manager: That’s far too long! We’ve only got one week. Programmer: I need at least three. Project Manager: I can give you two at most. Programmer: Deal!
* The programmer, at the end, comes up with an “estimate” that matches what is acceptable for the manager.
* since it is seen to be the programmer’s estimate, the manager will hold the programmer accountable to it.
* we need three definitions—estimate, target, and commitment:
* estimate is an approximate calculation or judgment of the value, number, quantity, or extent of something.
* A target is a statement of a desirable business objective
* A commitment is a promise to deliver specified functionality at a certain level of quality by a certain date or event.
* Estimates, targets, and commitments are independent from one another,
* primary purpose of software estimation is not to predict a project’s outcome; it is to determine whether a project’s targets are realistic enough to allow the project to be controlled to meet them.
* purpose of estimation is to make proper project management and planning possible, allowing the project stakeholders to make commitments based on realistic targets.
* What the manager in the preceding conversation was really asking the programmer was to make a commitment based on an unstated target that the manager had in mind, not to provide an estimate.
 

## Only the Code Tells the Truth - Peter Sommerlad

* “Oh, my comments will tell you everything you need to know.” But keep in mind that comments are not running code. They can be just as wrong as other forms of documentation
* If your code needs comments, consider refactoring it so it doesn’t. Lengthy comments can clutter screen space and might even be hidden automatically by your IDE
* Refactor mercilessly when you learn how to code a simpler, better solution. Make your code as simple as possible to read and understand.
* if you are a maintenance programmer and the code you are working on does not tell the truth easily, apply the aforementioned guidelines in a proactive manner. Establish some sanity in the code, and keep your own sanity.
 

## The Professional Programmer - Robert C. Martin (Uncle Bob)

* If you are a professional, then you are responsible for your own career. You are responsible for reading and learning. You are responsible for staying up to date with the industry and the technology. Too many programmers feel that it is their employer’s job to train them. Sorry, this is just dead wrong. Do you think doctors behave that way? Do you think lawyers behave that way? No, they train themselves on their own time, and their own nickel. They spend much of their off-hours reading journals and decisions. They keep themselves up to date. And so must we. The relationship between you and your employer is spelled out nicely in your employment contract. In short: your employer promises to pay you, and you promise to do a good job.
 

## Put Everything Under Version Control - Diomidis Spinellis

* Commit each logical change in a separate operation. Lumping many changes together in a single commit will make it difficult to disentangle them in the feature.
 

## Put the Mouse Down and Step Away from the Keyboard - Burk Hufnagel

* while you’re coding, the logical part of your brain is active and the creative side is shut out. It can’t present anything to you until the logical side takes a break.
* next time you hit a nasty problem, do yourself a favor. Once you really understand the problem, go do something involving the creative side of your brain—sketch out the problem, listen to some music, or just take a walk outside. Sometimes the best thing you can do to solve a problem is to put the mouse down and step away from the keyboard.
 

## The Road to Performance Is Littered with Dirty Code Bombs - Kirk Pepperdine

* Consider the case where you find an execution hot spot. The normal course of action is to reduce the strength of the underlying algorithm. Let’s say you respond to your manager’s request for an estimate with an answer of 3–4 hours. As you apply the fix, you quickly realize that you’ve broken a dependent part. Since closely related things are often necessarily coupled, this breakage is most likely expected and accounted for. But what happens if fixing that dependency results in other dependent parts breaking? Furthermore, the farther away the dependency is from the origin, the less likely you are to recognize it as such and account for it in your estimate. All of a sudden, your 3–4-hour estimate can easily balloon to 3–4 weeks. Often, this unexpected inflation in the schedule happens one or two days at a time. It is not uncommon to see “quick” refactorings eventually taking several months to complete. In these instances, the damage to the credibility and political capital of the responsible team will range from severe to terminal.
* Consider fan-out for classes: it is defined as the number of classes referenced either directly or indirectly from a class of interest. You can think of this as a count of all the classes that must be compiled before your class can be compiled. Fan-in, on the other hand, is a count of all classes that depend upon the class of interest. Knowing fan-out and fan-in, we can calculate an instability factor using I = fo / (fi + fo). As I approaches 0, the package becomes more stable. As I approaches 1, the package becomes unstable. Packages that are stable are low-risk targets for recoding, whereas unstable packages are more likely to be filled with dirty code bombs. The goal in refactoring is to move I closer to 0.
 
 
## Simplicity Comes from Reduction - Paul W. Homer

* The code should be simple. There should be a minimal number of variables, functions, declarations, and other syntactic language necessities. Extra lines, extra variables…extra anything, really, should be purged immediately. What’s there, what’s left, should be just enough to get the job done, completing the algorithm or performing the calculations. Anything and everything else is just extra, unwanted noise, introduced accidentally, obscuring the flow, and hiding the important stuff.
 

## Start from Yes - Alex Miller

* Usually, you’ll find that when you know the context of the request, new possibilities open up. It’s common for the request to be accomplished with the existing product in some other way, allowing you to say yes with no work at all: “Actually, in the user preferences, you can download the round, translucent windows skin and turn it on.”
* If you can voice a compelling explanation as to why the feature request is incompatible with the existing product, then you are likely to have a productive conversation about whether you are building the right product.
 

## Testing Is the Engineering Rigor of Software Development - Neal Ford

* A bridge builder would never hear from his boss, “Don’t bother doing structural analysis on that building—we have a tight deadline.”
 

## The Unix Tools Are Your Friends - Diomidis Spinellis

* Unix tools were developed in an age when a multiuser computer had 128KB of RAM. The ingenuity that went into their design means that nowadays they can handle huge data sets extremely efficiently. Most tools work like filters, processing just a single line at the time, meaning that there is no upper limit in the amount of data they can handle. You want to search for the number of edits stored in the half-terabyte English Wikipedia dump?
* Unix commands executing as pipelines, like the preceding one, will naturally distribute their load among the many processing units of modern multicore CPUs.
 

## Write Code As If You Had to Support It for the Rest of Your Life - Yuriy Zubarev

* Write code as if you had to support it for the rest of your life.
 

## Your Customers Do Not Mean What They Say - Nate Jackson

* Challenge your customers early, and challenge them often. Don’t simply restate what they told you they wanted in their words. Remember: they didn’t mean what they told you. I often implement this advice by swapping out the customer’s words in conversation with them and judging their reaction. You’d be amazed how many times the term customer has a completely different meaning from the term client. Yet the guy telling you what he wants in his software project will use the terms interchangeably and expect you to keep track as to which one he’s talking about. You’ll get confused, and the software you write will suffer.

