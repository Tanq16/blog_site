---
title: Plex Media Server in Home Lab
date: 2022-04-24 12:00:00 +0500
categories: [Home Server]
tags: [home-lab,plex]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

**Note:** Jellyfin is a pretty similar option with an easier setup and one that I switched over to from Plex. Check out [my guide](https://blog.tanishq.page/posts/homelab-jellyfin/).

## Deployment

[DockerHub](https://hub.docker.com/r/plexinc/pms-docker/)

The best way to use Plex Media Server in my opinion is to use a docker container. Plex offers a docker image that can be used. The command to run the container is as follows &rarr; 

```bash
mkdir -p $HOME/plex/{config,transcode}
```

```bash
# replace the media mount with an appropriate directory
docker run \
-d --restart=unless-stopped --net=host \
--name plex \
-e TZ="America/Chicago" -e PLEX_UID=1000 -e PLEX_GUID=1000 \
-e PLEX_CLAIM="" \
-v $HOME/plex/config:/config \
-v $HOME/plex/transcode:/transcode \
-v /media/tanq/Tanishq/Media:/data \
plexinc/pms-docker
```

Ensure that the `PLEX_CLAIM` token is accurate and taken from [this link](https://www.plex.tv/claim). The three main volume mounts here are needed as follows &rarr;

- `/config` and `/transcode` &rarr; used to maintain state, watch history, etc.
- `/data` &rarr; preferably mount an external hard disk with shows, movies, etc.

## Setup and Data Format

After starting the container, if the plex claim code was included, log in with the account at `http://<serverIP>:32400/web` to setup the server with your Plex account.

For the main media store, the data should be formated as best as possible. The way Plex works is it reads the metadata of files and names of folders and files to guess the TV show or movie name and pull details from multiple sources to display that information to the user. But it often misses, specially if that metadata isn't present or data has been downloaded from multiple sources. Therefore, it's best to follow a directory and naming structure that Plex can understand and get accurate details for.

For TV shows and Anime, [TVDB](https://thetvdb.com) is the best source and for movies, [TMDB](https://www.themoviedb.org) is the best. The idea is that the naming for each folder (or a single file if needed for movies) should be named with the following format &rarr;

```
<NAME> (Year) {tvdb-<id>}
```

>The `tvdb` and `tmdb` IDs can be obtained by visiting the show on the respective DB sites. The sites also list the full name and year, both of which are also important for perfect matches.
{: .prompt-tip }

Ensuring this naming scheme is the best way to avoid any issues with content match. After that, everything's good to go!
