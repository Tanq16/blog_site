---
title: Memos Notes in Home Lab
date: 2023-05-24 18:00:00 -0600
categories: [Home Server]
tags: [memos,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

[Memos](https://github.com/usememos/memos) is a note-taking application aimed at simplified note-taking with nothing but memos! The intent is - we store ideas quickly and forget, relying only on discovery and search based on text and tags to retrieve the information we need.

Not only is this application an awesome alternative to generic journaling, it can also work very well with technical people to store information in a concise and quick fashion. It supports markdown syntax, so it's easy to use, and is also lightweight with all data stored in an SQLite database. It supports pasting images for quick capture, and the notes can be exported as a PNG image too.

The app also supports AI operations when you add an OpenAI API token. All of this in a single Docker container!

To run this container, start by setting up the directory as follows &rarr;

```bash
mkdir -p $HOME/memos
```

Then, run the container as follows &rarr;

```bash
docker run --rm -d --name=memos \
-v $HOME/memos/:/var/opt/memos \
-p 5003:5230 \
ghcr.io/usememos/memos
```

An equivalent Docker compose template or a template to deploy using Portainer stacks is as follows &rarr;

```yaml
services: 
  memos:
    image: ghcr.io/usememos/memos
    container_name: memos
    networks:
      - servicesnet
    ports:
      - 5003:5230
    volumes:
      - /home/tanq/memos/:/var/opt/memos
```

Memos also has a Telegram bot, Raycast extension, and a Logseq plugin. Both computer and mobile platform app stores have apps to interact with a server. Memos has an authentication mechanism as well, which makes it easy to expose to the public internet as well.

And that's it! Enjoy taking notes (or rather memos)!
