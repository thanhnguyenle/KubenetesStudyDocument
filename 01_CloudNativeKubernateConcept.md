# What is Cloud Native ?

 **Key:** portable, modular, and isolated across different cloud environments and providers

**Common Confusion:** Cloud-Native vs Cloud-First
- **Cloud-Native:** Architecture-agnostic applications that can run on any cloud platform
- **Cloud-First:** Applications built specifically for a particular CSP's services (though CSPs often market this as "cloud-native")

# Cloud Native Computing Foundation (CNCF)
The Cloud Native Computing Foundation is a **Linux Foundation project** launched in 2015 with the original mission to advance container technology


# Kubernetes (K8s)

## What is Kubernetes?

**Kubernetes** is an open-source **container orchestration system** that automates the deployment, scaling, and management of containerized applications.

- **Created by**: Google (based on their internal systems)
- **Now maintained by**: CNCF (Cloud Native Computing Foundation)
- **Nickname**: **K8s** (the "8" represents the 8 letters between "K" and "s" in "Kubernetes")

## Key Advantage Over Docker

While **Docker** runs containers on a single machine, **Kubernetes** runs containerized applications **distributed across multiple VMs or servers**.

This means:
- **Docker** = Run containers on one machine
- **Kubernetes** = Orchestrate containers across many machines (a cluster)

## Pods - The Unique Component

The fundamental unit in Kubernetes is a **Pod**, not just a container.

**A Pod** is:
- A group of **one or more containers**
- With **shared storage**
- With **shared network resources**
- With other shared settings

Pods allow tightly coupled containers to run together as a single unit while still maintaining the benefits of containerization.

# Kubernetes Components Overview

## Core Organizational Structure

**Cluster**
- The **top-level grouping** of all Kubernetes components
- Contains both control plane and worker nodes

**Namespace**
- **Logical partitions within a cluster**
- Isolates workloads and manages resources (like folders organizing files)

**Node**
- A **virtual or physical machine** hosting workloads
- **Control Plane Node**: Manages the cluster
- **Worker Node**: Runs your applications

![alt text](media/image.png)

Everything flows through the API Server (the central switchboard).

- **CONTROL PLANE**:
  - API Server - Validates all requests
  - etcd - Stores cluster data
  - Scheduler - Picks which node runs each Pod
  - Controller Manager - Self-healing supervisor

- **WORKER NODES**:
  - Kubelet - Starts containers
  - Kube Proxy - Handles networking
  - CRI - Container runtime
  - Pods - Where containers live

**When you deploy**:
kubectl → API Server → etcd → Scheduler picks node → Kubelet starts container

**Key insight**: Control Plane decides, Worker Nodes execute. API Server coordinates everything.


## Application Components

**Pod**
- The **smallest deployable unit** in Kubernetes
- An **abstraction over containers** (wraps one or more containers)
- Containers in a pod share resources and networking

**Service**
- Provides a **stable IP and DNS name** for pods
- Ensures connectivity even when pods restart
- Acts as a **load balancer**

**Ingress**
- Manages **HTTP/S routing** from outside the cluster to services
- Handles TLS termination (SSL certificates)

## Control Plane Components (The Brain)

**API Server**
- **Entry point for all commands** (via kubectl or HTTP API)
- Central communication hub between all components

**Scheduler**
- **Assigns pods to nodes** based on available resources and requirements

**Controller Manager**
- **Watches cluster state** and fixes discrepancies
- Runs various controllers (Node, Replication, Endpoint)

**Cloud Controller Manager**
- **Connects K8s to cloud provider APIs** (AWS, Azure, GCP)

## Node Components (The Workers)

**Kubelet**
- **Agent running on each node**
- Ensures containers run as defined in pod specs
- Communicates with API Server

**Kube Proxy**
- Maintains **network rules** for routing and load balancing

## Configuration & Data Management

**ConfigMap**
- Stores **non-sensitive configuration** (key-value pairs)
- Keeps config separate from container images

**Secret**
- Stores **sensitive data** (passwords, tokens, keys)
- Base64-encoded (encrypt in production!)

**Volumes**
- Provides **persistent storage** for pods
- Can be local or cloud-based

## Workload Management

**Deployment**
- **Blueprint for creating pods**
- Automates rolling updates and rollbacks
- **Most common way to run applications**

**ReplicaSet**
- Ensures a **specified number of pod replicas** are running
- Provides high availability (usually managed by Deployments)

**StatefulSet**
- For **stateful applications** (databases)
- Provides stable IDs and persistent storage

## Security & Networking

**Network Policy**
- **Virtual firewall** for pods/namespaces
- Controls traffic flow

**Kubectl**
- **Command-line tool** for managing the cluster
- Your interface to the API Server

---

# Manifest Files in Kubernetes

## What "Manifest" Means

**General Definition**: A manifest is a document listing contents or items.

**In Kubernetes**: A **Manifest file** is any configuration file that defines how Kubernetes components should be set up and run.

## Types of Manifest Files

All of these are manifest files with specific purposes:

- **Deployment File** - Defines how to deploy and manage your application
- **PodSpec File** - Defines pod configuration
- **Network Policy File** - Defines network security rules
- And many more...

They're all "manifests" because they describe what you want Kubernetes to create and how it should be configured.

## File Formats

Manifest files can be written in:
- **YAML** (most common, human-readable)
- **JSON** (less common, more verbose)

## Multiple Components in One File

A **single manifest file can contain multiple Kubernetes components**.

In YAML, use `---` (three hyphens) to separate different components:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
```

## How to Use Manifest Files

**Deploy with kubectl**:
```bash
kubectl apply -f my-manifest.yaml
```

This command reads the manifest and creates/updates all the components defined in it.