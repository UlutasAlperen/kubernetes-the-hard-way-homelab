# kubernetes-the-hard-way-homelab
---
Why the "Very" Hard Way?

This course takes a comprehensive, hands-on approach that goes far beyond typical Kubernetes tutorials. You won't simply assemble the cluster and move on. Instead, you'll interact with every component as you install it.

This means:

    Testing each component individually before moving to the next
    Understanding the APIs and interfaces each component exposes
    Seeing how components communicate with each other
    Exploring the configuration options and their effects

This thorough, step-by-step interaction with each building block is what makes this course both comprehensive and incredibly educational.

By the end of this journey, you'll have a deep understanding of how Kubernetes works under the hood: not just how to use it, but how it actually functions internally.
What This Course is (and Isn't)

The primary goal of this course is educational: to explain the role of each Kubernetes component and how they work together to form a functioning cluster.

This course is NOT a guide for setting up production-ready clusters. In some instances, the examples deliberately use simplified or insecure configurations to make the learning experience more accessible and help you focus on understanding the core concepts.

For production deployments, always follow security best practices, use proper certificate management, implement network policies, and consider using established tools like kubeadm, kops, or managed Kubernetes services.
Course Structure

Each lesson in this course follows a consistent pattern:

    Download the necessary components for that lesson
    Configure them manually (with detailed explanations of each configuration option)
    Run them as systemd services
    Observe their behavior

Each lesson builds upon the previous ones, gradually introducing new concepts and components.

Additionally, each lesson starts with a playground environment that includes all previously installed components from earlier lessons, so you can focus on new material without rebuilding everything from scratch.

---

Lab Environment

The course includes dedicated lab environments, so you can focus entirely on learning without setting up or managing virtual machines.

Everything you need will be provided in a ready-to-use environment.
Versions

The course was tested with the following versions:

    containerd: 2.2.1
        runc: v1.4.0
        CNI plugins: v1.9.0
        nerdctl: 2.2.1
    etcd: v3.6.4
    Kubernetes: v1.34.0
    Network addon (flannel): v0.27.2
        CNI plugin: v1.7.1-flannel2
    CoreDNS: 1.12.2

These are the latest versions available at the time of writing, following the version compatibility matrices provided by the respective projects.
