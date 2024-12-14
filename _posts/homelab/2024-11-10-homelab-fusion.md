---
title: Leantime in Home Lab
date: 2024-11-10 13:37:19 -0400
categories: [Home Server]
tags: [rss,reader,fusion,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

[Fusion](https://github.com/0x2E/fusion) is an RSS feed aggregator which has an inbuilt reader interface for articles too. It's very easy to setup and is what I use for my reading day to day. It has a quick mark as read feature too, which allows me to manage my articles effectively.

To deploy this, use Docker in the command line like so &rarr;

```bash
docker run --rm -it -d -p 8080:8080 \
-v $(pwd)/fusion:/data \
-e PASSWORD="a strong password" \
rook1e404/fusion:latest
```

This will expose the app at `localhost:8080` and store related information in the `fusion` directory.

A better way to deploy it, specially using Portainer or Dockge is by using a compose template. First create a directory for it using &rarr;

```bash
mkdir $HOME/fusion
```

Then, launch the following template &rarr;

```yaml
services:
  fusion:
    image: rook1e404/fusion:latest
    ports:
      - 8080:8080
    environment:
      - PASSWORD=thisisasecurepassword
    volumes:
      - /home/username/fusion:/data
```

And that's it! Enjoy reading articles like a pro!
