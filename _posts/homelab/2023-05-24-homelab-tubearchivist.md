---
title: Tube Archivist in Home Lab
date: 2023-05-24 18:45:00 -0600
categories: [Home Server]
tags: [tubearchivist,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

[TubeArchivist](https://github.com/tubearchivist/tubearchivist) is a YouTube media management platform that allows indexing, searching, and storing YouTube videos. It uses [YT-DLP](https://github.com/yt-dlp/yt-dlp) to download videos. It's a comprehensive platform, but the primary advantage here is that it's deployable via Docker, making it an awesome and easy addition to home labs.

To run it as a containerized workload, start by setting up the directory as follows &rarr;

```bash
mkdir -p $HOME/tubearchivist/{media,cache,redis,es}
```

Then download the related Docker compose YAML from [GitHub](https://raw.githubusercontent.com/tubearchivist/tubearchivist/master/docker-compose.yml). Set the appropriate options in the file. An example final Docker compose template is as follows &rarr;

```yaml
services:
  tubearchivist:
    container_name: tubearchivist
    restart: unless-stopped
    image: bbilly1/tubearchivist
    ports:
      - 12008:8000
    volumes:
      - /home/tanq/tubearchivist/media:/youtube
      - /home/tanq/tubearchivist/cache:/cache
    environment:
      - ES_URL=http://tubearchivist-es:9200
      - REDIS_HOST=tubearchivist-redis
      - HOST_UID=1000
      - HOST_GID=1000
      - TA_HOST=192.168.1.82 # change this to your server IP
      - TA_USERNAME=tubearchivist # change this as necessary
      - TA_PASSWORD=tubearchivist # change this as necessary
      - ELASTIC_PASSWORD=tubearchivist # should be same as what's below
      - TZ=America/Chicago
    depends_on:
      - tubearchivist-es
      - tubearchivist-redis

  tubearchivist-redis:
    image: redis/redis-stack-server
    container_name: tubearchivist-redis
    restart: unless-stopped
    expose:
      - "6379"
    volumes:
      - /home/tanq/tubearchivist/redis:/data
    depends_on:
      - tubearchivist-es

  tubearchivist-es:
    image: bbilly1/tubearchivist-es
    container_name: tubearchivist-es
    restart: unless-stopped
    environment:
      - "ELASTIC_PASSWORD=tubearchivist" # should be same as what's above
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "xpack.security.enabled=true"
      - "discovery.type=single-node"
      - "path.repo=/usr/share/elasticsearch/data/snapshot"
    ulimits:
    memlock:
    soft: -1
    hard: -1
    volumes:
      - /home/tanq/tubearchivist/es:/usr/share/elasticsearch/data
    expose:
      - "9200"

volumes:
  media:
  cache:
  redis:
  es:
```

Running from this point here is pretty straightforward - simply run the following from the command line &rarr;

```bash
docker compose up # using compose v2
```

Otherwise, it can also be deployed as a stack in Portainer, which is my preferred way of deployment.

>One thing to note - in Portainer, if you specify a bind mount, it will automatically create the directories without the `mkdir` command. However, since Portainer usually runs as root within the container, it will prompt Docker to create the mount as root, which causes issues with the ElasticSearch and TubeArchivist setup due to permissions mismatch with the filesystem. So, it's best to run the `mkdir` command before deploying the stack on Portainer to fix that issue.
{: .prompt-info }

To be fair, while it can be used a full-fledged personal YouTube theater, for me the only use case is to download videos before a flight rather than getting YouTube premium. Still, the GUI helps a lot to get the videos easily and store them efficiently by channel in the HDD or a mounted flash drive (which is what I do). With that, go get your YouTube game in order!
