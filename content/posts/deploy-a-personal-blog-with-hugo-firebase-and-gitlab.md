---
title: "Deploy a personal blog with Hugo, Firebase and Gitlab"
date: 2019-05-03
draft: false
slug: "deploy-a-personal-blog-with-hugo-firebase-and-gitlab"
description: "Deploy a personal blog with Hugo, Firebase and Gitlab"
showToc: true
TocOpen: true
tags:
    - gohugo
    - firebase
    - gitlab
    - continuous-deployment
---

# Blog requirements

I recently evaluated what how writing content and carving out a little personal space on the internet would look like for me and came up with a list of requirements:

* Own my content (No third party platforms as only distribution method. There was a time before medium and there will be a time after medium.)
* Easy to set up and maintain (I don't want to set up and host databases or complicated CMS systems and when I come back to this in a month it should be easy to update.)
* Long term secure storage (By not hosting my content with a third party but in git I can easily back it up, keep old revisions and there is no worry of the hosting provider going down.)

All these requirements are solved by using a static site generator and keeping the original files versioned in git. When setting up some for of continuous integration (for example with gitlab) it should be reasonably easy to maintaining for a long time.

I chose gohugo as a site generator due to the excellent setup article written by Fabian Gruber here: https://www.fabiangruber.de/posts/setup-and-deployment



# Automated builds

I set up an automated build using gitlab using the following gitlab-ci.yml:

```yml
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

After creating a new project in firebase you'll need to get a token that allows gitlab to make deployments on your behalf:

```bash
firebase login:ci
```

Set it up as a secret variable in gitlab called "FIREBASE_TOKEN".

To add firebase to the gohugo project navigate to the directory in the CLI can call:
```bash
firebase init
```

Armed with the token that allows gitlab to deploy in your name as well as a configured firebase project we can add the last step to the gitlab-ci file - deployment.

You can see the full gitlab-ci.yml file here:
```yml
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

{{< aboutme >}}