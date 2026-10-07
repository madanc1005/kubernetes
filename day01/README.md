# Kubernetes Architecture

Kubernetes is a container orchestration platform used to deploy, manage, and scale containerized applications.

## Basic Kubernetes Architecture

The main components I have learned are:

* **kubectl**
* **API Server**
* **etcd**
* **Controller Manager**
* **Scheduler**
* **Kubelet**
* **Container Runtime**
* **Pods**

## Basic Request Flow

When we run:

```bash
kubectl create deployment nginx --image=nginx --replicas=3
```

We are telling Kubernetes that we want **3 replicas of the Nginx application**.

The basic flow is:

```text
kubectl
   ↓
API Server
   ↓
etcd
   ↓
Controller Manager
   ↓
Scheduler
   ↓
Kubelet
   ↓
Container Runtime
   ↓
Pods
```

## Kubernetes Components

### kubectl

`kubectl` is the command-line tool used to communicate with the Kubernetes cluster.

Example:

```bash
kubectl create deployment nginx --image=nginx --replicas=3
```

### API Server

The API Server is the main entry point for communication with the Kubernetes cluster.

It receives and validates requests from `kubectl` and other Kubernetes components.

### etcd

`etcd` is the key-value store used by Kubernetes to store cluster data and the desired state of resources.

### Controller Manager

The Controller Manager continuously monitors the cluster and works to make the actual state match the desired state.

For example:

```text
Desired State → 3 nginx Pods
Actual State  → 2 nginx Pods
```

The controller detects the difference and works to create the missing Pod.

### Scheduler

The Scheduler decides which worker node should run a newly created Pod based on available resources and scheduling requirements.

### Kubelet

Kubelet runs on worker nodes and is responsible for making sure the assigned Pods and containers are running correctly.

### Container Runtime

The Container Runtime is responsible for running containers.

Examples:

* containerd
* CRI-O

### Pod

A Pod is the smallest deployable unit in Kubernetes.

For example:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Nginx Container
```

## Desired State

One of the important concepts in Kubernetes is **desired state**.

For example:

```bash
kubectl create deployment nginx --image=nginx --replicas=3
```

The desired state is:

```text
3 nginx Pods should be running
```

Kubernetes continuously monitors the cluster and works to maintain this desired state.

If one Pod fails:

```text
Desired State → 3 Pods
Actual State  → 2 Pods
```

Kubernetes automatically works to create a replacement Pod.

## Commands Practiced

### Create a Deployment

```bash
kubectl create deployment nginx --image=nginx --replicas=3
```

### View Deployments

```bash
kubectl get deployments
```

### View Pods

```bash
kubectl get pods
```

### View detailed Pod information

```bash
kubectl get pods -o wide
```

### Describe a Deployment

```bash
kubectl describe deployment nginx
```

## Key Concepts

* Kubernetes follows a **desired state** model.
* The **API Server** is the central communication point.
* **etcd** stores Kubernetes cluster state.
* **Controllers** work to maintain the desired state.
* The **Scheduler** assigns Pods to nodes.
* **Kubelet** manages Pods on worker nodes.
* The **Container Runtime** runs containers.
* **Pods** run the application containers.
