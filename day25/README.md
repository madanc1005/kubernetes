# Kubernetes Pod Priority and Scheduling

## Overview

This project demonstrates how to configure **Pod priority and scheduling in Kubernetes** using `PriorityClass`.

It defines two priority classes:

- **high-priority** — priority value `15000`
- **low-priority** — priority value `100`, configured as the global default

The project includes one high-priority NGINX Pod and five low-priority NGINX Pods. Kubernetes uses Pod priority when scheduling Pods and, when necessary, deciding which lower-priority Pods may be preempted to make room for higher-priority Pods. Priority does not guarantee that a Pod will run if the cluster lacks the required resources or the Pod cannot be scheduled for other reasons.

## Repository structure

```text
.
├── deploy.yaml       # High-priority NGINX Pod
├── deploy1.yaml      # Low-priority NGINX Pod (pod1)
├── deploy2.yaml      # Low-priority NGINX Pod (pod2)
├── deploy3.yaml      # Low-priority NGINX Pod (pod3)
├── deploy4.yaml      # Low-priority NGINX Pod (pod4)
├── deploy5.yaml      # Low-priority NGINX Pod (pod5)
├── pc-high.yaml      # PriorityClass: high-priority
├── pc-low.yaml       # PriorityClass: low-priority
└── README.md
```

## Configuration details

| Resource | Name | Priority / configuration |
|---|---|---|
| PriorityClass | `high-priority` | Value: `15000`; `globalDefault: false` |
| PriorityClass | `low-priority` | Value: `100`; `globalDefault: true` |
| Pod | `high-priority-pod` | Uses `high-priority` |
| Pod | `pod1` | Uses `low-priority` |
| Pod | `pod2` | Uses `low-priority` |
| Pod | `pod3` | Uses `low-priority` |
| Pod | `pod4` | Uses `low-priority` |
| Pod | `pod5` | Uses `low-priority` |

Each Pod uses the `nginx` container image and requests `120Mi` of memory. The high-priority Pod requests `cpu: "12"`; the other five Pods each request `cpu: "1"`.

> **Resource note:** Kubernetes interprets `cpu: "12"` as 12 CPU cores, not 12 millicores. Confirm that this request is intended and that your cluster has enough available CPU. If you meant 12 millicores, use `cpu: "12m"` instead.

## Prerequisites

- A running Kubernetes cluster (for example, Minikube, kind, or a managed Kubernetes cluster).
- `kubectl` installed and configured to access the cluster.
- Sufficient cluster resources for the Pods' declared CPU and memory requests.

Check your cluster connection:

```bash
kubectl cluster-info
kubectl get nodes
```

## Deploy the resources

Apply the PriorityClasses first, then create the Pods:

```bash
kubectl apply -f pc-high.yaml
kubectl apply -f pc-low.yaml

kubectl apply -f deploy.yaml
kubectl apply -f deploy1.yaml
kubectl apply -f deploy2.yaml
kubectl apply -f deploy3.yaml
kubectl apply -f deploy4.yaml
kubectl apply -f deploy5.yaml
```

You can also apply all manifests in the current directory:

```bash
kubectl apply -f .
```

Applying all manifests from the directory is convenient, but applying the PriorityClasses first makes the dependency order explicit.

## Verify the deployment

List the Pods and their status:

```bash
kubectl get pods
kubectl get pods -o wide
```

Inspect the PriorityClasses:

```bash
kubectl get priorityclass
kubectl describe priorityclass high-priority
kubectl describe priorityclass low-priority
```

Inspect the priority and scheduling details of the high-priority Pod:

```bash
kubectl describe pod high-priority-pod
kubectl get pod high-priority-pod -o jsonpath='{.spec.priorityClassName}{"\n"}{.spec.priority}{"\n"}'
```

Check scheduling events if a Pod remains pending:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe pod high-priority-pod
```

## Understanding the result

- The `high-priority-pod` references the `high-priority` PriorityClass, whose value is `15000`.
- Pods `pod1` through `pod5` reference the `low-priority` PriorityClass, whose value is `100`.
- The `low-priority` class is marked `globalDefault: true`. This applies as the default priority class to Pods that do not specify a `priorityClassName`; the five Pods in this project explicitly specify `low-priority`.
- Higher-priority Pods are considered ahead of lower-priority Pods during scheduling. Preemption may be used when needed, but it is not guaranteed to resolve every scheduling constraint.

## Cleanup

Remove the Pods and PriorityClasses when you no longer need the demonstration:

```bash
kubectl delete -f deploy.yaml
kubectl delete -f deploy1.yaml
kubectl delete -f deploy2.yaml
kubectl delete -f deploy3.yaml
kubectl delete -f deploy4.yaml
kubectl delete -f deploy5.yaml

kubectl delete -f pc-high.yaml
kubectl delete -f pc-low.yaml
```

Alternatively, from the project directory:

```bash
kubectl delete -f .
```

## Push the project to GitHub

If this directory is already a Git repository, save `README.md` alongside the YAML files and run:

```bash
git status
git add README.md *.yaml
git commit -m "Add Kubernetes pod priority scheduling example"
git push
```

If Git has not been initialized in this directory yet, initialize it and add your GitHub repository as the remote:

```bash
git init
git add README.md *.yaml
git commit -m "Add Kubernetes pod priority scheduling example"
git branch -M main
git remote add origin https://github.com/<YOUR-USERNAME>/<YOUR-REPOSITORY>.git
git push -u origin main
```

Replace the placeholder repository URL with your actual GitHub repository URL.

## License

Add a license file if you intend to distribute or reuse this project publicly.
