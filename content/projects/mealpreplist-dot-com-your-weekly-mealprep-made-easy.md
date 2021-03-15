---
title: "mealpreplist.com - Your weekly mealprep, made easy"
date: 2017-09-27T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["spa","mealprep", "react", "firebase"]
slug: "mealpreplist-dot-com-your-weekly-mealprep-made-easy"
author: "Philip Heltweg"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "mealpreplist.com - Your weekly mealprep, made easy"
disableHLJS: false # to disable highlightjs
disableShare: false
searchHidden: true
cover:
    image: "" # image path/url
    alt: "" # alt text
    caption: "" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page

---

![mealpreplist.com Landingpage](/img/projects/mealpreplist-dot-com-your-weekly-mealprep-made-easy/landing.png#center)

Meal prepping is the practice of preparing multiple meals at once and either freezing them or keeping them in the fridge to eat over the upcoming days. There is an active community on reddit ([r/MealPrepSunday/](https://www.reddit.com/r/MealPrepSunday/)) and various blogs online.

![Combining three base recipes into one meal](/img/projects/mealpreplist-dot-com-your-weekly-mealprep-made-easy/choose-recipe.png#center)

Often mealprep containers are separated in three sections for a carb source, a protein source and vegetables. 

With mealpreplist I wanted to save my own, simple meal components for each of these sections and allow myself an easy way of combining them to a meal to prepare. 

Because I felt easy filtering of recipes and various other functions might have worth for other people to I decided to built the product with user accounts and the ability to pay for a premium account with additional features.

![Premium users can filter recipes by tags and see combined macros](/img/projects/mealpreplist-dot-com-your-weekly-mealprep-made-easy/filter-macros.png#center)
  
# Technology

## Google Firebase
With mealpreplist I wanted to allow users to submit their own recipes, rate and comment on them as well as buy premium accounts for additional functionality with Stripe. All of this would normally mean a fairly complex backend but by relying on Google's firebase I was able to implement everything without my own servers.

![Login with social providers using Firebase](/img/projects/mealpreplist-dot-com-your-weekly-mealprep-made-easy/login.png#center)

- I used Firestore to store recipe and ingredient data as well as user comments
- Firebase Storage allowed me to let users upload pictures of their recipes
- Firebase User Accounts/Login was used for user management
- Firebase Cloud Functions handle the interaction with Stripe and update user's premium status

![Firebase Cloud Functions and Stripe allow for premium accounts without ever handling passwords or CC data](/img/projects/mealpreplist-dot-com-your-weekly-mealprep-made-easy/premium.png#center)

## Frontend
Mealpreplist was my personal project to get experience in React development so the frontend is written in React using Typescript. During development I built the UI with react material ui though I later switched to some custom built components.

![Users can add their own recipe and upload images](/img/projects/mealpreplist-dot-com-your-weekly-mealprep-made-easy/add-recipe.png#center)

The data flow is based on redux using react-redux and redux-saga for data loading and other side effects like analytics tracking.

I made sure to make the app mobile friendly from the start, relying on flexbox to shrink and expand the display depending on screen size and media queries to hide not needed information on smaller screens.

![Displaying all selected recipes in one overview](/img/projects/mealpreplist-dot-com-your-weekly-mealprep-made-easy/recipe-display.png#center)

## CI
CI runs on gitlab, building every commit and allowing me to have an automated deploy to Firebase hosting.

# Marketing
For marketing I relied on building an email list using mailchimp and posting to the appropriate subreddit. The feedback was very good and the spike in visitors allowed me to gather a lot of email list subscribers but converted incredibly poorly (only one subscription from 6000 visitors).

![Frontpage reddit post on r/Mealprepsunday](/img/projects/mealpreplist-dot-com-your-weekly-mealprep-made-easy/reddit.png#center)

I kept up sending out regular email updates about development (mostly regarding feature requests from the reddit feedback) during the time spend on the project but never saw any improvement in subscriptions or visitor numbers.

![Analytics from 2018](/img/projects/mealpreplist-dot-com-your-weekly-mealprep-made-easy/analytics.png#center)

Without any continued marketing mealpreplist still gets a low number of regular visitors that use it for the recipes.

# Conclusion
I mainly built mealpreplist to save recipes for myself and to learn React and I am happy with the outcome for both. I learned a lot about the kind of products I want to build from it (e.g. I don't want to be in a business were I need to create a lot of content myself) and I also cemented what I learned from [Whatchr](../whatchr-dot-com-a-better-youtube-playlist-experience) regarding being more sure about the concrete value proposition of a product before trying to sell it.