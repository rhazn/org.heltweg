---
title: "Personal cloud with docker & kubernetes: Hardware is dead"
date: 2019-08-15
draft: true
description: "Personal cloud with docker & kubernetes: Hardware is dead"
tags:
    - kubernetes
    - docker
    - cloud
    - gke
    - gitlab
    - google cloud
---

# Why invest in a personal cloud with docker & kubernetes
##
- invest in a techstack you can work with so it gets boring to work with, you can focus on other things
- docker&kubernetes is future proof: widespread adoption, knowledge is worth something in any case
- can do anything from delivering SPAs to hosting databases to backends, no provider lock in like using Amazon SQS or firebase db structure
# Infrastructure as code
- enforces infrastructure description that is always correct and versioned in git
- if you come back in a while your infrastructure is always in one place, no need to search around per project
- can easily move infrastructure (locally minikube, different providers for kubernetes)
# Cloud services: Traefik working with Let's encrypt and Zookeeper
- ingress router for own control, gke ingress expensive
- https is standard and needed for PWA in any case, needs to be automated
- zookeeper easy to set up as k/v store for traefik, needed in cluster mode anyway
- cheaper instances in gke go down all the time (nice chaos monkey) so storage of certificates needs to happen in any case
# Organizing your projects
- central cloud you deploy into
- every project has it's own infrastructure otherwise (easy to set up anyway since infrastructure in code!)
- every project own docker repo
- user service accounts to allow cloud to pull the image
- deployment etc is in every project using service account that allows changing of that project in kubernetes only
# Further reading
- manuels blog

