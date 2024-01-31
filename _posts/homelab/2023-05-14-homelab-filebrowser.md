---
title: FileBrowser in Home Lab
date: 2023-05-14 15:41:00 -0600
categories: [Home Server]
tags: [filebrowser,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

[FileBrowser](https://filebrowser.org) is a file management service that can run within the local network through a browser. It can be thought of as a "create your own cloud storage" solution, with the cloud being the self-hosted home lab. It has a slick UI for mobile and desktop web browsers and allows all basic functionalities that a service like Google Drive would provide. In addition to that, FileBrowser also has the ability to edit the text in files with syntax highlighting for code files.

All of these features are very helpful for local file shares. The entire home directory should be shared with the container to enable editing files relating to other containers. Multiple files can also be zipped, gzipped, or converted into tarballs.

To run this container, start by setting up the directory as follows &rarr;

```bash
mkdir -p $HOME/filebrowser
touch $HOME/filebrowser/filebrowser.db
```

Then, run the container as follows &rarr;

```bash
docker run --rm -d --name=filebrowser \
-v $HOME/filebrowser/:/srv `# can just be $HOME as well` \
-v $HOME/filebrowser/filebrowser.db:/database.db \
-u $(id -u):$(id -g) \
-p 8090:80 \
filebrowser/filebrowser
```

An equivalent Docker compose template or a template to deploy using Portainer stacks is as follows &rarr;

```yaml
services:
  filebrowser:
    image: filebrowser/filebrowser
    container_name: filebrowser
    # expanding $HOME in volumes so that Portainer can deploy correctly
    # since $HOME means something else in the Portainer container
    volumes:
      - /home/tanq/filebrowser/:/srv
      - /home/tanq/filebrowser/filebrowser.db:/database.db
    user: "1000:1000"
    ports:
      - 8090:80
```

You may also choose not to pass the `-u` argument; however, passing that ensures that the files created and modified are done so by the user id of the defualt user of the linux system that the container is deployed on. Without the `-u` argument, the files will be created by the `root` user instead.

One place where not passing specific user ID might be helpful is if the main server home directory needs to be edited by root, specifically container volumes which default to root ownership due to containers using the root user. Editing those files needs root privileges, so you could probably use 2 FileBrowser container - one for all home lab services with root and the volume being `/home/<user>` for the server, and the other being one with default UID and GID values and a specific directory for storage of files.

FileBrowser is an amazing addition to home lab services, and I highly recommend deploying it and using it.
