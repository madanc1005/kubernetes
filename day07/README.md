# Kubernetes DaemonSet

## 1. DaemonSet YAML

In a Kubernetes DaemonSet, containers must be inside `spec.template.spec`, not directly inside `spec`.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset
  labels:
    app: web
    version: v1
spec:
  selector:
    matchLabels:
      app: nginx-daemon
  template:
    metadata:
      labels:
        app: nginx-daemon
    spec:
      containers:
      - name: web
        image: nginx
```

## 2. Understand the YAML hierarchy

- `spec` — DaemonSet configuration.
- `template` — Defines the Pods that Kubernetes creates.
- `template.spec` — Defines how each Pod runs.
- `containers` — Defines the container image, name, and other settings.

## 3. Important fields

### `metadata.name: daemonset`

**Purpose:** Name of the DaemonSet.

```yaml
metadata:
  name: daemonset
```

This is the name you use to identify and manage the DaemonSet.

For example:

```bash
kubectl get daemonset
kubectl describe daemonset daemonset
```

### `spec.selector.matchLabels`

**Purpose:** Identify which Pods belong to the DaemonSet.

```yaml
selector:
  matchLabels:
    app: nginx-daemon
```

Think of this as a rule: “Manage Pods that have the label `app: nginx-daemon`.”

### `spec.template.metadata.labels`

**Purpose:** Give that label to the Pods that will be created.

```yaml
template:
  metadata:
    labels:
      app: nginx-daemon
```

This tells Kubernetes to create Pods with the label `app: nginx-daemon`.

## 4. How the labels work together

- **DaemonSet:** `daemonset` — The resource name.
- **Selector:** `app: nginx-daemon` — Finds the Pods belonging to the DaemonSet.
- **Pod template labels:** `app: nginx-daemon` — Assigns the matching label to each new Pod.

**Labels match!** Kubernetes can associate the Pods with the DaemonSet.

## 5. Apply and check the DaemonSet

Save the YAML in a file such as `demo.yaml`, then run:

```bash
kubectl apply -f demo.yaml
kubectl get daemonset
kubectl get pods -o wide
kubectl describe daemonset daemonset
```
