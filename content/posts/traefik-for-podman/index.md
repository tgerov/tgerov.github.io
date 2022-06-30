---
categories:
- Linux
date: "2022-02-03T10:45:58+02:00"
draft: false
tags:
- linux
- podman
- traefik
- fedora
title: How To Use Traefik as a Reverse Proxy for Podman (rootless)
summary: Use Traefik as Reverse Proxy for Podman rootless containers
---

## Introduction
Traefik is a leading modern reverse proxy and load balancer that makes deploying microservices easy. Traefik integrates with your existing infrastructure components and configures itself automatically and dynamically. [1]

Podman is a daemonless, open source, Linux native tool designed to make it easy to find, run, build, share and deploy applications using Open Containers Initiative (OCI) Containers and Container Images. Podman provides a command line interface (CLI) familiar to anyone who has used the Docker Container Engine.  Containers under the control of Podman can either be run by root or by a non-privileged user. [2]

In this tutorial I'm using Fedora 35 for host OS with enabled SELinux and Firewalld. This means that we will need to execute few commands with root to allow rootless container (traefik) to listen on privileged ports (80, 443). We also will enable LetsEncrypt and HTTPS redirect in Treafik.

## Install podman

You can install it using the following command:

```bash
root# dnf install -y podman
```

After installation, run the hello world image to ensure everything is working:

```bash
root# podman run hello-world
```

## Create a regualr user for containers

Create a regular user or use existing one.  In this tutorial I will use user "container" with UID 1000.

```bash
root# useradd containers
root# passwd containers
```

Note: We need to login via SSH or Console with the new user. You cannot use "su - user" as this does not create a proper user session and custom exports for DBUS are required.  In this tutorial I will use ssh containers@localhost.

## Allow privileges ports binding for non-root users
```bash
root# echo 'net.ipv4.ip_unprivileged_port_start=80' >> /etc/sysctl.conf
root# sysctl -p
```

## Allow port 80 and 443 in Firewalld
```bash
root# firewall-cmd --add-service={http,https} --permanent
root# firewall-cmd --reload
```

## Enable lingering for user container

```bash
root# loginctl enable-linger containers
```

This allows users who are not logged in to run long-running services, otherwise containers will stop once you logout.

## Enable Podman.socket

```bash
containers$ systemctl --user enable --now podman.socket
```

This will create a podman socket in /run/user/1000/podman/podman.sock, where 1000 is your UID.

## Create a Traefik container
### Create podman netwok web
```bash
containers$ podman network create web
```

### Create an empty acme.json file with privileges 600
```bash
containers$ touch acme.json
containers$ chmod 0600 acme.json
```

### Run Traefik
Make sure that email is properly set, otherwise you will get an error.

We need to run this container with an SELinux context that allows it to talk the podman socket, otherwise SELinux will block traefik communitaction with the podman.socket.

If you don't add security opt label, you will get a similar error in SELinux

```bash
type=AVC msg=audit(1643875799.990:1912): avc:  denied  { connectto } for  pid=26594 comm="traefik" path="/run/user/1000/podman/podman.sock" scontext=system_u:system_r:container_t:s0:c41,c864 tcontext=unconfined_u:system_r:container_runtime_t:s0-s0:c0.c1023 tclass=unix_stream_socket permissive=0
```

### Create Traefik container
```bash
containers$ podman run --name=traefik -d \
  --security-opt label=type:container_runtime_t \
  -v /run/user/1000/podman/podman.sock:/var/run/docker.sock:z \
  -v ./acme.json:/acme.json:z \
  --net web \
  -p 80:80 \
  -p 443:443 \
  --restart always \
  docker.io/library/traefik:latest \
  --entrypoints.web.address=":80" \
  --entrypoints.web.http.redirections.entryPoint.to=websecure \
  --entrypoints.web.http.redirections.entryPoint.scheme=https \
  --entrypoints.websecure.address=":443" \
  --providers.docker=true \
  --certificatesresolvers.le.acme.email=your@email.local \
  --certificatesresolvers.le.acme.storage=/acme.json \
  --certificatesresolvers.le.acme.tlschallenge=true 	
```
### Create a systemd unit for the traefik container
```bash
containers$ mkdir -p ~/.config/systemd/user/
containers$ cd ~/.config/systemd/user/
containers$ podman generate systemd -f -n traefik
containers$ systemctl --user enable container-traefik.service
```
Now,  lets stop podman container and start it via systemd

```bash
containers$ podman stop traefik
containers$ systemctl --user start container-traefik.service
```

You can verify that the traefik is running with the one of the following commands:

```bash
containers$ podman ps
CONTAINER ID  IMAGE                             COMMAND               CREATED        STATUS            PORTS                                     NAMES
759a25c8c203  docker.io/library/traefik:latest  --entrypoints.web...  2 minutes ago  Up 2 minutes ago  0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp  traefik
```

or

```bash
containers$ systemctl --user status container-traefik.service
● container-traefik.service - Podman container-traefik.service
     Loaded: loaded (/home/containers/.config/systemd/user/container-traefik.service; enabled; vendor preset: disabled)
     Active: active (running) since Thu 2022-02-03 07:39:02 UTC; 2min 48s ago
       Docs: man:podman-generate-systemd(1)
    Process: 21755 ExecStart=/usr/bin/podman start traefik (code=exited, status=0/SUCCESS)
   Main PID: 21920 (conmon)
      Tasks: 16 (limit: 1112)
     Memory: 30.4M
        CPU: 510ms
		... cut
```

## Examples
After configuring the traefik reverse proxy, we will create a few examples. Examples will not follow the best practices, they will be run only for test purpose, so no persistent volumes will be used.

In order for Traefik to route our traffic correctly, we will need a few things:

- Container should be in network "web"
- Labels should be set
- Port should be exposed
- Traefik should known on which port to route traffic

### Ghost CMS
Our Ghost platform will run with the following host: ghost.podman.gerov.eu, so you should replace hostname in the example below:

```bash
containers$ podman run -d --name ghost \
  --network=web \
  -l traefik.http.routers.ghost.rule=Host\(\`ghost.podman.gerov.eu\`\) \
  -l traefik.http.services.ghost.loadbalancer.server.port=2368 \
  -l traefik.http.routers.ghost.tls.certresolver=le \
  -l traefik.http.routers.ghost.tls=true \
  -e url=https://ghost.podman.gerov.eu \
  --expose=2368 \
  docker.io/library/ghost:latest
```

We will wait a minute and test if Ghost is running over HTTPS with valid Let's Encrypt certificate.

```bash
containers$ curl -I https://ghost.podman.gerov.eu/
HTTP/2 200
cache-control: public, max-age=0
content-type: text/html; charset=utf-8
date: Thu, 03 Feb 2022 07:58:54 GMT
etag: W/"5eac-sKMD98knbLhfiGj86JrLrs+J4Ow"
vary: Accept-Encoding
x-powered-by: Express
content-length: 24236
```

### PHP 7.4 & Apache

```bash
containers$ podman run -d --name php74-apache \
  --network=web \
  -l traefik.http.routers.php74.rule=Host\(\`php.podman.gerov.eu\`\) \
  -l traefik.http.services.php74.loadbalancer.server.port=80 \
  -l traefik.http.routers.php74.tls.certresolver=le \
  -l traefik.http.routers.php74.tls=true \
  --expose=80 \
  php:7.4-apache
```


## References:
[1] https://traefik.io/traefik/  
[2] https://docs.podman.io/en/latest/  
[3] Photo: [Robin Pierre](https://unsplash.com/photos/dPgPoiUIiXk)


