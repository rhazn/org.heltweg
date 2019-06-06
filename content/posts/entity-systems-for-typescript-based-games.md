---
title: "Entity-Systems for typescript based games"
date: 2019-06-06
tags: ["ecs", "entity component system", "game development", "gamedev", "typescript"]
draft: false
---

# Entity-Systems for typescript based games

For my latest game project [Frozzen](https://frozzen-client.appspot.com/) I want to explore how an external UI, build with Angular, would work for a browser based game. Since Angular is written in Typescript that means ideally the game should also use the same.

![Code of a typical data only component in Frozzen](/img/frozzencode.png)

I have used [Artemis ODB](https://github.com/junkdog/artemis-odb) as framework for a Java based game in the past and liked it a lot. Entity-Systems are much better introduced by any of the huge amount of articles out there (for example the classic on [T=Machine](http://t-machine.org/index.php/2007/09/03/entity-systems-are-the-future-of-mmog-development-part-1/) but I feel they are especially well suited to Javascript/Typescript development.

If you work with a strict separation of logic into systems and data only into components there is a very natural way to serialize components, JSON. Whole levels can be expressed as an array of JSON data that is used to set up components. That is why I prefer a very basic but strict implementation like artemis over similar frameworks like PhaserJS.

![Frozzen is a turn based strategy game written in Typescript and artemists](/img/frozzenscreen.png)

I started my development with artemists, a Typescript port of artemis by darkoverlordofdata. Unfortunately the code is a bit outdated and does not use import/export and can not be directly imported for newer Typescript versions (since it extends the built-in Array).

With darkoverlordofdata’s permission I did a quick update to the Typescript parts of the code only, adding import/export support and fixing the build for newer Typescript versions. [You can find the updated version here](https://github.com/rhazn/artemis-ts). If you are looking for an example of that framework in action you can play an example level of Frozzen [here](https://frozzen-client.appspot.com/).

<!-- Begin Mailchimp Signup Form -->
<link href="//cdn-images.mailchimp.com/embedcode/slim-10_7.css" rel="stylesheet" type="text/css">
<style type="text/css">
	#mc_embed_signup{background:#fff; clear:left; font:14px Helvetica,Arial,sans-serif; }
	/* Add your own Mailchimp form style overrides in your site stylesheet or in this style block.
	   We recommend moving this block and the preceding CSS link to the HEAD of your HTML file. */
</style>
<div id="mc_embed_signup">
<form action="https://appspot.us16.list-manage.com/subscribe/post?u=aa19a09de542be6eb75be25f6&amp;id=11ebcbdb71" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <div id="mc_embed_signup_scroll">
	<label for="mce-EMAIL">Get notified about new content</label>
	<input type="email" value="" name="EMAIL" class="email" id="mce-EMAIL" placeholder="email address" required>
    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_aa19a09de542be6eb75be25f6_11ebcbdb71" tabindex="-1" value=""></div>
    <div class="clear"><input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button"></div>
    </div>
</form>
</div>

<!--End mc_embed_signup-->