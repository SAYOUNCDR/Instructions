# Docker — Notes and Quick Reference

This document consolidates Docker notes, commands, examples, and best practices. It mirrors the contents of the original `Docker.txt` and is organized for easier reading and quick lookups.

---

## Problem statement

- "Works on my machine" failures often happen because not all tools or dependencies are installed on the target system, or because dependency versions mismatch.
- Some development environments depend on Linux-specific tooling, which may not run natively on macOS or Windows.
- Reproducible, consistent deployment across machines and auto-scaled servers is tedious without packaging; Docker containers simplify this by packaging the OS, tools, and dependencies together.

## Solution overview

Docker wraps up dependencies, tools, and a lightweight userland (not a full VM hypervisor) into an image. When run, an image becomes a container that acts like a lightweight, isolated environment for your app or service.

- Image: a standalone, executable package containing an application and everything it needs to run.
- Container: a runtime instance of an image. Multiple containers can be created from the same image; each container is isolated with its own filesystem and state.

---

## Quick concepts

- Images are immutable; containers are writable instances of images (changes in a container do not affect the image).
- Containers are isolated from one another by default (separate filesystems and runtime state).
- Images can be reused to run many containers.

In real development you usually build custom images that include the OS layer plus your runtime and dependencies (for example: Ubuntu + Node + MongoDB + Redis in separate images/services).

---

## Installation

- Docker Desktop: provides the Docker CLI and GUI for macOS and Windows.
- Docker Daemon: the background service that runs containers, builds images, and manages Docker resources.

Check Docker version:

```
docker -v
```

Example output:

```
Docker version 28.4.0, build d8eb465
```

---

## Running an image (example: Ubuntu)

Start an interactive Ubuntu container:

```
docker run -it ubuntu
```

If the image is not found locally, Docker will pull it from Docker Hub. Example output when pulling:

```
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
4b3ffd8ccb52: Pull complete
Digest: sha256:...
Status: Downloaded newer image for ubuntu:latest
root@ef82f012dc9c:/#
```

Notes:

- `-it` stands for interactive + TTY. You get a shell inside the container.
- Images are the canonical artifacts; containers are runtime instances.

---

## Common container commands

```
docker container ls           # list running containers
docker ps                    # same as above
docker container ls -a       # list all containers (including stopped)
docker start <container>     # start a stopped container
docker stop <container>      # stop a running container
docker rm <container_or_id>  # remove a container (must be stopped)
```

Execute commands inside a running container:

```
docker exec <container> <cmd>         # run a command inside the container (non-interactive)
docker exec -it <container> bash     # start an interactive shell inside the container
```

Examples of `docker exec` usage:

```
docker exec myapp cat /var/log/app.log
docker exec mydb mysql -uroot -psecret -e "SHOW DATABASES;"
docker exec myapp printenv
docker exec myapp nginx -s reload
```

---

## Images

```
docker images                # list local images
docker rmi <image_or_id>     # remove an image (image must not be used by any container)
```

Notes on image deletion and renaming:

- Renaming (tagging) an image does not duplicate the image; it creates another reference. Deleting a tag may only remove the reference, not the underlying image layers.
- Use the image ID to ensure you remove the actual image when needed.

Renaming / retagging example:

```
docker pull ubuntu:22.04
docker tag ubuntu:22.04 sayoun/dev-base:1.0
docker images
```

---

## Port mapping and access

Containers run services in their own network namespace. If a service runs on port 8000 inside the container, that port is not automatically exposed to the host machine. You must map the container port to a host port when you run the container.

Example (host port 4000 -> container port 3000):

```
docker run -it -p 4000:3000 <image_name>
```

You can also set environment variables at runtime rather than baking them into images. Example:

```
docker run -it -p 4000:3000 -e KEY=VALUE -e ANOTHER=VAL <image_name>
```

---

## Building and running a custom image (Node.js example)

Example Dockerfile:

```
FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
		apt-get install -y curl ca-certificates gnupg && \
		curl -fsSL https://deb.nodesource.com/setup_22.x | bash - && \
		apt-get install -y nodejs && \
		apt-get clean && \
		rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

CMD ["node", "index.js"]
```

Build the image and run it:

```
docker build -t <image_name> .   # run in the directory containing Dockerfile
docker run -p <host_port>:<container_port> <image_name>
```

Notes:

- Port mapping is set when creating a container from an image; to change ports you must create a new container with different mapping or remove and recreate the container.
- To enter a running container:

```
docker exec -it <container_id_or_name> bash
```

---

## Build cache

Docker builds images layer by layer. Each instruction in the Dockerfile creates a layer. Docker will reuse cached layers when the exact same instruction and inputs were used previously.

If an instruction changes (or its inputs change), the cache is invalidated from that instruction onward and the remaining steps are re-executed. Proper Dockerfile ordering maximizes cache reuse (for example, copy package.json and run npm install before copying source files that change frequently).

---

## Publishing images to Docker Hub

1. Create an account on hub.docker.com
2. Log in with the Docker CLI: `docker login`
3. Tag and push your image:

```
docker push <image_name>
```

---

## Docker Compose

Docker Compose manages multi-container applications with a single YAML definition (`docker-compose.yml` or Compose V2 `docker-compose.yml`). It describes services, networks, volumes, and configuration for every container in the app.

Example `docker-compose.yml` (version 3.9):

```
version: "3.9"
services:
	db:
		image: postgres:15
		environment:
			POSTGRES_PASSWORD: secret
		volumes:
			- db-data:/var/lib/postgresql/data

	backend:
		build: ./backend
		ports:
			- "5000:5000"
		depends_on:
			- db

	frontend:
		build: ./frontend
		ports:
			- "3000:3000"
		depends_on:
			- backend

volumes:
	db-data:
```

Useful commands:

```
docker compose up             # start services (foreground)
docker compose up -d          # start services (detached)
docker compose down           # stop and remove containers and networks
docker compose down --rmi all # remove containers and images
docker compose down -v        # remove containers, networks, and volumes
docker compose up --build -d  # build images and start in detached mode
```

Notes:

- `docker compose down` removes containers and networks by default but leaves named volumes intact. Use `-v` to remove volumes. `--rmi all` removes images.

---

## Networking

Docker networks allow containers to communicate. Two common network modes are bridge (default) and host.

- Bridge (default): Docker creates a virtual bridge between the host and containers. Port mapping (-p) is required to expose container ports to the host.
- Host: The container uses the host network stack directly; no port mapping is required.

Commands:

```
docker network ls
docker network inspect <network_name>
```

Example output from `docker network inspect bridge` shows container details like IP addresses and MAC addresses. Containers attached to the same user-defined network can reach each other by container name (DNS provided by Docker).

- Create a user-defined network (bridge mode):

```
docker network create -d bridge myNetwork
```

- Run a container on a specific network:

```
docker run -it --network=myNetwork --name mycontainer -p 3000:3000 <image_name>
```

Note: Containers on the same user-defined network can communicate without mapping ports to the host.

---

## Volumes and persistent storage

When you remove a container, the data inside it is lost unless you use volumes or bind mounts.

```
docker run -it -v <host_path>:<container_path> <image_name>   # bind mount a host directory

docker run -it --name <container> --mount source=<vol_name>,target=/app <image_name>  # use a named volume
```

Named volumes are managed by Docker and are the recommended way to persist data between container recreations.

---

## Multi-stage builds

Multi-stage builds help keep final images small by separating build and runtime stages. Only the final stage is included in the final image; build artifacts can be copied from earlier stages.

Example:

```
# Stage 1: Build
FROM node:20 AS builder
WORKDIR /app
COPY package*.json tsconfig*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Run
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --only=production

# Copy built files from builder
COPY --from=builder /app/dist ./dist

# Run compiled JS
CMD ["node", "dist/index.js"]
```

Key points:

- `AS builder` names the build stage so you can copy from it.
- `COPY --from=builder` copies artifacts from the earlier build stage.
- Great for frontend and compiled backends where you don't want build tools in the production image.

---

## Examples and handy commands

- Get inside a MongoDB container using mongosh:

```
docker exec -it mongo_db mongosh
```

---

## Best practices / notes

- Prefer passing secrets and environment-specific values at runtime (`docker run -e ...`) or use Docker secrets and environment files; avoid baking secrets into images.
- Arrange Dockerfile instructions to maximize layer cache reuse (install dependencies before copying frequently changing code).
- Use multi-stage builds to reduce final image size and attack surface.
- Use named volumes for persistent data.
- Use Docker Compose to coordinate multi-service applications.

---

## References

- Docker documentation: https://docs.docker.com

---

End of document.
