---
title: "Build a progressive web app in docker with nginx to deploy to kubernetes or docker swarm"
date: 2019-08-24
lastmod: 2024-02-10
tags: ["software-engineering"]
slug: "build-a-progressive-web-app-in-docker-with-nginx-to-deploy-to-kubernetes-or-docker-swarm"
description: "Build a progressive web app delivered by nginx with docker to deploy it to docker swarm or kubernetes."
---

**This post was updated on 2024-02-10 to work with up-to-date nginx and node versions.**

With my personal cloud setup based on kubernetes done (you can read about it here: [https://heltweg.org/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper/](https://heltweg.org/posts/run-a-personal-cloud-with-traefik-lets-encrypt-and-zookeeper/)) it is time to actually deploy the first project into it. 

The easiest application to deploy is a pure client side single page application, packaged in a docker container with a webserver like nginx to deliver the files. Packaging the application into it's own container allows us to build a standardized container that can be run locally for testing or deployed to docker swarm and kubernetes.

Setting up and configuring our own HTTP server also allows for fine tuning of caching to achieve good lighthouse scores:

![Lighthouse scores, don't let the bugged checkmark fool you it passes the PWA tests ;)](/img/posts/build-a-progressive-web-app-in-docker-with-nginx-to-deploy-to-kubernetes-or-docker-swarm/scores.png#center)

# Building in docker
For this setup we build the app using docker. That way the app is always built with the same node version and can be consistently reproduced, regardless of installed software on the local computer.

The project here is a react application based on vite but it works similarly with any frontend framework:

```docker
FROM node:lts-alpine3.18 AS build

COPY . .

RUN npm ci

RUN npm run build
```

# Configuring nginx
For the nginx config I placed a config file into the project and checked it in. This config file is later on copied into the container that serves the SPA. To achieve good performance we

- enable gzip for HTML/CSS and JS files
- set up caching for any file for one year (because vite builds new file names with each production build that invalidates the cache on deploy)
- disable the cache for the actual index.html file (since we need to make the browser request the newest files)
- Redirect any request to index.html so the SPA router can handle them

You can see the complete config file here:
```
events {
}

http {
    include mime.types;
    default_type  application/octet-stream;
    sendfile on;

    server {
        listen 80;
        server_name _;

        gzip on;
        gzip_types text/html text/css application/javascript;

        root /var/www/;
        index index.html;

        # Force all paths to load either itself (complete filenames) or go through index.html.
        location / {
            try_files $uri /index.html;

            expires 1y;
            add_header Cache-Control "public";
        }

        location /index.html {
            try_files $uri /index.html;

            add_header Cache-Control "no-store, no-cache, must-revalidate";   
        }

        # Add custom cache headers for specific files
        location ~* \.(?:css|js|jpg|svg)$ {
            expires 30d;
            add_header Cache-Control "public";
        }
    }
}
```

# Building the final container
The end result will be a combination of a) building the SPA in  docker in the "build" step and then setting up a container from the nginx image and copying the JS from the build step as well as the checked in nginx config described above.



Finally we expose port 80 and start nginx to serve the files.

```docker
FROM node:lts-alpine3.18 AS build

COPY . .

RUN npm ci

RUN npm run build

FROM nginx:1.25.3

COPY --from=build /dist /var/www/
COPY ./_infrastructure/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

{{< aboutme >}}