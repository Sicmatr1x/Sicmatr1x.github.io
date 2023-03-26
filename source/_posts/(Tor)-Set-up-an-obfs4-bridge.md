---
title: (Tor) Set up an obfs4 bridge
date: 2018-08-16 10:39:13
categories:
    - Network
tags: 
    - Tor
---

## [Bridge set up guide](community.torproject.org/relay/setup/bridge/ )

This guide will help you set up an obfs4 bridge to help censored users connect to the Tor network. The requirements are:

    24/7 Internet connectivity
    The ability to expose TCP ports to the Internet (make sure that NAT doesn't get in the way)

Note: If you're running a platform that is not listed on this page, you can [compile obfs4 from source](https://gitlab.com/yawning/obfs4#installation).

- [Debian / Ubuntu](https://community.torproject.org/relay/setup/bridge/debian-ubuntu/): How to deploy an obfs4 bridge on Debian / Ubuntu
- [Windows](https://community.torproject.org/relay/setup/bridge/windows/): How to deploy an obfs4 bridge on Windows
- [Fedora](https://community.torproject.org/relay/setup/bridge/fedora/): How to deploy an obfs4 bridge on Fedora
- [CentOS / RHEL / OpenSUSE](https://community.torproject.org/relay/setup/bridge/centos-rhel-opensuse/): How to deploy an obfs4 bridge on CentOS / RHEL / OpenSUSE
- [DragonflyBSD](https://community.torproject.org/relay/setup/bridge/dragonflybsd/): How to deploy an obfs4 bridge on DragonflyBSD
- [Docker](https://community.torproject.org/relay/setup/bridge/docker/): How to deploy an obfs4 bridge using a docker container
- [Post-install](https://community.torproject.org/relay/setup/bridge/post-install/): How to find your bridge in Relay Search and connect manually
- [NetBSD](https://community.torproject.org/relay/setup/bridge/netbsd/): How to deploy an obfs4 bridge on NetBSD

## Set up bridge by using Docker

### 1. Deploy a container

We provide a docker-compose file that helps you deploy the container. First, download docker-compose.yml: 

```yml
# This file assists operators in (re-)deploying an obfs4 bridge Docker
# container.  You need the tool 'docker-compose' to use this file.  You
# can find it in the Debian package 'docker-compose'.
#
# First, you need to create a configuration file, ".env", in the same directory
# as this file, "docker-compose.yml".  Add the following environment variables
# to this configuration file.  EMAIL is your email address; OR_PORT is your
# onion routing port; and PT_PORT is your obfs4 port:
#
#   EMAIL=you@example.com
#   OR_PORT=XXX
#   PT_PORT=XXX
#
# Next, pull the Docker image, by running:
#
#   docker-compose pull obfs4-bridge
#
# And finally, to (re-)deploy the container, run:
#
#   docker-compose up -d obfs4-bridge

version: "3.4"
services:
  obfs4-bridge:
    image: thetorproject/obfs4-bridge:latest
    environment:
      # Exit with an error message if OR_PORT is unset or empty.
      - OR_PORT=${OR_PORT:?Env var OR_PORT is not set.}
      # Exit with an error message if PT_PORT is unset or empty.
      - PT_PORT=${PT_PORT:?Env var PT_PORT is not set.}
      # Exit with an error message if EMAIL is unset or empty.
      - EMAIL=${EMAIL:?Env var EMAIL is not set.}
    volumes:
      - data:/var/lib/tor
    ports:
      - ${OR_PORT}:${OR_PORT}
      - ${PT_PORT}:${PT_PORT}
    restart: unless-stopped

volumes:
  data:
    name: tor-datadir-${OR_PORT}-${PT_PORT}
```

and then write your bridge configuration to a new file, `.env`, which is in the same directory as `docker-compose.yml`. Here's a template:

```
# Your bridge's Tor port.
OR_PORT=X
# Your bridge's obfs4 port.
PT_PORT=Y
# Your email address.
EMAIL=Z
```

Replace `X` with your desired OR port, `Y` with your obfs4 port (make sure that **both** ports are forwarded in your firewall), and `Z` with your email address, which allows us to get in touch with you if there are problems with your bridge. With your bridge configuration in place, you can now deploy the container by running:

```
docker-compose up -d obfs4-bridge
```

This command will automatically load your docker-compose.yml file while considering the environment variables in `.env`.

You should now see output similar to the following:

```
Starting docker-obfs4-bridge_obfs4-bridge_1 ... done
```

That's it! Your container is now bootstrapping your new obfs4 bridge.

### 2. Upgrade your container

Upgrading to the latest version of our image is as simple as running:

```
docker-compose up -d obfs4-bridge
```

Note that your bridge's data directory (which includes its key material) is stored in a docker volume, so you won't lose your bridge's identity when upgrading to the latest docker image. If you are running multiple bridges on your computer, you need to repeat this step for each bridge. We will announce new image versions on the [tor-dev](https://lists.torproject.org/cgi-bin/mailman/listinfo/tor-dev) mailing list.

### 3. Monitor your logs

You can inspect your bridge's logs by running:

```
docker logs CONTAINER_ID
```

To use your new bridge in Tor Browser, you need its "bridge line". Here's how you can get your bridge line:

```
docker exec CONTAINER_ID get-bridge-line
```

This will return a string similar to the following:

```
obfs4 1.2.3.4:1234 B0E566C9031657EA7ED3FC9D248E8AC4F37635A4 cert=OYWq67L7MDApdJCctUAF7rX8LHvMxvIBPHOoAp0+YXzlQdsxhw6EapaMNwbbGICkpY8CPQ iat-mode=0
```

Make sure to check out the **post-install notes**. If you are having troubles setting up your bridge, have a look at **our help section**.

#### [post-install notes](https://community.torproject.org/relay/setup/bridge/post-install/)

```
Congrats!

If you get to this point, it means that your obfs4 bridge is running and is being distributed by BridgeDB to censored users. Note that it can take several days or weeks until you see a consistent set of users, so don't get discouraged if you don't see user connections right away. BridgeDB uses four buckets for bridge distribution: HTTPS, Moat, Email, and manual. Some buckets are used more than others, which also affects the time until your bridge sees users. Finally, there aren't many bridge users out there, so you cannot expect your bridge to be as popular as a relay.

If you want to connect to your bridge manually, you will need to know the bridge's obfs4 certificate. See the file /var/lib/tor/pt_state/obfs4_bridgeline.txt and paste the entire bridge line into Tor Browser:

Bridge obfs4 <IP ADDRESS>:<PORT> <FINGERPRINT> cert=<CERTIFICATE> iat-mode=0
You'll need to replace <IP ADDRESS>, <PORT>, and <FINGERPRINT> with the actual values, which you can find in the tor log. Make sure to use <FINGERPRINT>, not <HASHED FINGERPRINT>; and that <PORT> is the obfs4 port you chose - and not the OR port.

Finally, you can monitor your obfs4 bridge's usage on Relay Search. Just enter your bridge's <HASHED FINGERPRINT> in the form and click "Search". After having set up the bridge, it takes approximately three hours for the bridge to show up in Relay Search.
```

#### [our help section](https://community.torproject.org/relay/getting-help/)

If you run into problems while setting up your relay, you can ask your questions on the public [tor-relays mailing list](https://lists.torproject.org/cgi-bin/mailman/listinfo/tor-relays). The list is a great resource for asking (and answering) questions, and for getting to know other relay operators. Make sure to check out the archives!

You can also get help by joining the IRC channel #tor-relays in the network [irc.oftc.net](https://support.torproject.org/get-in-touch/#irc-help).

