---
layout: post
title: GSoC 2019 - Second Milestone
description: "Google Summer of Code 2019: Reaching the Second Milestone"
modified: 2019-07-23
tags: [gsoc19]
categories: [code]
comments: false
image:
    feature: gsoc19/post-header.png
    credit: casals    
---

This week marks the end of the second month of <abbr title="Google Summer of Code">[(GSoC)](https://summerofcode.withgoogle.com/)</abbr> - which also means reaching our second milestone. It's been less than four weeks since the first milestone was achieved, but a lot happened. Once again, this is a long post... 

<!-- more -->

## The future is there... looking back at us

When we reached [our first milestone]({{ site.url }}/code/gsoc-reaching-first-milestone/), the group structure necessary for the collective reasoning mechanism was ready (after a lot of tests and discussion). We had RGB deers, ready to be tweaked. The evolving scenario was defined. The hivemind model was solved. The first phase of GSoC actually had five weeks, and since I was slightly ahead of schedule I took my time to experiment with core commands, polish the evolving scenario, clean up my workspace and prepare things for the collective reasoning mechanism _per se_. Life was good.

The first week of phase two was pretty uneventful - I was comfortable with the group structure from the previous milestone, so I dedicated a few hours to explore other functionalities within the engine that would be useful later. Since I had been able to experiment with probabilistic decisions for groups ahead of time, [the remaining tasks for this milestone](https://github.com/orgs/Terasology/projects/20) were all about collective behavior:

* T1: Assign identical behavior to all group members (unison behavior)
* T2: Test inherited behavior, behavior convergence and inheritance handling
* T3: Create identities for groups and group members
* T4: Group assignment by entity type and on-demand

The solution I found for group structures involved simply assigning a component to an entity and creating additional mechanisms to handle the group logic at the system level. This was a zero-impact approach from the core API perspective, preserving the original association between behavior trees and individual entities. Considering the evolving scenario (and all the previous discussions on Discord), however, I already knew that it would be necessary to create a few additional mechanisms to allow future functionalities such as cumulative behavior and individual behavior recovery (for entities that leave a group) - both wishlisted. This was all about tasks T2 and T3: I needed to solve both the identity and behavior inheritance problems _while_ working on unison behavior to avoid refactoring in the future. The plan was sound, so I went with it.

## Leave no path untaken

As I said before, I had a few hours per week allocated for exploration - what I call "lab time". A few interesting things were happening, (marginally) related to this project:

* By the end of week 5 the community found a bug related to the `stray` movement behavior. This was something fun to debug, and it allowed me to learn more about the game.

* There was some discussion around the pathfinding mechanism. I found this particularly interesting because this was the first game mechanism I studied in-depth (and pathfinding problems are naturally appealing). 

* While studying inherited behavior, I took some time to study prefabs and prefab inheritance as well. This is something that was bugging me since the RGB Deers extension - I found some problems when combining (module-)local/referenced component descriptions, which is why I decided to extend the `WildAnimals` component at the time. As a side-product, now there's a small addition to the prefab explanation on the wiki and RGB Deers are (also) being used as a sample case for prefab inheritance. It also allowed me to properly develop the new creatures I wanted to test group and collective behavior assigning - and now we have CYMK Deers (it's about the small things, you know).

* A performance problem related to the `FindNearbyPlayers` mechanism was identified. This is closely related to my project since at some point we want "organic" group assignment - which means detecting similar creatures around (according to some criteria) - and this was the mechanism I was planning to use as a reference.

Well, this was all good and nice, but on week 7 I started finding a few bumps on the road. I had concluded the unison behavior mechanism at the time, but my sample module repository was still empty - working on multiple tasks at the same time also means greatly increasing the junk code, and I wanted to finish the behavior shifting/state persistence task before moving to the last two. I already had an idea of how to implement them, but I wanted to release a proof of concept on week 8 - which meant at least cleaning up T1 and T2 in two or three days (the meeting for week 7 was exceptionally held on Friday, so this was the status at that time). At the same time, I was already starting to question a few decisions from the first milestone.

## The truth shall make you mad 

Let me begin by saying that there was no crisis. Exploring the system was exactly what allowed me to quickly overcome the difficulties I found on week 8 - but due to these same difficulties I was only able to release the POC at the beginning of week 9 (which means today, Tuesday, and it also means little time for my mentors to evaluate the progress for this milestone since the official evaluation takes place in three days). 

Since week 7's meeting was held on Friday, there was not much to tell on the next meeting (Wednesday) - but I was still working on T2, which was supposed to take less than one week. Behavior trees are great - they provide a simple mechanism for AI mechanisms, cleaner than Finite State Machines and without the extra complexity of Hierarchical Task Networks. However, like the other two, they are heavily dependent on states. Let us suppose, for example, that we have the following behavior for an NPC described by a behavior tree:

* Walk around and look for enemies
* If an enemy is found, increase one point on a threat level meter (but do nothing).
* If a second enemy is found, increase the threat level again, but still - do nothing.
* If a third enemy is found, increase the threat level again - and now it's high enough, so attack the enemy.

This behavior involves a counter, which translates into a state. We talked before about different types of group behaviors, and one of the scenarios involves replacing an individual behavior by a collective one. Now - imagine that this NPC is at threat level two, but then it joins a group. Now the NPC doesn't care anymore about enemies and threats, it just sits idly somewhere (it's a nap group). But once the NPC leaves the group, it needs its original behavior again. Restoring the behavior tree itself is easy - but what if I want to return the NPC to its original state?

When I closed the first milestone, the group structure was implemented as a module attached to an `Actor`. It solves a lot of impact issues, yes, but it also means it is a single point of failure for data. Let us consider a scenario where an entity can join a group by proximity (if a stray deer gets close enough of an existing deer group, it becomes part of the group). From an ECS perspective, the entity will be assigned a `GroupTag` module. If all my group logic is on the system level (which holds logic, not data) and I want to save my actual state (let's say "memories"), I can hold this data within the newly assigned module. However, it is expected for the entity to leave the group at some point. Again from the ECS perspective, that means the entity will likely have this module removed. So the "memory transfer" logic must be implemented at the system level.

While the logic itself is easy to understand, the hard part was to figure out what exactly I needed to save as "memory". We already had previous discussions about performance issues, and I initially considered saving a state descriptor. This would allow us to "follow the path" within the behavior tree until the original state of the entity (before joining the group) was reached. This, however, involves re-executing a trace - which is OK when you have a counter from zero to three, but not necessarily as well when you have a more complex tree. At the same time, the tree itself already has a descriptor (behavior trees are described in JSON structures and compiled in runtime), but there is always a trade-off between loading things from the disk and saving them in the memory. 

After running some tests, I decided to save everything in memory. The behavior tree was pretty straightforward, but I had to retrieve the state somehow. I found out that I could get it from the `DefaultBehaviorTreeRunner` instance, but reinstating it was more complicated. So - considering API impact as well - I set for saving the `Interpreter` instance since it required only the addition of a copy constructor to the original class, and the original entity state can be recovered in three lines of code.

Looking closely into this mechanism, however, opened a few possibilities. I had decided to completely focus on a system-level logic solution since my initial assessment led me to believe that the impact on the existing API was too great. However, after studying the behavior state problem, I realized that I only needed to create three new classes in the core API to have behavior trees assigned to (and processed by) groups of entities, instead of having a 1:1 relationship. It also opened room for some things that we had originally discarded due to their estimated complexity.

Having a collective behavior tree allows a much simpler mechanism for scenarios involving "invisible hiveminds" - to the point that the logic can be separated according to the scenario, and the complexity involved in implementing group logic is compartmentalized. If I want a simple group according to the entity type (which is exactly task T4), I can just assign the entities a `GroupTag` component. Simple behavior substitution still works in the same way as described above, and most group scenarios are satisfied by this. 

However - if I want a scenario where a single entity can not only belong to multiple groups but need to _ponder_ about conflicting behaviors - having a true collective behavior tree greatly simplifies the logic behind it. Collective behavior logic can then be separated into the following scenarios:

* Scenario 1: Entities that join a group receive the same behavior mechanism, regardless of state. This can be illustrated by the situation described before: if an NPC joins a group and just sits idly, there's no need for it to be coordinated with the other group members - it's enough that they are behaving _similarly_. Bringing this illustration to our context, it's equivalent to assigning all deers in a group the `critter` behavior.

* Scenario 2: Entities that join a group receive the same behavior mechanism, and they have to be _somewhat_ coordinated. Let's say that the same group of deers gets startled by some reason; the behavior description may require that all deers have their speed multiplier increased at the same time. Observe that doesn't mean changing the behavior (or the behavior tree), but rather assigning the same _behavior change_ to all entities within a group.

* Scenario 3: Entities that join a group receive the same behavior tree, and they _must_ act the same way. This can be illustrated by behavior changes that depend on a probability. Suppose that there's a behavior change that is bound to a probability roll of 50%. If we consider the previous scenario, that means that all entities in the group will go through the probability roll, but each with its own - so some of the entities within the group will change their behavior, and some won't. In scenario 3, however, there will be only one probability roll, and all entities in the group will behave as one. 

Without the use of a collective behavior tree processing, unison behavior (scenario 3) was emulated by forcing the state of one entity into others (something like "follow the leader"). That means one entity decides for the rest. This is OK in simpler scenarios, but we start to have a problem when we consider that one entity can belong to multiple groups at the same time. If the "leader" doesn't below to all groups to which another fellow entity belongs, there will be a problem down the road when conflicting behavior changes must be applied. Having a collective behavior tree, on the other hand, allows the pondering to be done individually by each entity, according to the groups to which they belong. These advantages were already discussed last month - what changed since then was the impact assessment on the existing API.

The last task for this milestone (T3) involved group and entity identity. Separating scenarios also helped here: `GroupTag` and `Hivemind` components were designed to work together, and not exclusively. I created a group asset that can be described in JSON format (similarly to behaviors and prefabs) and compiled in runtime. This allows the developer to meta-specify the groups (using group labels, declaring which behaviors will be used, and especially indicating if a `Hivemind` component will be necessary). At the same time, both `GroupTag` and `Hivemind` components can be directly assigned to the entities prefab files. This provides greater flexibility for dealing with different collective behavior scenarios. 

After solving these issues I was finally able to showcase everything in the [sample module](https://github.com/casals/WildAnimalsMadness). Due to how the components and supporting system logic were designed, I was also able to create a sample of individual behavior reassignment (which was one of the wishlist tasks). I also included simple movement behaviors (`run` and `stay`) to see how they could be changed using the new system logic. The new CMYK Deers are (partially) used in the examples:

![CYMK Deers]({{ site.url }}/images/gsoc19/CYMK-deers.png)
{: .image-center} 

## Weekly report

Since multiple tasks were handled at the same time (and pretty much solved at the same time as well), this comprises the reports for the past four weeks:

### What have you achieved in the last week?

#### Week 6:
* Closed M1
* Closed new core command set
* Started working on the collective behavior

#### Week 7:
* Finished basic unison behavior for groups

#### Week 8:
* Created the individual behavior state caching mechanism
* Created the `GroupTag` component
* Created the asset mechanism for group descriptions

#### Week 9:

* Extended the original logic API with collective behavior tree processing
* Created the `HiveMind`component
* Created identities for groups and group members
* Finalized group assignment basic mechanisms and commands
* Updated and committed the sample module showcasing the new functionalities

### What are you currently working on?
* Solving a bug in the group asset mechanism
* Looking into the `FindNearbyPlayer` issue

### What problems are you currently facing?
* RW bug in the group asset mechanism      

### Is anything blocking you from making progress?
* (none)

### List of PRs and opened/closed Issues
* [WildAnimals - PR #31](https://github.com/Terasology/WildAnimals/pull/31)
* [Terasology - PR #3708](https://github.com/MovingBlocks/Terasology/pull/3708)

### Something else (pictures of new content, code snippets, new wiki content, …)
* Small wiki additions to the [Gestalt](https://github.com/MovingBlocks/gestalt/wiki/Gestalt%20Asset%20Core%20Quick%20Start) and [ECS concepts](https://github.com/MovingBlocks/Terasology/wiki/Entity-system-concepts) wikis.
* Sample module [publicly available](https://github.com/casals/WildAnimalsMadness)
* Fun concepts explored: changing the skin of a deer while playing

## Permanent links

* [GSoC proposal]({{ site.url }}/docs/gsoc19/GSoC 2019 - Arthur Casals.pdf)
* [GitHub project board](https://github.com/orgs/Terasology/projects/20)
* [Forum thread](https://forum.terasology.org/threads/gsoc-2019-collective-behavior.2255/)
* [Bibliography (BibTex format, subject to constant change)]({{ site.url }}/docs/gsoc19/gsoc-references.bib)
