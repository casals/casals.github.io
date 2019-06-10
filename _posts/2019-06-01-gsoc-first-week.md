---
layout: post
title: GSoC 2019 - first week
description: "Google Summer of Code 2019: First week"
modified: 2019-06-01
tags: [gsoc19]
categories: [code]
comments: false
image:
    feature: gsoc19/post-header.png
    credit: casals    
---

After an entire month of community bonding (which also included playing the game, setting up the development environment, and overloading the mentors with questions) it was finally time to start the official development period of <abbr title="Google Summer of Code">[(GSoC)](https://summerofcode.withgoogle.com/)</abbr>. 

<!-- more -->

## A plan depends as much upon execution as it does upon concept

Before starting the actual coding, there were two things to be done (for the record: [there are always things to be done before you start coding](https://www.khanacademy.org/computing/computer-programming/programming/good-practices/a/planning-a-programming-project)). In this case, [Terasology's GSoC project structure](https://github.com/MovingBlocks/Terasology/wiki/GSOC-project-structure) includes a list of "Pre-GSoC action items" related to project management. From a macro perspective, the list items were:

* Setting up the communication and reporting mechanisms (Slack channel, Forum thread, blog, etc.);
* Arranging a weekly meeting with all mentors;
* Creating and populating the associated [GitHub board](https://github.com/orgs/Terasology/projects/20); and
* Considering the API impact of the future code.

The first two items were painless, but the other two - oh, boy. After some time working in software development, one usually learns that [development scheduling works better with enough evidence](https://www.joelonsoftware.com/2007/10/26/evidence-based-scheduling/). Still, it is important to plan in advance what features you will be programming, how you intend to do it, what is your timeline, etc. And yet - it will change, probably (definitely). With that in mind, I created the initial tasks based on my [GSoC proposal]({{ site.url }}/docs/gsoc19/GSoC 2019 - Arthur Casals.pdf). Now, all that was left was considering the API impact. "All that was left".

## It begins, as most things begin, with a song

Another of those things that you eventually learn is that [one spends much more time reading than actually writing code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship-ebook-dp-B001GSTOAM/dp/B001GSTOAM/). This is especially true for the project at hand: we are messing with things under the hood, and that alone involves a lot of consideration. Also - the objective is to implement something that can be done in many different ways, from both the theory and the coding perspectives. That means reading _a lot_.

I tend to work well with (ever-changing) lists, so I created one to prioritize - well, the reading - considering both the project topic and context:

* Everything about how the current system works (always a good starting point);
* Articles related to Behavior Trees and their use in game development;
* Existing Behavior Tree implementations used in games (it is always nice to see what everybody is doing before trying to reinvent the wheel); and
* Alternatives to Behavior Trees for group-oriented behavior in games (it is also nice to know why people are not doing what you think is a good idea at the moment).

Terasology uses an [Entity-component-system Architecture](https://en.wikipedia.org/wiki/Entity_component_system), which is an architectural pattern mostly used in games. This pattern promotes strict separation between logic and data, and it is composed by:

* Entities (used to _identify_ each unique object in the world, including players and NPCs)
* Components (used to store all _data_ related to an Entity, including features and behaviors)
* System (providing all _logic_ behind a component or a combination of components)

You can find more information on Terasology's ECS architecture [here](https://github.com/MovingBlocks/Terasology/wiki/Entity-system-concepts) and [here](https://github.com/MovingBlocks/Terasology/wiki/Entity-System-Architecture).

After considering different alternatives (as in "long story short"), I finally decided to adopt a hivemind-like structure. Behavior Trees are used in Terasology to assign pre-defined behaviors to single Entities. The objective of this project is to develop a collective behavior mechanism for NPCs, which translates into "making a group of Entities behave as one". The keyword here is _group_ - before creating a collective reasoning mechanism, the grouping mechanism must be defined. We discussed a few ideas on Discord - for example, a group could be befined by a leader entity, which would also determine the behavior for the rest of the group - but this would lead to other problems (leadership transfer - in the case the leader dies, or superposing group behaviors - I will not go through the entire list here, but you are invited to access the #ai channel on Discord and follow the discussion).


## Weekly report

Also following Terasology's GSoC project structure, the students are encouraged to publish a weekly report following a specific template:

### What have you achieved in the last week?
* Read a lot of articles related to BT (full biblio to be documented on the blog post)
* Checked existing code to see how other people were doing it
* Determined a general hivemind structure
* Housekeeping: updated Trello, created blog, structured planning, updated forum thread
   * Overview: initial planning

### What are you currently working on?
* Creating the collective behavior model

### What problems are you currently facing?
* (none)

### Is anything blocking you from making progress?
* (none)

### List of PRs and opened/closed Issues
* (none) 

### Something else (pictures of new content, code snippets, new wiki content, …)
* Blog address: https://casals.io 

### In the weekly meeting, we:
* Went over the initial planning
* Discussed the different models that could be used for grouping Entities

## Permanent links

* [GSoC proposal]({{ site.url }}/docs/gsoc19/GSoC 2019 - Arthur Casals.pdf)
* [GitHub project board](https://github.com/orgs/Terasology/projects/20)
* [Forum thread](https://forum.terasology.org/threads/gsoc-2019-collective-behavior.2255/)
* [Bibliography (BibTex format, subject to constant change)]({{ site.url }}/docs/gsoc19/gsoc-references.bib)
