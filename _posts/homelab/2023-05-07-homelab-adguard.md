---
title: AdGuard in Home Lab
date: 2023-05-07 10:22:00 -0600
categories: [Home Server]
tags: [adguard,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

[AdGuard Home](https://adguard.com/en/adguard-home/overview.html) is a network-wide DNS sink hole, like PiHole. Like most services, this can be deployed as a container, which is the easiest way to set it up.

## Deployment

AdGuard provides a [wiki](https://github.com/AdguardTeam/AdGuardHome/wiki/Docker) to help with the installation through docker. It's best to install AdGuard Home in a home lab on Linux, which usually has `systemd-resolved` running. `resolved` will prevent deploying the container with port 53 bound, so the local stub resolver would need to be disabled.

Begin by deploying the home directory structure &rarr;

```bash
mkdir -p $HOME/adguard/{work,conf}
```

Next, run the following to figure out if `resolved` is active &rarr;

```bash
service systemd-resolved status
```

If `resolved` is inactive, you should skip to the `docker run` step. Otherwise, perform the following commands serially &rarr;

```bash
sudo mkdir -p /etc/systemd/resolved.conf.d/
```

```bash
sudo mv /etc/resolv.conf /etc/resolv.conf.backup
```

```bash
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

```bash
# get a root shell
sudo su
```

```bash
cat << EOF > /etc/systemd/resolved.conf.d/adguardhome.conf
[Resolve]
DNS=127.0.0.1
DNSStubListener=no
EOF
```

Then, exit out of the root shell and restart `systemd-resolved` as follows &rarr;

```bash
sudo systemctl reload-or-restart systemd-resolved
```

Finally, run the docker container as follows &rarr;

```bash
docker run --name adguardhome \
    --restart unless-stopped \
    -v $HOME/adguard/work:/opt/adguardhome/work \
    -v $HOME/adguard/conf:/opt/adguardhome/conf \
    -p 53:53/tcp -p 53:53/udp \
    -p 80:80/tcp -p 443:443/tcp -p 443:443/udp -p 3001:3000/tcp \
    -p 853:853/tcp \
    -d adguard/adguardhome
```

An equivalent Docker compose template or a template to deploy using Portainer stacks is as follows &rarr;

```yaml
services:
  adguardhome:
    image: adguard/adguardhome
    container_name: adguardhome
    restart: unless-stopped
    # expanding $HOME in volumes so that Portainer can deploy correctly
    # since $HOME means something else in the Portainer container
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
```

I've omitted some ports used for DNS-over-QUIC and DHCP, but those can be included as necessary following the [documentation](https://github.com/AdguardTeam/AdGuardHome/wiki/Docker#create-and-run-the-container).

Now, visit `http://<server>:3001` to begin configuring an account for AdGuard. Follow the instructions and log in to the web interface as port 80/443.

## Usage & Setup

AdGuard works as a sink-hole, such that DNS requests to domains present in denylists are answered with a bogus value or `0.0.0.0` such that the devices making DNS requests cannot load appropriate assets from the translated IP addresses. To do that, AdGuard (and many other ad blockers) use filter lists, which are text files that contain a list of domains.

Go to `Filters > DNS blocklists` on the AdGuard interface. Select "Add blocklist" and check all the sources curated by AdGuard and disable each one from the imported collection. Then, enable the following &rarr;

- AdGuard DNS filter
- AdAway Default Blocklist
- Phishing URL Blocklist
- NoCoin Filter List
- OISD Blocklist Small

Lastly, add two more custom lists from public sources as follows &rarr;

- EasyList &rarr; `https://easylist.to/easylist/easylist.txt`
- FanBoy's Annoyances &rarr; `https://secure.fanboy.co.nz/fanboy-annoyance.txt`

Of course, the DNS sinkhole also supports a log of all queries that have been answered or denied. *DNS Rewrite* is a feature that allows setting custom IP addresses as resolutions for specific domain names, so things like personal cloud servers or local servers can be granted a domain name for easy connections.

One of the most useful features in AdGuard is the option to disable the filtering for a short period of time. On the home page of the webapp, filtering can be disabled for a time period determined by a dropdown of time period (30 seconds to 10 minutes). This is extremely useful when certain sites need to be loaded without worrying about ads or trackers because the protection is automatically enabled after the time period of selection.

AdGuard Home can also function as a DHCP server, though I prefer using the home router as the DHCP server and restrict AdGuard for just DNS filtering. AdGuard can be provided custom upstream DNS providers like Cloudflare and Quad9, and also supports advanced configurations such as load balancing among multiple upstream providers.

## Outro

You can verify the blocking status as [AdBlock Test](https://d3ward.github.io/toolz/adblock.html). This test is rigorous, so don't expect a 100 result. A perfect score isn't necessary, but if you really want to, add additional lists like "OSID Blocklist Big". Keep in mind that an excessive number of filters can cause performance overload on the container (and the host).

AdGuard helps blocks ads for all devices in a local network while protecting the processing power of the devices themselves to block the ads. Enjoy an adless experience with AdGuard today!
