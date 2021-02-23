---
title: "Kubernetes for sideprojects: Hardware is dead"
date: 2019-08-15
draft: false
description: "Kubernetes for sideprojects: Hardware is dead"
showToc: true
TocOpen: true
tags:
    - kubernetes
    - docker
    - cloud
    - gke
    - gitlab
    - google cloud
    - devops
    - traefik
    - infrastructure
    - infrastructure in code
    - container
    - personal-cloud
---

# Why invest in a personal cloud
It has never been easier to host your personal side projects. Tools like surge.sh or Heroku make it painless to run your code. And if all else fails the old reliable "drag and drop files to a ftp" is always there - so why invest time into setting up your own personal cloud with kubernetes?

My goal for technology is typically to find a setup that gets boring to work with because I know it well and can focus on delivering new functionality. For that a setup needs to be future proof (so it continues to work for a long time), generic (so I can use it for a wide range of applications and do not need to switch for every project) and not bound to any company or product.

![Photo by Taylor Vick on Unsplash](/img/posts/kubernetes-for-sideprojects-hardware-is-dead/taylor-vick-M5tzZtFCOfs-unsplash.jpg#center)

# Docker & kubernetes
Docker and Kubernetes check most of these boxes. Kubernetes has de-facto won the orchestration war for containerized applications and managed kubernetes offerings from all major cloud providers means there is no provider lock-in. As an open source project that is not bound to individual company or setup you retain flexibility. Lastly learning more about it is useful in any case - if you stop developing your side projects the devops knowledge you gained still looks good on a CV.

A further important point that separates kubernetes from similar offerings is that you can 100% forget about hardware while still being free to use standard technology. That means you do not need to maintain a set of servers if you use a managed kubernetes offering (like you would need to with docker swarm) but you are still not locked into any provider (like you would be by using AWS Beanstalk, Firebase or Heroku).

{{< freelance >}}

# Why not...?
My own personal journey has made me try out the following alternatives in the order they are written down here and I have abandoned them all.

1. Own servers
    - You need to maintain and update your own servers. If you do it without automation you will forget what you did.
    - The complexity and learning curve is close to kubernetes anyway. Instead of learning about deployments and stateful sets you will need to know about processes or package managers.
    - Scaling is harder.
2. Services (e.g. Firebase)
    - Very easy to set up and use, probably advisable if you work on one or a few projects
    - Using vendor specific tools (like firebase realtime database) locks you and the logic of your app into that vendor. What if they change their offering (price? functionality?)
    - Services are the hammer that makes very problem look like a nail. Instead of picking the best technology for a job you will start trying to shoehorn your problems to be solved by what is available.
3. Docker Swarm
    - I really liked it, very close to kubernetes but considerably easier
    - Sadly still forces you to manage your own servers to set up the swarm cluster, I could not find a "managed docker swarm" solution (if you know one, <a href="mailto:pheltweg@gmail.com">let me know</a>!)

# The value of infrastructure as code
By choosing kubernetes you also commit to keeping your infrastructure in code which has a plethora of benefits:

- it automatically and correctly documents your infrastructure and any changes you made by reading git logs
- you can easily redeploy it on new providers or e.g. locally for development
- all your infrastructure is in one place so you don't need to think about how each project solved a problem

# Organizing your projects
![Google Cloud setup for personal cloud](/img/posts/kubernetes-for-sideprojects-hardware-is-dead/google-cloud-setup.svg#center)

For my personal cloud I chose:

- A Google Cloud Kubernetes Engine Cluster as cloud
  - Traefik as ingress router to forward requests to my projects
  - Zookeeper for shared state between Traefik nodes
  - Let's Encrypt to automatically set up HTTPS
- Every project has it's own Google Cloud Container Registry as a private docker repository
- Service accounts allow the GKE cloud to pull the docker images of individual projects from the respective repository
- Infrastructure descriptions for the generic GKE setup and services is in one place, infrastructure descriptions for the individual projects is in their respective code
- Deployment is handled with gitlab



# Further reading
- Read my blog post about the actual setup of this cloud: [/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper](/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper).
- Also read this helpful article: [Traefik on a Google Kubernetes Engine Cluster managed by Terraform by Manuel Zapf](https://medium.com/google-cloud/traefik-on-a-google-kubernetes-engine-cluster-managed-by-terraform-ad871be8ee26)

{{< aboutme >}}