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

After writing the code, the developer hands it over to testers or SREs, who repeat the same steps on test and production servers,install dependencies, set configurations, and run the application.

![traditional-develpment](/images/blog/why-docker-and-k8s/traditional-development.jpg)

### Common Problems

You’ve probably heard this sentence many times:

> It works on my machine!

This happens when:

- Dependency versions differ between environments
- Configuration is not consistent
- Something is missed during manual setup

Manual deployments are slow, error-prone, and difficult to scale, especially when microservices entered the picture. In a microservices world, every service may use a different language and dependencies, making things even more complicated.

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

### 1. Where Should Containers Run?

When you have multiple machines (servers), and need to start a new container:

- Should it run on Server A?
- Does Server A has enough RAM and CPU to run new container?
- Would Server B be better?

Manually deciding where containers go becomes too hard as the number of servers grows. This problem is called **scheduling** problem.

---

### 2. How Do We Keep Services Running?

Containers can fail due to various issues ex: Panics in code. that’s normal.

When a container stops:

- How do we notice it?
- Who restarts it?
- How do we make sure there are always running?

Doing this manually for hundreds of containers is not practical.

---

### 3. How Do Services Find Each Other?

Imagine one service (like a backend) needs to talk to another (like a notification).

In simple setups, you might connect with an IP address. But in a real system:

- Containers start and stop dynamically
- IP addresses can change
- Multiple copies of a service might exist

Hardcoding IPs doesn’t work, we need **service discovery** that tracks where each service is currently running.

---

### 4. How Is Traffic Distributed?

If there are multiple copies of a service:

- Which copy should handle incoming requests?
- How do we make sure the load is balanced?

Manually sending all traffic to one instance can overload it. We need systems that **spread requests evenly** across replicas.

---

### 5. How Do We Handle Failures and Recovery?

In a real environment:

- A server might crash
- Network issues can occur
- Containers might stop unexpectedly

We want systems that can automatically detect these failures and **self-heal** without human intervention.

---

## These Are Orchestration Problems

All of the problems above are not solved by Docker itself. They belong to a category called **orchestration problems**, issues that arise when you try to manage many containers across many machines.

This is where **Kubernetes steps in**.

Kubernetes is designed specifically to solve these orchestration issues:

- Deciding where containers should run
- Restarting failed services automatically
- Ensuring consistent service discovery
- Balancing traffic across containers
- Maintaining the desired state of the system

By automating these tasks, Kubernetes helps make systems **reliable, scalable, and easier to operate**.

---

## What Kubernetes Actually Manages

Rather than directly managing individual containers, Kubernetes works with a higher-level concept called a **Pod**.

A **Pod** is the smallest unit in Kubernetes and can contain one or more containers. Kubernetes schedules, monitors, and manages Pods, not containers directly. This abstraction helps Kubernetes handle groups of containers together.

---

## The Declarative Approach of Kubernetes

One powerful idea in Kubernetes is this:

> **You tell Kubernetes what you want, not how to do it.**

Instead of writing scripts that list steps, you define your **desired state** in a manifest file:

- How many copies (replicas) you want
- Which container image to use
- What resources are required

Kubernetes continuously works in the background to ensure the real system matches your desired state, restarting, rescheduling, and healing if needed.

---

## Kubernetes: The Orchestrator

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

## How Everything Comes Together

Let’s say you want to deploy a new microservice:

1. You create a YAML file with desired state of application.
2. You apply a YAML file (using kubectl apply).
3. The API server receives the request.
4. The information is stored in etcd.
5. The scheduler selects the best node to run it.
6. Kubelet on that node creates a pod using the container runtime.
7. Controller Manager keeps watching the pod. If it dies, a new one is created.
8. Kube Proxy ensures networking works.

Everything is handled automatically.
This is the power of Kubernetes.

## Final Thoughts

Docker solved the packaging and consistency problem.
Kubernetes solved the orchestration and reliability problem.

Together, they form one of the most powerful platforms for building and running applications at scale.
