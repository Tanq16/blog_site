---
title: MD Dumpster in Home Lab
date: 2023-04-28 22:18:00 -0600
categories: [Home Server]
tags: [services,home-lab]
---

This is a self-made application that renders MD files or keeps MD pastes available for any device on the local network. More details can be found at the repository [MD Dumpster: Share-N-Render](https://github.com/tanq16/share-n-render).

Use the following commands in order to build and run the container &rarr;

```bash
git clone https://github.com/tanq16/share-n-render
cd share-n-render
```

```bash
docker build -t local_dumpster .
```

```bash
docker run --name md_dumpster --rm -p 80:5000 -d -t local_dumpster
```

Deploy the container on a server and then visit `http://<server>` to add markdown text to the dumpster. Then visiting the application webpage from any other device will present the same list of markdown text dumps with the options to view the raw text, render in GitHub style (light and dark mode), and delete the particular dump. Additionally, visiting `http://<server>/print` or `http://<server>/darkprint` to just render markdown text without adding to the dump.

This is generally helpful for sharing text between several home netork devices, especially if the devices have firewall limitations enforced by oganizations to block WebRTC conections. This application is a server-side focussed application where data is transmitted only over HTTP and stored only on the server i.e., the most basic HTTP connection, so all devices with a valid non-APIPA DHCP granted IP address in the local network would be able to use it.

To stop the container, do the following &rarr;

```bash
docker stop md_dumpster -t 0
```

Remember that the history of items lives inside the container so it is not persistent across container restarts. If that functionality is necessary, then `app.py` should be modified to perform all file operations at a base directory or `/data` and the `docker run` command must have another argument to mount a local directory to the container like so - `-v ./mdd_data:/data`. Then all data will persist in the `md_data` directory locally.
