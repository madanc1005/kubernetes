# Kubernetes Services

Kubernetes Service provides a stable way to access Pods.

## Types of Services

### 1. ClusterIP

Used for communication **inside the Kubernetes cluster**.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cluster-ip-service
spec:
  type: ClusterIP
  selector:
    env: dep-pod
  ports:
    - port: 80
      targetPort: 80
```

**Flow:**

```text
Service → Pods
```

---

### 2. NodePort

Used to access the application from **outside the cluster** using the Node IP and NodePort.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    env: dep-pod
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

**Flow:**

```text
Node-IP:30080 → Service → Pods
```

---

### 3. LoadBalancer

Used to expose the application through an **external Load Balancer**, mainly in cloud environments.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    env: dep-pod
  ports:
    - port: 80
      targetPort: 80
```

**Flow:**

```text
Internet → LoadBalancer → Service → Pods
```

---

### 4. ExternalName

Used when an application inside Kubernetes needs to access a service **outside the cluster**.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-database
spec:
  type: ExternalName
  externalName: database.example.com
```

**Flow:**

```text
Pod → my-database → database.example.com
```

---

## Important Commands

```bash
kubectl get pods
kubectl get services
kubectl get endpoints
kubectl describe service <service-name>
kubectl get nodes -o wide
```

## Service Selector

The Service uses `selector` to find matching Pods.

Service:

```yaml
selector:
  env: dep-pod
```

Pod:

```yaml
labels:
  env: dep-pod
```

The labels must match.

## Service Types Summary

| Type | Purpose |
|---|---|
| ClusterIP | Internal access |
| NodePort | Access using Node IP + port |
| LoadBalancer | External Load Balancer |
| ExternalName | Access external service |