---
title: "Single page application hosting: Progressive web app and 100 percent lighthouse scores"
date: 2019-08-18
draft: true
description: "Single page application hosting: Progressive web app and 100 percent lighthouse scores"
tags:
    - spa
    - pwa
    - docker
    - kubernetes
    - lighthouse
    - react
    - webworker
    - single page application
    - progressive web app
---

{{< figure src="/img/posts/single-page-application-hosting-progressive-web-app-and-100-percent-lighthouse-scores/scores.png" caption="Lighthouse scores">}}

{{< highlight docker >}}
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
{{< / highlight >}}

{{< highlight conf >}}
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
{{< / highlight >}}

{{< aboutme >}}