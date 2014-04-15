---
layout: docs
slug: guides
title: Install Debugging Tools
category: cluster_management
sub_category: debugging
weight: 7
---

# Install Debugging Tools

You can utilize common debugging software like tcpdump or  through [Toolbox](https://github.com/coreos/toolbox). Toolbox will launch you directly into the namespace of a docker container that you specify. This container has full system privileges and should provide the same access to network cards and other hardware that you're used to on other Linux distributions.

## Quick Debugging

By default, Toolbox uses the stock Fedora docker container. To start using it, simply run:

```
/usr/bin/toolbox
```

You're now in the namespace of Fedora and can install any software you'd like via `yum`. For example, if you'd like to use `tcpdump`:

```
[root@srv-3qy0p ~]# yum install tcpdump
[root@srv-3qy0p ~]# tcpdump -i ens3
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens3, link-type EN10MB (Ethernet), capture size 65535 bytes
```

### Specify a Custom Docker Image

Create an `.toolboxrc` in the user's home folder to use a specific docker image:

```
$ cat .toolboxrc
TOOLBOX_DOCKER_IMAGE=index.example.com/debug
TOOLBOX_USER=root
$ /usr/bin/toolbox
Pulling repository index.example.com/debug
...
```

## SSH Directly Into A Toolbox

Advanced users can SSH directly into a toolbox by setting up an `/etc/passwd` entry:

```
useradd bob -m -p '*' -s /usr/bin/toolbox
```

To test, SSH as bob:

```
ssh bob@hostname.example.com

   ______                ____  _____
  / ____/___  ________  / __ \/ ___/
 / /   / __ \/ ___/ _ \/ / / /\__ \
/ /___/ /_/ / /  /  __/ /_/ /___/ /
\____/\____/_/   \___/\____//____/
[root@srv-3qy0p ~]# yum install emacs
[root@srv-3qy0p ~]# emacs /media/root/etc/systemd/system/docker.service
```