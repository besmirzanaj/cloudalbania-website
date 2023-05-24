---
title: 'Hosting your own public IP echo service'
date: "2022-08-16"
description: "implement a public ip echo service"
draft: false
tags: 
  - wan
  - myip
  - go
  - docker
---

- [Introduction](#introduction)
- [The service](#the-service)
- [Building the docker image](#building-the-docker-image)
- [Testing the container](#testing-the-container)
- [Implementing echoip as a docker-compose service](#implementing-echoip-as-a-docker-compose-service)
  - [Explaining the docker-compose Traefik options](#explaining-the-docker-compose-traefik-options)
- [Adding GeoIP data](#adding-geoip-data)
- [Using the service](#using-the-service)
- [Update 2023-05 - Implementing with Podman instead of Docker](#update-2023-05---implementing-with-podman-instead-of-docker)
  - [Installing podman packages](#installing-podman-packages)
  - [Adopting docker-compose files](#adopting-docker-compose-files)
  - [Autostarting Podman containers](#autostarting-podman-containers)
    - [Implement container autostart with podman](#implement-container-autostart-with-podman)

## Introduction

This article describes some learning efforts to deploy a service in a container enviroment, The service is deployed right now at http://ifconfig.cloudalbania.com/. 

Repo for this article: https://github.com/besmirzanaj/echoip
Service URL: http://ifconfig.cloudalbania.com/

This work has been adopted from https://ifconfig.co/.

## The service

Echoip is a service that will echo back your public IP address. Original repo is hosted at https://ifconfig.co/ and as soon as I found out it was an opensource project that I could host myself I took the task to implement it in one of my VPS servers running the docker daemon.

Furthermore, the original author has implemented capabilities to read GeoIP databases and return more information on the visiting IP, such as Country, City, ASN and reverse PTR check, see the section below that describes how to gat this data.

## Building the docker image

Since I wanted to host my own container image, I opted to rebuild the container and host it in my Docker Hub account. After authenticating with docker hub with `docker login`, I ran the following from the root directory using the exisitng `Dockerfile`:

```
docker build -t beszan/echoip:v1.0 .                  # Builds the image
docker tag beszan/echoip:v1.0 beszan/echoip:latest    # tags with latest so it is easy to use
docker push beszan/echoip                             # push versions in docker hub
```

The image is now hosted in my Docker Hub account. You can do the same with your docker username.

## Testing the container

Since this is a very experimental service for me, I used one of the VPS hosts that I currently have installed docker daemon. This host by the way is also hosting a [software version of RIPE Atlas](https://labs.ripe.net/author/alun_davies/ripe-atlas-software-probes/) probe that you can see [here](https://atlas.ripe.net/probes/1000720/).

After reviewing the repository I was able to quickly run the service as a docker detached container.


```shell
$  docker run --rm --detach \
> --log-driver json-file --log-opt max-size=10m \
> --cpus=1 --memory=64m --memory-reservation=64m \
> --name echoip --hostname "$(hostname --fqdn)" \
>   -p 8080:8080 \
> beszan/echoip -H X-Real-IP

Unable to find image 'beszan/echoip:latest' locally
latest: Pulling from beszan/echoip
676ee85450f5: Pull complete 
28d7554a8df6: Pull complete 
Digest: sha256:5d113ef52bf90290758681fdb2790ff65645f20cf37ee3532440a7e66681baa2
Status: Downloaded newer image for beszan/echoip:latest
238bba267a61ef38669b7ae05e50d2b7e0621af45c8044e7031feba72f373d5a

$  docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS                                       NAMES
238bba267a61   beszan/echoip   "/opt/echoip/echoip …"   5 seconds ago   Up 2 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   echoip

$  curl localhost:8080
172.17.0.1
```

This allowed me to quickly test the image capabilties and make sure the container could run, but since the service would only listen to port 8080, it is not very practical for accessing it. I needed a way to expose the service to a more reachable port such as 80 and 443, a dedicated hostname and have the capability to implement a SSL cert in front of the service.

## Implementing echoip as a docker-compose service

Since I don't have a k8s cluster in production yet, I opted to run the service through a `docker-compose` file. This enabled me to place a frontend to the `echoip` container and use a more friendly domain name.

I chose to use `ifconfig.cloudalbania.com` for the service hostname so I added a DNS A record pointing to the VPS server where docker is running. As a frontend reverse proxy for the container I used [Traefik](https://traefik.io/) since it was very easy to configure and setup.

Initial tests were started from the [quickstart](https://doc.traefik.io/traefik/getting-started/quick-start/) and then I was able to deep dive in their documentation website. We are clearly using the [docker](https://doc.traefik.io/traefik/providers/docker/) provider to allow Traefik to discover docker services based on labels and the [Let's Encrypt](https://doc.traefik.io/traefik/https/acme/#configuration-examples) free SSL for our service.

We will be listening to both HTTP and HTTPS ports since requests might be coming from both web browsers and CLI clients such as [curl](https://curl.se/).

After some back and forth with the configs, in the end this is the final `docker-compose.yml` file that worked for me:

```docker-compose
version: "3.3"
services:
  traefik:
    image: "traefik:v2.8"
    container_name: "traefik"
    restart: always
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--entryPoints.web.forwardedHeaders.trustedIPs=127.0.0.1/32,172.18.0.1/16,172.17.0.1/16"
      - "--entryPoints.websecure.forwardedHeaders.trustedIPs=127.0.0.1/32,172.18.0.1/16,172.17.0.1/16"
      - "--certificatesresolvers.myresolver.acme.httpchallenge=true"
      - "--certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.myresolver.acme.email=besmirzanaj@gmail.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "./letsencrypt:/letsencrypt"

  echoip:
    image: beszan/echoip:v1.5
    container_name: echoip
    restart: always
    hostname: "ifconfig.cloudalbania.com"
    command: ["-r", "-H", "X-Real-IP", "-a", "/usr/share/GeoIP/GeoLite2-ASN.mmdb", "-c", "/usr/share/GeoIP/GeoLite2-City.mmdb", "-f", "/usr/share/GeoIP/GeoLite2-Country.mmdb"]
    volumes:
      - "/usr/share/GeoIP/:/usr/share/GeoIP/"
    deploy:
      resources:
        limits:
          memory: 64M
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.echoip-https.rule=Host(`ifconfig.cloudalbania.com`)"
      - "traefik.http.routers.echoip-https.entrypoints=websecure"
      - "traefik.http.routers.echoip-https.tls=true"
      - "traefik.http.routers.echoip-https.service=echoip-https"
      - "traefik.http.routers.echoip-https.tls.certresolver=myresolver"
      - "traefik.http.services.echoip-https.loadbalancer.server.port=8080"
      - "traefik.http.services.echoip-https.loadBalancer.passHostHeader=true"
      - "traefik.http.routers.echoip-http.rule=Host(`ifconfig.cloudalbania.com`)"
      - "traefik.http.routers.echoip-http.entrypoints=web"
      - "traefik.http.services.echoip-http.loadbalancer.server.port=8080"
      - "traefik.http.routers.echoip-http.service=echoip-http"
      - "traefik.http.services.echoip-http.loadBalancer.passHostHeader=true"

```

To start the service you can just run once `docker-compose up -d`. Since I am using `restart=always` for the containers, this service will start automatically in case the underlaying OS restarts.

```
[root@hostname echoip]# docker-compose up -d
WARNING: Some services (echoip) use the 'deploy' key, which will be ignored. Compose does not support 'deploy' configuration - use `docker stack deploy` to deploy to a swarm.
Starting traefik ... 
Starting traefik ... done
```

Checking containers

```
[root@hostname echoip]# docker ps | grep -E 'echo|traefik|CONTA'
CONTAINER ID   IMAGE                        COMMAND                  CREATED         STATUS              PORTS                                                                      NAMES
8683b586bd15   traefik:v2.8                 "/entrypoint.sh --pr…"   8 hours ago     Up About a minute   0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   traefik
59eb6310a309   beszan/echoip:v1.5           "/opt/echoip/echoip …"   8 hours ago     Up About a minute   8080/tcp                                                                   echoip
[root@hostname echoip]#
```

### Explaining the docker-compose Traefik options

I am skipping the `docker-compose` options/configs as it is out of the scope of this article and focusing more on the reverse proxy config. The Traefik daemon can be configured with either `ENV` variables, a static configuration `yaml` or `toml` file or with command arguments. In my case we are using command arguments as it is the easiest way to spin the container service.

```
# Tells Traefik to use docker service discovery. This is achieved through the volume mount at `/var/run/docker.sock:/var/run/docker.sock:ro`
--providers.docker=true
# This parameter does not allow any other docker service exposed unless told to do so
--providers.docker.exposedbydefault=false
# listen on port 80 HTTP
--entrypoints.web.address=:80
# listen on port 443 HTTPS
--entrypoints.websecure.address=:443
# forward received headers on port 80 to localhost and the docker containers only
--entryPoints.web.forwardedHeaders.trustedIPs=127.0.0.1/32,172.18.0.1/16,172.17.0.1/16
# forward received headers on port 443 to localhost and the docker containers only
--entryPoints.websecure.forwardedHeaders.trustedIPs=127.0.0.1/32,172.18.0.1/16,172.17.0.1/16
```

**Let's Encrypt section**

I am using the [HTTP01 challenge](https://letsencrypt.org/docs/challenge-types/#http-01-challenge) for domain verification.

```
# Use the HTTP01 ACME challenge
--certificatesresolvers.myresolver.acme.httpchallenge=true
# allow port 80 for Let's encrypt to do file verification
--certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web
# Let's encrypt needs a valid email account
--certificatesresolvers.myresolver.acme.email=besmirzanaj@gmail.com
# Where to save the challenge information and the generated certificate
--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json
```

**Echoip service labels section**

These labels are under the echoip service in the `docker-compose.yml` file

```
# allow service to be discovered by Traefik
traefik.enable=true

# Listen to this hostname on HTTPS port. Notice the backticks for the hostname variable.
# Then when matching that hostname let Traefik know this is a TLS service and use the "myresolver"
# certresolver for ACME Let's Encrypt automatic certificate generation.
traefik.http.routers.echoip-https.rule=Host(`ifconfig.cloudalbania.com`)
traefik.http.routers.echoip-https.entrypoints=websecure
traefik.http.routers.echoip-https.tls=true"
traefik.http.routers.echoip-https.service=echoip-https"
traefik.http.routers.echoip-https.tls.certresolver=myresolver"
traefik.http.services.echoip-https.loadbalancer.server.port=8080"

# This section is the same but for port 80 HTTP. Traefik needs separate services for each published ports/services
traefik.http.services.echoip-https.loadBalancer.passHostHeader=true"
traefik.http.routers.echoip-http.rule=Host(`ifconfig.cloudalbania.com`)"
traefik.http.routers.echoip-http.entrypoints=web"
traefik.http.services.echoip-http.loadbalancer.server.port=8080"
traefik.http.routers.echoip-http.service=echoip-http"
traefik.http.services.echoip-http.loadBalancer.passHostHeader=true"

```

## Adding GeoIP data

If you notice the `echoip` service in the `docker-compose.yml` file, it contains the `command` section which has some parameters to load GeoIP databases for countries, cities and ASNs so it can then display on the client side. I wrote a `BASH` script that will pull these databases located in `scripts/geoip_update.sh` to pull the neccessary databases. You have to run this script first before running the `docker-compose` command.

The script pulls the database files from this repo: https://github.com/P3TERX/GeoLite.mmdb. Script content below.

```bash
# Database directory
DBDIR=/usr/share/GeoIP

# Files to download
FILES="https://git.io/GeoLite2-ASN.mmdb https://git.io/GeoLite2-City.mmdb https://git.io/GeoLite2-Country.mmdb"

# If http proxy needed
#https_proxy="http://foo.bar:3128"

# DB directory
test -w $DBDIR && cd $DBDIR 2>/dev/null || { echo "Invalid directory: $DBDIR"; exit 1; }

# Sleep 0-600 sec if started from cron
if [ ! -t 0 ]; then sleep $((RANDOM/54)); fi

export https_proxy
for f in $FILES; do
  wget -nv -N -T 30 $f --directory-prefix=$DBDIR
  RET=$?
  if [ $RET -ne 0 ]; then
    echo "wget $f failed: $RET" >&2
    continue
  fi
done
```

## Using the service

The service can be used from any web client, be it a gui or a cli. Example with Chrome:

![ifconfig.cloudalbania.com](/ifconfig_website.png)

Example with CLI:

```bash
$ curl ifconfig.cloudalbania.com
64.231.237.254
```

Example with json output

```
$  curl ifconfig.cloudalbania.com/json
{
  "ip": "64.231.237.254",
  "ip_decimal": 1088941566,
  "country": "Canada",
  "country_iso": "CA",
  "country_eu": false,
  "region_name": "Ontario",
  "region_code": "ON",
  "zip_code": "L6S",
  "city": "Brampton",
  "latitude": 43.7387,
  "longitude": -79.7271,
  "time_zone": "America/Toronto",
  "asn": "AS577",
  "asn_org": "BACOM",
  "hostname": "lnsm5-toronto12-64-231-237-254.internet.virginmobile.ca",
  "user_agent": {
    "product": "curl",
    "version": "7.68.0",
    "raw_value": "curl/7.68.0"
  }
}
```

## Update 2023-05 - Implementing with Podman instead of Docker

Recently I migrated my servers from CentOS 7 to RockyLinux 9 and one of the built-in tools in the default (AppStream) repos is [Podman](https://podman.io/). There are several benefits to using podman where the main two are the capability to run containers without being an admin (root) and the other to not rely on a daemon for the container service.

### Installing podman packages

To make the docker-compose files work with podman we will need to install podman-compose from the EPEL repo:

```bash
dnf install -y epel-release
```

Then install the `podman` packages:

```bash
dnf install -y podman-compose podman
```

From here we are ready to start working with containers. See? No need to start any service/daemon.

### Adopting docker-compose files

The main difference in our `docker-compose.yml` file is the socket path for podman.

```bash
[root@hostname ]# diff docker-compose.yml podman-compose.yml

<       - "/var/run/podman/podman.sock:/var/run/docker.sock:ro"
---
>       - "/var/run/docker.sock:/var/run/docker.sock:ro"
```

After this change is implemented we can now start the containers with `podman-compose up -d`.

### Autostarting Podman containers

One issue with podman is the restart policy: The containers will not restart automatically after our server reboots. If we look at the `podman-run` [`--restart` man page](https://docs.podman.io/en/latest/markdown/podman-run.1.html#restart-policy) we will notice that the `--restart` option will not start the container(s) again when the system reboots. It says:

> Please note that `--restart` will not restart containers after a system reboot.

This is different than Docker and the reason behind this behavior is Podman's daemon-less architecture. The containers managed by Docker respect this for every reboot because the Docker daemon starts at boot and starts the specified containers. On the same `man` page you will notice the recommendation:

> Podman provides a systemd unit file, `podman-restart.service`, which restarts containers after a system reboot.
> If container will run as a system service, generate a systemd unit file to manage it. See podman generate systemd.

#### Implement container autostart with podman

In order to allow containers to autostart with podman we will follow the recommended way: enable the `podman-restart.service` systemd unit.

```bash
[root@hostname ~]# systemctl enable --now podman-restart.service
Created symlink /etc/systemd/system/default.target.wants/podman-restart.service → /usr/lib/systemd/system/podman-restart.service.
```

Now we can run our `podman-compose` commands and the `restart` policy will be applied from the newly enabled `podman-restart.service` systemd unit.

The next time the system will reboot, the containers will be up and running.
