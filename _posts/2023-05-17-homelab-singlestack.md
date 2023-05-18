---
title: Deploy Services via a Single Stack in Home Lab
date: 2023-05-17 23:03:00 -0600
categories: [Home Server]
tags: [services,home-lab]
---

As discussed in the [Container Management blog post](https://blog.tanishq.page/posts/homelab-container-mgmt/), Portainer can be used to deploy containers using [Docker's compose plugin](https://docs.docker.com/compose/install/linux/) through the use of Docker compose YAML template files. This feature is called ***Stacks***.

While each service can be deployed as a separate stack, it's best to deploy all your home lab services as a single stack. Maintaining a single compose YAML template helps to start all of the services in a single-click fashion. Additionally, you can deploy containers in specific networks easily (though it's generally best to isolate them).

An example compose YAML template for some of the services mentioned in my blog is as follows &rarr;

```yaml
services:
 adguardhome:
 image: adguard/adguardhome
 container_name: adguardhome
 networks:
 - adguardnet
 restart: unless-stopped
 volumes:
 - /home/tanq/adguard/work:/opt/adguardhome/work
 - /home/tanq/adguard/conf:/opt/adguardhome/conf
 ports:
 - "53:53/tcp"
 - "53:53/udp"
 - "80:80/tcp"
 - "443:443/tcp"
 - "443:443/udp"
 - "3001:3000/tcp"
 - "853:853/tcp"

 filebrowser:
 image: filebrowser/filebrowser
 container_name: filebrowser
 networks:
 - servicesnet
 volumes:
 - /home/tanq/:/srv
 - /home/tanq/filebrowser/filebrowser.db:/database.db
 ports:
 - 5002:80

 homepage:
 image: ghcr.io/benphelps/homepage
 container_name: homepage
 networks:
 - servicesnet
 volumes:
 - /var/run/docker.sock:/var/run/docker.sock
 - /home/tanq/homepage:/app/config
 ports:
 - 5001:3000

 local_dumpster:
 image: tanq16/local_dumpster:main
 container_name: local_dumpster
 networks:
 - servicesnet
 ports:
 - 5000:5000

 jellyfin:
 image: jellyfin/jellyfin
 container_name: jellyfin
 networks:
 - jellynet
 restart: unless-stopped
 volumes:
 - /home/tanq/jellyfin/config:/config
 - /home/tanq/jellyfin/cache:/cache
 - /media/tanq/Tanishq/Media/:/data/media
 ports:
 - 8096:8096

networks:
 adguardnet:
 servicesnet:
 jellynet:
```

Now that is useful, if nothing else! Of course, specific situations may require that containers be deployed via individual stacks, so it's easy to spin them down or up. I use a master stack because I have two server machines, one for all the primary services I use daily, and the other for development and trial runs. So, the master stack is useful for deploying the primary services on the main server.

In conclusion, I think ***Stacks*** is an awesome feature of Portainer!
