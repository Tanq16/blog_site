---
title: Traggo Time Tracker in Home Lab
date: 2023-05-07 22:30:00 -0600
categories: [Home Server]
tags: [traggo,home-lab]
---

[Traggo](https://traggo.net) is a time-tracking tool with a list and a calendar view. For basic functionality, it's an open-source alternative to Toggl Track. Since it's open-source, it's deployable in a home lab for local-only functionality.

To deploy this as a docker container, first, create a persistence directory as follows &rarr;

```bash
mkdir -p $HOME/traggo
```

Then deploy the container as follows &rarr;

```bash
docker run --rm -d -p 3010:3030 \
-v $HOME/traggo:/opt/traggo/data \
traggo/server:latest
```

For Docker compose or via Portainer stacks, the following template can be used &rarr;

```yaml
services:
  traggo:
    image: traggo/server
    container_name: traggo
    # expanding $HOME in volumes so that Portainer can deploy correctly
    # since $HOME means something else in the Portainer container
    volumes:
      - /home/tanq/traggo:/opt/traggo/data
    ports:
      - 3010:3030
```

And that's it! Pretty straightforward, right? The easiest way to use it is via the List view where you can start an event based on tags, and the app records the time till you stop the event. An example view of the UI is as follows &rarr;

![Traggo List View UI](/assets/post-images/traggoui.png)

>Eventually, using tags may become odd since the application expects a value after the `:` when you tag an event. To avoid this, each entry can be a `.` character after the `:` so only the tag name is visible. All other details can live in the Note section for every event.
{: .prompt-tip }

Traggo also has a dashboard view where you can display several charts with specific statistics based on how you track time. The default credentials for the application are `admin:admin`. Feel free NOT to change it for security reasons 😉.
