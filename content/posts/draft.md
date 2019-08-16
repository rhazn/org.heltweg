---
title: "Personal cloud with docker & kubernetes: Cloud setup"
date: 2019-08-16
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
# Kubernetes ingress with Traefik
As mentioned in my [last blog post](/posts/kubernetes-for-sideprojects-hardware-is-dead) 
- ingress router for own control
- gke supports load balancing for http by default but it is expensive
# HTTPs and Let's encrypt
- https is standard and needed for PWA in any case, needs to be automated
- traefik has support for requesting https certificates from let's encrypt
- link to docs
# GKE Preemptible nodes, your own chaos monkey
- "Preemptible VMs are Google Compute Engine VM instances that last a maximum of 24 hours and provide no availability guarantees."
# Shared K/V store for Traefik with Zookeeper
- zookeeper easy to set up as k/v store for traefik, needed in cluster mode anyway
- cheaper instances in gke go down all the time (nice chaos monkey) so storage of certificates needs to happen in any case