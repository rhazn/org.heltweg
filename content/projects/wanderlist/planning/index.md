---
title: "Wanderlist - Planning"
date: 2023-09-05
tags: ["project", "software-engineering", "startup", "wip"]
slug: "planning"
author: "Philip Heltweg"
description: "Wanderlist - Planning"
---

# Wanderlist

One of my favorite ways to explore a new city are free walking tours in which a local guide takes you around the city, shows you interesting places and shares their knowledge with you. At the end of these tours, guides often share their favorite spots and 'secret tips' with you, usually as a way to keep contact or get you to follow their Instagram. We'd take these recommendations and, if there is time, plan some of our travel days around visiting them, eating there or whatever else makes sense.

I always thought, sharing local recommendations would be a cool idea for an app. You gather curated places from friends and locals and have an easy way to plan short trips centered around them. When a lazy Sunday is coming up, just open your app and plan (or let it generate) an interesting afternoon trip to some local point of interest and finishing in a good restaurant.

I am sure there are a million apps that do this, let's add one more.

# Planning

With these thoughts in mind, I sat down for some sketches with [GoodNotes](https://www.goodnotes.com/) on my iPad. This is currently my favorite way for very low fidelity mockups, basically just a way to quickly get my initial ideas 'on paper'. I've done some follow up mockups in [Excalidraw](https://excalidraw.com/) and the overkill-solution I am aware of is of course [Figma](https://www.figma.com/). If anyone has one suggestions for good mockup tools, I'd be delighted to hear about them.

We will need some form of navigation, which will also define the first functionality we want to build. If done well, I think a bit of gamification by collecting points of interests would be nice. So I added a tab to manage your point of interests and a tab to manage 'wanderlists' (basically lists of points of interests to combine into a trip). Then we just need some way to create a new wanderlist and some stats and account management views for good measure.

{{< figure src="layout.png" title="Screen 1: The initial navigation and planning a new Wanderlist" >}}

For every screen, we just need some high-level idea of what it is about. Both managing points of interests and wanderlists are just lists with additional filters. For the points of interests, we also need a way to find new ones. Initially, I assume this will be adding a code from someone to get their list of local suggestions, maybe later on this could be scanning QR codes or opening some random chests (I have been damaged by playing mobile games).

{{< figure src="lists.png" title="Screen 2: Point of interest and Wanderlist management" >}}

Finally, let us add the boring screens we can actually start working on. We will start by creating a skeleton app with a coming soon screen and some user log-in. Because 'As a user, I want to be able to log in so that I can use the app.' is the first story (and lie) every product owner writes to start a new product.

{{< figure src="comingsoon-and-account.png" title="Screen 3: Coming soon screen and account management" >}}

# Up next
Next up, we need to actually write some scaffolding code to get the app running and show some coming soon screens :).

[Next post: Setup](/projects/wanderlist/setup)

# Read the series
[Project Wanderlist](/projects/wanderlist/)