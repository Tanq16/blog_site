---
title: Services running in my Home Server
date: 2022-04-24 12:00:00 +0500
categories: [Home Server]
tags: [services]
---

# BEING DEPRECATED in favor of single posts

The following services runs on my Home Server &rarr; 

1. Plex Media Server
2. Snapdrop (local network variant)
3. PiHole DNS
4. LinkDing
5. MD Dumpster

# Plex Media Server

Command to run the server &rarr; 

```bash
docker run \
-d --restart=unless-stopped --net=host \
--name plex \
-e TZ="America/Chicago" -e PLEX_UID=1000 -e PLEX_GUID=1000 \
-e PLEX_CLAIM="" \
-v /home/tanq/plex_data/config:/config \
-v /home/tanq/plex_data/transcode:/transcode \
-v /media/tanq/Tanishq/Media:/data \
plexinc/pms-docker
```

The server is running off the docker image built by LinuxServerIO &rarr; 

Resource &rarr; [LSIO Plex Image](https://docs.linuxserver.io/images/docker-plex)

# Snapdrop (local network variant)

Command to run the server &rarr; 

```bash
docker run -d \
--name=snapdrop_variant \
-e PUID=1000 \
-e PGID=1000 \
-e TZ=America/Chicago \
-p 80:80 \
-p 443:443 \
-v /home/tanq/snapdrop_config:/config \
--restart unless-stopped \
tanq16/linuxserver_snapdrop_local_only
```

The server is running off a personal local network only variant of the LinuxServerIO image &rarr; 

1. [Modified Snapdrop Image](https://github.com/Tanq16/docker-snapdrop)
2. [LSIO Snapdrop Image](https://docs.linuxserver.io/images/docker-snapdrop)

This repository should be cloned, after which the image can be built using the command &rarr; `docker build -t linuxserver_snapdrop_local_only .` (CI is enabled so building may not be necessary).

# PiHole DNS

Command to run this server &rarr; 

```bash
docker run -d --name pihole \
-p 53:53/tcp -p 53:53/udp -p 8090:80 \
-e TZ="America/Chicago" -e WEBPASSWORD="<pw>" -e DNSSEC="true" -e WEBTHEME="default-auto" \
-v "/home/tanq/pihole/etc-pihole:/etc/pihole" -v "/home/tanq/pihole/etc-dnsmasq:/etc/dnsmasq.d" \
--dns=1.1.1.1 --hostname pi.hole --restart=unless-stopped \
pihole/pihole:latest
```

The server is running off the official docker image from the maintainers &rarr; 

Resource &rarr; [PiHole Docker](https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker)

# LinkDing Bookmark Manager

This is a bookmark manager that can be run using &rarr;

```bash
docker run --name linkding -p 9090:9090 -v ~/linkding_data:/etc/linkding/data -d sissbruecker/linkding:latest
```

Next, it needs a default account the first time it is setup, which is done like this &rarr;

```bash
docker exec -it linkding python manage.py createsuperuser --username=tanishq --email=tanishq16@proton.me
```

Resource &rarr; [LinkDing](https://github.com/sissbruecker/linkding)

# MD Dumpster

This is a self-made application that renders MD files or keeps MD pastes available for any device on the local network. More details can be found at the repository referenced below.

Commands to build and run &rarr;

```bash
git clone https://github.com/tanq16/share-n-render
cd share-n-render

docker build -t local_dumpster .
docker run --name md_dumpster --rm -p 80:5000 -d -t local_dumpster
```

Resource &rarr; [MD Dumpster: Share-N-Render](https://github.com/tanq16/share-n-render)
