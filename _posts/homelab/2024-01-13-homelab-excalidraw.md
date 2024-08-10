---
title: Excalidraw in Home Lab
date: 2024-01-13 10:29:00 -0600
categories: [Home Server]
tags: [budgeter,excalidraw,whiteboard,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

## Excalidraw

Excalidraw is an open source infinite-canvas whiteboard software. It is primary local-first and the public hosted version is end to end encrypted, with support for live collaboration as well. The online version of the software has some additional features as well, such as generative AI features, a laser pointer for presentations, and mermaid diagram to excalidraw functionality.

For me though, I want to avoid sending what I draw over the internet. So, the local-network version of the application is a much better alternative for me. I don't want to use AI features or the laser pointer. That makes this application an awesome addition to my home lab services.

## Deployment

Here is a one-liner deployment command &rarr;

```bash
docker run --name excalidraw --rm -p 80:80 -d -t excalidraw/excalidraw:latest
```

After this, the container will be deployed at `http://localhost/`. A better way to deploy it is using docker-compose YAML either directly or through Dockge/Portainer/Yacht. Use this stack definition &rarr;

```yaml
version: "3.8"
services:
  excalidraw:
    restart: unless-stopped
    ports:
      - 5004:80
    image: excalidraw/excalidraw:latest
networks: {}
```

## Fin

Enjoy creating local-first charts on this beautiful whiteboard-style application!
