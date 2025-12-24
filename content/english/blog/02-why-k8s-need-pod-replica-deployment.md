---
title: "Why Kubernetes Needs Deployments, ReplicaSets, and Pods"
meta_title: ""
description: "Understanding k8s"
date: 2025-04-04T05:00:00Z
image: "/images/image-placeholder.png"
categories: ["Kubernetes", "Docker"]
author: "Deepak Bansode"
tags: ["Docker", "Kubernetes"]
draft: false
---

_This is the second article in the Kubernetes series, continuing from the earlier posts on Why We Needed Docker and Kubernetes in the First Place._

In the previous article, we understood **why Kubernetes exists** and the orchestration problems it solves. Now that Kubernetes is managing containers at scale, the next natural question is:

> How does Kubernetes actually run applications in a reliable and scalable way?

To answer this, we need to understand a few **core building blocks** of Kubernetes:

- Pods
- ReplicaSets
- Deployments

This article explains **why these abstractions exist**, what problems they solve, and how they fit together.

---

## Requirement

Imagine you have requirement to deploy 3 tier todo application on kubernets cluster. The frontend is built using React, and backend is built using Golang. MySql database is used to store the todos data. The architecute is as follows;

![architecture](/images/blog/pod-replica-deployment/architecture.jpg)

The application team has build the application and docker images. We have followings things;

1. Frontend docker image
2. Backend docker image
3. MySql image
4. NGINX image
5. Kubernets cluster setup

What we want to do?

1. Deploy these image on k8s cluster i.e. create container from those images
2. Setup reverse proxy
3. Setup database
4. Setup networking between components i.e backend can talk to DB
5. Make application accessible over the internet.

Final outcome;
![final-result](/images/blog/pod-replica-deployment/final-result.jpg)

## Deploying Frontend

We have frontend docker image and goal is to deploy on k8s cluster.

![Deploy Frontend ](/images/blog/pod-replica-deployment/frontend-image-to-container.jpg)

As we learned kubernes doesn't manages container. We need **pod** to run the container.

## Pods: The Smallest Unit Kubernetes Understands

A **Pod** is:

- A wrapper around one or more containers
- The smallest unit Kubernetes can create, schedule, and manage
- Assigned a **single IP address**

You can think of a Pod like a very lightweight virtual machine:

- Multiple containers in a Pod share the same network (IP and ports)
- They can communicate using `localhost`
- They share storage if configured

In practice, **most Pods run only one container**. Multiple containers in a Pod are used only when they are tightly coupled and must run together.

---

## Why Kubernetes Doesn’t Manage Containers Directly

Kubernetes is not a container runtime.

- Docker, containerd, and similar tools already know how to create containers
- Kubernetes does not want to re‑solve that problem

Instead:

- Kubernetes delegates container creation to a runtime using **CRI (Container Runtime Interface)**
- Kubernetes focuses only on **orchestration**

This separation allows Kubernetes to:

- Work with different container runtimes
- Stay flexible and future‑proof
- Focus on reliability and scale

That’s why Pods exist as an abstraction above containers.

---

## Creating first pod

Lets tell kuberntes that we want a pod and it should run the container from frontned docker image. We can tell this to kubernetes by either running command `kubctl run frontend --image=frontend` or create yaml file and run command `kubectl apply -f pod.yaml`

```yaml
//pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
    - name: frontend
      image: frontend
      ports:
        - containerPort: 80
```

You use `kubectl` commmand line tool to issue these commands.

### What happens behind the sceen when you run command `kubectl apply -f pod.yaml`

![Pod Creation Flow ](/images/blog/pod-replica-deployment/pod-creation-flow.jpg)

1. Kubectl (command line tool) talks to K8S API server and provides details(pod.yaml) to create pod.
2. API Server create object in etcd databse as follows

```json
{
  "kind": "pod",
  "node": "?",
  "status": "pending"
}
```

3. Scheduler receives the event about the pod. It find the right node to run the pod and update the etcd database.

```json
{
  "kind": "pod",
  "node": "worker-node-1",
  "status": "pending"
}
```

5. Kubelet running on worker-node-1 recieves the event. It creates a pod and the container using CRI(Container runtime interface).
6. Kubelet uses CNI plugin to assign the IP address to pod.
7. Pod is created with container inside it.

```json
{
  "kind": "pod",
  "node": "worker-node-1",
  "status": "running"
}
```

Cluster after pod is created

![Cluster after pod creation ](/images/blog/pod-replica-deployment/cluster-after-pod-creation.jpg)

## Pod Lifecycle and Health

When you ask Kubernetes to create a Pod, it doesn’t appear instantly.

A Pod goes through multiple **phases**:

- `Pending` – Kubernetes is still setting things up
- `Running` – Containers are running
- `Succeeded` – Completed successfully
- `Failed` – Something went wrong
- `Unknown` – Node communication is lost

![Cluster after pod creation ](/images/blog/pod-replica-deployment/pod-lifecycle.jpg)

Similarly, containers inside the Pod also have states like:

- Waiting
- Running
- Terminated

![Cluster after pod creation ](/images/blog/pod-replica-deployment/container-lifecycle.jpg)
Understanding this lifecycle helps explain why Kubernetes needs health checks.

---

## Container Probes

Just because a container is running doesn’t mean the **application inside it is ready**.

For example:

- A web server may take time to connect to a database
- A Java application may still be warming up

Kubernetes solves this using **Probes**. Kubelet perform few checks either by calling HTTP endpoint or GRPC method or running command inside container. These checks are called Probes.

### Liveness Probe

- Checks if the application is still alive
- If it fails, Kubelet kills container. It decides whether to restart based on restartPolicy.

### Readiness Probe

- Checks if the application is ready to accept traffic
- If it fails, traffic is temporarily stopped

### Startup Probe

- Used for slow‑starting applications
- Prevents premature restarts during startup

These probes allow Kubernetes to make **smart decisions** about restarting containers and routing traffic.

---

## Pods Are Disposable, and That’s a Problem

Pods are not designed to live forever.

- A Pod can die if a node crashes
- A Pod can be deleted during updates
- A Pod name and IP can change

If Pods are fragile, then who ensures:

- The correct number of Pods are always running?
- Failed Pods are recreated automatically?

This is where **ReplicaSets** come in.

---

## ReplicaSets: Keeping Pods Alive

You can think ReplicaSet is wrapper on pod. This wrapper makes sure pod is up and running.

![ReplicaSet](/images/blog/pod-replica-deployment/replicaset.jpg)

A **ReplicaSet** is responsible for:

- Ensuring a specific number of identical Pods are running

If you say:

> I want 3 replicas of my application

The ReplicaSet will:

- Create Pods if fewer than 3 exist
- Delete Pods if more than 3 exist
- Recreate Pods if some fail

ReplicaSets continuously **watch the cluster state** and act to maintain the desired number of Pods.

However, ReplicaSets solve only **one problem** availability.

They don’t handle deployments or updates.

---

## Deployments: Managing Change Safely

In real systems, applications evolve.

- New versions are released
- Bugs are fixed
- Configurations change

You don’t want to:

- Kill all Pods at once
- Bring the system down during updates

This is why Kubernetes introduces **Deployments**.

**Deployment**:

You can think of Deployment as wrapper on Replicaset.

![Deployment](/images/blog/pod-replica-deployment/deployment.jpg)

Deployment is reponsible for;

- Manages ReplicaSets
- Enables rolling updates
- Supports rollbacks
- Handles versioning

When you update a Deployment:

- A new ReplicaSet is created
- New Pods are started gradually
- Old Pods are terminated slowly

This ensures **zero‑downtime deployments**.

The hierarchy in Kubernetes looks like this:

```bash
Deployment
  └── ReplicaSet
        └── Pod
              └── Container
```

Each layer has a clear responsibility:

- **Container** → Runs your program
- **Pod** → Groups containers and networking
- **ReplicaSet** → Maintains availability
- **Deployment** → Manages updates and lifecycle

This layered design is intentional and powerful.

---

## Applying the knowledge learned so far

1. Create yaml file for deployment

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  labels:
    app: app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: frontend:1.0
        ports:
        - containerPort: 80

```

2. Run the command `kubectl apply -f deployment.yaml`
3. State after deployment

   ![Deployment Flow](/images/blog/pod-replica-deployment/state-after-deployment.jpg)

## What happens behind the sceen when you run command `kubectl apply -f deployment.yaml`

![Deployment Flow](/images/blog/pod-replica-deployment/deplyment-flow.jpg)

1. Kubectl (command line tool) talks to K8S API server and provides details (deployment.yaml) to create deployment.
2. API Server create object in etcd databse as follows

```json
{
  "kind": "Deployment",
  "replicaSet": "",
  "pods": []
}
```

3. Deployment Controller receives the event about the deployment. It updates the replicaSet information in etcd.

```json
{
  "kind": "Deployment",
  "replicaSet": "app-replica",
  "pods": []
}
```

4. Replication controller recieves the event and update the pod information in etcd.

```json
{
  "kind": "Deployment",
  "replicaSet": "app-replica",
  "pods": [
    {
      "kind": "pod",
      "node": "?",
      "status": "pending"
    },
    {
      "kind": "pod",
      "node": "?",
      "status": "pending"
    }
  ]
}
```

5. Scheduler receives the event. It find the right node to run the pod and update the etcd database.

```json
{
  "kind": "Deployment",
  "replicaSet": "app-replica",
  "pods": [
    {
      "kind": "pod",
      "node": "w-node-1",
      "status": "pending"
    },
    {
      "kind": "pod",
      "node": "w-node-2",
      "status": "pending"
    }
  ]
}
```

6. Kubelet running on respective worker-node-1 and workder-node-2 recieves the event. It creates a pod and the container using CRI(Container runtime interface). Assigns IP to each pod using CNI plugins.

```json
{
  "kind": "Deployment",
  "replicaSet": "app-replica",
  "pods": [
    {
      "kind": "pod",
      "node": "w-node-1",
      "status": "Running"
    },
    {
      "kind": "pod",
      "node": "w-node-2",
      "status": "Running"
    }
  ]
}
```

---

## Lets go ahead and deploy rest other deployment similar way

## ![Final state after deployment](/images/blog/pod-replica-deployment/state-after-deployment.jpg)

---

## Final Thoughts

Kubernetes is built around **simple, composable abstractions**.

Instead of one complex object, Kubernetes uses:

- Small building blocks
- Clear responsibilities
- Continuous reconciliation toward desired state

Once you understand **why Pods, ReplicaSets, and Deployments exist**, Kubernetes stops feeling complicated, and starts feeling logical.

In the next article, we’ll move into **networking** and answer questions like:

- Why Pods cannot talk directly to each other reliably
- Why Services exist
- How traffic is routed inside a cluster
