# Single Machine Kubernetes Cluster

A single machine Kubernetes cluster is a perfect development environment for writing applications against Kubernetes and an easy way to get started with your first Kubernetes cluster. This document will give some background first, but you can [skip to the cloud-config][full-cloud-config] if you aren't interested.

## Using the Kubelet to Bootstrap the Cluster

The Kubernetes kubelet is an agent that runs on machines in your cluster to start and stop [pods][pod-overview] (groups of containers). In normal operation, the kubelet communicates with the Kubernetes API server to receive commands &mdash; start this pod, create an iptables rule for this service, etc.

In addition to normal events, the kubelet is also responsible for registering a node with a Kubernetes cluster, sending events and pod status, and reporting resource utilization.

The kubelet has one last feature that we're going to use to bootstrap our single-machine cluster. In addition to starting and stopping pods via commands from the API server, the kubelet also watches a local directory for pod manifests that it should start. The location of this directly is configured via a the `--config` flag.

Since the kubelet is already installed in CoreOS, we're going to use it to run all of the Kubernetes components (API server, controller manager, etc) in a single pod by dropping a single pod manifest on disk. But first, we need to start the kubelet.

## Create a Kubelet Unit

```
sudo vim /etc/systemd/system/kubelet.service
```

```
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStartPre=/usr/bin/mkdir -p /etc/kubernetes/manifests
ExecStart=/usr/bin/kubelet \
  --api-servers=http://127.0.0.1:8080 \
  --allow-privileged=true \
  --config=/etc/kubernetes/manifests \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Here's a quick breakdown of the flags that we're settings:

| Flag | Description |
|:-----|:------------|
| --api-servers | We're pointing the Kublet at the API server over localhost, since our Kubernetes components are going to run on this machine. |
| --allow-privileged | The Kubernetes proxy must run within a privileged container. |
| --config | The location where we are going to place our pod manifest that runs the Kubernetes components. |

Now we're ready to enable the unit, so it starts after a reboot, and then start it:

```
$ sudo systemctl daemon-reload
$ sudo systemctl start kubelet
$ sudo systemctl enable kubelet
```

The Kubelet should now be running, and you can verify this by checking the status:

```
sudo systemctl status kubelet
```

## Creating our Kubernetes Pod Manifest

Pod manifests are written in the JSON or YAML file formats and describe a set of volumes and one or more containers. Normally pod manifests are submitted through the API server, but as we described above, we can also use the kubelet to start a pod directly.

We're going to start the rest of the required Kubernetes components in a single pod by placing this manifest in the configured manifest directory. After this, we can submit regular pod manifests through the newly started API server.

You can find the full manifest on Github. SSH onto your CoreOS machine and download it:

```
wget https://raw.githubusercontent.com/coreos/pods/master/kubernetes.yaml
```

As always, check the contents of the file you downloaded before running it:

```
cat kubernetes.yaml
```

Now copy the manifest into the directory watched by the Kublet:

```
sudo cp kubernetes.yaml /etc/kubernetes/manifests/
```

After a few seconds you should see the Kubelet start downloading and running the containers:

```
sudo docker images
sudo docker ps
```

The last step is to download and configure `kubectl` so we can interact with our new cluster.

## Interact with the Cluster

Kubernetes ships a command line tool called `kubectl` that allows you to create and destroy objects, see the status of your pods, etc. You can run `kubectl` from the CoreOS machine (covered below) or remotely from your laptop by configuring it with the API endpoint of your cluster (not covered).

First, download the latest release:

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.0.3/bin/linux/amd64/kubectl
chmod +x kubectl
```

Since our API is running at the default location (`http://localhost:8080`) we don't have to do any configuration:

```
$ ./kubectl cluster-info
Kubernetes master is running at http://localhost:8080
```

Now you're ready to start launching stuff:

```
./kubectl run nginx --image=nginx
```

You should immediately see the pod:

```
./kubectl get pods
```

After the nginx image downloads, you should be able to curl the pod's IP address from the CoreOS machine and see the default page.


## Full Cloud-Config

Here are the same steps self-contained within a single cloud-config file. For a local development environment, use this in conjunction with the [CoreOS Vagrant guide][vagrant-guide].

Be sure to add in an SSH-key so you can connect to the server.

```yaml

```

<div class="co-m-docs-next-step">
  <p><strong>Are you familiar with pods and services?</strong></p>
  <a href="services.md" class="btn btn-default">Services overview</a>
  <a href="pods.md" class="btn btn-default">Pods overview</a>
  <a href="index.html" class="btn btn-link">Back to Listing</a>
</div>

[full-cloud-config]: #full-cloud-config
[upstream-rc]: http://kubernetes.io/v1.0/docs/user-guide/replication-controller.html
[pod-overview]: pods.md
[service-overview]: services.md
[vagrant-guide]: {{site.baseurl}}/os/docs/latest/booting-on-vagrant.html