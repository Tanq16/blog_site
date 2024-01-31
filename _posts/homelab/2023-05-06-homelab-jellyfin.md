---
title: Jellyfin in Home Lab
date: 2023-05-06 12:01:00 -0600
categories: [Home Server]
tags: [jellyfin,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

## Deployment

[Jellyfin Container Installation](https://jellyfin.org/docs/general/installation/container)

Plex is a great home media server, but Jellyfin has recently gained a lot of traction. Also, it is much simpler to setup directly and doesn't need the activation for mobile devices. Plex requires a web-based account to claim a server and then use the account to login and interact with the server. Frontend webapps/apps are needed to interact with the sevrer. Jellyfin is different as it just loads the web service on the web UI that directly interacts with the backend, rather than being a separate app component that can connect to multiple backends. Basically, Jellyfin is pretty similar to Plex in its offering but with added simplicity.

The best way to use Jellyfin Media Server is to use its docker container. Jellyfin offers a docker image that can be used to run the container. First create a directory as follows &rarr;

```bash
mkdir -p $HOME/jellyfin/{config,cache}
```

Then launch the container as follows &rarr;

```bash
# replace the media mount with an appropriate directory
docker run -d \
--name=jellyfin \
-p 8096:8096 \
-v $HOME/jellyfin/config:/config \
-v $HOME/jellyfin/cache:/cache \
-v /media/tanq/Tanishq/Media/:/data/media \
--restart unless-stopped \
jellyfin/jellyfin
```

An equivalent Docker compose template or a template to deploy using Portainer stacks is as follows &rarr;

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    restart: unless-stopped
    # expanding $HOME in volumes so that Portainer can deploy correctly
    # since $HOME means something else in the Portainer container
    volumes:
      - /home/tanq/jellyfin/config:/config
      - /home/tanq/jellyfin/cache:/cache
      - /media/tanq/Tanishq/Media/:/data/media
    ports:
      - 8096:8096
```

The good part here is that there is no need to do a claim and link an online account like in Plex. The three main volume mounts here are quite similar to the ones used by Plex needed for the follows &rarr;

- `/config` and `/cache` &rarr; used to maintain state, watch history, etc.
- `/data` &rarr; preferably mount an external hard disk with shows, movies, etc.

## Setup and Data Format

For the main media store, the data should be formated as best as possible. Jellyfin has a similar show processor to Plex. It reads the metadata of files and names of folders and files to guess the TV show or movie name and pull details from multiple sources to display that information to the user. Like Plex, this is not often accurate; therefore, it's best to follow a naming structure just like for Plex.

For TV shows and Anime, [TVDB](https://thetvdb.com) is the best source and for movies, [TMDB](https://www.themoviedb.org) is the best. TMDB also seems to contain everything related to shows like TVDB. The idea is that the naming for each folder (or a single file if needed for movies) should be named with the following format &rarr;

```
<NAME> (Year) [tvdbid-<id>]
```

>The `tvdbid` and `tmdbid` IDs can be obtained by visiting the show on the respective DB sites. The sites also list the full name and year, both of which are also important for perfect matches.
{: .prompt-tip }

Ensuring this naming scheme is the best way to avoid any issues with content match. After that, everything's good to go!

Music can also be added to Jellyfin and the nomenclature of that is pretty simple - just Artist/Album/Song. Also, like Plex, Jellyfin also has support for Live TV and Photos media.

## Migration from Plex

If you have data named according to Plex with `{tvdb-xxxxx}` and `{tmdb-xxxxx}` format, all the files can easily be converted to Jellyfin-supported format. Run the following python code via an interpreter in the directory that contains your media (one type at a time like tv-shows, then movies, then others).

```python
import os
data = os.listdir()
for i in data:
    os.rename(i, i.replace('tmdb-', 'tmdbid-').replace('tvdb-', 'tvdbid-').replace('{', '[').replace('}', ']'))
```

This converts it into the correct format, then Jellyfin should be able to recognize all media accurately.
