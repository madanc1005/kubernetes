## Kubernetes – ReplicationController

A **ReplicationController (RC)** ensures that the desired number of Pods are always running.

### Types of Kubernetes Workload Resources

* ReplicationController
* ReplicaSet
* Deployment

### Important RC Commands

```bash
# Create/Apply ReplicationController
kubectl apply -f ReplicationController.yaml

# Check ReplicationControllers
kubectl get rc

# Check Pods
kubectl get pods

# Get detailed information about RC
kubectl describe rc nginx-rc

# Scale Pods up
kubectl scale rc nginx-rc --replicas=5

# Scale Pods down
kubectl scale rc nginx-rc --replicas=2

# Delete ReplicationController
kubectl delete rc nginx-rc
```

### What I Learned

* How to create a ReplicationController
* How to check RC and Pod status
* How to describe an RC
* How to scale Pods up and down
* How to delete an RC


