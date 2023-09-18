---
title: "Wanderlist - Setup"
date: 2023-09-18
tags: ["project", "software-engineering", "startup", "wip"]
slug: "setup"
author: "Philip Heltweg"
description: "Wanderlist - Setup"
---

# Overview
I am a big fan of separating a web client (typically as a single page application) from a backend that can be accessed using a REST api. The setup feels intuitive for me and can easily be extended for branching out into different clients, like mobile apps.

For development, I package everything as simple as I can using docker and docker compose. Docker compose is a nice middle ground that is easier to set up and run than kubernetes but means you are already "halfway there" if you want to deploy into a k8s cluster later on.

For Wanderlist, I wanted to try out some new tech that I had heard or read about but never used: [TailwindCSS](https://tailwindcss.com/) for the client and [Pocketbase](https://pocketbase.io/) for the backend.

# Pocketbase: The young gun
[Pocketbase](https://pocketbase.io/) is a very new open source "backend for your next SaaS and Mobile app". What makes it special is that it is just one self-contained file. Data is stored in a local SQLite database, uploads in the local file system or a S3 compatible store.

I have gotten to really enjoy working with SQLite, either during developing our open-source data pipeline modeling language [Jayvee](https://jvalue.github.io/jayvee/) or in the [JValue](https://jvalue.org/) project.

{{< figure src="pocketbase.png" title="Pocketbase backend" >}}

After a decade of wrestling complex backend setups or terrible serverless experiences like Firebase, I am excited to work with some simpler tools. And if I manage to outgrow SQLite, I am hyped. I'll write more about my experiences with pocketbase down the line.

# React with TailwindCSS: Pretty fly for an old man
Much has been written about which frontend framework is the best, I just like React. Fun fact, did you know it was open-sourced ten years ago in 2013? Time flies.

For the frontend styling, I wanted to try out [TailwindCSS](https://tailwindcss.com/). I am not sure how much I like the inline definition of styles but so far it has been weirdly productive. The code itself looks terrible, but the guideline seems to be to refactor any component you use multiple times into a reusable react component. Makes sense to me, we'll see.

For icons, I've found and enjoyed [Feather Icons](https://feathericons.com/), I think they look great.

{{< figure src="client.png" title="Scaffolded client with coming soon page" >}}

Not much to see there yet, so far, so good.

# Deploying a react single page application with pocketbase using docker compose: Group hug
Building and deployment of the whole app (both the SPA using react and the backed with pocketbase) is done using two simple files: A dockerfile and the docker compose file.

The dockerfile:
- builds the react frontend
- downloads and extracts pocketbase
- moves the built frontend assets into a folder that pocketbase serves
- starts the pocketbase server

```dockerfile
FROM node:lts-alpine3.18 AS build-web

COPY client-web client-web

WORKDIR client-web

RUN npm ci

RUN npm run build

FROM node:lts-alpine3.18

ARG PB_VERSION=0.18.2

RUN apk add --no-cache \
    unzip \
    ca-certificates

# download and unzip PocketBase
ADD https://github.com/pocketbase/pocketbase/releases/download/v${PB_VERSION}/pocketbase_${PB_VERSION}_linux_amd64.zip /tmp/pb.zip
RUN unzip /tmp/pb.zip -d /pb/

COPY --from=build-web client-web/dist /pb/pb_public

EXPOSE 8080

# start PocketBase
CMD ["/pb/pocketbase", "serve", "--http=0.0.0.0:8080"]
```

Finally, using docker compose the previously built docker image is instantiated as a service and exposed on port 80. To make sure no data is lost when restarting the service, a volume is defined that I can map to whereever I want on deployment.

```yaml
version: "3.9"
services:
    wanderlist:
        build: .
        ports:
            - "80:8080"
        volumes:
            - pocketbase_data:/pb/pb_data
volumes:
    pocketbase_data:
```

# Up next
- User sign in / log in

# Read the series
[Project Wanderlist](/projects/wanderlist/)