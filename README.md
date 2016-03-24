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
    - [Copying Images to Other Hosts](#copying-images-to-other-hosts)
    - [Top writeable layer of containers](#top-writeable-layer-of-containers)
    - [One Process per Container](#one-process-per-container)
    - [Commands for working with Containers](#commands-for-working-with-containers)

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

### Copying Images to Other Hosts

Later in the course, will use Docker Hub to push and pull images. But can also do this manually by saving container image to a tar file and exporting it.

To run a container with a short lived command (run container, run command, then process exits). This is detached mode, i.e. not using `-it` flags:

```shell
docker run ubuntu /bin/bash -c "echo 'cool content' > /tmp/cool-file"
```

This makes a _change_ to the container because it created a file. To verify, run `docker ps -a`. Sample output:

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
c583c0b6afab        ubuntu              "/bin/bash -c 'echo '"   6 minutes ago       Exited (0) 35 seconds ago                       amazing_keller
bd04a65de4fd        ubuntu              "bash"                   24 hours ago        Exited (0) 24 hours ago                         agitated_goldberg
4bd18fa5121f        ubuntu              "bash"                   4 days ago          Exited (127) 3 days ago                         pedantic_stonebraker
7240d4bb88c0        hello-world         "/hello"                 4 days ago          Exited (0) 4 days ago                           insane_poitras
```

To create a new image from the changes just made to the container, where `fridge` is the name to be assigned to the image:

```shell
docker commit c583c0b6afab fridge
```

Now to  verify newly created image, run `docker images`, sample output:

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
fridge              latest              6a2caa759236        8 seconds ago       188 MB
ubuntu              latest              07c86167cdc4        2 weeks ago         188 MB
hello-world         latest              690ed74de00f        5 months ago        960 B
```

To see all the commands that were used to create an image, run `docker history fridge`.

To save our newly created image:

```shell
docker save -o /tmp/fridge.tar fridge
```

To look inside the contents of the tar:

```shell
tar -tf /tmp/fridge.tar
```

To import the tar file on another machine runningDocker:

```shell
docker load -i /tmp/fridge.tar
```

### Top writeable layer of containers

Containers are run-time instances of images. When a container is launched with `docker run`, the Docker engine reads the image and any metadata, then builds the container by stacking the different image layers, as per the instructions in the image metadata.

_Each container gets its own thin writeable layer on top of the read-only image layers below it_ . All changes to a container are made in this top writeable layer, for example, installing and updating applications, writing new files, config changes like ip address. All container state is stored in this top writeable layer. This layer is initially empty, it only consumes space as changes are made to the container.

The `rootfs` of a container is never made writeable. But due to union mounts, end up with "look and feel" of a regular writeable file system.

### One Process per Container

Generally it's good practice for containers run a single app or process.

When the process running inside the container exits, so does the container. For example, run an ubuntu container in detached mode (`-d` to make it run in the background) and execute a single command (`-c` specifies the command) to ping Google's 8.8.8.8, 10 times:

```shell
docker run -d ubuntu /bin/bash -c "ping -c 10 8.8.8.8"
```

 Hitting return from above displays the container's ID. Running `docker ps` while container is still running will show the status. When container finishes running the command, `docker ps` will show nothing because there is no more active docker process running.

 To see top running processes inside a running container:

 ```shell
 docker top {containerID}
 ```

 It's good practice to be _very specific_ about which image to run. For example, instead of simply `ubuntu`, specify which version/tag like `ubuntu:14.04`. Otherwise it will download `latest` tag which could be different from today to tomorrow.

### Commands for working with Containers

Docker `run` command has many switches. For example:

* `-it` to run in interactive mode and with a shell
* `-d` to run container detached and in the background
* `--cpu-shares` to control how many cpu shares container gets (1024 is all, 256 is a quarter), by default, all containers on a host get equal access to shares.
* `memory=1g` to specify how much memory to allocate to container, for example 1g.

To get detailed information about a container:

```shell
docker inspect {containerID}
```
