---
layout: post
title: GSoC 2019 - First Milestone
description: "Google Summer of Code 2019: Reaching the First Milestone"
modified: 2019-06-23
tags: [gsoc19]
categories: [code]
comments: false
image:
    feature: gsoc19/post-header.png
    credit: casals    
---

This week marks the end of the first month of <abbr title="Google Summer of Code">[(GSoC)](https://summerofcode.withgoogle.com/)</abbr> - which also means reaching our first milestone. This is a long post condensing all the discussions we had in the past month, detailing the evolution of the development with all of its failures and sudden realizations. 

<!-- more -->

## Recap

According to the official GSoC timeline, each month corresponds to a project phase, and by the end of each phase, both students and mentors must submit a mutual evaluation to Google. In our initial planning, we also established milestones for each of the phases (the full task list can be found in the GitHub project board - link below). The goal of our project is to create a mechanism for collective reasoning in Terasology - which means allowing a group of entities to reason as a unit. In order to achieve this goal we established the following milestones:

* Phase 1 - Create a group structure for entities
* Phase 2 - Create a collective reasoning mechanism
* Phase 3 - Integrating the new mechanism with the existing code

In this first month, our goal was to create a way of grouping structures in a manner that they could use a shared behavior mechanism and act as one. In practical terms, this involved:

* Exploring possible alternatives for the group structure
* Choosing/Modeling the new group structures and mapping its impact on the existing API
* Implementing the new structure and modifying the existing API as necessary
* Creating an evolutive scenario to be used in the rest of the project
* Implementing a new module to be used for tests along with the project

While these tasks seem simple enough for a milestone, there was a lot to be considered and discussed - and even after that, different alternatives should be tested if we were to create something that could be used in the long run.

## Discussion

In order to effectively establish collective reasoning, there were two dimensions to be considered: (i) how group behavior can be _manifested_ within the game, and (ii) how the game is actually being implemented. 

The first dimension involves recognizing the different manifestations of group behavior, and which of them are pertinent to our domain. There are different ways to look at it, but in a general manner, we can classify it from the _coordination_ perspective. Group behavior can be either actively coordinated, or it can _emerge_ from the actions of its members. As an illustration, we can consider the difference between group movement patterns: _flocking_ is something that can emerge from the individual behaviors of each of the group members, driven by the necessity of being close enough of their neighbors, or moving towards the same direction they are moving. Emerging behavior can be _active_ or _passive_, depending on the interaction between the group members. Flocking does not require any form of interaction: each group member perceives its neighbors and takes the appropriate action (getting closer or further). This can be seen as passive emergent behavior. An example of active emergent behavior would be a stock market operated by multiple entities: while each entity has its own set of goals and can make their own decisions (coordinated or not), the market behavior emerges without any coordination, as a result of the decisions taken by the entities. 

Active coordination can also be _centralized_ or _decentralized_. Following a leader is an example of centralized coordination since the decision process is held by a single entity. This entity can be a part of the group (the alpha wolf in a wolf pack) or an external entity, that does not take part in the actions resulting from the decision process (a general can order the dislocation of troops without having to leave the HQ). Decentralized decision processes, on the other hand, can account for multiple group members actions.
If the entire group votes on their destination (considering an entirely democratic process), the destination selection can be seen as a decentralized coordination mechanism. 

From the game perspective - and taking this dimension into consideration, ideally we want to have:

* Passive emergent group behavior (mainly for movement)
* Decentralized coordination
* Centralized coordination, considering a member of the group
* Centralized coordination, considering a member external to the group

Emergent movement behavior, however, does not necessarily involve the creation of a group structure, since the group itself can emerge from the movement patterns. Also, decentralized coordination requires each group member to be capable of communicating with each other - which is currently not (explicitly) possible in Terasology. For these reasons, we focused our problem on enabling centralized group coordination, and by the end of the first week, we had decided to model group behavior following a [hivemind-like structure]({{ site.url }}/code/gsoc-first-week/).

We also need to consider that a single entity can be a member of more than one group at the same time. That means group behavior can also be _cumulative_: in the case of an entity belonging to two different groups with centralized coordination, for example, and in the event of the two different group leaders choosing conflicting behaviors - which behavior should the entity avoid? We tackled this subject in different discussions but in order to simplify our scenario, we decided that at this point it would be enough for a single entity to belong to multiple groups simultaneously.

The second dimension involves the ECS architectural pattern used in Terasology (which we briefly introduced [in a previous post]({{ site.url }}/code/gsoc-first-week/)). We did not want to break this pattern, which means that the group implementation should be done either as an entity or as a system module. We also wanted to impact the API as less as possible to maintain backward compatibility, and to take into consideration other development initiatives within the game. With that in mind, [in the second week]({{ site.url }}/code/gsoc-second-week/) we chose three implementation models to be tested:

* A "transparent" Entity to be used as an Entity aggregator ("ex-machina parent");
* A Sector-derived implementation for Entity grouping; and
* A specialized Entity register within the code.

## Third week

Having defined the base alternatives for implementing a group structure, the next step was creating a sandbox module for testing each of the alternatives. I used the [WildAnimals](https://github.com/Terasology/WildAnimals) module as a base due to its stability/simplicity and created a secondary module to implement the proposed alternatives. I felt the need of creating different As a result, we got _sick deers_:

![Sick Deers]({{ site.url }}/images/gsoc19/sick-deers.jpg)
{: .image-center}

The first alternative I explored was the Sector-derived implementation for Entity grouping (`Sector` and `Pool` are referenced [here]({{ site.url }}/code/gsoc-second-week/)). After playing with the Sector/Pool implementation for a couple of days, however, it became clear that I would still need a secondary Entity for behavior processing. As it is implemented now, behaviors are associated with single entities (Actors) and processed accordingly. Transferring this to the System level would require to transfer most of the behavior structure to the system level as well, or to make a few bridges that could lead to an even higher level of complexity. So I decided to work on the Entity aggregator model.

During the rest of the third week, I created and tested a class called `ActorGroup` in the core, which would work both as a set of actors and a set of ActorGroupp The idea was to create behavior actions in the module (no messing with the Behavior Tree structure) and making it work for groups. For that, however, I needed to solve two points in the near future. The first was how to assign new entities to existing groups. I hard-coded an event that would create an ActorGroup instance every time a sick deer was spawned, which gave me a group of one sick deer - but didn't solve the assignment of newly spawned sick deers to an existing group.

The second point was related to group movement. I chose "flocking" as the initial goal for group movement to find possible hiccups, and I realized that the basic "stray" behavior was bound to a random neighbor choosing that I could not bypass from above. My idea at the time was to create a ContainedStray behavior (stray within an area) so I could use a point of reference (first spawn location of the first member of the group) to make animals flee to the same direction, for example.

In theory, using the ActorGroup would solve most of the problems associated with group behavior scenarios (above). But the beauty of implementing a separate module for testing was that it helped me visualize a few of the uncertainty points discussed before. Testing movement behaviors, for example, was helpful to confirm some of the predicted difficulties related to probabilistic actions. Working with a Pool did not help in that sense, and my first group approach (a sick deer having a BrainDeer as a parent) hit the same wall. 

I was able to determine that this would always happen because of how the current behavior structure is modeled - the `Action.modify(Actor, BehaviorState)` method used by `BehaviorNode` is the bottleneck. In the end, no matter how I grouped the entities, the behavior implementation logic would always be applied to its associated Actor. The perfect example of this situation is the `SetTargetToNearbyBlockNode` behavior action (Behavior module). Imagine that I wanted to every single member of a group to move to the same target (and I need it to be probabilistically defined, just as it's done in this class). In SetTargetToNearbyBlockNode, the target (after determined) is set on line 48 of the class implementation, with `actor.save(moveComponent);`. 


The most direct way to solve this was to create the ActorGroup as originally intended, and modifying all the behavior processing as it was (action methods, for example, should be overloaded to support both Actor and ActorGroup). The API impact would be huge (pretty much everything in the core behavior implementation), so I decided to hold on before going this direction.


I was looking for an alternative to minimize the API impact when I realized that I could just modify the `Actor` class by making it self-nesting (Actor with Actor[]). This would simplify the behavior processing since I wouldn't need to refactor it to support a new class type. In addition, it would solve *a lot* of the issues related to cumulative behavior: I could assign behaviors to entities seamlessly, and the responsibility of deciding what to do could be delegated to the composite system logic. 


With that in mind, I extended the Actor class with self-nesting but didn't create any methods related to group functionalities - this should be delegated to behavior logic (instead of creating a method for mass-assigning components to group members within the actor, this should be done as recursion in the behavior implementation). Also - any hivemind behavior could be implemented using the existing behavior structure, and this would also allow the same Actor to simultaneously belong to multiple groups. 

In order to test this idea, I went back to the [Behaviors] module, focusing on the _follow_ behavior. It is one of the simplest behaviors in the game, and it was easy enough to set a recursion to make every entity in the list follow a single leader. It worked as intended, and I had decided to close the matter and proceed with clean-up and documentation. I saved my work, satisfied with a solution that had a minimal API impact, and went for a fresh pot of coffee. And that was when I realized that I had broken one of the original premises.

I had spent almost four weeks going back and forth on the theory behind group behavior, experimenting with the existing code, and I finally had something that worked the way I wanted - and it was simple. Really, really simple. As simple as modifying a single class, and giving Actors the ability of self-nesting. As simple as including a list field in an entity class. As simple as forcing an entity to hold data. _As simple as breaking the ECS architectural pattern_.

Given the implementation at hand, it was simple enough to solve - instead of modifying the `Actor` class, I simply had to create a component to hold the group data. I could still use the behavior logic to affect multiple actors in the same manner, just by modifying how the recursion was made. But I was still baffled on how simple it was to stray from one of the original premises. I thought about this, and - for some reason - I was convinced that it was OK at the time (I even said it out loud in one of the Discord discussions). And realizing a mistake so simple (and yet so important) by the end of the fourth week of the project is part of the learning experience that I'd like to share.

## lessons learned and results

We had an apparently simple milestone for this first phase: creating a group structure to allow the implementation of a collective reasoning mechanism. This boils down to choosing the best way to represent groups in the system. The final solution was creating a specific component to register all of the group-related data. And yet - it took me a lot of reading, *a lot* of code that went straight to the bin (sick deers are no more), and a few mistakes and walls along the way (up to the very last moment). I hadn't had this much fun for ages. I've been developing software for a while now, but I never really ventured into game development. I had only read about ECS but never had any hands-on experience. So - if you're reading this post as a fellow GSoC student (perhaps a few years in the future), or if you just started contributing (or considering to contribute) to a project outside of your comfort zone, here are a few pointers illustrated by my personal recent experience:

* Take your time. Plan. Think of what you are trying to accomplish, and how you are doing it. You will eventually have to implement and test it - but don't be in a rush. Having a clear image of where you want to be is important in every step of the process (even if it changes along the way).

* Don't ever be afraid of deleting code. Ever. You can read a lot about this in books or posts, so it's pretty easy to assume this is now something new - but doing so when you are working outside your comfort zone is not _that_ easy. You must have the ability to say "great - this is what I learned - and with that, the code I wrote in the past days is utterly useless". Enjoy your journey, discard the code, embrace the experience.

* Reach for the community. I can't really stress this enough. During one of the discussions, I shared one of the potential problems I could have in the future, and one of the mentors said: "but this is _super_ easy". And it was, indeed. Sometimes you are so focused on your perspective that you fail to circle around your problem, considering it from a different point of view - one where you can see the solution. Different people think differently; use this for your advantage.

Back to the technical part of the text - after cleaning up the code and consolidating the tests, here are the results of this milestone:

* The group structure was implemented in the form of a component. This allows any `Actor` to act as the hivemind structure, with zero impact on the existing behavior API. All group behavior logic can be defined at the system level without the need for extra entity structures. This also allows the hivemind to be implemented as a virtual entity (without a physical representation in the world), or to be implemented in the form of a group leader (a deer that holds a leadership role, for example). Transferring leadership roles between entities might require a secondary system-level implementation, but this is justified by the minimized overall API impact. 

* In order to allow the instantiation of virtual hiveminds in runtime, we felt the need to implement a new core command called `summonVirtualPrefab`. This implementation impacts the core API.

* Instead of creating different creatures in the sandbox module, the WildAnimals module was extended with the implementation of RBG Deers. This allows greater segregation of base/experimental code for the next milestones, and the new creatures can be used by any module that currently depends on WildAnimals. The new deer types are shown below:

![RGB Deers]({{ site.url }}/images/gsoc19/RGB-deers.png)
{: .image-center}

As part of this milestone, I also created the evolving scenario for the entire project. It starts with the creation of groups of different creatures, and it evolves towards applying group behaviors in different conditions. The idea is to use this scenario to test the different steps of the upcoming implementation. Using the group module implementation as a basis, the next steps are:

#### Milestone 2:

* Creating a situation where newly-spawned creatures are assigned to an existing virtual hivemind. If the hivemind does not exist, it must be instantiated.

* Creating a situation where a creature can belong to two different groups.

* Implementing a behavior that can be applied to multiple group members at the same time. 

* Creating a movement behavior and implementing it both in the group and in the individual levels.

#### Milestone 3:

* Creating a situation with conflicting behaviors between two different groups, affecting a common member of both groups.

* Creating a situation to exploit the scenario where leadership must be transferred between entities.

## Weekly report

(this refers to both the third and fourth weeks of the project)

### What have you achieved in the last week?
* Finalized testing the discussed alternatives for the hivemind model
* Finalized group base implementation       
* Created a base module for scenario expansion
* Expanded the WildAnimals module with RBG Deers
* Finalized the evolving scenario for group behavior


### What are you currently working on?
* New core command
* Base module 

### What problems are you currently facing?
* (none)      

### Is anything blocking you from making progress?
* (none)

### List of PRs and opened/closed Issues
* RGB Deers (PR)

### Something else (pictures of new content, code snippets, new wiki content, …)
* Behavior module: `SetTargetToNearbyBlockNode` - added movement probability as a parameter (PR)

## Permanent links

* [GSoC proposal]({{ site.url }}/docs/gsoc19/GSoC 2019 - Arthur Casals.pdf)
* [GitHub project board](https://github.com/orgs/Terasology/projects/20)
* [Forum thread](https://forum.terasology.org/threads/gsoc-2019-collective-behavior.2255/)
* [Bibliography (BibTex format, subject to constant change)]({{ site.url }}/docs/gsoc19/gsoc-references.bib)
