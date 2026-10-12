# Kubernetes Health Probes

## Overview

Health probes in Kubernetes are used to check the health and availability of applications running inside containers.

Kubernetes provides three types of health probes:

1. **Startup Probe** – Checks whether the application has started successfully.
2. **Liveness Probe** – Checks whether the application is still running properly. If the probe fails repeatedly, Kubernetes restarts the container.
3. **Readiness Probe** – Checks whether the application is ready to receive traffic.

## Objectives

- Understand Kubernetes health probes.
- Implement Startup, Liveness, and Readiness probes.
- Monitor application health using Kubernetes.
- Learn how Kubernetes handles unhealthy and unready applications.

## Prerequisites

- A Kubernetes cluster (Minikube, Kind, or another cluster).
- `kubectl` installed and configured.
- Basic knowledge of Kubernetes Pods and YAML files.

## Project Structure

```text
kubernetes-health-probes/
├── README.md
└── health-probe.yaml
```

## Kubernetes Configuration

Create a file named `health-probe.yaml` with the following configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: health-probe-demo
spec:
  containers:
    - name: app
      image: nginx:latest
      ports:
        - containerPort: 80

      startupProbe:
        httpGet:
          path: /
          port: 80
        failureThreshold: 30
        periodSeconds: 5

      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 10

      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 5
```

## Explanation of Health Probes

### 1. Startup Probe

The Startup Probe checks whether the application has completed its startup process.

Configuration:

```yaml
startupProbe:
  httpGet:
    path: /
    port: 80
  failureThreshold: 30
  periodSeconds: 5
```

- `httpGet`: Sends an HTTP request to the application.
- `path: /`: Checks the root URL.
- `port: 80`: Sends the request to port 80.
- `periodSeconds: 5`: Performs a check every 5 seconds.
- `failureThreshold: 30`: Allows up to 30 consecutive failures before Kubernetes considers startup unsuccessful and restarts the container.

The configured startup-probe failure allowance is approximately 150 seconds, though actual timing can vary.

### 2. Liveness Probe

The Liveness Probe checks whether the application is still functioning.

Configuration:

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 10
  periodSeconds: 10
```

- `initialDelaySeconds: 10`: Sets an initial delay of 10 seconds before liveness checks begin, provided the startup probe has already succeeded.
- `periodSeconds: 10`: Checks the application every 10 seconds.
- If the probe fails repeatedly, Kubernetes restarts the container.

### 3. Readiness Probe

The Readiness Probe checks whether the application is ready to handle requests.

Configuration:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

- `initialDelaySeconds: 5`: Sets an initial delay of 5 seconds, subject to the startup probe succeeding first.
- `periodSeconds: 5`: Checks readiness every 5 seconds.
- If the probe fails, the Pod is removed from normal Service traffic endpoints until it becomes ready again.
- A readiness failure does not, by itself, restart the container.

## Deployment Steps

### Step 1: Check the Kubernetes Cluster

```bash
kubectl get nodes
```

Verify that your Kubernetes node is in the `Ready` state.

### Step 2: Apply the YAML Configuration

```bash
kubectl apply -f health-probe.yaml
```

This creates the Pod with all three health probes configured.

### Step 3: Check the Pod Status

```bash
kubectl get pods
```

Expected output:

```text
NAME                READY   STATUS    RESTARTS
health-probe-demo   1/1     Running   0
```

The output may differ while the container is starting or if a probe fails.

### Step 4: Inspect the Pod

```bash
kubectl describe pod health-probe-demo
```

Check the `Containers` and `Events` sections for probe configuration, readiness information, and errors.

### Step 5: View the Pod YAML

```bash
kubectl get pod health-probe-demo -o yaml
```

This displays the Pod configuration, including its health probes and current status.

### Step 6: Test the Nginx Application

```bash
kubectl port-forward pod/health-probe-demo 8080:80
```

Open the following URL in your browser:

```text
http://localhost:8080
```

You should see the Nginx welcome page if the application is running correctly.

Press `Ctrl+C` to stop port forwarding.

## How It Works

1. Kubernetes starts the Nginx container.
2. The Startup Probe checks whether Nginx responds successfully.
3. After the Startup Probe succeeds, the Liveness and Readiness Probes begin.
4. The Liveness Probe detects an application that stops responding and can trigger a container restart.
5. The Readiness Probe determines whether the Pod should receive normal Service traffic.

**Important:** This example uses the Nginx root URL (`/`) for all three probes. It verifies that Nginx responds over HTTP, but it does not guarantee that every application dependency is healthy.

## Cleanup

Delete the Pod after completing the practice:

```bash
kubectl delete -f health-probe.yaml
```

## Key Takeaways

| Probe | Purpose | Action on repeated failure |
|---|---|---|
| Startup Probe | Checks application startup | Container is restarted |
| Liveness Probe | Checks whether the application is functioning | Container is restarted |
| Readiness Probe | Checks whether the application can receive traffic | Pod is removed from normal Service endpoints |

## Conclusion

Kubernetes health probes help maintain application reliability by checking startup, liveness, and readiness. They allow Kubernetes to restart containers that fail health checks and prevent traffic from being sent to Pods that are not ready.

This project demonstrates how to configure and inspect all three probes in a Kubernetes Pod using an Nginx container.



