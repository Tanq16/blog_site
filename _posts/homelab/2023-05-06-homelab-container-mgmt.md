---
title: Container Management in Home Lab - Portainer, Dockge & Yacht
date: 2023-05-06 22:46:00 -0600
categories: [Home Server]
tags: [portainer,yacht,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

## Reasoning & Candidates

Home Labs come in different shapes and sizes. We could have a multi-node setup running Kubernetes and a couple of other servers running Proxmox with specialized virtual machines and containers, all interacting with each other over TLS; or it can be a simple raspberry pi running a single docker container like Plex or PiHole. But irrespective of the size of a home lab, the intention is - to make your home network cool and automate some parts of your life or improve experiences related to your everyday activities. With this intent in mind, a container management service can help debug issues and observe logs, manage running services by spinning them up or down, obtain command line access to running containers, and organize and monitor all the images/volumes/containers from an eagle-eyed view.

All of this can make our home lab life easy. So, it's another quality of life improvement. IMO, the two best container management services that have the most engaging communities are [Portainer](https://www.portainer.io) and [Yacht](https://yacht.sh). Another one that popped up in the end of 2023 is [Dockge](https://github.com/louislam/dockge/tree/master).

All these services offer installation via Docker and provide helpful information sets and container management toolsets. One can even end up running all of them together! The main differences are as follows &rarr;

- Portainer initially started as built-for-containers but slowly expanded its horizons to other technologies like remote host Docker container management, Docker Swarm management, and Kubernetes cluster management; whereas Yacht and Dockge are purpose-built for containers.
- Portainer is well established and has a business offering with advanced features while Yacht and Dockge are relatively new and have a limited but refined feature set.
- Portainer also provides the ability to exec into customers directly from the web UI, even in the community version. Dockge supports the same functionality.

My choice here is Portainer for three reasons - to deploy using the latest Docker compose plugin, the ability to exec into containers directly from within the browser, and expand control to a Kubernetes cluster or Docker Swarm. However, after learning about Dockge, I also shifted to that *instead* of Portainer because it just turned out to be very straightforward and provided me exactly the feature set I wanted.

## Deployment

### Portainer

First, setup a local config directory as follows &rarr;

```bash
mkdir -p $HOME/portainer
```

Then launch the container as follows &rarr;

```bash
docker run -d \
-p 8000:8000 -p 9443:9443 \
--name portainer \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $HOME/portainer:/data \
portainer/portainer-ce:latest
```

Then access the service on port 9443 and set up a strong password. Portainer has the concept of environments. The machine that the Portainer container is deployed on becomes the local environment. The most common way of adding new environments is to deploy a Portainer image in the other machines, which will sync to the main server in the local environment. The Portainer Agent can be deployed as follows on the other machines &rarr;

```bash
docker run -d \
-p 9001:9001 \
--name portainer_agent \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /var/lib/docker/volumes:/var/lib/docker/volumes \
portainer/agent
```

After that, add the agent to the container UI deployed via the local environment.

### Dockge

First, setup a local config directory. Unlike the rest, for this one, certain reasons demand creation of a directory in `/opt`. There could be ways to get around that but it's fruitless effort. So I do the following &rarr;

```bash
sudo mkdir -p /opt/dockge/{data,stacks}
```

Then setup the container with a docker run command as follows &rarr;

```bash
docker run --name dockge --rm -d \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /opt/dockge/data:/app/data \
-v /opt/dockge/stacks/:/opt/dockge/stacks/ \
-e DOCKGE_STACKS_DIR=/opt/dockge/stacks \
-p 9441:5001 \
louislam/dockge:1
```

I went with port `9441` but of course that can be changed. The default `5001` is generally used for other services in my network.

### Yacht

Yacht is competitive with Portainer but has limited functionalities. To run the container, first, create a directory for config as follows &rarr;

```bash
mkdir -p $HOME/yacht
```

Then deploy the container as follows &rarr;

```bash
docker run --name yacht \
-d -p 8000:8000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $HOME/yacht:/config
selfhostedpro/yacht
```

The port can be changed to 8001 on the host side if Portainer is needed to run simultaneously.

I recommend checking out the other two projects as well and ensuring they work for your use case. Then, have fun!

## Portainer/Dockge Stacks

Since I primarily use Portainer/Dockge, I also want to highlight this - they can be used to deploy containers using [Docker's compose plugin](https://docs.docker.com/compose/install/linux/) through the use of Docker compose YAML template files. This feature is called ***Stacks***. This makes it very easy to maintain and port a home lab. This is because starting Portainer on a server is a single command like mentioned above under "Deployment", and all other containers can simply be maintained as compose (or stack) definitions.

While each service can be deployed as a separate stack, it's also easy to deploy everything as a single stack if it's small enough. Maintaining a single compose YAML template helps to start all of the services in a single-click fashion. Alternatively, separate stacks allow easy debugging on the home server. Additionally, you can deploy containers in specific networks easily (though it's generally best to isolate them).

>Stack are defined via YAML syntax and this is very useful for deploying services together. However, keep in mind that if Portainer is started as the `root` user, then the volume binds will also be maintained and inherited by `root`, which might be an issue if other services/containers need to access the same volume mounts.
{: .prompt-tip }

The stacks also allow editing the definitions on the browser UI. One of the features I use very often is the checkbox of "Re-pull and deploy" on Portainer and a similar option in Dockge when updating a stack. This can be done without actually making a modification to the definition, allowing a simple update of the containers in a given stack. Again, there are other services like [Watchtower](https://github.com/containrrr/watchtower) that are great at updating containers, but I just like to do that manually for specific stacks.

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
