---
title: "Deploy to google kubernetes engine using gitlab ci"
date: 2019-09-04
draft: false
description: "Deploy to google kubernetes engine using gitlab ci"
tags:
    - gitlab
    - gitlab ci
    - gke
    - google kubernetes engine
    - ci
    - continuous integration
    - continuous deployment
    - kubernetes
    - devops
---

In a previous blogpost I showed how I build and publish docker images on gitlab ci ([Build a docker image on gitlab ci](https://rhazn.com/posts/build-a-docker-image-on-gitlab-ci-and-publish-it-to-google-container-registry/))

Make sure to read that post first for an overview and permission setup.

# Update the kubernetes service with the new docker image

You can easily set up a deploy step using google's own cloud SDK docker images. Note the service account with the permissions to change the kubernetes setup is saved as "GCLOUD_K8S_KEY" variable here.

{{< freelance >}}

This job changes the image of my deployment for the app. You will need to change the last line in the script to whatever change you want to make to your kubernetes setup on deploy.

{{< highlight yml >}}
deploy:
  stage: deploy
  image: google/cloud-sdk:257.0.0
  script:
    - echo $GCLOUD_K8S_KEY | base64 -d > ${HOME}/gcloud-k8s-key.json
    - gcloud auth activate-service-account --key-file ${HOME}/gcloud-k8s-key.json
    - gcloud config set project personal-cloud-project-id
    - gcloud config set compute/zone your-compute-zone
    - gcloud container clusters get-credentials production
    - kubectl set image deployment/???-app ???-app=eu.gcr.io/docker-project-id/app:${CI_COMMIT_SHA}
  only:
    - master
  when: manual
{{< / highlight >}}

{{< mailinglist >}}

{{< aboutme >}}