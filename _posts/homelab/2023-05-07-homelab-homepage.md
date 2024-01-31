---
title: Homepage Dashboard in Home Lab
date: 2023-05-07 18:53:00 -0600
categories: [Home Server]
tags: [homepage,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

[Homepage](https://gethomepage.dev) is a slick-looking dashboard that I use in my home lab. Out of at least 10 other dashboard applications I tried, Homepage is the only one that struck the right balance of elegance, simplicity, and functionality.

## Deployment

As usual, Docker is the best way to deploy the dashboard application. Make a local directory for it as follows &rarr;

```bash
mkdir -p $HOME/homepage
```

Then, deploy the container as follows &rarr;

```bash
docker run --rm -d \
--name homepage \
-p 3000:3000 \
-v $HOME/homepage:/app/config \
-v /var/run/docker.sock:/var/run/docker.sock \
ghcr.io/benphelps/homepage
```

For Docker compose or via Portainer stacks, the following template can be used &rarr;

```yaml
services:
  homepage:
    image: ghcr.io/benphelps/homepage
    container_name: homepage
    # expanding $HOME in volumes so that Portainer can deploy correctly
    # since $HOME means something else in the Portainer container
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /home/tanq/homepage:/app/config
    ports:
      - 3000:3000
```

And that's it! All that's left is to configure it!

## Usage & Setup

Homepage is a static-style dashboard, i.e., we need to edit the configuration files to make changes. That's not the optimal way to do things, but some multi-cursor magic in VS-Code or Sublime-Text can help. It's highly configurable and has a ton of functionality involving widgets, service integrations, etc.

For me, the main sections of interest are as follows &rarr;

- **Services** &rarr; These are the big boxes that we can configure to provide basic information about the resource usage of our containers.
- **Bookmarks** &rarr; As the name suggests, it's a list of websites. The good part is there's a huge library of icons available at [Dashboard Icons](https://github.com/walkxcode/dashboard-icons/) that are supported, and a couple of other sources mentioned in the documentation.

Adding services from a different server than where the dashboard is loaded needs the application to communicate with the Docker socket of that server. There are multiple options for doing that, but the two best options are - to expose the port in the local network or use [docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy). I went with the first option, but I want to caution you: *exposing the socket is not safe, but it isn't an issue if it's in the home network only*. To do that, I used the following steps as `root` on the server &rarr;

```bash
cat << EOF > /etc/docker/daemon.json
{"hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]}
EOF
```

```bash
mkdir -p /etc/systemd/system/docker.service.d/
```

```bash
cat << EOF > /etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
EOF
```

```bash
systemctl daemon-reload && systemctl restart docker.service
```

The steps and discussions on the topic are present in [this gist](https://gist.github.com/styblope/dc55e0ad2a9848f2cc3307d4819d819f). After these instructions, the server's Docker socket will be available in the local network.

## Fin

An example setup of my Homepage dashboard is as follows &rarr;

![Homapege Example](/assets/post-images/homepageexample.png)

Maybe that provides some inspiration. I also want to mention a couple of other dashboards that are famous and might be worth looking into &rarr;

- [Dashy](https://github.com/lissy93/dashy)
- [Heimdall](https://heimdall.site)
- [Dashboard](https://github.com/phntxx/dashboard)
- [Flame](https://github.com/pawelmalak/flame)

Good luck with your dashboard setup!

If most of these services are accessed via mobile devices and browsers, it may also be easy to integrate the IP:port combinations or internal domain names with the browser bookmarks or home screen app icons for mobile.
