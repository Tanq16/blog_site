---
title: Wallabag in Home Lab
date: 2024-01-20 10:37:19 -0600
categories: [Home Server]
tags: [reader,bookmarks,wallabag,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

Wallabag is a read-it-later application for storing bookmarks. It automatically extracts the readable text and images and stores in a reader-format, which can also be exported as PDF, TXT, EPUB, and more. The best way to use it is to throw in blogs and links we find everyday and get back to them later when free.

Wallabag can be deployed using docker-compose. I recommend doing it with a container management service like Dockge or Portainer. First, create a directory for persistence as follows &rarr;

```bash
mkdir -p $HOME/wallabag/{data,pgdata} && \
mkdir -p $HOME/wallabag/data/db && \
chmod -R 777 $HOME/wallabag/
```

The `chmod` step is needed because wallabag tries to create an SQLite DB within the `data/db` directory and without the necessary permissions to the "other" groups, it will fail.

Then create the compose stack. The stack definition to use is as follows &rarr;

```yaml
version: "3.8"
services:
  wallabag:
    image: wallabag/wallabag
    ports:
      - 5002:80
    volumes:
      - /home/tanq/wallabag/data:/var/www/wallabag/data
      - /home/tanq/wallabag/images:/var/www/wallabag/web/assets/images
    environment:
      - SYMFONY__ENV__DOMAIN_NAME=http://192.168.86.32:5002
```

Another thing to mention here is that the above compose file will launch wallabag with a simple SQLite backend. Wallabag also supports MariaDB/MySQL and Postgres with Redis. The templates for those define additional containers for the DB and cache-DB. My use case is relatively limited, so I went with the simplest deployment. If you need a more robust backend, the appropriate config file can be found [here](https://hub.docker.com/r/wallabag/wallabag).

Well, that was a quick 101 for deploying Wallabag locally! Happy content-reading!
