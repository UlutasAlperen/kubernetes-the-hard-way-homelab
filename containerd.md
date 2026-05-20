---
## Overview

Kubernetes needs a container runtime to actually run containers. Orchestrates container execution through OCI runtimes and CNI plugins.

Objectives:

    Understand containerd's role within a Kubernetes cluster
    Install and configure containerd from scratch
    Explore the tools and ecosystem surrounding containerd
    Understand how containerd relies on open standards such as CNI (Container Network Interface) and OCI (Open Container Initiative) to perform container operations

By the end of the day, you'll have a fully configured container runtime capable of pulling images and running containers.

---

## What is containerd

containerd is an industry-standard container runtime that provides the fundamental tools for running containers.

Originally developed by Docker, containerd is now maintained by the CNCF and has become one of the most widely adopted container runtimes in the cloud-native ecosystem.

containerd provides several essential services to Kubernetes:

    Container Lifecycle Management: Creating, starting, stopping, and deleting containers
    Image Management: Pulling, storing, and managing container images from registries
    Storage Management: Handling container filesystems and volume mounts
    Network Management: Coordinating with CNI plugins for container networking
    Runtime Management: Interfacing with low-level runtimes

## install containerd

containerd can be downloaded from the project's [GitHub Releases page](github.com/containerd/containerd/releases).

Download and install containerd:
```bash
CONTAINERD_VERSION=2.2.1

curl -fsSLO "https://github.com/containerd/containerd/releases/download/v${CONTAINERD_VERSION?}/containerd-${CONTAINERD_VERSION?}-linux-amd64.tar.gz"

sudo tar xzvofC "containerd-${CONTAINERD_VERSION?}-linux-amd64.tar.gz" /usr/local
```
Download the systemd unit file to run containerd as a systemd service:

```bash
sudo wget -P /etc/systemd/system "https://raw.githubusercontent.com/containerd/containerd/v${CONTAINERD_VERSION?}/containerd.service"
```
Start the containerd service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

## interacting with containerd

*containerd* includes a CLI tool called ctr for basic container operations.

Think of ctr as similar to the Docker CLI, but designed for lower-level container operations.

Unlike higher-level tools like the Docker CLI, ctr requires you to manually pull an image before running it:

```bash
sudo ctr images pull ghcr.io/sagikazarmark/docker-hello-world:latest
```

With the image pulled, you can run the container:

```bash
sudo ctr run --rm ghcr.io/sagikazarmark/docker-hello-world:latest hello
```
>This command will fail because no OCI runtime is installed yet. The error output shows what's missing. 

```bash
ctr: failed to create shim task: OCI runtime create failed:
unable to retrieve OCI runtime error
(open /run/containerd/io.containerd.runtime.v2.task/default/hello/log.json: no such file or directory):
exec: "runc": executable file not found in $PATH
```
---

## installing oci runtime

*containerd* is a high-level container runtime that provides a wide range of container management services (like image management, storage, and networking), but delegates certain tasks to specialized components. One such task is actually running the container process, which containerd delegates to a low-level OCI runtime.

In practical terms, containerd manages the "what" (which container to run, with what image, network, and storage), while OCI runtimes handle the "how" (the underlying isolation mechanisms like cgroups, namespaces, or VMs).

Historically, container management used a highly integrated, monolithic architecture (think Docker Engine).

As the ecosystem matured and standardization efforts advanced, the need for a more modular and flexible approach became clear.

This evolution led to the development of various high-level container runtimes and the establishment of the Open Container Initiative (OCI).

The OCI Runtime Specification standardizes how containers should be created, started, and managed at the low level, abstracting away platform-specific isolation details. This allows high-level runtimes like containerd to work with any OCI-compliant (aka. low-level) runtime without knowing the implementation details.

- What is container-shim

```m
The containerd-shim is a lightweight process that acts as a bridge between containerd and OCI runtimes.

It provides a stable interface for containerd to interact with the runtime, keeps containers running even if containerd crashes, handles container I/O, and reaps processes when they exit.

```
The reference OCI runtime implementation is [runc](https://github.com/opencontainers/runc), which serves as the default runtime for containerd.

Other OCI runtimes provide different isolation mechanisms: *crun* (also uses cgroups/namespaces but with better performance) and gVisor (uses a user-space kernel for VM-like isolation).

Although runc remains the most widely used and well-tested option, the beauty of OCI is that containerd can use any of these runtimes interchangeably without code changes.

Download and install runc:

```bash
RUNC_VERSION=v1.4.0

curl -fsSLO "https://github.com/opencontainers/runc/releases/download/${RUNC_VERSION?}/runc.amd64"

sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

Configure containerd to use the systemd cgroup driver with runc:

```bash
/etc/containerd/config.toml

version = 3

[plugins."io.containerd.cri.v1.runtime".containerd.runtimes.runc.options]
SystemdCgroup = true
```
**cgroups** (control groups) limit and isolate resource usage (CPU, memory, I/O) for container processes.

When using systemd as the init system, it's recommended to use the systemd cgroup driver so both systemd and the container runtime manage cgroups consistently.

This ensures the container runtime and systemd coordinate through a single cgroup hierarchy, rather than managing resources separately, which can cause instability under memory or CPU pressure.


Restart the containerd service to apply the configuration changes:

```bash
sudo systemctl restart containerd```

With runc installed and configured, the container now runs successfully:

```bash
sudo ctr run --rm ghcr.io/sagikazarmark/docker-hello-world:latest hello
```
---
## installing nerdctl

While ctr is an excellent tool for low-level interaction with containerd, it isn't the most user-friendly tool, especially for users familiar with the Docker CLI.

Fortunately, there's an alternative tool called nerdctl (contaiNERD CTL) that provides a Docker CLI-like interface for interacting with containerd.

Download and install nerdctl:


```bash
NERDCTL_VERSION=2.2.1

curl -fsSLO "https://github.com/containerd/nerdctl/releases/download/v${NERDCTL_VERSION?}/nerdctl-${NERDCTL_VERSION?}-linux-amd64.tar.gz"

tar xzvof "nerdctl-${NERDCTL_VERSION?}-linux-amd64.tar.gz"

sudo install -m 755 nerdctl /usr/local/bin

nerdctl completion bash | sudo tee /etc/bash_completion.d/nerdctl
```
Unlike *ctr*, *nerdctl* automatically handles image pulling and network setup.

Verify this by running a container:


```bash
sudo nerdctl run --rm ghcr.io/stefanprodan/podinfo:latest /home/app/podinfo --version
```
 This command will fail because CNI plugins are not installed yet.


 ```bash

FATA[0000] failed to create shim task: OCI runtime create failed:
runc create failed: unable to start container process:
error during container init: error running createRuntime hook #0:
exit status 1, stdout: , stderr: time="2025-07-17T12:39:15Z" level=warning msg="Container failed starting. Removing allocated network configuration."

time="2025-07-17T12:39:15Z" level=fatal msg="failed to call cni.Setup: plugin type=\"bridge\" failed (add): failed to find plugin \"bridge\" in path [/opt/cni/bin]"
 ```
While *ctr* doesn't automatically create a network for containers, nerdctl attempts to create a bridge network by default. Without CNI plugins installed, this network creation fails.

---

## installing CNI Plugins

Just as containerd delegates container process execution to OCI runtimes, it delegates network configuration to specialized components through the Container Network Interface (CNI).

CNI is a specification and set of libraries for configuring network interfaces in Linux containers. It provides a standardized way for container runtimes to set up container networking, including creating network namespaces, configuring IP addresses, and establishing connectivity between containers and the host system.

Various CNI plugins are available, many of which can be found in the official [CNI plugins repository](CNI plugins repository).

*nerdctl*, *containerd* uses the bridge CNI plugin by default to set up the container's network namespace.

Download and install CNI plugins:

```bash
CNI_PLUGINS_VERSION=v1.9.0

curl -fsSLO "https://github.com/containernetworking/plugins/releases/download/${CNI_PLUGINS_VERSION?}/cni-plugins-linux-amd64-${CNI_PLUGINS_VERSION?}.tgz"

sudo mkdir -p /opt/cni/bin

sudo tar xzvofC "cni-plugins-linux-amd64-${CNI_PLUGINS_VERSION?}.tgz" /opt/cni/bin

```
> CNI plugins are installed in /opt/cni/bin by convention.
 Container runtimes and Kubernetes network add-ons look for them in this directory by default.


With CNI plugins installed, containerd is now fully functional.

Run the container again using nerdctl:

```bash
sudo nerdctl run -d --name podinfo ghcr.io/stefanprodan/podinfo:latest
```

Verify that the container is running:

```bash
sudo nerdctl ps
```

Verify that podinfo is working correctly:

```bash
PODINFO_IP=$(sudo nerdctl inspect --format '{{ .NetworkSettings.IPAddress }}' podinfo)
curl -f "http://${PODINFO_IP}:9898"
```
---
### Summary

Learned about containerd, the high-level container runtime that serves as the foundation for container management on Kubernetes worker nodes.

Key takeaways:

    Industry standard: containerd is a CNCF-maintained, industry-standard container runtime that originated from Docker and powers container execution in Kubernetes clusters
    Modular architecture: containerd orchestrates container operations while delegating specific tasks to specialized components: OCI runtimes (like runc) handle process execution and CNI plugins manage networking
    Essential services: Provides container lifecycle management, image management, storage coordination, and network interface setup for Pods
    Management tools: Use ctr for low-level debugging and nerdctl for Docker-like commands when working directly with containerd

With containerd now running and fully configured with runc and CNI plugins, you're ready to set up kubelet, the Kubernetes node agent that will use containerd to manage Pods.

