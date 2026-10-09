# Kubernetes Namespaces

## Introduction
A Namespace in Kubernetes is used to organize and separate resources within a single Kubernetes cluster. It helps manage resources for different environments, such as development, testing, and production.

## Common Namespace Commands

```bash
# List namespaces
kubectl get ns

# Create a namespace
kubectl create namespace dev

# View namespace details
kubectl describe ns dev

# Create a pod inside a namespace
kubectl run myapp --image=nginx -n dev

# List pods in a namespace
kubectl get pods -n dev

# List pods in all namespaces
kubectl get pods -A

# Delete a pod
kubectl delete pod myapp -n dev

# Delete a namespace
kubectl delete namespace dev
```

## Advantages
- Organizes Kubernetes resources.
- Separates development, testing, and production environments.
- Helps manage access permissions and resource quotas.

## Reference
[Kubernetes Namespaces Documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)