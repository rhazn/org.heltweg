---
title: "Entity-Systems for typescript based games"
date: 2019-06-06
slug: "entity-systems-for-typescript-based-games"
description: "Entity-Systems for typescript based games"
tags: ["software-engineering"]
---

# Entity-Systems for typescript based games

For my latest game project [Frozzen](https://frozzen-client.appspot.com/) I want to explore how an external UI, build with Angular, would work for a browser based game. Since Angular is written in Typescript that means ideally the game should also use the same.

![Code of a typical data only component in Frozzen](/img/posts/entity-systems-for-typescript-based-games/frozzencode.png#center)

I have used [Artemis ODB](https://github.com/junkdog/artemis-odb) as framework for a Java based game in the past and liked it a lot. Entity-Systems are much better introduced by any of the huge amount of articles out there (for example the classic on [T=Machine](http://t-machine.org/index.php/2007/09/03/entity-systems-are-the-future-of-mmog-development-part-1/) but I feel they are especially well suited to Javascript/Typescript development.

If you work with a strict separation of logic into systems and data only into components there is a very natural way to serialize components, JSON. Whole levels can be expressed as an array of JSON data that is used to set up components. That is why I prefer a very basic but strict implementation like artemis over similar frameworks like PhaserJS.

![Frozzen is a turn based strategy game written in Typescript and artemists](/img/posts/entity-systems-for-typescript-based-games/frozzenscreen.png#center)

I started my development with artemists, a Typescript port of artemis by darkoverlordofdata. Unfortunately the code is a bit outdated and does not use import/export and can not be directly imported for newer Typescript versions (since it extends the built-in Array).

With darkoverlordofdata’s permission I did a quick update to the Typescript parts of the code only, adding import/export support and fixing the build for newer Typescript versions. [You can find the updated version here](https://github.com/rhazn/artemis-ts). If you are looking for an example of that framework in action you can play an example level of Frozzen [here](https://frozzen-client.appspot.com/).



{{< aboutme >}}