---
title: "Personal cloud with docker & kubernetes: Cloud setup"
date: 2019-08-15
draft: true
description: "Personal cloud with docker & kubernetes: Cloud setup"
tags:
    - kubernetes
    - docker
    - cloud
    - gke
    - gitlab
    - google cloud
    - devops
---
# Cloud services: Traefik working with Let's encrypt and Zookeeper
- ingress router for own control, gke ingress expensive
- https is standard and needed for PWA in any case, needs to be automated
- zookeeper easy to set up as k/v store for traefik, needed in cluster mode anyway
- cheaper instances in gke go down all the time (nice chaos monkey) so storage of certificates needs to happen in any case