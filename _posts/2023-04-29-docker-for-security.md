---
title: Streamlining Security-Related Workflows with Docker Containers
date: 2023-04-29 23:39:00 -0600
categories: [Computers and Programming]
tags: [docker,container,productivity]
---

# WORK IN PROGRESS

I'm generally always looking for ways to improve my workflow and make my work as a cybersec professional more efficient. One of the tools that has had the biggest impact on my work is [Docker](https://www.docker.com).

## What is Docker?

Docker uses cgroups and namespaces to dedicate a limit-enforced set of resources for containers to run in an isolated environment as a process. Docker uses layers of filesystem, together called an image, which acts as the filesystem for the isolated process. Containers are also not a virtual OS rather a process running in the host OS. This allows containers to have a completely isolated environment with a separate process tree and filesystem. Images are essentially a snapshot of the filesystem and containers created with those images add a writeable layer on top to make local changes to the filesystem. Therefore, it enables developers to build, ship, and run applications in containers packaged as images. The writeable filesystem layer is destroyed after the container stops, therefore, images are basically self-contained snapshots that package an application and its dependencies together, making it easier to deploy the application across different environments.

## Using Docker for Security-Related Work

As mentioned before, the writeable layer is what makes the image a container in simple terms. But it isn't persistent since containers are created from images. So when a container is stopped, all those changes are removed. One way of keeping filesystem changes persistent is using volume mounts. Essentially, a local host-directory is mounted on to the container allowing that directory to be a state-preserving directory present on the host. This can be used to provide persistence in containers. Now on to how and what to use this for.

As a security engineer, I work with a variety of different tools in seveeral kinds of environments. Usually, security engineers would install tools and libraries on their workstations, or maybe create a virtual machine that they keep up to date and use for cloning into other VMs that they use for specific work. I would often do the same but encounter issues like maintaining a base virtual machine, face errors stemming from conflicting Python environments, OS type or version issues, and module mismatches. Docker, or rather, containers have solved most of these issues for me by providing me with a standardized environment that I can use across all my projects and any machine.

In my Github repository for [Containerized Security Toolkit](https://github.com/tanq16/containerized-security-toolkit#image-builds), I have shared a Docker image that I use for all my security-related work. This image is based on Ubuntu and contains all the tools that I use for security reviews and pentesting, including stuff like Nmap, Project Discovery tools, and Scout Suite, Trivy, etc. The repo also has a CI process that builds the image regularly and pushes it to Docker Hub. Using a single Docker container for all my security-related work has been a game-changer.

<!-- ## How is it Helpful?

#### Standardized Environment

The Docker container provides me with a standardized environment that I can use across all my projects. This means that all my tools are updated and consistent, making it easier for me to switch between different environments without having to install all the necessary tools each time. I don't have to worry about managing dependencies or libraries, which saves me a lot of time and effort.

#### Portability

The Docker container can be deployed anywhere I want to do the work. This means that I can easily switch between different environments without having to install all the necessary tools each time. I can also share my container with other security engineers, making it easier for us to collaborate on projects.

#### Community Support

Ubuntu is a widely used operating system with a great online community. If I encounter any issues, I can easily find solutions online through forums, blogs, and other resources. This makes troubleshooting much easier and faster.

#### Up-to-Date Tools

I don't need to worry about updating any of my tools as everything is built directly via CI. This means that my container is always up to date with the latest security patches and updates. This is important as it ensures that my tools are secure and reliable, and I don't have to spend time manually updating them.

#### Simplified Workflow

Using a Docker container has simplified my workflow and made my job more efficient. I no longer have to worry about managing dependencies, environments, or libraries, and I can focus on my work. This has allowed me to be more productive and effective in my job.

## How to Use My Container

If you're interested in using my Docker container for security-related work, it's straightforward to get started. All you need to do is clone my repository and build the Docker image. Once the image is built, you can run the container and start using the tools.

## Conclusion

Docker has become an essential tool for security engineers and other professionals in the tech industry. Using a Docker container has allowed me to simplify my workflow, save time, and focus on my work. I highly recommend using Docker containers for anyone involved in security-related work. My Docker container is a great starting point, and I hope it helps you to improve your workflow and productivity as well. -->
