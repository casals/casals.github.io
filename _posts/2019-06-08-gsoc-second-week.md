---
layout: post
title: GSoC 2019 - second week
description: "Google Summer of Code 2019: First week"
modified: 2019-06-08
tags: [gsoc19]
categories: [code]
comments: false
image:
    feature: gsoc19/post-header.png
    credit: casals    
---

Continuing on the first month of <abbr title="Google Summer of Code">[(GSoC)](https://summerofcode.withgoogle.com/)</abbr> it was finally time to start coding. 

<!-- more -->

## Thou shalt not make a machine in the likeness of a human mind

After some discussion with the mentors, in the previous week I had determined that the group structure would be implemented using the hivemind model. The next logical step was to start working on the actual group structure implementation, so - following previous discussions on the topic - I started exploring the existing work on _Sectors_. The concept of Sectors was explored and implemented [in a previous GSoC project](https://vizaxo.github.io/2017/08/26/gsoc-2017-final-post.html), and the basic idea involves grouping Entities into groups (which is exactly what I need). Based on this implementation, the hivemind structure could work as a Pool, minus the different simulation rate (please check the associated GSoC project link for a full understanding on the structure of Sectors). 

I presented this idea during our weekly meeting, and the following discussion on the use of Sectors was really good - there were, however, other possible group structure mechanisms that could be implemented (a result of past discussions on Discord). Each of these mechanisms was discussed due to possible future constraints in different scenarios - which are very difficult to predict at this point, of course, so all discussions were based in previous experiences with similar mechanisms. By the end of the discussion, we had three different implementation paths to follow:

* Creating a "transparent" Entity to be used as an Entity aggregator ("ex-machina parent");
* Creating a Sector-derived implementation for Entity grouping; and
* Creating a specialized Entity register within the code.

Since we had reached the limits of a theoretical discussion, I decided to start working on a proof-of-concept (PoC) module so I could actually test each of the above alternatives. I chose the existing WildAnimals module as a primary reference for the PoC. The objective is to use the existing entities in the reference module (deers) and create entity groups without modifying the spawning mechanism as it is right now. That means: when the first deer is created, it is assigned to a group. If a second deer is created, it should be able to recognize the existence of a local group and join it. Crerating a separate model allow us to test the different group structure alternatives without interfering with the existing (working) module. As soon as a group implementation is chosen, I can then proceed to assign behaviors to groups and start the next task (how a group behavior can superpose an individual behavior). 

Management: I'm not planning on solving any issues related to Sectors, but I know there's a macro issue on Multi-world support (#3146) - I'll revisit it as I work to see if anything can be reused. As for other issues: at this point it's all new - but I may raise new issues depending on the questions I compiled (below). In terms of repo impact, /MovingBlocks/Terasology should be the only one impacted (apart from the creation of a new module within /Terasology). (Issue suggestions?) 

## Weekly report

### What have you achieved in the last week?
* Determined how the group structure would be implemented 
* Started working on the group structure implementation       

### What are you currently working on?
* Documentation on model/API
* Sample module (using WildAnimals as a base)

### What problems are you currently facing?
* Just a few doubts about:
    * prefab structure: Trigger
    * Sectors use case?
    * spawnPrefab mechanism 
    * FindNearbyPlayersSystem       

### Is anything blocking you from making progress?
* (none)

### List of PRs and opened/closed Issues
* (none)

### Something else (pictures of new content, code snippets, new wiki content, …)
* (Updated the Github project board)

## Permanent links

* [GSoC proposal]({{ site.url }}/docs/gsoc19/GSoC 2019 - Arthur Casals.pdf)
* [GitHub project board](https://github.com/orgs/Terasology/projects/20)
* [Forum thread](https://forum.terasology.org/threads/gsoc-2019-collective-behavior.2255/)
* [Bibliography (BibTex format, subject to constant change)]({{ site.url }}/docs/gsoc19/gsoc-references.bib)
