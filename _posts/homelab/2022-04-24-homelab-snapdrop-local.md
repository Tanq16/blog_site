---
title: SnapDrop Local Container in Home Lab
date: 2022-04-24 12:00:00 +0500
categories: [Home Server]
tags: [home-lab,snapdrop]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

>As explained below, the WebRTC-based Snapdrop uses TURN (Traversal Using Relays around NAT) for the publicly available [instance](https://snapdrop.net) when a peer-peer connection cannot be established, and SnapDrop uses its public TURN servers for that. Now, all that and more has been solved by another project that I created - [Local Content Share](https://github.com/Tanq16/local-content-share). To learn more about how that works, read my blog in the Home Lab series - [Local Content Share in Home Lab](https://tanishq.page/blog/posts/homelab-local-dumpster/).
{: .prompt-danger }

## Deployment

Use this [repo](https://github.com/Tanq16/docker-snapdrop) for deployment. [SnapDrop](https://snapdrop.net) is an open-source WebRTC based app inspired by Apple's AirDrop. It works out of the box by visiting the link. However, WebRTC apps rely on STUN and TURN servers and may follow a network path that is not secure or completely local. Therefore, I made an edit to the [LinuxServer.IO image](https://docs.linuxserver.io/images/docker-snapdrop) dockerfile in the repo mentioned earlier to modify it such that the app just uses the local network and relies on the network address as that of the local router. The command to run the modified image is as follows &rarr;

```bash
mkdir -p $HOME/snapdrop
```

```bash
docker run -d \
--name=snapdrop_variant \
-e PUID=1000 \
-e PGID=1000 \
-e TZ=America/Chicago \
-p 80:80 \
-p 443:443 \
-v $HOME/snapdrop:/config \
--restart unless-stopped \
tanq16/linuxserver_snapdrop_local_only
```

## PreBuilt

The docker hub variant is not something I regularly update and build, so a local build may be necessary by cloning the original repository, making the change that I made (check the latest commit on my repository if needed), and building using the command &rarr; 

```bash
docker build -t linuxserver_snapdrop_local_only .
```

Once built and deployed on the home-lab server, just visit `https://<server>` on two devices that need to share text or files and use the naming scheme and the UI to perform the sharing. Look at the MD Dumpster blog post for another local sharing implementation that I made. However, I still recommend using [Local Content Share in Home Lab](https://tanishq.page/blog/posts/homelab-local-dumpster/).
