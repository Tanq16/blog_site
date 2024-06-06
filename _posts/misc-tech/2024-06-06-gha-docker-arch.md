---
title: GitHub Actions & ARM Architecture
date: 2024-06-06 17:52:00 -0500
categories: [Computers and Fun]
tags: [github-actions,docker,container,cicd]
image:
  path: /assets/img/covers/ghaarch-cover.jpeg
  alt: GHA Arch Artwork
pin: true
---

> This blog post covers my troubleshooting efforts and research on how we can use GitHub workflows for various architectures. It focuses more on how to build container images for ARM in the absence of native ARM runners and the state of such things in mid-2024 (this means any improvements made after mid-2024 were not considered when writing this content).
{: .prompt-tip }

## GitHub Actions Primer

GitHub allows building projects and code artifacts from a given project using GitHub Actions (GHA). Actions can be configured using YAML configuration files within the directory structure `.github/workflows/` to specify what to build or test and how. These "workflows" can be initiated by specific triggers or based on schedules (i.e., cron). A workflow defines multiple jobs independent of or dependent on one or more other jobs. Jobs usually have steps, where each step is something that the runner executes.

Steps can be usual Linux (shell) code that we define, or they can be prepared procedures, also called actions, that GitHub and the open-source community maintain in the actions marketplace. One example is the `actions/checkout@v3`, which is used as a starting step to checkout code from the repository into the runner. Many different kinds of actions can aid in several tasks, such as code analysis, linting and testing, building and pushing container images, sending notifications, and many other things.

> Quick thing to note: if you have hawk-eyes, you realize some actions you might perform on the runners require internet access. In Organizations, these actions and how they can be initiated (like from non-restricted branches) are some of the most common ways CI/CD systems are compromised.
{: .prompt-danger }

## Runners and the State of ARM

GitHub runners can run Linux or MacOS. We can also define the architecture, but things get complicated here. By default, though, it's x86-64. Usually, most laptops and business machines used to be x86-64 (generally Intel processors). Most cloud machines also defaulted to the same architecture. ARM64 was primarily used in embedded devices, Raspberry Pis, and smartphones - none of which captures the essence of deployments and running applications because that's what GitHub actions and other CI/CD constructs are used for.

All of this changed, and the state of ARM observed a massive shift in adoption at the end of 2020 due to Apple launching the M1 series of processors, which can be seen as the big brother of Apple's high-performant chipsets from previous iPad pros. With the popularity of those M-series laptops, everyone who bought a laptop, including most businesses that relied on Apple computers had ARM64 processors. That's where issues with cross-architecture builds came to light.

Apple launched Rosetta - an emulator to support x86-64 code by performing dynamic translation of the instruction set from x86-64 to ARM64 before an x86-64 process is executed. However, several apps still didn't run on this, and things like debugging or writing code that runs for everyone became harder and harder. Many open-source projects don't support ARM (ARM64 or aarch64) today! Projects can be built manually wherever needed, but that had two issues &rarr;

- Building from scratch takes time as compared to using pre-built binaries
- Projects can depend on other projects that don't have ARM variants, which will cause the build to fail

With all that, we're in this state of limbo where some things have ARM variants, and others don't. This was one of the primary issues I faced when building my [Containerized Security Toolkit](https://github.com/tanq16/containerized-security-toolkit). The good thing is that GitHub runners immediately started supporting the ARM MacOS machine type, likely through some sort of Mac Mini farm. However, the drawback is that the MacOS runners do not support containerization, so we can't build ARM Docker images, for example.

> All this is almost old news now, as GitHub just launched native ARM runners for everyone.
{: .prompt-tip }

## Solutions for Running ARM Images

So, with GitHub's latest launch of ARM64 runners, why do I want to talk about old solutions? Because it shows what I did in the absence of ARM runners to maintain my tool and represented a good learning experience worth documenting. The only thing I'll focus on to demonstrate solutions is building container images. They're a great way to escape dependency hell and the easiest first step to universalizing software builds. Projects sometimes have releases of both ARM64 and x86-64, but it's possible by using these solutions.

The ***easiest*** solve for any simple project was to switch to a DevOps provider that supported ARM64 runners. CircleCI was one such provider, and I switched to it for my [Containerized Security Toolkit](https://github.com/tanq16/containerized-security-toolkit) project. Native ARM runners work great and do not compromise on speed at all. However, this only solved some things; what about organizations already on GitHub that didn't want to subscribe to CircleCI? What about the runner limits, which could be much more forgiving on GitHub?

I maintain a similar container-based project in my organization and couldn't rely on CircleCI. So, I took the next best solution - using Rosetta and other emulators locally to build and push images to container registries. Of course, this is an imperfect solution because of the operational effort, so the next step was to run virtualized builds on x86-64 GitHub runners. Specifically, this would involve using QEMU (keemu, q-e-m-u, I don't know at this point) and Docker BuildX. I initially implemented this as a solution for my [Containerized Security Toolkit](https://github.com/tanq16/containerized-security-toolkit) project before switching to CircleCI. The only drawback is the slow build time due to virtualization, which is why many projects don't have releases for ARM64. The other drawback is that Docker build operations for multi-stage builds can consume all disk space and fail.

> Quick tip - Apple's Rosetta is great on Macs for emulating some things, but it's not flawless. Docker natively supports Rosetta and can use that to run x86-64 images; however, some specific images may still not work. It's still very convenient and impressive.
{: .prompt-tip }

Let's explore emulated builds in more detail.

## Building ARM Images in x86-64 Runners

As I mentioned above, building an ARM image in the x86-64 runners requires emulation through QEMU and Docker BuildX. [Docker Docs](https://docs.docker.com/build/ci/github-actions/multi-platform/) also mentions that the `build-push-action` action can be used to specify the platforms for which the image is built. However, several times, there are multi-stage builds involved, and it's more complex than calling the action. Therefore, I'll detail this via a build script in the workflow file. Before getting into the details of that, the caveats with using QEMU are as follows &rarr;

- Slow build times due to QEMU
- The need for a Dockerfile rewrite depending on what software we're running because not many tools and libraries are written in multi-platform (this doesn't go away with the native runners, though)
- Easier to run out of disk space - may need multi-stage builds, which can't be done using the `build-and-push` action; for example - the image I build in [Containerized Security Toolkit](https://github.com/tanq16/containerized-security-toolkit) uses multi-stage build and a build script to aid the build process

When used in GitHub, a multi-stage build script would need a code snippet like so for initiating the build of an image &rarr;

```bash
docker buildx build -o type=docker --platform=linux/arm64 -f Dockerfile -t image:tag
```

Let's dissect this command for understanding &rarr;

- `docker buildx build` &rarr; this part tells Docker to use `buildx` to build the image
- `-o type=docker` &rarr; this part is necessary because the intermediate images in a multi-stage build do not get cached into the local disk of the runner, so when the next build stage is triggered, Docker attempts to pull the previous stage from Docker Hub instead of locally; setting type to `docker` sets the export type to an image format which will be cached locally
- `--platform=linux/arm64` &rarr; this specifies the architecture for which the image is being built
- `-f Docker.file` &rarr; this can be omitted if the file is named `Dockerfile`, but is needed for other filenames, especially in multi-stage builds
- `-t image:tag` &rarr; default tag is `latest`, but that does not work for local caches between multi-stage builds for `buildx` (works on local computers but not inside GitHub runners); so, a tag must be specified and used in the next stage

In totality, additional actions are also needed to set up the workflow with QEMU and BuildX. An example workflow file is as follows &rarr;

```yaml
name: Build ARM Image
on:
  push:
    branches:
      - 'main'
  workflow_dispatch:

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_ACCESS_TOKEN }}
      - run: docker buildx build -o type=docker --platform=linux/arm64 -t image:tag .
      - run docker push image:tag
```

## Conclusion

We've explored how building an ARM docker image would look on GitHub actions. Multiple options are discussed, but my true recommendation is to use the new and shiny GitHub ARM runners or CircleCI. The QEMU/BuildX method could still be useful for limited self-hosted runners. But really, all this was just about sharing the adventure.
