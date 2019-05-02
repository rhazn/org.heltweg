---
title: "Deploy a personal blog with Hugo, Firebase and Gitlab"
date: 2019-05-02
tags: ["hugo", "firebase", "gitlab", "continuous-deployment"]
draft: false
---

# Blog requirements

I recently evaluated what how writing content and carving out a little personal space on the internet would look like for me and came up with a list of requirements:

* Own my content (No third party platforms as only distribution method. There was a time before medium and there will be a time after medium.)
* Easy to set up and maintain (I don't want to set up and host databases or complicated CMS systems. They are useless for the low volume writing I'll be doing and I have to do it often enough at my day job.)
* Long term secure storage (By not hosting my content with a third party but in git I can easily back it up, keep old revisions and there is no worry of the hosting provider going down.)

Basically all these requirements are solved by using a static site generator and keeping the original files versioned in git. When setting up some for of continuous integration (for example with gitlab) it should be reasonably easy to maintaining for a long time.

I chose gohugo.io as a site generator due to the excellent setup article written by Fabian Gruber here: https://www.fabiangruber.de/posts/setup-and-deployment

# Automated builds

I set up an automated build using gitlab using the following gitlab-ci.yml:

```
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
```

This uses a docker image for Hugo and builds the static page from the sources provided.

# Continuous delivery

I chose firebase to host the website because I have previous experience with it. It is very easy to create and deploy a static website as well as wire it up with a HTTPS domain.

To enable it in the gohugo.io project I created a new project in firebase and called
```
firebase init
```
in the CLI afterwards.

With gitlab CI it is possible to provide secret variables to builds. I generated a firebase token by calling:

```
firebase login:ci
```

and set it up as a secret variable called "FIREBASE_TOKEN".

You can see the full gitlab-ci.yml file here:
```
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
```