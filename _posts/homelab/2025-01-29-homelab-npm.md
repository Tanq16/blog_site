---
title: Reverse Proxy in Home Lab
date: 2025-01-29 20:11:00 -0400
categories: [Home Server]
tags: [reverse-proxy,nginx-proxy-manager,cloudflare-tunnel,home-lab]
image:
  path: /assets/img/covers/homelab-cover.jpeg
  alt: HomeLab Artwork
---

## Whats and Whys?

A *forward proxy* sends a legitimate request made by a user to the origin of the service that the user requested. It also forwards back the response from the origin server and is considered part of the "client" when dumbing down client-server communication. Forward proxies (usually just called a proxy) is very commonly used for various purposes.

A *reverse proxy* does a similar passthrough of communication but instead sits in front of one or more servers, i.e., is considered part of the "server" when dumbing down client-server communication. Requests are made to the origin servers but intercepted by the reverse proxy and then it does the transitive communication like a forward proxy.

The reason reverse proxies are common for homelab use is that it allows creating a single entrypoint to all services contained within. It also helps exposing it to the internet and can be used to enforce TLS on all services even if a service itself doesn't use HTTPS.

## My Intentions

I run my homelab on a Debian machine within Docker containers. I also have a relatively less flexible ISP, so I don't want to expose anything directly from my router. Lastly I own a domain (purchased from *Cloudflare*) for my homelab use (different from the one you're reading this post on). For the remainder of the post, I'll refer to my domain as `revp.home`.

Given this setup, my intention was as follows &rarr;

- I want to reduce what ports I expose on my Debian server, ideally such that only the reverse proxy management, reverse proxy web ports, and 53 (for DNS, yes I don't use DoH or DoT *yet*) are exposed.
- I'd like most of my containers to not even expose ports on localhost, instead stay within the Docker network.
- All my services should be exposed via HTTPS irrespective of how I access them.
- My domain should be reachable externally with the convention of `SERVICE.revp.home` providing me access to my service.
- I should be able to use the same domain inside my home network and it should directly route to my Debian server rather than going over the internet,
- My services exposed externally should have authentication, while those within the network should not.

The Software Stack I'll be using contains of the following &rarr;

- Nginx Proxy Manager
    - There are a lot of alternatives to this like Caddy or even configuring vanilla Nginx for it.
    - Pick your poison, what you need to do doesn't change either way.
- Cloudflare tunnels
    - This makes it easy to expose things externally without messing with your router or upsetting your ISP.
    - There are some cons like limited bandwidth and throttling, making it a no-go for services like Plex or Jellyfin; not an issue for me personally, so do your research.

> Another way connect to your home network services is by using a VPN. This isn't something explored in this post, but is a very popular option that allows streaming too. However, requires additional server setup, so it wasn't something I required.
{: .prompt-tip }

## Implementation

There's a couple steps to implement this kind of a setup, starting with setting up Cloudflare tunnels. Cloudflare also acts as a reverse proxy, however, has its own agent-server infrastructure in place to allow visibility via zero trust. Following the tunnel setup, there needs to be a common Docker network to allow connectivity. This would allow using NPM (Nginx Proxy Manager) to reach all services.

### Cloudflare Tunnels

> Setting up Cloudflare tunnels is a bit outside the scope of what I want to include in this blog. It's also not complicated so I've given high level steps here instead of a detailed walkthrough.
{: .prompt-warning }

You should already have a domain configured and an account made with Cloudflare. If your domain is not registered with Cloudflare, add it to Cloudflare and change nameservers as needed.

With the domain active, choose **Zero Trust** on the left hand side and naviaget to **Networks > Tunnels** to setup your tunnels. Here you can create a `cloudflared` tunnel (establishes an outbound-only connection to Cloudflare) associated with your domain.

Configure the tunnel and Cloudflare will walk you through installing and running your connector agent. Select a Docker environment and store the token in an `.env` file for your compose file. Ensure this new service in your homelab gets the network configuration detailed under the *Linking Services to NPM* subheading below. The connector will show up and you can finish off the setup by routing a random `localhost:7777` at the end.

> Finish off the rest of the local service setup in the following subsections first before continuing with this subsection.
{: .prompt-danger }

Once the tunnel is saved, go back to your tunnel configuration page and add your services under the **Public Hostname** tab. This is what helps your homelab services become reachable. However, these are all publicly available at this point without any authentication. So, go to **Access > Applications** and add an application with the URL `*.revp.home`, and configure a **Policy** to allow access to your subdomains.

There are several ways of doing this; I prefer using emails for authentication, which sends an OTP to the included email addresses and authenticates with a JWT for the configured length of time. Explore and do what best suits you!

### Setting up NPM

With our Cloudflare tunnel in place and ready to point to services, we need to configure Nginx Proxy Manager. NPM is a reverse proxy that makes it easy to manage reverse proxy routes in a UI and can also help set up SSL certificates using LetsEncrypt for the local network.

NPM can be setup with a Docker container using a compose file. First, setup the persistence directory as follows &rarr;

```bash
mkdir -p $HOME/npm/{data,letsencrypt}
```

Then, deploy the stack &rarr;

```yaml
version: "3.8"
services:
  nginxproxymanager:
    image: jc21/nginx-proxy-manager:latest
    restart: unless-stopped
    ports:
      - 8080:80
      - 443:443
      - 81:81
    environment:
      DISABLE_IPV6: "true"
    volumes:
      - /home/tanq/npm/data:/data
      - /home/tanq/npm/letsencrypt:/etc/letsencrypt
    networks:
      - npmnet
networks:
  npmnet:
    name: npmnet
```

After this stack is deployed, the control plane is available at `IP:81`, where default credentials are `admin@example.com:changeme`. It auto-prompts to change the password and set an email.

> The `npmnet` here is a new Docker network setup for NPM. We'll use this to help route to our services without exposing ports unnecessarily. Unfortunately, that means modifying your existing stacks and redeploying all services.
{: .prompt-info }

### Setting up SSL Certificates

The next step is to setup a Cloudflare domain with NPM and prove ownership. This can be done via a DNS-01 verification. To do this, go to the domain and create an API token. Select a custom token there and then name it `Homelab SSL`. Set the permissions to `Zone, DNS, Edit` and create the token.

After this, go to SSL Certificates on the NPM control plane UI and write the domain names as `revp.home` and `*.revp.home`. This will create a wildcard certificate, so all subdomains will support SSL. Select the DNS provider in the control plane and add the token. NPM will trigger the DNS-01 verification and handle certificate creation.

> We use DNS-01 verification instead of HTTP, because this is what allows for the wildcard certificate.
{: .prompt-tip }

### Linking Services to NPM

With NPM setup, the other services need to be configured to use the same Docker network (`npmnet`). To do this, we add the network to an existing stack and remove the exposed ports from the compose definition. As an example, an updated stack for **Vikunja** would look like this &rarr;

```yaml
version: "3.8"
services:
  vikunja:
    image: vikunja/vikunja
    container_name: vikunja
    volumes:
      - /home/tanq/vikunja/db:/db
      - /home/tanq/vikunja/files:/app/vikunja/files
      - /path/to/config/on/host.yml:/app/vikunja/config.yml:ro
    networks:
      - npmnet
networks:
  npmnet:
    external: true
```

The `external: true` being added here signifies the use of a network defined for another container i.e., a pre-existing network.

> Remember to take a note of the ports on which the services run before removing the exposed ports part, so they can be added to NPM and you don't have to hunt for what port it was.
{: .prompt-tip }

Now, each service can be added to NPM control plane as a new proxy host using the service container name (as defined in the stack) as the hostname and the service's operating port for the port.

After this step, your service should be reachable via the domain, example - `vikunja.revp.home` and present a valid SSL certificate. At this point, Cloudflare tunnels can be configured properly to point to the the same addresses as setup in the NPM control plane (`http(s)://container_name:service_port`). Additionally, keep in mind that the container running the Cloudflare connector agent should also use the same network as the services so it can route to those containers.

> Actually, it won't be reachable just yet because `revp.home` routes to Cloudflare instead of the home server. Read the next subsection to fix that.
{: .prompt-info }

### Finishing off Domain Setup

To allow NPM to be reached via the same domain locally, use a DNS sinkhole to rewrite requests to the domain. That way all local network requests to `revp.home` will be routed to the IP of the server hosting the homelab.

## Fin

With NPM and Cloudflare Tunnels, you can ensure your homelab services are reachable both internally and externally, with authentication enforced for external connections, and SSL enforced and validated everywhere.

## Resources

- [CloudFlare - Reverse Proxy](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/)
- [Nginx Proxy Manager](https://nginxproxymanager.com/)
- [Caddy](https://caddyserver.com/)
- [Cloudflare Tunnels](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
