---
title: "Build a progressive web app in docker with nginx to deploy to kubernetes or docker swarm"
date: 2019-08-24
draft: false
slug: "build-a-progressive-web-app-in-docker-with-nginx-to-deploy-to-kubernetes-or-docker-swarm"
description: "Build a progressive web app delivered by nginx with docker to deploy it to docker swarm or kubernetes."
showToc: true
TocOpen: true
tags:
    - spa
    - pwa
    - docker
    - kubernetes
    - lighthouse
    - react
    - create-react-app
    - webworker
    - single page application
    - progressive web app
    - personal-cloud
---

With my personal cloud setup based on kubernetes done (you can read about it here: [https://heltweg.org/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper/](https://heltweg.org/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper/)) it is time to actually deploy the first project into it. 

The easiest application to deploy is a pure client side single page application, packaged in a docker container with a webserver like nginx to deliver the files. Packaging the application into it's own container allows us to build a standardized container that can be run locally for testing or deployed to docker swarm and kubernetes.

Setting up and configuring our own HTTP server also allows for fine tuning of caching to achieve good lighthouse scores:

![Lighthouse scores, don't let the bugged checkmark fool you it passes the PWA tests ;)](/img/posts/build-a-progressive-web-app-in-docker-with-nginx-to-deploy-to-kubernetes-or-docker-swarm/scores.png#center)

# Building in docker
For this setup we build the app using docker. That way the app is always built with the same node version and can be consistently reproduced, regardless of installed software on the local computer.

{{< freelance >}}

The project here is a react application based on create-react-app but it works similarly with any frontend framework:

```docker
FROM node:12.6.0 AS build

WORKDIR /

COPY package.json package-lock.json tsconfig.json ./

RUN npm ci

COPY ./src ./src
COPY ./public ./public

RUN npm run build --prod
```

# Configuring nginx
For the nginx config I placed a config file into the project and checked it in. This config file is later on copied into the container that serves the SPA. To achieve good performance we

- enable gzip for HTML/CSS and JS files
- set up caching for any file for one year (because create-react-app builds new file names with each production build that invalidates the cache on deploy)
- disable the cache for the actual index.html file (since we need to make the browser request the newest files)
- Redirect any request to index.html so the SPA router can handle them

You can see the complete config file here:
```conf
server {
    listen 80;
    server_name _;

    gzip on;
    gzip_types text/html text/css application/javascript;

    root /var/www/;
    index index.html;

    # Force all paths to load either itself (js files) or go through index.html.
    location /index.html {
        try_files $uri /index.html;

        add_header Cache-Control "no-store, no-cache, must-revalidate";    
    }

    location / {
        try_files $uri /index.html;

        expires 1y;
        add_header Cache-Control "public";
    }
}
```

# Building the final container
The end result will be a combination of a) building the SPA in  docker in the "build" step and then setting up a container from the nginx image and copying the JS from the build step as well as the checked in nginx config described above.



Finally we expose port 80 and start nginx to serve the files.

```docker
FROM node:12.6.0 AS build

WORKDIR /

COPY package.json package-lock.json tsconfig.json ./

RUN npm ci

COPY ./src ./src
COPY ./public ./public

RUN npm run build --prod

FROM nginx:1.16.1

COPY --from=build /build /var/www/
COPY ./k8s/config/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

{{< aboutme >}}