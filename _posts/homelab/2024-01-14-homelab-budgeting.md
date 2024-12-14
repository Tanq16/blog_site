---
title: Personal Finance & Budgeting in Home Lab
date: 2024-01-14 14:53:00 -0600
categories: [Home Server]
tags: [budgeting,personal-finance,actualbudget,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

## Intro

Personal finance is more than just a practice - it's a skill. But at the same time, finding a good, privacy-focused application to help manage personal finances is very hard. Many applications are either paywalled for tiers of features, or hold your data hostage to make migration harder. Of course, there are good paid services as well, but an open-source application that can be deployed in our home labs - that's what I'm after. [Actual Budget](https://github.com/actualbudget/actual) is such an application.

I also wrote a tool called [Budgeter](https://github.com/Tanq16/budgeter) for very straightforward tracking of expenses and nothing else. For anything more complicated than that, Actual Budget is amazing.

## Deployment

The Actual-Server is a self-contained backend and a web application container. Here is a one-liner deployment command &rarr;

```bash
mkdir -p $HOME/actualbudget && \
docker run -v $HOME/actualbudget/:/data --name actual-budget --rm -p 5006:5006 -d -t actualbudget/actual-server:latest
```

However, opening this on the browser might lead to an error that requires visiting over `https`. For that, we need to set up a certificate and configure the server to use that self-signed certificate. The following commands can be used to do that &rarr;

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout actual.key -keyout actual.crt
```

```bash
cat << EOF > $HOME/actualbudget/config.json
{
  "https": {
    "key": "/data/actual.key",
    "cert": "/data/actual.crt"
  }
}
EOF
```

```bash
mv actual.key actual.crt $HOME/actualbudget/
```

After this, the container will be deployed at `localhost:5006` over HTTPS. A better way to deploy it is using docker-compose either directly or through Dockge. Use this stack definition &rarr;

```yaml
version: '3.9'
services:
  actual-server:
    image: 'actualbudget/actual-server:latest'
    ports:
      - '5006:5006'
    container_name: actual-budget
    volumes:
      - '/home/tanq/actualbudget/:/data' # Change appropriately
```

## Backup and Fin

Backups are another piece of the puzzle. I recommend manual backups for your home lab server, so this data is included automatically. Still, you can easily set up `rsync` for backing up to local NAS or other more sophisticated backup utilities triggered via cron jobs. In the spirit of automating everything, you could also use Dropbox API and Python in a script triggered by cron jobs to make a tarball from the expense data and back that single file to the cloud.

With that, you can deploy your own amazing personal finance software. Good luck!
