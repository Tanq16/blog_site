---
title: HomeBox Asset Inventory in Home Lab
date: 2023-05-24 18:15:00 -0600
categories: [Home Server]
tags: [services,home-lab]
---

[HomeBox](https://hay-kot.github.io/homebox/) is an inventory and organization software aimed at providing a simple and easy-to-use platform. It's also very lightweight with SQLite as the database backend, and runs everything in a single container. Its primary use for me is maintaining an inventory of the items I buy/use/store at my home. It also gives a rough estimate of the total value of all items in my house.

The software has pretty comprehensive options if needed, like purchase dates, prices, warranty, selling information, etc. It also has some slick themes to help you enjoy the inventorying experience.

Deploying this is also very simple - start with setting up a directory for persistence &rarr;

```bash
mkdir -p $HOME/homebox
```

Then, run the container as follows &rarr;

```bash
docker run -d --rm \
--name homebox \
--restart unless-stopped \
--publish 3100:7745 \
--env TZ=America/Chicago \
--volume /home/tanq/homebox/:/data \
ghcr.io/hay-kot/homebox:latest
```

An equivalent Docker compose template or a template to deploy using Portainer stacks is as follows &rarr;

```yaml
services:
 homebox:
 image: ghcr.io/hay-kot/homebox
 container_name: homebox
 environment:
 - TZ=America/Chicago
 - HBOX_LOG_LEVEL=info
 - HBOX_LOG_FORMAT=text
 - HBOX_WEB_MAX_UPLOAD_SIZE=10
 volumes:
 - /home/tanq/homebox/:/data/
 ports:
 - 3100:7745
```

And that's it! Enjoy inventorying!
