---
title: Streamlining Security-Related Workflows with Docker Containers
date: 2023-04-29 23:39:00 -0600
categories: [Computers and Security]
tags: [docker,container,productivity,security]
image:
  path: /assets/img/covers/docker-cover.jpeg
  alt: Docker Artwork
---

I'm generally always looking for ways to improve my workflow and make my work as a cybersec professional more efficient. One of the tools that has had the biggest impact on my work is [Docker](https://www.docker.com).

## What is Docker?

Docker uses cgroups and namespaces to dedicate a limit-enforced set of resources for containers to run in an isolated environment as a process. Docker uses layers of filesystem, together called an image, which acts as the filesystem for the isolated process. Containers are also not a virtual OS, rather a process running in the host OS. This allows containers to have a completely isolated environment with a separate process tree and filesystem. Images are essentially a snapshot of the filesystem and containers created with those images add a writeable layer on top to make local changes to the filesystem.

With all this, it enables developers to build, ship, and run applications in containers packaged as images. The writeable filesystem layer is destroyed after the container stops, therefore, images are basically self-contained snapshots that package an application and its dependencies together, making it easier to deploy the application across different environments.

## Using Docker for Security-Related Work

As mentioned before, the writeable layer is what makes the image a container in simple terms. But it isn't persistent since containers are created from images. So when a container is stopped, all those changes are removed. One way of keeping filesystem changes persistent is using volume mounts. Essentially, a local directory on the host is mounted on the container allowing that directory to be a state-preserving directory present on the host. This can be used to provide persistence in containers. Now on to how and what to use this for.

As a security engineer, I work with a variety of different tools in several kinds of environments. Usually, security engineers would install tools and libraries on their workstations, or maybe create a virtual machine that they keep up to date and use for cloning into other VMs that they use for specific work. I would often do the same but encounter issues like maintaining a base virtual machine, facing errors stemming from conflicting Python environments, OS type or version issues, and module mismatches. Docker, or rather, containers have solved most of these issues for me by providing me with a standardized environment that I can use across all my projects and any machine.

In my Github repository for [Containerized Security Toolkit](https://github.com/tanq16/containerized-security-toolkit), I have shared a Docker image that I use for all my security-related work. This image is based on Ubuntu and contains all the tools that I use for security reviews and pentesting, including stuff like Nmap, Project Discovery tools, Scout Suite, Trivy, etc. The repo also has a CI process that builds the image regularly and pushes it to Docker Hub. Using a single Docker container for all my security-related work has been a game-changer.

> To learn more about advanced workflows with the Containerized Security Toolkit, read the [companion guide in my blog](https://tanishq.page/blog/posts/cst-guide/).

## How is it Helpful?

### Standardized Environment

A containerized instance provides a standardized environment that I can use across all my projects. This means that all my tools are updated and consistent, which makes it easier for me to switch between different environments without having to install all the necessary tools each time. All the tools are installed directly via CI in the image and are quite up-to-date. The CI ensures that my tools in the latest image are completely reliable. Aside from the tooling, even the shell experience, or as otherwise put in the community as dotfiles, can also be baked into the container and provide a seamless experience.

### Portability

The Docker container can be deployed absolutely anywhere. This means that it's extremely simple to take my work to another laptop or a cloud VM without having to install all the necessary tools each time. It's also easy to share my container with others. Since all of it is basically maintained by a single Dockerfile, it's also easy to customize workflows like the Lazy script or the AutoRecon workflow and deploy them as container tasks so that input can be passed to cloud functions which then execute the containerized workflows and return the result easily without having to go through installing or updating tooling to take to different execution environments.

### Community Support

I prefer using Ubuntu as the base image layer since it is the most widely used Linux distro with a very well-established online community. Any issues encountered can easily be solved since people discuss a whole lot of issues surrounding tooling or software in Ubuntu on platforms like forums, blogs, etc. This makes troubleshooting much easier and faster.

### An Isolated & Simplified Workflow

Managing dependencies, environments, or libraries is one of the most frustrating parts of installing several tools on a given OS. Special attention to things like Python environments or OS version mismatch. One very common issue with MacOS is that of Python versions since there is a MacOS-specific Python and a Brew-installed Python, both of which sometimes share the same `pip` executable or some STL, and installing something with the Brew-Python installs it in the MacOS Python path, calling which via the MacOS Python then doesn't sit well with its version and Brew-Python executables get symlinked to MacOS Python. After that, suddenly the only Python version available is an older MacOS Python which can't access any modules installed using brew-python. Yes, something like this did happen once.

Of course, using virtual environments reduces the frequency of encountering such an error, but what do you do when you encounter such an issue? Try to fix it? Definitely not, because we're all on a clock. Running these things inside the container solves this issue. And if something goes wrong, the container can just be restarted from a base working condition without any issue. So, I don't have to worry about managing dependencies or libraries when working off of a container, which saves me a lot of time and effort.

## Using A Container

My Docker container is available publicly on Docker Hub aimed at security-related workflows with the GitHub repo at [Containerized Security Toolkit](https://github.com/tanq16/containerized-security-toolkit). It's straightforward to get started and all that's needed is to pull the image and start using it or clone the repository and build a modified version of the Dockerfile. It's just that simple! There are more considerations and nuances such as using SSH instead of `docker exec` to get a shell to the container, but all of that's well explained in the repo. I recommend the quick-start section. But even outside of using that image, there's a whole host of possibilities for running workflows from docker which significantly improve the quality and efficacy of security-related work.

## Conclusion

Docker is an essential tool for me and I think should be on the minds of other professionals in the tech industry. Using a Docker container has allowed me to simplify my workflow, save time, and focus on my work. I highly recommend using Docker containers for anyone involved in security-related work. Cheers!
