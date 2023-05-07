---
title: Container Management in Home Lab - Portainer & Yacht
date: 2023-05-06 22:46:00 -0600
categories: [Home Server]
tags: [services,home-lab]
---

## Reasoning & Candidates

Home Labs come in different shapes and sizes. We could have a multi-node setup running kubernetes and a couple other servers running Proxmox with specialized virtual machines and containers, all interacting with each other with TLS enforced; or it can be a simple raspberry pi running a single docker container like Plex or PiHole. But irrespective of the size of a home lab, the intention really is - make your home network cool and automate some parts of your life or improve certain experiences related to your everyday activities. With the same intent in mind, a container management service can help debug issues and observe logs, manage running services by spinning them up or down, obtain command line access to running containers, and organize and observe all images/volumes/containers from a bird's eye view.

All of that can definitely make our home lab life easier. So, it's another quality of life improvement. IMO, the two best container management services that are the best and have the most engaging communities are [Portainer](https://www.portainer.io) and [Yacht](https://yacht.sh).

Both the services offer installation via Docker and provide a very helpful information set and management tools for containers. One can even end up running both together! The main differences are as follows &rarr;

- Yacht is purpose-built for containers
- Portainer started out as built for containers, but slowly expanded horizons to other technologies like remote host Docker container management, Docker Swarm management and Kubernetes cluster management
- Portainer is well established and has a business offering with advancesd features
- Yacht is relatively new and has a limited but refined feature set
- Portainer also provides the ability to exec into customers directly from the web UI even in the community version
- Yacht loads up instantly and provides a very nice dashboard with helpful metrics for all containers

My personal choice here is Portainer mainly because of two reasons - the ability to exec into containers firectly from the browser, and that expanding to a kubernetes cluster and Docker Swarm infrastructure is easy to do. But I've also sometimes run both just for name-sake.

## Deployment

### Portainer

First setup a local config directory as follows &rarr;

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

Then access the service on port 9443 and setup a strong password. Portainer has the concept of environments. The machine that the Portainer container is deployed on is the local environment. More environments can also be added and the most common way to do that is to deploy a Portainer image in the other machines which will sync to the main server in the local environment. The Portainer Agent can be deployed as follows on the other machines &rarr;

```bash
docker run -d \
-p 9001:9001 \
--name portainer_agent \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /var/lib/docker/volumes:/var/lib/docker/volumes \
portainer/agent
```

After that, add the agent to the container UI deployed via the local environment.

### Yacht

Yacht was basically a competition to Portainer, but with limited functionalities. To run the container, first create a directory for config as follows &rarr;

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

The port can be changed to 8001 on the host side if Portainer is also to be run simultaneously.

I recommend checking out both the projects and ensure they work for your use case, just be sure to look at it yourself once.
