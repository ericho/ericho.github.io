---
layout: post
title: "Using docker as building environment"
date: 2018-02-01
---

The normal usage of docker is to be used as a container of an application. That's great because you can have all the dependencies inside the container and then just run that as a another command. Also, docker can be used as a container of dependencies to build software. This usage can be seen in Travis CI, for example, sometime ago I worked to [enable a C/C++ project in Travis CI](https://travis-ci.org/intel-ctrlsys/sensys) with a [nightmare of dependencies involved](https://github.com/intel-ctrlsys/infra-helpers/blob/master/Dockerfiles/sensys-bld/Dockerfile). I solved that by creating a Dockerfile with all the dependencies needed and then build the software inside the container.

But, can we use the same building model from our development systems? And yes, of course is possible!. Imagine you have Fedora (as my case) or macOS or even Windows and you need to develop and test software for CentOS 7.2/7.3 and openSUSE. You can achieve this with docker. In the following sections I'll describe how to setup a docker image to build C++17 software under CentOS 7.3.

## Build the image

First, you need to define a Dockerfile. This one will install the devtoolset to have c++17 as well as other needed tools.

```
FROM centos:latest

RUN yum install -y git make sudo centos-release-scl gdb && \
    yum install -y devtoolset-7-gcc-c++

RUN echo "%wheel        ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers

ENV HOME /home/user

RUN groupadd --gid 1000 user
RUN useradd --create-home --home-dir $HOME --uid 1000 --gid 1000 user

WORKDIR $HOME

USER user
```

Noticed that the user `user` was created with `--uid 1000`. The user and group id should match with the user of your host machine. This is because we'll share the source folder between the container and the host system.

Once the image finished to build, we can tag it.

```
Removing intermediate container 950f60014a93
Successfully built af466b33ba83
-bash-4.2$ docker tag af466b33ba83 centos-building
```

## Running the image

The proposed workflow here would be,
    1. To have a folder with the source code.
    2. Run `/bin/bash` from the container and share that folder with the host system.
    3. Build our program inside the container.
    
### Disable SELinux

Sharing folders between the container and the host system is something that won't make SELinux happy. Therefore we need to disable SELinux (temporarily or permanent, as you decided). To do this, at least in fedora, run:

```
sudo setenforce 0
```

### Run the container.

Now, let's say we have a folder under `/home/erich/src/` then you can run the container as follows:

```
$ docker run -ti -v /home/erich/src:/home/user/src centos-building /bin/bash
```

The C++17 toolset needs to be enabled, to that run: 

```
source /opt/rh/devtoolset-7/enable
```

With the user created inside the container we can work freely with the folder in the host system.

## Write some code

I'm using this C++ code as an example: 

```c++
#include <iostream>
#include <vector>

using namespace std;

template<typename... Args>
auto sum(Args... args)
{
    return (0 + ... + args);
}

template<typename F, typename... T>
void for_each(F fun, T&&... args)
{
    (fun (std::forward<T>(args)), ...);
}


int main()
{
    cout << sum(1,2,3,4) << endl;
    
    for_each([](auto elem) {
        cout << elem << endl;
    }, 1, "hello", 10.0);
}
```

and I saved that into the test.cc file. This code compiles with: 

```
g++ test.cc -Wall -Wextra -o test
```

Now we have a binary `test` in the host system.

### Need gdb?

You may need to run some `gdb` and you'll need to do it from the container. For this you'll need to pass some security options into docker.

```
docker run -ti --security-opt seccomp=unconfined -v /home/erich/src:/home/user/src centos-building /bin/bash
```

And you can use `gdb` freely.

## Drawbacks

Here are some possible problems using this mode: 
    1. The image size is huge: As there is a lot of dependencies installed it's expected to have big images.
    2. I only testing this mode in Linux, I tried in Windows 10 using the Linux subsystem, but unfortunately, sharing folders between Linux and Windows doesn't work as expected.
