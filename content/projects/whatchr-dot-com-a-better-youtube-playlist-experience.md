---
title: "Whatchr.com - a better youtube playlist experience"
date: 2016-10-04
draft: false
description: "Whatchr.com - a better youtube playlist experience"
tags:
    - spa
    - youtube
---
Whatchr was born of a personal problem. I regularly watch let's play content on youtube (following a group of content creators with their series), mainly following the channels of [The Yogscast](http://yogscast.com/).

Youtube's focus is on either suggesting content it thinks you find interesting on your "Suggestions" or content divided in daily chunks in your "Feed". There is no nice way to watch multiple series other than bookmarking playlists with watch positions.

I set out to change that for myself, building a small "Netflix for Youtube".

{{< figure src="/img/projects/whatchr-dot-com-a-better-youtube-playlist-experience/landingpage.png" caption="Whatchr's landingpage - I did not want to use a gaming looking product, looking for a clean look instead">}}

Originally I had also planned to tag videos with the content creator appearing in them so you can follow your favorite content creator over multiple channels.

# Technology

Watchr would need to solve a few typical web app problems: Managing and displaying data and  allow users to create accounts and log in (to save their individual watch positions). In addition - since all data about playlists and videos came from the youtube API - I also needed to build a caching layer to speed up response time and not run into API limits.

## The frontend - Angular with Typescript, SCSS, HTML

At the time of creation Angular was releasing Beta versions. Since I had previous experience with AngularJS and Typescript I chose Watchr as personal learning project for the new stack of Angular and Observables with RXJS. 

{{< figure src="/img/projects/whatchr-dot-com-a-better-youtube-playlist-experience/search.png" caption="I implemented a series search to play around with observables and debouncing">}}

While I build all features and a basic material design the talented [Simon Brix](https://simonbrix.dk/) helped me with the final design and did most of the implementation of it in SCSS/HTML. Thanks!

## The API - NodeJS with express/Typescript, PostgreSQL

With the goal of staying in a unified language stack I decided to write the API in NodeJS (using Typescript) with the express framework. 

An initial version of the App used CouchDB as a database but the added overhead of writing views for every data selection seemed tedious. During development I switched to PostgreSQL and their JSON support. It allowed for fast development (since I am familiar with PostgreSQL), is performant and allows for easy query building due to the built-in JSON operators.

For user accounts I decided to implement Google login using Passport. That allowed me to not directly save any user data (always a good idea to save as little important data as possible) but the user id and name.

## The Import - NodeJS with Typescript, PostgreSQL, Youtube API, Cronjobs

Importing new data would need to happen regularly to keep the database up to date. While the youtube API is very good I needed to focus on a manually curated set of channels to import. I therefor decided on writing a small script that could be called by a cronjob and import all playlists and videos of a specific channel.

{{< figure src="/img/projects/whatchr-dot-com-a-better-youtube-playlist-experience/explore.png" caption="The explore page showed all recently updated content, new content etc">}}

The script was written in Typescript (again, keeping the same techstack) and used the same database as the API to save JSON objects directly from the youtube API. I used a combination of observables (with RXJS) and functional programming to make the import code easily manageable while being very efficient,

## Hosting - DigitalOcean

When everything was done I implemented deployment on a digital ocean droplet using gitlab CI. 
I disabled any log in on the droplet but public key authentication, configured a private key as CI variable in gitlab and set it up during the build job. After the project was build gitlab rsynced the files to the droplet and restarted the node process.

{{< figure src="/img/projects/whatchr-dot-com-a-better-youtube-playlist-experience/seriesdetail.png" caption="A series detail screen">}}

I configured the import cronjobs manually with a channel id for each job. The database was hosted on the same droplet and the whole server backed up automatically with daily snapshots.

If I would redo this project I would definitely improve the deployment workflow so I do not need to manually configure cronjobs as getting back to the knowledge how and where to do that was one of the main pain points of the project.

# Marketing

I posted once to the respective reddit [https://www.reddit.com/r/Yogscast/comments/5tdtio/follow_yogscast_playthroughs_easier_give_me_your/](https://www.reddit.com/r/Yogscast/comments/5tdtio/follow_yogscast_playthroughs_easier_give_me_your/) but did not do any marketing otherwise.

Not surprisingly I was the only active user for the time Whatchr was alive.

# Conclusion

Whatchr was a fun project and allowed me to learn a big set of technologies while scratching my own itch. I learned a lot about the youtube API and the internal organisation of their data.

{{< figure src="/img/projects/whatchr-dot-com-a-better-youtube-playlist-experience/dashboard.png" caption="The dashboard shows all the series you are currently watching">}}

Overall the project gave me an appreciation for better planning of a value proposition beforehand (because I regularly had problems articulating what problem it solves) and marketing.

{{< aboutme >}}