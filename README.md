<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [Docker Deep Dive](#docker-deep-dive)
  - [Major Docker Components](#major-docker-components)
    - [Images](#images)
    - [Containers](#containers)
    - [Registries and Repositories](#registries-and-repositories)
  - [Closer look at Images and Containers](#closer-look-at-images-and-containers)
    - [Image Layers](#image-layers)
    - [Union Mounts](#union-mounts)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Docker Deep Dive

> Learning Docker with [Pluralsight course](https://app.pluralsight.com/library/courses/docker-deep-dive/description)

## Major Docker Components

### Images

Docker containers are launched from Docker images. i.e. image is a build-time concept, and containers are runtime.

For example, to launch an Ubuntu container and run a bash shell inside of it:

```shell
docker run -it ubuntu bash
```

`it` specifies interactive and terminal.

`ubuntu` specifies which image to base container on.

`bash` which process or application to run. Can be anything that is installed on the container.

Sample output

```
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
5a132a7e7af1: Pull complete
fd2731e4c50c: Pull complete
28a2f68d1120: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:4e85ebe01d056b43955250bbac22bdb8734271122e3c78d21e55ee235fc6802d
Status: Downloaded newer image for ubuntu:latest
```

`Unable to find image 'ubuntu:latest' locally` Since this is the first time an ubuntu container is being launched, don't have a local copy, therefore it gets pulled from Docker Hub, which is the public Docker registry.

`5a132a7e7af1, fd2731e4c50c, etc.` Images are comprised of multiple layers, each number represents a layer (more details later in course).

`ubuntu:latest` On Docker hub, there are many different versions of ubuntu, if a version number is not specified as part of the `run` command, then Docker will grab the image that's tagged as the `latest` version.

Docker `pull` command is used to pull images from Docker hub and install them locally. Saves time when running containers, don't have to wait for download.

To pull all versions of an image (rather than just latest which is the default)

```shell
docker pull -a ubuntu
```

To list all the images locally

```shell
docker images ubuntu
```

IMAGE ID uniquely identifies each image. Note the same image can have multiple different tags.

Docker images contain all the data and metadata required to fire up a container.

### Containers

In order to launch a container, need an image. `docker run` is used to launch a container.

`docker ps` to see a list of running images.

`docker ps -a` to see all containers that have been run on the host.

To attach to an existing running image `docker attach {containerid}`

If you're in an interactive container, can exit it without killing it by pressing Ctrl + P + Q

If you don't specify a name on the docker run command, docker will assign a name.

### Registries and Repositories

Images are pulled from _repositories_, which live inside a _registry_.

The default public registry for Docker is _Docker Hub_.

Within Docker Hub, there are many repo's, for example, the official (i.e. trusted) repos for Fedora, Ubuntu, Redis, MongoDB, etc.

Each of these repos contains different images, for example for each version such as Ubuntu 12.0.4, 14.0.4, etc.

[Browse docker hub](https://hub.docker.com/explore/)

User repos look like, for example `docker pull radial/nginx`. Be careful when pulling from a non official repo!

## Closer look at Images and Containers

### Image Layers

Images are layered (or stacked). Sometimes layers are referred to as images, which can be confusing.

For example, 3 layers stacked on top of each other, with 0 on the bottom:

```
Layer 2 (Image 2)
Layer 1 (Image 1)
Layer 0 (Image 0)
```

Together, these layer/images form a single image. i.e. a single image comprised of three layered images. An example stack of layers:

At the bottom layer, there is the _Base Image (rootfs)_. This has the root file system, which is all the files and directories required to make up a container's stripped down, bare minimum OS, for example, Ubuntu.

The next layer, _Layer 1_ could be the application layer, for example nginx. And next _Layer 2_ might have some updates or config files.

A single image can be shared by multiple containers. The layered approach allows for tweaks and updates to higher layers, without touching the base layer.

### Union Mounts

Each image or layer gets its own unique id. These id's are listed inside the Docker image, plus metadata that tells Docker how to build the container at run time. If there are any conflicts, higher layer overrides lower layer.

Image layering is accomplished through _union mounts_. The ability to mount file systems on top of each other, combining all the layers into a single view.

All of the layers in the image are mounted as read only. Then an additional layer is added when container is launched, which is the _pnly writeable_ layer.

All changes to the container at run time are committed to this top layer, via copy on write behaviour.
