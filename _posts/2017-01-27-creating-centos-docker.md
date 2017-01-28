---
layout: post
title: "Creating a CentOS 7 docker image"
date: 2017-01-27
---

This post is not particularly innovator. There is a lot of pages and tutorials on how to create docker images. I noticed that most of these tutorials focus on projects for web technologies. Although this is one the biggest usages of Docker it is not the only one.

This week a needed to build [Sensys](https://github.com/intel-ctrlsys/sensys), a C/C++ on which I've been work the last year, on [Travis-CI](https://travis-ci.org). The main challenge here is that Sensys is intended to compile and run on CentOS 7.x, while the build environment of Travis runs Ubuntu.

The first approach was to try to build Sensys on Ubuntu but I failed, not all the dependencies are available on Ubuntu's repos. Sensys is also an old project that has some dependencies that needs to be installed manually.

## Docker to the rescue

The solution was simple:
  - Download a CentOS 7 docker image.
  - Build a new docker image with all the dependencies.
  - Push the image into https://hub.docker.com/
  - Build Sensys whenever I want without the need of a Centos system :smile:

### The Dockerfile

The [Dockerfile](https://github.com/ericho/Dockerfiles/blob/master/sensys-bld/Dockerfile) only has two statements:

```
FROM centos:centos7

RUN <ALL COMMANDS SEPARATED BY &&>
```

In my poor knowlegde of Docker, every time a `RUN` statement is run, it runs in a separate layer. So putting all the commands in the same RUN statement will reduce the docker image build time as only one layer is needed.

### Building with docker

I tagged this image as sensys-bld, as is intended just to build Sensys. The image is avaiable in the docker hub [here](https://hub.docker.com/r/erichcm/sensys-bld/) and can be obtained with:

```
docker pull erichcm/sensys-bld
```

Let's say that we have the source of on `/home/user/src`, so the following steps will build the source code using Docker.

```
cd /home/user/src
docker run -t -i -v "$PWD":/build erichcm/sensys-bld sh -c "cd /build && ./configure && make"
```
