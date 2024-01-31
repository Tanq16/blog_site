---
title: Bookmark Manager in Home Lab
date: 2024-01-15 00:03:10 -0600
categories: [Home Server]
tags: [bookmarks,linkwarden,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

## Intro

There are probably a hundred different ways to manage links and bookmarks. People use Chrome's bookmarks, some use an extension like Toby, and others store them on GitHub. There are also many bookmark manager apps, dashboard applications, and even command-line tools that can manage bookmarks. Once again, I turn to the amazing open-source community in search of projects.

I identified 3 projects that I liked the best &rarr;

- [Linkwarden](https://github.com/linkwarden/linkwarden)
- [Linkding](https://github.com/sissbruecker/linkding)
- [LinkAce](https://github.com/Kovah/LinkAce)

Of these, I went with Linkwarden, but really - all of them are amazing. Why not use Chrome or something simpler? Just because I wanted something in my home lab and easily accessible across multiple computers and accounts. Plus all of this is local-first, so maintaining security becomes easy.

## Deployment

Linkwarden can be deployed using docker-compose. I recommend doing it with a container management service like Dockge or Portainer. First, create a directory for persistence as follows &rarr;

```bash
mkdir -p $HOME/linkwarden/{data,pgdata}
```

Then create the compose stack. The stack definition to use is as follows &rarr;

```yaml
version: "3.8"
services:
  postgres:
    image: postgres:16-alpine
    env_file: .env
    volumes:
      - /home/tanq/linkwarden/pgdata:/var/lib/postgresql/data
  linkwarden:
    env_file: .env
    environment:
      - DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres:5432/postgres
    image: ghcr.io/linkwarden/linkwarden:latest
    ports:
      - 5007:3000
    volumes:
      - /home/tanq/linkwarden/data:/data/data
    depends_on:
      - postgres
networks: {}
```

The `.env` file (Dockge, for example, provides a section to specify the contents easily) should contain the following &rarr;

```
NEXTAUTH_SECRET=linkwarden # change
NEXTAUTH_URL=http://localhost:5007/api/v1/auth # port must match the exposed one
POSTGRES_PASSWORD=linkwarden # change
```

With this, the applications will be deployed.

## Fin

Well, hope that's a quick 101 for deploying Linkwarden locally! One of the best features of Linkwarden (and several other apps too) is link archives. Using this functionality, Linkwarden can store a screenshot, image, and archive.org snapshot for each of the links. This is pretty useful. Happy bookmarking!
