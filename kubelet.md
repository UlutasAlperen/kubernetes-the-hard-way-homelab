---
# Overview

A container runtime alone can't join a cluster or accept work from a control plane. That's kubelet's job. In this lesson, you'll install kubelet, the node agent that bridges Kubernetes and the container runtime.

Objectives:

- Understand kubelet's role within a Kubernetes cluster
- Install and configure kubelet from scratch
- Observe how kubelet interacts with containerd via the CRI (Container Runtime Interface)
- Understand the relationship between Pods and containers
- Run Pods using kubelet without an API server
- Learn basic debugging techniques for kubelet

> you'll have kubelet running standalone, managing Pods without a control plane.


## What is Kubelet

**kubelet** is the **primary node agent** that runs on every worker node in a Kubernetes cluster. As one of the core components that makes Kubernetes work, it acts as the bridge between the Kubernetes control plane and the container runtime on each node.

kubelet operates in a continuous reconciliation loop:

1. **Watches** for Pod specifications from the API server
2. **Compares** the desired state (what should be running) with the actual state (what is running)
3. **Takes action** to bring the actual state in line with the desired state
4. **Reports** the current status back to the control plane

In essence, kubelet is the **"hands and feet"** of Kubernetes on each node: it takes declarative Pod specifications from the control plane and makes them a reality by managing containers on the worker nodes.

However, kubelet doesn't run containers directly. Instead, it delegates this responsibility to a **container runtime** like [containerd](https://containerd.io) or [CRI-O](https://cri-o.io). To support multiple runtimes and provide flexibility in runtime selection, kubelet communicates with these runtimes through the [Container Runtime Interface (CRI)](https://kubernetes.io/docs/concepts/architecture/cri/)
