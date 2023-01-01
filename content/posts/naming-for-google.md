---
title: "Choose names for Google, not people"
date: 2022-01-22
slug: "chose-names-for-google-not-people"
tags: ["software-engineering"]
author: "Philip Heltweg"
description: ""
---

> "There are only two hard things in Computer Science: cache invalidation and naming things." 
> 
*- Phil Karlton*

AngularJS was released in 2010 and revolutionized modern frontend development. In 2016, the AngularJS team published a new framework, written in TypeScript and incompatible with AngularJS. They also made the baffling choice to call this framework Angular 2.

Mayhem ensued, confused developers talked to each other about incompatible frameworks. The excellent community documentation AngularJS had built in the form of blogs and StackOverflow answers became an active hindrance for people trying to learn Angular. Undeterred by this, the Angular team doubled down and kept releasing incompatible framework versions, only distinguished by a number.

Five years later, the team announced that AngularJS would be discontinued. Immediately they needed to clarify that they would not discontinue Angular but only AngularJS.

![Angular clearing up confusion, five years later](/img/posts/naming-for-google/tweet.png#center)
<p align="center"><i>Angular clearing up confusion, five years later</i></p>

Even with the best documentation, users are going to google for information. They are going to read blogs, ask on StackOverflow and copy answers. So to make it easy on them, please:

- Use unique names for major versions ("Android KitKat" instead of "Android 4.4"). They are easier to include in searches.
- Use error codes (like "Error 34") or slugs (like "ERROR_SOME_NAME") in addition to localized, readable messages. Unique strings enable users with different languages than English to contribute.
- As a user, use Software in English. It makes it easier to google for errors ;).

{{< aboutme >}}