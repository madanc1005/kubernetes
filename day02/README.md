# Day 02 - Kubernetes Pods & kubectl Commands

Today I learned the basics of creating, managing, and inspecting Kubernetes Pods using `kubectl`.

## 1. Create a Pod

```bash
kubectl run nginx-pod --image=nginx
```

Creates a Pod named `nginx-pod` using the `nginx` Docker image.

---

## 2. Apply a YAML File

```bash
kubectl apply -f demo.yaml
```

Creates the Kubernetes resources defined in the YAML file.

I used `demo.yaml` to create an NGINX Pod.

---

## 3. List Pods

```bash
kubectl get pod
```

Displays the Pods running in the current Kubernetes namespace.

Example:

```text
NAME         READY   STATUS    RESTARTS   AGE
nginx-pod    1/1     Running   0          10s
```

---

## 4. Get More Information About Pods

```bash
kubectl get pod -o wide
```

Displays additional information about Pods, including:

* Pod IP address
* Node where the Pod is running
* Pod status
* Number of restarts

---

## 5. Delete a Pod

```bash
kubectl delete pod nginx-pod
```

Deletes the `nginx-pod` from the Kubernetes cluster.

---

## 6. Dry Run

```bash
kubectl run nginx-pod --image=nginx --dry-run=client
```

Performs a client-side dry run.

It shows what Kubernetes would create without actually creating the Pod.

---

## 7. Generate YAML Using kubectl

```bash
kubectl run nginx-pod --image=nginx --dry-run=client -o yaml
```

Generates the YAML configuration for the Pod without creating it.

This is useful for quickly creating a Kubernetes YAML template.

---

## 8. Pod YAML

The `demo.yaml` file contains the following Pod configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    env: demo
    type: prod
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```

### Important Parts

| Field           | Description                                 |
| --------------- | ------------------------------------------- |
| `apiVersion`    | Specifies the Kubernetes API version        |
| `kind`          | Defines the type of Kubernetes resource     |
| `metadata`      | Contains the Pod name and labels            |
| `spec`          | Defines the desired configuration           |
| `containers`    | Defines the containers inside the Pod       |
| `image`         | Specifies the container image               |
| `containerPort` | Specifies the port exposed by the container |

---

## Commands Summary

| Command                                                        | Purpose                           |
| -------------------------------------------------------------- | --------------------------------- |
| `kubectl run nginx-pod --image=nginx`                          | Create a Pod                      |
| `kubectl apply -f demo.yaml`                                   | Create/update resources from YAML |
| `kubectl get pod`                                              | List Pods                         |
| `kubectl get pod -o wide`                                      | Get detailed Pod information      |
| `kubectl delete pod nginx-pod`                                 | Delete a Pod                      |
| `kubectl run nginx-pod --image=nginx --dry-run=client`         | Test without creating             |
| `kubectl run nginx-pod --image=nginx --dry-run=client -o yaml` | Generate YAML                     |

## What I Learned

* How to create a Kubernetes Pod using `kubectl run`
* How to create a Pod using a YAML manifest
* How to apply Kubernetes YAML files
* How to list and inspect Pods
* How to delete Pods
* How to use `--dry-run=client`
* How to generate YAML using `-o yaml`
* Basic structure of a Kubernetes Pod manifest
* The purpose of `apiVersion`, `kind`, `metadata`, and `spec`

## Practice

```bash
kubectl apply -f demo.yaml
kubectl get pod
kubectl get pod -o wide
kubectl delete pod nginx-pod
```

---

**Day 02 completed — Kubernetes Pods and basic `kubectl` commands.**
