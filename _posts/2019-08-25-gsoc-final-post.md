---
layout: post
title: GSoC 2019 - Final Report
description: "Google Summer of Code 2019: It's a wrap"
modified: 2019-08-25
tags: [gsoc19]
categories: [code]
comments: false
image:
    feature: gsoc19/post-header.png
    credit: casals    
---

This week marks the end of the third (and last) month of <abbr title="Google Summer of Code">[(GSoC)](https://summerofcode.withgoogle.com/)</abbr>. Here is a summary of what happened in these past weeks.

<!-- more -->

## Sometimes we can choose the paths we follow

This summer has been really fun. Getting accepted into GSoC [was kind of unexpected]({{ site.url }}/code/gsoc-here-we-go/), and working on this project involved stepping out of my comfort zone - which I really like. I hadn't ventured into game development before, and the architecture used by Terasology was completely new to me. At the same time, however, it was close to home in terms of research subject: "collective behavior for NPCs" was too good to pass. On top of that, it had been a while since I worked in an open-source project, so I decided to give it a shot. 

Best. Decision. Ever.

## Conspiracy theory's got to be simple

My detailed GSoC proposal can be found in the permalinks section below, but the main idea was to “extend the behavior mechanisms of non-playable characters (NPCs) to support collective behavior” in Terasology. In this context, "collective behavior" means that NPCs should be able to form (and act as) groups. For that to happen, it was necessary to:

* **Create a group structure** to support grouping of multiple NPCs;
* **Create a collective mind mechanism** to reflect a collective behavior among the group members;
* **Improve the existing behavior mechanism** to make it compatible with collective behaviors; and
* **Create a usage example** and add it to the official repository.

Terasology's behavior mechanism uses something called [behavior trees](https://github.com/Terasology/Behaviors/wiki/High-Level-Overview/), which are great to define complex behaviors for NPCs. These trees, however, depend on states and are usually assigned individually. Even with two or more NPCs have the same tree, they will not necessarily execute the same action under the same conditions, since their internal state may be different from each other. 

Working on the points listed above meant dealing with the behavior tree mechanism, extending/modifying them when necessary. Also - since our context involves playing a game - there were a few other things that we wanted:

* **Automatic group creation** as a result of existing behaviors;
* **Multiple groups for each NPC**, so that each NPC could belong to multiple groups at the same time; and
* **A supergroup structure** to make groups discoverable.

All of these points comprised the "goals" and "stretched goals" sections of the GSoC project. I had researhed as much as I could before submitting the proposal, and the first week of the project was [all about organizing the next steps]({{ site.url }}/code/gsoc-first-week/). In the end, we established milestones for each phase of GSoC: 

* Phase 1 - Create a group structure for entities
* Phase 2 - Create a collective reasoning mechanism
* Phase 3 - Integrate the new mechanism with the existing code

The GitHub project board (link below) was organized according to this plan, and we ended up with the following tasks:

* **Phase 1:**
   * Explore possible implementations for the group structure, considering an initial set of application scenarios and existing development initiatives in the game that could help us reach our goals
   * Determine the group structure to be used, considering (i) the classical/in-use behavior tree structure, (ii) existing implementations using behavior trees (i.e. HALO), and (iii) possible alternatives to behavior trees
   * Model the new group structure to be used, mapping the impact on the existing structures and current API
   * Create a progressive scenario and implement it as a module
* **Phase 2:**
   * Create a collective hive-like structure to process collective behaviors for NPCs
   * Implement guiding behavior and probabilistic decision mechanisms for a group
   * Create a collective behavior mechanism that would allow assigning identical behavior to all group members (so they would behave in unison)
   * Create an identification mechanism for groups and group members
   * Create different group assignment mechanisms
* **Phase 3:**
   * Reflect new behavior structure on a test module
   * Document all changes on existing wikis and repositories
   * Improve the existing Behaviors wiki and tutorial

This list was, of course, adjusted along the course of the project. We also added a few wishlist items that appeared during our discussions, related to the project scope - they are detailed in the GitHub board project.

## Hard tasks need hard ways

It was a fun, but not a smooth ride. Weighting all the possible scenarios in the first phase, in particular, involved a lot - **a lot** - of discarded code. I was also discovering the architecture at the same time, so there were a lot of experiments involved - [I wrote all about it before]({{ site.url }}/code/gsoc-reaching-first-milestone/). Even after that, [during the second phase]({{ site.url }}/code/gsoc-reaching-second-milestone/) I found that the group structure would need to be broken in two pieces and re-implemented, so all collective behavior scenarios discussed would be possible.

Another interesting aspect of the project was the collateral production of code. Creating a module to showcase the new structures required new behaviors, for example - and since I was exploring the environment I decided to venture into pretty much all I could. So, in the end, we had:

* New creatures - RGB/CYMK/Steel deers and a (paralyzed) wolf. This involved messing with existing prefabs and components from other modules, textures, animations, playing with Blend a bit.
* New behaviors - from simple things (such as staying still) to a flocking behavior, so we could demonstrate a scenario involving decentralized coordination in a group.
* A couple of experiments involving movement and pathfinding mechanisms (result from a joint effort to fix a particularly annoying bug)
* A new asset structure for groups (which was actually a sub-product from one of the requirements - and understanding how to load pre-defined assets from the disk was also pretty nice)
* Custom commands and effects (such s creatures changing colors when joining a group)

There were also a few experiments that will probably never see the light of the day, mostly done in the first phase. One of them is of particular interest to me since it involves k-d trees - I worked with this structure in the past and it was too much of a rabbit hole to be useful for the project, but this is something that I expect to explore after GSoC (among other things).

![The sad, sad truth]({{ site.url }}/images/gsoc19/sad-truth.jpg)
{: .image-center} 

## He who controls the spice controls the universe

We are at the end of the project, and I'm glad to say that we were able to achieve all of the proposed and stretched goals. Due to the implementation process, a few of them naturally emerged, which actually saved us time. ut even if that was the case, there was still not enough time for two of the wishlisted tasks: improving the in-game behavior tree editor and a mechanism to create consolidated behavior trees. 

As concrete results from the project, we have a list of pull requests [here](https://github.com/search?q=org%3ATerasology+org%3AMovingBlocks+author%3Acasals+type%3Apr). The most important ones are:

* Core API, [PR #3708](https://github.com/MovingBlocks/Terasology/pull/3708): Implements a collective behavior structure capable of interpreting a single behavior tree for a group of actors. This was necessary due to some restrictions found in the original group model while working on the last tasks of Milestone #2. The scenario is discussed in [the GSoC Milestone 2 blog post](https://casals.io/code/gsoc-reaching-second-milestone/). This structure required the following classes to be added to the engine:

   - `org.terasology.logic.behavior.core.CollectiveBehaviorTreeRunner`
   - `org.terasology.logic.behavior.DefaultCollectiveBehaviorTreeRunner`
   - `org.terasology.logic.behavior.CollectiveInterpreter`

* Core API, [PR #3717](https://github.com/MovingBlocks/Terasology/pull/3717): Contains new asset definitions for Groups. The new Group asset allows group configurations to be loaded from JSON files (assets/groups/.group).  This structure required the following classes to be added to the engine:
   - `org.terasology.logic.behavior.asset.Group`
   - `org.terasology.logic.behavior.asset.GroupBuilder`
   - `org.terasology.logic.behavior.asset.GroupData`
   - `org.terasology.logic.behavior.asset.GroupFactory`
   - `org.terasology.logic.behavior.asset.GroupFormat`

* Core API, [PR #3727](https://github.com/MovingBlocks/Terasology/pull/3727): Group components, designed to work together and cover all collective behavior scenarios studied. The new components implemented are described in detail in [the GSoC Milestone 2 blog post](https://casals.io/code/gsoc-reaching-second-milestone/):
   - `org.terasology.logic.behavior.GroupMind`
   - `org.terasology.logic.behavior.GroupTag`

* WildAnimals module, [PR #28](https://github.com/Terasology/WildAnimals/pull/28) and [PR #31](https://github.com/Terasology/WildAnimals/pull/31): new creatures (RGB deers) used in our test scenarios and documents. This prefab package includes:
   - New prefabs: `redDeer`, `greenDeer`, and `blueDeer`. These new prefabs were created to be used later in different tutorials and examples:
      - `redDeer`: has the original `deer` as parent and contains only the `skeletalmesh`and the `Behavior` components (assigned behavior: `Behavior:critter`)
      - `greenDeer`: has the same content of the original `deer` prefab, minus the `Behavior` component
      - `blueDeer`: has `greenDeer` as parent and contains only the `skeletalmesh`and the `Behavior` components (assigned behavior: `Behavior:critter`)
   - Correspondent textures and materials
   - `WildAnimalsSpawnSystem` modifications so it could recognize the new prefabs

Another contribution was the improvement on the existing [Behaviors wiki](https://github.com/Terasology/Behaviors/wiki), featuring a modified topics structure and new content related to groups and collective behavior scenarios (a detailed changelog can be found [here]({{ site.url }}/docs/gsoc19/behaviors-wiki-changelog.txt)).

In addition to the pull requests, I also created a two new modules. The first one, called [WildAnimalsMadness](https://github.com/Terasology/WildAnimalsMadness/), was created to showcase the new structures. This module was integrated as part of the official organization repositories and it is publicly available to be used by game players and developers. The second module, called [TutorialGroups](https://github.com/Terasology/TutorialGroups/), was created to serve as a reference for the improved wiki.

This post marks the end of GSoC 2019, but definitely not the end of contributing with Terasology. We already have a few topics being discussed for the nest steps, originated mostly from the experiments performed in the first and second phases:

* Creating a merging mechanism for behavior trees. We discussed two methods so far:
   - Creating a new, empty behavior tree with a single parallel node under the root and then nesting the concurrent behavior trees. The main advantage of this method is that we can maintain different behavior trees combinations by using metadata (describing which trees are part of the merge), but we still need additional experiments to check situations in which conflicting nodes might appear.
   - Translating behavior trees into FSMs and using specific techniches for monimizing the resulting state machine. There's the obvious overhead of going back and forth to FSMs, though (and limitations to what can actually be translated).

* Creating an external behavior tree editor, instead of improving the existing one. During our experiments we identified some issues related to the serialization of the existing `.behavior` file format, so there's an attached discussion related to the choice between customizing our serializers *versus* moving to a standard format (JSON, YAML).

* The use of k-d trees within the core API as a general locator mechanism. Implementing a k-d tree as a mechanism to geographically contain groups was one of the experiments executed in the first phase of the project, and it was halted due to the necessity of additional, complex structures that were outside the GSoC project scope. The original implementation, however, can be used in conjunctions with other existing mechanisms and systems to increase the performance of proximity-related logic within the game.

![Because, well, wolves]({{ site.url }}/images/gsoc19/wolves.png)
{: .image-center} 

## Permanent links

* [GSoC proposal]({{ site.url }}/docs/gsoc19/GSoC 2019 - Arthur Casals.pdf)
* [GitHub project board](https://github.com/orgs/Terasology/projects/20)
* [Forum thread](https://forum.terasology.org/threads/gsoc-2019-collective-behavior.2255/)
* [Bibliography (BibTex format, subject to constant change)]({{ site.url }}/docs/gsoc19/gsoc-references.bib)
