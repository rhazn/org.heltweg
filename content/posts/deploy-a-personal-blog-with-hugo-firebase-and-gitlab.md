---
title: "Deploy a personal blog with Hugo, Firebase and Gitlab"
date: 2019-05-03
tags: ["hugo", "firebase", "gitlab", "continuous-deployment"]
draft: false
---

# Blog requirements

I recently evaluated what how writing content and carving out a little personal space on the internet would look like for me and came up with a list of requirements:

* Own my content (No third party platforms as only distribution method. There was a time before medium and there will be a time after medium.)
* Easy to set up and maintain (I don't want to set up and host databases or complicated CMS systems and when I come back to this in a month it should be easy to update.)
* Long term secure storage (By not hosting my content with a third party but in git I can easily back it up, keep old revisions and there is no worry of the hosting provider going down.)

All these requirements are solved by using a static site generator and keeping the original files versioned in git. When setting up some for of continuous integration (for example with gitlab) it should be reasonably easy to maintaining for a long time.

I chose gohugo.io as a site generator due to the excellent setup article written by Fabian Gruber here: https://www.fabiangruber.de/posts/setup-and-deployment

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

# Automated builds

I set up an automated build using gitlab using the following gitlab-ci.yml:

{{< highlight yml >}}
stages:
  - build

build:
  image: monachus/hugo:v0.55.3
  stage: build
  script:
    - hugo
  artifacts:
    paths:
      - public/
{{< / highlight >}}

This uses a docker image for Hugo and builds the static page from the sources provided.

# Continuous delivery

I chose firebase to host the website because I have previous experience with it. It is very easy to create and deploy a static website as well as wire it up with a HTTPS domain.

After creating a new project in firebase you'll need to get a token that allows gitlab to make deployments on your behalf:

{{< highlight bash >}}
firebase login:ci
{{< / highlight >}}

Set it up as a secret variable in gitlab called "FIREBASE_TOKEN".

To add firebase to the gohugo project navigate to the directory in the CLI can call:
{{< highlight bash >}}
firebase init
{{< / highlight >}}

Armed with the token that allows gitlab to deploy in your name as well as a configured firebase project we can add the last step to the gitlab-ci file - deployment.

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

You can see the full gitlab-ci.yml file here:
{{< highlight yml >}}
stages:
  - build
  - deploy

build:
  image: monachus/hugo:v0.55.3
  stage: build
  script:
    - hugo
  artifacts:
    paths:
      - public/

deploy:
  image: devillex/docker-firebase:slim
  stage: deploy
  only:
    - master
  script:
    - firebase use <project-name> --token $FIREBASE_TOKEN
    - firebase deploy --only hosting -m "Pipe $CI_PIPELINE_ID Build $CI_BUILD_ID" --token $FIREBASE_TOKEN
  environment:
    name: production
    url: https://<project-name>.firebaseapp.com
  dependencies:
    - build
{{< / highlight >}}