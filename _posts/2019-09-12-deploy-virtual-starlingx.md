---
layout: post
title: "Deploy a virtual StarlingX Simplex node"
date: 2019-09-12
---

I've been working in this project for one year and a half and has been a great experience in many ways. Currently the [StarlingX 2.0 was announced](http://lists.starlingx.io/pipermail/starlingx-discuss/2019-September/005881.html), were some big changes and features were introduced.

I install and deploy [StarlingX](https://www.starlingx.io) almost every day to test the system and sometimes just to reinstall it when I broke it. Although the project is intended to be run in baremetal systems, it is possible to create virtual deployments for testing and evaluation purposes.

The testing team has created a [really cool test suite](https://opendev.org/starlingx/test/src/branch/master/automated-robot-suite) that deploys StarlingX in baremetal and virtual environments. In this post I'll explain how to use this test suite to deploy a Simplex StarlingX installation.

## Requirements

I'm using a system with the following specs: 

  - Processor Intel Xeon 3.8 GHz
  - 64Gb of RAM
  - 1 TB disk.
  
The requirements might look excessive but in this system I'm able to run all the StarlingX configurations. The Simplex configuration is the least demanding one and I've run it in a system with 32GB of RAM. The configuration with external storage requires to run 6 VMs and it's the most demanding.

I'm using Fedora 30 but the test suite can run in Ubuntu 16.04 and 18.04. 

Also, ensure that you can run `sudo` commands without password.

## Install dependencies

The following packages are required: 

```
$ sudo dnf install virt-manager libvirt-daemon qemu-system-x86 python-pip python-devel redhat-rpm-config libvirt-client xterm
$ sudo pip install virtualenv
```

### Configure libvirt/qemu

This can be the most tricky part of this process, to get qemu work with the test suite.

Ensure you have this values set in the `/etc/libvirt/qemu.conf` file.

```
user = "root"
group = "root"
dynamic_ownership = 0
```

And then restart the libvirtd service

```
$ sudo systemctl restart libvirtd
```

## Prepare test suite

To clone the repository run:

```
git clone https://opendev.org/starlingx/test.git
```

The files under `Config` folder needs to be edited to define or remove the docker http proxy definition. To me it worked like this.

```
dns_servers:
  - 8.8.8.8

#docker_http_proxy: http://<url>:<port>
#docker_https_proxy: http://<url>:<port>

external_oam_subnet: 10.10.10.0/24
external_oam_gateway_address: 10.10.10.1
external_oam_floating_address: 10.10.10.3

ansible_become_pass: ANSIBLE_PASS
admin_password: ADMIN_PASS
```

## Download files

Before running the test suite, two files needs to be present in the same directory, the StarlingX ISO image and the armada file. The ISO is used to install the base OS and the armada file to deploy the containerized environment.

To download the StarlingX 2.0 release run: 

```
wget http://mirror.starlingx.cengn.ca/mirror/starlingx/release/2.0.0/centos/outputs/iso/bootimage.iso
```

and the armada file:

```
wget -O stx-openstack.tgz http://mirror.starlingx.cengn.ca/mirror/starlingx/release/2.0.0/centos/outputs/helm-charts/helm-charts-stx-openstack-centos-stable-versioned.tgz
```

## Run suite

First, install test suite dependencies in a virtual environment.

```
$ virtualenv myenv
$ source myenv/bin/activate
(myenv) $ pip install -r requirements.txt
(myenv) $ pip install -r test-requirements.txt
```

Now we can run the test suite:

```
(myenv) $ python runner.py --run-suite Setup --configuration 1 --environment virtual
```

Here we are telling the test suite to run the `Setup` section which creates the VMs and installs the ISO. With `--configuration 1` that we want a simplex configuration and with `--environment virtual` that we want a virtual environment execution.

If everything goes right, you should see something like this:

![Launch test suite](https://ericho.github.io/img/stx-vid.gif)

Now we can wait until the installation completes. If you want to see more you can use `virt-manager` to see something like this:

![virt-manager](https://ericho.github.io/img/stx-1.png)

![virt-manager](https://ericho.github.io/img/stx-3.png)

After `Setup` finishes we need to run the `Provision` set. 

```
(myenv) $ python runner.py --run-suite Provision
```

This may take a while to finish, depending on your Internet speed connection. In this process a set of containers will be downloaded. After the finished you should be able to load the StarlingX admin interface. 

![Starlingx admin page](https://ericho.github.io/img/stx-4.png)

![Starlingx host inventory](https://ericho.github.io/img/stx-5.png)

