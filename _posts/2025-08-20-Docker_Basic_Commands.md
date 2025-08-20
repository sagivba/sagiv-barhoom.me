---
layout: post
title:  "Docker Basic Commands"
author: "Sagiv Barhoom"
date:   2025-08-20
categories: Dockers 
background: '/img/posts/Dockers.jpg'
---


# Docker Basic Commands (note to myself)

## What is an Image and a Container?
- **Image**: A lightweight, stand-alone package that contains everything needed to run a piece of software, including the code, runtime, libraries, and dependencies. Think of it as a template.
- **Container**: A running instance of an image. Containers are isolated environments that run the application based on the image.

## Images

- `docker images` - List all images available locally.
- `docker pull <repository:tag>` - Download (pull) an image from a registry.
- `docker rmi <image_id>` or `docker rmi <repository:tag>` - Remove an image from the local machine.
- `docker image prune` - Remove dangling (unused) images.

## Containers

- `docker ps` - List all running containers.
- `docker ps -a` - List all containers (running and stopped).
- `docker run <repository:tag>` - Create and start a new container from an image.
- `docker stop <container_id or name>` - Gracefully stop a running container.
- `docker kill <container_id or name>` - Immediately stop a container (force).
- `docker rm <container_id or name>` - Remove a stopped container.
- `docker stop $(docker ps -q)` - Stop all running containers.
- `docker rm $(docker ps -aq)` - Remove all containers (running and stopped).

## Useful Flags with `docker run`

- `-d` - Run container in detached (background) mode. Detached mode means the container runs in the background without attaching to your terminal, allowing you to keep using the shell while the container keeps running.
- `-p <host_port>:<container_port>` - Map a port on the host to a port in the container.
- `--name <name>` - Assign a name to the container.
- `-v <host_path>:<container_path>` - Mount a host directory as a volume inside the container.

## Example: Download and Run an Nginx Image
### Download the official Nginx image
```bash
docker pull nginx:latest
```
### List images (check that the image is ready to use)
```bash
docker images
REPOSITORY    TAG        IMAGE ID       CREATED       SIZE
nginx         latest     33e0bbc7ca9e   7 days ago    279MB
```

### Run the container 
- Run the container in detached mode (so it runs in the background without blocking the terminal),
- Map port 8080 on the host to port 80 in the container,
- assign it a name "mynginx", and mount a local folder as a volume.

```bash
# on windows  ${PWD}  
docker run -d -p 8080:80 --name mynginx -v  ${PWD}/html:/usr/share/nginx/html nginx:latest
# on linux
docker run -d -p 8080:80 --name mynginx -v  $(pwd)/html:/usr/share/nginx/html nginx:latest
```

### Check running containers
```bash
docker ps
```

```bash
docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS                                     NAMES
0cc4aa2b3d2d   nginx:latest   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   mynginx
```
### Stop the container
```bash
docker stop mynginx          # Stop by name
docker stop 0cc4aa2b3d2d     # Stop by ID
```

## Logs
Docker logs are stored by the Docker engine (not inside the container’s filesystem). You can view them using the `docker logs` command. By default, logs are managed by Docker and not written to standard files on the host.

### View logs of the container named mynginx
```bash
docker logs mynginx
```
### Follow logs in real-time
```bash
docker logs -f mynginx
```

## Execute Commands Inside a Container
You can run commands inside a running container using `docker exec`.

### Open an interactive shell inside the container
```bash
docker exec -it mynginx /bin/bash
```
### Check Nginx version inside the container
```bash
docker exec -it mynginx nginx -v
```

## Restart a Container
You can restart an existing container without removing it.
```bash
docker restart mynginx        # Restart by name
docker restart 0cc4aa2b3d2d   # Restart by ID
```

## Clean Up Resources
Remove all unused containers, networks, images, and optionally volumes.
```bash
docker system prune
```
