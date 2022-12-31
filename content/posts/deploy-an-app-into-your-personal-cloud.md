---
title: "Deploy an app into your personal cloud"
date: 2019-09-04
draft: false
slug: "deploy-an-app-into-your-personal-cloud"
description: "Deploy an app into your personal cloud"
showToc: true
TocOpen: true
tags:
    - docker
    - gitlab
    - gitlab ci
    - gcr
    - google container registry
    - google kubernetes engine
    - gke
    - ci
    - continuous integration
    - continuous deployment
    - kubernetes
    - devops
    - personal-cloud
---

As the final step in this long series of blogposts we are going to deploy a simple webapp in a docker container to my personal cloud. For context here is the personal cloud setup with Traefik/Let's Encrypt ([Run a personal cloud with Traefik, Let's encrypt and Zookeeper](https://heltweg.org/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper/)).

In previous blogposts I also described how I built the app ([Build a PWA in docker](https://heltweg.org/posts/build-a-progressive-web-app-in-docker-with-nginx-to-deploy-to-kubernetes-or-docker-swarm/)).

# App deployment

The deployment runs the docker container. If you follow my personal cloud setup make sure to use more than one replica as the nature of preemtive VMs means one replica might randomly get shut down.

Note that the image will be set and updated by gitlab ci as described here: ([Deploy to google kubernetes engine using gitlab ci](https://heltweg.org/posts/deploy-to-google-kubernetes-engine-using-gitlab-ci/)).

```yml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: ???-app
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: ???-app
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: ???-app
        image: "eu.gcr.io/???/app:latest"
```

# The service and traefik ingress
```yml
apiVersion: v1
kind: Service
metadata:
name: ???-app
spec:
selector:
    app: ???-app
ports:
- name: web
    port: 80
    targetPort: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
name: ???-app
annotations:
    kubernetes.io/ingress.class: traefik
    traefik.frontend.passHostHeader: "false"
    traefik.frontend.priority: "1"
spec:
rules:
- host: app.???.com
    http:
    paths:
    - path: /
        backend:
        serviceName: ???-app
        servicePort: web
```

# Updating the traefik config
Updating the traefik config is important so traefik requests a new HTTPS certificate for the app from Let's encrypt. You will need to add this line to the traefik toml file that is described here ([Run a personal cloud with Traefik, Let's encrypt and Zookeeper](https://heltweg.org/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper/)):

```yml
[[acme.domains]]
    main = "app.???.com"
```

# DNS
Now you can just point the A record of your domain to the traefik external IP and the rest will automatically be handled by your personal cloud :).

![The final result: The app runs in your personal cloud <3](/img/posts/deploy-an-app-into-your-personal-cloud/final-result.png#center)



{{< aboutme >}}