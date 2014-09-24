---
layout: docs
title: Cluster Architectures
category: cluster_management
sub_category: setting_up
forkurl: https://github.com/coreos/docs/blob/master/cluster-management/setup/cluster-architectures/index.md
weight: 5
---

# CoreOS Cluster Architectures

## Overview

Depending on the size of your cluster and how it's going to be used, there are a few common cluster archictures that can be followed. Each are described below.

Most of these scenarios set aside a few machines dedicated to running central cluster services, such as etcd and the distributed controllers for applications like Kubernetes, Mesos, and OpenStack. Separating out these services into a few known machines allows you to make sure they are distributed across cabinets/availability zones and easily set up static networking to allow for easy bootstrapping. If you're worried about relying on the discovery service, this architecture will remove your worries.

## Small Cluster

[diagram]

| Cost | Great For | Set Up Time | Production |
|------|-----------|-------------|------------|
| Low | Small clusters, trying out CoreOS | Minutes | Yes |

For small clusters between 3-9 machines, running etcd on all of the machines allows for high availability without paying for extra machines that just run etcd.

Getting started is easy &mdash; a single cloud-config can be used to start all machines on a cloud-provider.

### Configuration

cloud-config here...

## Easy Development/Testing Cluster

[diagram]

| Cost | Great For | Set Up Time | Production |
|------|-----------|-------------|------------|
| Low | Development/Testing | Minutes | No |

When you're first getting started with CoreOS, it's common to frequently tweak your cloud-config which requires booting/rebooting/destroying many machines. Instead of being slowed down and distracted by generating new discovery urls and bootstrapping etcd, it's easier to start a single etcd node.

You are now free to boot as many machines as you'd like as test workers that read from the etcd node. All features of such as fleet, locksmith and etcdctl will continue to work properly, but will connect to the etcd node instead of using a local etcd instance. Since etcd isn't running on all of the machines, you'll gain a little bit of extra CPU and RAM to play with.

This environment is now set up to take a beating. Pull the plug on a machine and watch fleet reschedule the units, max out the CPU, etc.

## Production Cluster with Central Services

[diagram]

| Cost | Great For | Set Up Time | Production |
|------|-----------|-------------|------------|
| High | Large bare-metal installations | Hours | Yes |

For clusters larger than 9 machines, it's recommended to set aside 3-5 machines to run central services. Once those are set up, you can boot as many workers as your heart desires. Each of the workers will use the distributed etcd cluster on the central machines.

`fleet` can be used to bootstrap both the central services and jobs on the worker machines by using fleet's machine metadata and global units.

| Unit Function | Target | X-Fleet Parameters |
|---------------|--------|--------------------|
| Monitoring      | All Machines     | `Global=true`                                    |
| Kubernetes      | Central Services | `Global=true`<br/>`MachineMetadata=role=central` |
| WebApp          | Any Worker       | `MachineMetadata=role=worker`                    |
| Dist. Database  | Worker in Cab 1  | `MachineMetadata=role=worker,cabinet=one`        |


- 3-5 machines running central services
 - separate out into different cabinets if possible
 - fleet metadata
 - static networking
 - stable channel
- many worker machines with fleet metadata
 - workers are etcd proxies in 0.5, env in 0.4x
 - can be used to autoscale easily
 - any channel

## Large-Scale Cluster with Central Services

[diagram]

| Cost | Great For | Set Up Time | Production |
|------|-----------|-------------|------------|
| High | Sophisticated environments with multiple availability zones | Hours | Yes |

- central services in different AZs
- workers are etcd proxies in 0.5, env in 0.4x
- PXE booted, DHCP