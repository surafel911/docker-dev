# Table of Contents
## [Quick Links](#Quick-Links)
## [Introduction](#Introduction)
### [Docker](#Docker)
### [Containers](#Containers)
### [Docker Engine](#Docker-Engine)
### [Docker Compose](#Docker-Compose)
## [Docker Tips](#Docker-Tips)
### [Docker](#Docker)
### [Containers](#Containers)
### [Dockerfile](#Dockerfile)
#### [Multistage Builds](#Multistage-Builds)


# Quick Links
* (https://github.com/wsargent/docker-cheat-sheet/blob/master/README.md)
* (https://www.docker.com/sites/default/files/d8/2019-09/docker-cheat-sheet.pdf)
* (https://dockerlabs.collabnix.com/docker/cheatsheet/)

# Introduction

## Docker

Docker is a tool and platform for packaging, deploying, and running
applications that can by used and run on any system and host OS. The main
advantage of Docker is for developers to easily develop applications,
ship them into a standardized unit (i.e. a sandbox
called a **_container_**) and deploy them anywhere. This works by packaging
an application with all of its dependencies intofor development and deployment.

## Containers

Containers are what make Docker. **_Containerization_** is a kernel
technology that's existed for a long time, but has become powerful with Docker.

A **_container_** is a running process on your machine that's isolated from the rest
of the machine; conceptually similar to a VM but with much less overhead. It
contains all the code, dependencies, configurations, and runtime environment to
to execute the application.

It's important to note that Docker containers don't run in their own
virtual machine, rather they share the machine's OS kernel and therefore
don't require a kernel per container; driving up resource efficiency.

Containers can either be based on Linux or Windows kernels. Containers based
on one OS can run on the other, so there is no developer-snamee overhead.

You *might* have to finesse to get Linux containers running on Windows.

![Docker Containers Diagram](https://www.docker.com/sites/default/files/d8/styles/large/public/2018-11/container-what-is-container.png?itok=vle7kjDj)

Containers are built from a container images (i.e. **_images_**), a
standardized unit of software that packages up all code with all of its
dependencies so that the application can run quickly and reliably
from one environment to another.

Images become containers when they are run on the **_Docker Engine_**.

## Docker Engine

**_Docker Engine_** is the industry standard container runtime and server
that runs on various Linux distrubutions and Windows operating systems. Docker
Engine allows containerized applications to run consistently on any infrastructure.

Docker Engine acts as a client-server application with:
* A server as a daemon process: `dockerd`
* Docker APIs for programs to instruct the Docker daemon
* A CLI client `docker`

![Docker Engine diagram](https://www.docker.com/sites/default/files/d8/styles/large/public/2018-11/Docker-Website-2018-Diagrams-071918-V5_a-Docker-Engine-page-first-panel.png?itok=TFiL1wtt)

## Docker Compose

`docker-compose` is a CLI tool to automatically create Docker images through
commands; conceptually similar to Makefiles

# Docker Tips

## Docker

* Only use Linux containers.
* Use alpine Docker images when possible.
* Useful for testing out new programs without installing them locally.

## Containers

* Data persists _only_ when stopped/paused. Data _doesn't_ persist if container is destroyed.

## Docker CLI

Create a `docker` group and add user to avoid using `sudo` (_warning: gives
same privleges as `sudo`_). Alternatively run docker daemon in [rootless mode](https://docs.docker.com/engine/security/rootless/).

Some `docker` commands have tab-completion.

Find container from name:
```shell
name=$(docker ps -a | grep patient-service | awk '{printf $1}')
```

Run image:
* `-d`: detach process
* `-p` {host port}:{container port}`: map host port to container port
* `-it`: run in interactive mode w/ shell after container starts
  * must have {path to shell} arg
  * cannot be used w/ `-d`
* `--rm`: Remove container on exit
* `--name {name}`: name container
```shell
docker run {options} {image name} {args}
```
List all containers
```shell
docker ps -a
```

List all container (only IDs):
```shell
docker ps -aq
```

Force remove running containers:
```shell
docker rm -f {container name}
```

Fetch logs:
* `-f`: stream output
```shell
docker logs [-f] {container name}
```

Remove image:
```shell
docker rmi {image name}:{tag}
```

Build image:
* `--build-arg {arg}={value}`: send build-time args to Dockerfile
  * Must be called for every build-time arg
  * {path to Dockerfile} can be just a `.`, but must be present
* `{tag}` used to track versions. can be left blank, auto set to `latest` or set to version #
```shell
docker build -t {image name}:{tag} {path to Dockerfile}
```
Remove all images (optional force):
```shell
docker rmi [-f] $(docker images -aq)
```

Remove unused images (optional force):
```shell
docker image prune -a [-f]
```

Remove all containers (optional force):
```shell
docker rm [-f] $(docker ps -aq)
```

Remove unused containers (optional force):
```shell
docker container prune -a [-f]
```

Remove all unused data (optional force):
```shell
docker system prune -a [-f]
```

## Dockerfile Tips

* [Dockerfile best practices](https://www.docker.com/blog/intro-guide-to-dockerfile-best-practices/)
* Only use alpine images when possible.


Example:
```shell
FROM mcr.microsoft.com/dotnet/sdk:5.0-alpine AS build
WORKDIR /source

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN rm -rf bin/
RUN dotnet publish -c Release

FROM mcr.microsoft.com/dotnet/runtime:5.0-alpine AS runtime
WORKDIR /DataAtThePointOfCare
COPY --from=build /source/ ./
```

Initialize build stage:
* Must always be called first
```shell
FROM {image name}:{tag} [AS {stage}]
```

Arguments:
Build-time variables/arguments for dynamic image creation.
Can't be used in a build stage if declared outside the build stage,
so either declare before or after build stage

Copying:
* Host to image
```shell
COPY {path from Dockerfile} {image path from `WORKDIR`}
```
* Stage to image
```shell
COPY --from={stage} {source} {path from root} {image path from `WORK DIR`}

### Multistage Builds

Using multistage builds and best-practices:
* (https://docs.docker.com/develop/develop-images/multistage-build/)

Multiple build stages (i.e. **_multi-stage builds_** can be used to
combine files.

e.g.

```shell
FROM surafel911/data-at-the-point-of-care-lib AS dep
FROM mcr.microsoft.com/dotnet/sdk:5.0-alpine AS build

COPY --from=dep /DataAtThePointOfCare ../DataAtThePointOfCare
```

Alternatively [copy from environment](https://stackoverflow.com/questions/53687769/including-an-external-library-in-a-docker-cmake-project).
