---
title: "Why Kubernetes Needs Ingress"
meta_title: ""
description: "Understanding k8s"
date: 2025-04-04T05:00:00Z
image: "/images/image-placeholder.png"
categories: ["Kubernetes", "Docker"]
author: "Deepak Bansode"
tags: ["Docker", "Kubernetes"]
draft: true
---

_This article is the next part in the Kubernetes series, continuing from Services and networking._

In the previous article, we understood **why Kubernetes needs Services** and how Services provide stable networking inside the cluster. At that point, applications can talk to each other reliably **within Kubernetes**.

But one important question still remains:

> How does traffic from the outside world reach an application running inside Kubernetes?

This article explains **why Kubernetes needs Ingress**, what problems it solves, and how it fits into the overall request flow — from the internet to a Pod.

---

## The Problem: Services Are Not Enough for External Traffic

A Service gives us:

- Stable IP address
- Stable DNS name
- Load balancing across Pods

But Services are mainly designed for **internal communication**.

If we want users on the internet to access our application, we still need answers to questions like:

- Which URL should users hit?
- How do we handle HTTP paths like `/login` or `/api`?
- How do we terminate TLS (HTTPS)?
- How do we route traffic to different services?

Trying to solve these problems using only Services quickly becomes messy.

---

## The Naive Approach: Exposing Every Service

One simple approach is:

- Create a `NodePort` or `LoadBalancer` Service for every application

This works for small setups, but it has serious drawbacks:

- Every service gets its own external IP
- Cloud load balancers are expensive
- No easy way to do path-based or host-based routing
- TLS configuration becomes duplicated and hard to manage

As the number of services grows, this approach does not scale.

---

## Ingress: A Smart Entry Point

Ingress was introduced to solve this exact problem.

An **Ingress** provides:

- A single entry point into the cluster
- HTTP and HTTPS routing
- Path-based routing (`/api`, `/admin`, `/shop`)
- Host-based routing (`api.example.com`, `app.example.com`)
- Centralized TLS configuration

Instead of exposing every Service, you expose **one Ingress**.

---

## Ingress Is a Rule, Not a Server

This is a very important concept.

An **Ingress object**:

- Does not handle traffic by itself
- Is just a set of routing rules

Something must **implement** those rules.

That component is called an **Ingress Controller**.

---

## Ingress Controller: The Actual Worker

An **Ingress Controller** is a real running component (usually a Pod) that:

- Watches Ingress objects
- Reads routing rules
- Configures an underlying proxy or load balancer

Common Ingress Controllers include:

- NGINX Ingress Controller
- HAProxy Ingress
- Traefik
- Cloud-native controllers (AWS ALB, GCP Ingress)

Ingress defines _what_ should happen. The controller decides _how_ it happens.

---

## Request Flow: From Internet to Pod

A typical request flow looks like this:

```
User → DNS → Load Balancer → Ingress Controller → Service → Pod
```

Step by step:

1. User hits `https://app.example.com`
2. DNS points to a cloud load balancer or node IP
3. Traffic reaches the Ingress Controller
4. Ingress rules decide where the request should go
5. The request is forwarded to the correct Service
6. The Service load-balances traffic to a Pod

Each layer has a single responsibility.

---

## Why Ingress Is HTTP-Focused

Ingress is designed mainly for:

- HTTP
- HTTPS
- Web applications
- APIs

This allows Ingress to understand:

- URLs
- Headers
- Cookies
- TLS certificates

For non-HTTP traffic, other mechanisms like `LoadBalancer` Services are still used.

---

## Centralized TLS and Security

Without Ingress:

- Every service manages its own TLS
- Certificates are duplicated
- Security policies are inconsistent

With Ingress:

- TLS is terminated at one place
- Certificates are managed centrally
- Security rules are applied consistently

This simplifies both **operations** and **security**.

---

## Ingress vs Service: Clear Separation of Roles

Understanding this difference is key:

- **Service** → Stable access _inside_ the cluster
- **Ingress** → Controlled access _into_ the cluster

Ingress always sits **in front of Services**, never behind them.

---

## Why Kubernetes Did Not Build This Into Services

Kubernetes follows a strong design principle:

> Keep primitives simple and composable.

If Services tried to handle:

- Internal discovery
- External routing
- TLS
- HTTP rules

They would become complex and hard to evolve.

Ingress keeps concerns separated:

- Services stay simple
- Ingress focuses on edge traffic

---

## The Big Picture

Ingress completes the networking story in Kubernetes:

- Pods run applications
- Services provide stable internal access
- Ingress provides controlled external access

Together, they form a clean and scalable model.

---

## Final Thoughts

Ingress exists because exposing applications safely and cleanly is hard.

Ingress:

- Reduces operational complexity
- Centralizes routing and security
- Scales well as services grow

Once you understand **why Ingress exists**, the full Kubernetes request flow — from the internet to a Pod — becomes clear.

In the next article, we can explore **Ingress Controllers in detail**, or move into **ConfigMaps and Secrets** to understand configuration management.
