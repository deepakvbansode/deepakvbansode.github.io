---
title: "Why We Needed Docker and Kubernetes in the First Place"
meta_title: ""
description: "Why part of docker and kubernetes"
date: 2025-04-04T05:00:00Z
image: "/images/blog/why-docker-and-k8s/docker-k8s-title.png"
categories: ["Kubernetes", "Docker"]
author: "Deepak Bansode"
tags: ["Docker", "Kubernetes"]
draft: false
---

When we learn any new technology, it’s easy to jump straight into commands or tools. But I strongly believe we should first understand the problem that technology is trying to solve. Once the problem is clear, the solution ie Docker, Kubernetes, or anything else, makes much more sense.

This article walks through the challenges in modern software development and explains how Docker and Kubernetes help us overcome them.

## How Software Was Traditionally Developed and Deployed

In a typical development process, developers work on their local machines. To run an application, they need:

1. An operating system
2. The right compilers or interpreters (Node.js, Python, Go, etc.)
3. Frameworks and libraries
4. Application code and its configuration (like database URLs, ports, etc.)

After writing the code, the developer hands it over to testers or SREs, who repeat the same steps on test and production servers—install dependencies, set configurations, and run the application.

![traditional-develpment](/images/blog/why-docker-and-k8s/traditional-development.jpg)

### Common Problems

You’ve probably heard this sentence many times:

> It works on my machine!

This happens when:

- Dependency versions differ between environments
- Configuration is not consistent
- Something is missed during manual setup

Manual deployments are slow, error-prone, and difficult to scale—especially when microservices entered the picture. In a microservices world, every service may use a different language and dependencies, making things even more complicated.

## Enter Docker: Containers to the Rescue

The main root cause of deployment issues is human intervention. When humans manually install dependencies, mistakes are bound to happen.

Docker solves this by giving us a simple idea:

> Package everything needed for your application into a single unit called a image.

How Docker Works (In Simple Words)

1. You write a Dockerfile. It contains a set of instructions about:

- What to install
- How to configure the app
- How to run it
- What ports to expose

2. You build an Image. It is a blueprint created from the Dockerfile.
3. You run Containers. It is an actual running instances created from the image.

![traditional-develpment](/images/blog/why-docker-and-k8s/docker-steps.jpg)

Think of:

**Image** = _Class_

**Container** = _Object_

Once an image works on your system, it will work on any system (same OS), because everything is packaged inside it.

![traditional-develpment](/images/blog/why-docker-and-k8s/docker-working.jpg)

Why Docker Is Powerful

- Same behavior everywhere
- No need to install compilers or libraries on test/production servers
- Fast, consistent deployments
- Great for microservices
- Efficient resource usage
- Lightweight compared to virtual machines

## But Docker Alone Is Not Enough

Imagine you have 50 microservices running as Docker containers on multiple servers.

![traditional-develpment](/images/blog/why-docker-and-k8s/micro-service-cluster.jpg)

Now think of these challenges:

1. **Scheduling**

   Where should a new container run?
   On Server 1, Server 2, or Server 3?

2. **Self-healing**

   What if a container crashes?
   Who will restart it?

3. **Service Discovery**

   How do services find each other?
   If a new service is added, who informs the rest?

4. **Load Balancing & High Availability**

   If one instance of a service goes down, traffic should move to another.
   If you have two instances, requests must be distributed properly.

5. **Networking**

   How do containers across different machines communicate securely and reliably?

To solve all this manually, you would have to write scripts, monitors, registries, load balancers—basically build your own orchestration system.

This is exactly the problem Kubernetes solves.

## Kubernetes: The Orchestrator

Kubernetes is a system designed to automate all the challenges we discussed:

- **Scheduling**
- **Restarting failed containers**
- **Service discovery**
- **Load balancing**
- **Networking**
- **Scaling**
- **And much more**

To manage everything cleanly, Kubernetes divides machines into two groups:

### 1. _Control Plane (Master Nodes)_

This is the brain of the cluster.
It contains:

- **API Server** → Entry point for all commands
- **Scheduler** → Decides which server to run a pod on
- **Controller Manager** → Watches the system and fixes issues (like restarting pods)
- **etcd** → A database storing cluster state
- **Cloud Controller Manager** → Interacts with cloud providers (AWS, GCP, etc.)

### 2. _Worker Nodes (Data Plane)_

These nodes actually run your applications.

They run:

- **Kubelet** → Communicates with API server and creates pods
- **Kube Proxy** → Handles networking
- **Container Runtime (Docker, containerd, etc.)** → Creates containers

![traditional-develpment](/images/blog/why-docker-and-k8s/k8s.jpg)

**Note**:

> In Kubernetes, you don’t directly create containers.
> You create Pods, which are wrappers around containers. A pod can contain one or multiple containers.

## How Everything Comes Together

Let’s say you want to deploy a new microservice:

1. You apply a YAML file (using kubectl apply). YAML contains the desired state of your application.
2. The API server receives the request.
3. The information is stored in etcd.
4. The scheduler selects the best node to run it.
5. Kubelet on that node creates a pod using the container runtime.
6. Controller Manager keeps watching the pod. If it dies, a new one is created.
7. Kube Proxy ensures networking works.

Everything is handled automatically.
This is the power of Kubernetes.

## Final Thoughts

Docker solved the packaging and consistency problem.
Kubernetes solved the orchestration and reliability problem.

Together, they form one of the most powerful platforms for building and running applications at scale.
