---
title: "Build a docker image on gitlab ci and publish it to google container registry"
date: 2019-09-03T11:30:03+00:00
slug: "build-a-docker-image-on-gitlab-ci-and-publish-it-to-google-container-registry"
# weight: 1
# aliases: ["/first"]
tags: ["software-engineering"]
author: "Philip Heltweg"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "Build a docker image on gitlab ci and publish it to google container registry"
disableHLJS: false # to disable highlightjs
disableShare: false
searchHidden: true
cover:
    image: "" # image path/url
    alt: "tagged images" # alt text
    caption: "tagged images" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page

---

In previous blogposts I explained my concept of a personal cloud for my own projects ([Kubernetes for Sideprojects](https://heltweg.org/posts/kubernetes-for-sideprojects-hardware-is-dead/)) and how I set it up ([Run a personal cloud with Traefik, Let's encrypt and Zookeeper](https://heltweg.org/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper/)). I also showed how I packaged a PWA project with docker ([Build a PWA in docker](https://heltweg.org/posts/build-a-progressive-web-app-in-docker-with-nginx-to-deploy-to-kubernetes-or-docker-swarm/)).

With all those ingredients ready to go the last hurdle to solve is building the docker image automatically as well as publishing it to a private container registry so I can deploy it to my cloud from there.

# Overview

The goal of the setup is to:

- build a docker image from any git hash using gitlab ci
- push that docker image to a google container registry tagged with the git hash
- update the kubernetes service description in the personal cloud project to pull the new docker image

# Permissions

To enable gitlab to do these actions on our behalf we need to set up service accounts. We need

- one account that allows gitlab to upload a new docker image to the container registry
- an account that allows it to change our kubernetes setup in the personal cloud registry

You can create these service accounts in the "IAM & admin" -> "Service accounts" section of google cloud. Make sure to download and save the generated json file.

We will also need to allow the personal cloud project to pull the docker image from the container registry that is in a different project. For that I followed this excellent blogpost by Alexey Timanovskiy ([Using single Docker repository with multiple GKE projects](https://medium.com/google-cloud/using-single-docker-repository-with-multiple-gke-projects-1672689f780c)).

In general the permissions needed to interact with docker images in the container registry are related to only "Storage" so for pulling docker images the account will need "Storage Object Viewer" permissions only. For adding images to the registry "Storage Admin".

# Publish a docker image with gitlab ci

To allow gitlab ci to use your service account you need to save the content of the json files as a base64 encoded variable in the backend. You can find the setting under "Settings" -> "CI /CD" -> "Variables". Be careful with this data since it is security relevant. The variables here will be available as environment variables during your jobs.

![The service account variables](/img/posts/build-a-docker-image-on-gitlab-ci-and-publish-it-to-google-container-registry/gitlab-ci-variables.png#center)

I use the following gitlab ci stage to build and publish a project. Note that it only runs manually and for master. In this case it uses the service account saved in GCLOUD_SERVICE_KEY:

```yml
publish:
stage: publish
image: docker:19.03.1
services: - docker:dind
variables:
DOCKER_DRIVER: overlay
script: - echo $GCLOUD_SERVICE_KEY | base64 -d > ${HOME}/gcloud-service-key.json - docker login -u \_json_key --password-stdin https://eu.gcr.io < ${HOME}/gcloud-service-key.json
    - docker build -t eu.gcr.io/projectid/app:${CI_COMMIT_SHA} . - docker push "eu.gcr.io/projectid/app:\${CI_COMMIT_SHA}"
only: - master
when: manual
```

![The resulting tagged images in gcr](/img/posts/build-a-docker-image-on-gitlab-ci-and-publish-it-to-google-container-registry/tagged-images.png#center)

{{< aboutme >}}
