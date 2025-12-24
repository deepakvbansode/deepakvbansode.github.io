---
title: "Why Kubernetes Needs Services and Networking"
meta_title: ""
description: "Understanding k8s"
date: 2025-04-04T05:00:00Z
image: "/images/image-placeholder.png"
categories: ["Kubernetes", "Docker"]
author: "Deepak Bansode"
tags: ["Docker", "Kubernetes"]
draft: true
---

_This is the next article in the Kubernetes series, continuing from the earlier posts on Pods, ReplicaSets, and Deployments._

In the previous article, we understood how Kubernetes runs applications using Pods, ReplicaSets, and Deployments. At that point, applications are running — but they are still **isolated**.

The next big question is:

> How do these applications talk to each other reliably?

This article explains **why Kubernetes needs Services and networking primitives**, what problems they solve, and how Kubernetes hides networking complexity behind simple abstractions.

---

## The Basic Networking Promise of Kubernetes

Kubernetes makes a very strong promise:

> **Every Pod gets its own IP address, and every Pod can talk to every other Pod directly.**

It does not matter:

- Which node the Pod is running on
- Whether the Pods are on the same node or different nodes

From an application’s point of view, networking should "just work".

However, making this promise true in real systems is **not simple**.

---

## The Real Problem: Pods Are Dynamic

Pods are not permanent.

- Pods can be destroyed
- New Pods can be created during scaling
- Pods can move to different nodes
- Pod IP addresses can change

If applications directly talk to Pod IPs, they would constantly break.

For example:

- A frontend Pod needs to call a backend Pod
- Backend Pods scale from 2 to 4
- One backend Pod crashes and is recreated

Now the frontend has **no reliable way** to know:

- How many backend Pods exist
- Which ones are healthy
- Which IP addresses to use

This problem appears everywhere — reverse proxies, frontends, and internal services.

---

## Why Not Let Applications Handle This?

One option is to push this responsibility to applications:

- Track all Pod IPs
- Perform health checks
- Implement load balancing
- Handle failures and retries

But this creates serious issues:

- Every application must re-implement the same logic
- Business logic gets mixed with infrastructure logic
- Systems become hard to maintain and debug

Kubernetes solves this problem **once**, at the platform level.

---

## Services: A Stable Way to Reach Pods

A **Service** is a Kubernetes object that provides:

- A **stable virtual IP address**
- A **stable DNS name**
- Automatic load balancing
- Automatic health-aware routing

Even though Pods come and go, the Service **remains stable**.

Applications talk to the Service — not directly to Pods.

---

## How a Service Knows Which Pods to Route To

Services do not magically discover Pods.

They use **labels and selectors**.

- Pods are created with labels (for example: `app=backend`)
- A Service defines a selector using the same label

Kubernetes continuously watches:

- Which Pods match the selector
- Which Pods are healthy

This information is stored internally as **Endpoints**.

If a Pod is added or removed, the endpoint list is updated automatically.

---

## Services Are Logical — Not Physical

A Service is **not a running process**.

It does not run as a container or Pod.

Instead:

- The Service exists as a logical object in the control plane
- Traffic routing is implemented using **kube-proxy** on each node
- kube-proxy programs **iptables or IPVS rules**

These rules ensure that:

- Requests sent to the Service IP
- Are transparently forwarded to healthy Pods

Applications never see this complexity.

---

## Service Discovery Using DNS

Kubernetes provides built-in **service discovery** using DNS.

When you create a Service:

- A DNS entry is automatically created
- The DNS name follows this pattern:

```
<service-name>.<namespace>.svc.cluster.local
```

Applications only need to know the **Service name**.

They do not need:

- Pod IP addresses
- Node IP addresses
- Load balancing logic

DNS resolves the Service name to the Service IP, and Kubernetes handles the rest.

---

## Namespaces and Isolation

As clusters grow, managing everything in one place becomes difficult.

**Namespaces** provide logical isolation:

- Group related applications together
- Avoid name collisions
- Improve clarity and management

Namespaces do **not block communication by default** — they organize resources.

Namespaces also become part of the Service DNS name, making service discovery predictable.

---

## ClusterIP: Internal-Only Services

By default, Services are created as **ClusterIP**.

This means:

- The Service is reachable only inside the cluster
- It cannot be accessed directly from the internet

ClusterIP is ideal for:

- Frontend → Backend communication
- Backend → Database communication
- Internal APIs

Most Services in Kubernetes are ClusterIP.

---

## Reaching the Cluster from Outside

Eventually, external users need access.

Kubernetes provides multiple options:

- **NodePort** – Exposes a port on every node (simple but limited)
- **LoadBalancer** – Uses cloud load balancers when available
- **Ingress** – Provides HTTP routing and advanced control

In most production systems:

- LoadBalancer or Ingress is preferred
- NodePort is rarely used directly

These options build on top of Services — not instead of them.

---

## How Networking Actually Works Under the Hood

Kubernetes itself does **not implement networking**.

Instead, it relies on **CNI (Container Network Interface) plugins**.

CNI plugins are responsible for:

- Assigning Pod IP addresses
- Connecting Pods to the node network
- Enabling cross-node communication

Different CNI plugins use different techniques:

- Virtual Ethernet pairs
- Linux bridges
- IP-in-IP tunnels
- VXLAN overlays

All of this complexity is hidden behind the Kubernetes networking model.

---

## The Big Idea

Kubernetes networking follows one core principle:

> **Make distributed systems feel simple to applications.**

Applications:

- Talk using DNS names
- Assume stable endpoints
- Do not care where Pods run

Kubernetes and CNI handle:

- IP management
- Routing
- Load balancing
- Failure handling

---

## Final Thoughts

Services are one of the most important Kubernetes concepts.

They turn:

- Unstable Pods
- Dynamic IPs
- Distributed systems

Into something **predictable and reliable**.

Once you understand why Services exist, Kubernetes networking becomes much easier to reason about.

In the next article, we will go deeper into **Ingress and external traffic flow**, and understand how requests travel from the internet to a Pod.
