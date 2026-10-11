# Kubernetes Taints, Tolerations, and Node Selector

## 1. What is a Taint?

A **Taint** is applied to a Kubernetes Node to prevent Pods from being scheduled on that Node unless they have a matching toleration.

**Syntax:**

```bash
kubectl taint node <node-name> <key>=<value>:<effect>
```

## 2. What is a Toleration?

A **Toleration** is added to a Pod to allow it to be scheduled on a Node that has a matching taint.

**Note:** A toleration allows a Pod to be scheduled on a tainted Node, but it does not guarantee that the Pod will run on that particular Node.

## 3. Practical: Adding Taints to Two Worker Nodes

In this practical, we have two worker nodes:

- `desktop-worker`
- `desktop-worker2`

Apply taints to both worker nodes.

### Add a Taint to the First Worker Node

```powershell
kubectl taint node desktop-worker gpu=true:NoSchedule
```

**Output:**

```text
node/desktop-worker tainted
```

### Add a Taint to the Second Worker Node

```powershell
kubectl taint node desktop-worker2 gpu=false:NoSchedule
```

**Output:**

```text
node/desktop-worker2 tainted
```

Now, both worker nodes have different taints.

| Node | Taint |
|---|---|
| `desktop-worker` | `gpu=true:NoSchedule` |
| `desktop-worker2` | `gpu=false:NoSchedule` |

The `NoSchedule` effect prevents new Pods without matching tolerations from being scheduled on these nodes.

## 4. Creating a Pod Without a Toleration

Create a file named `demo.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
```

Apply the YAML file:

```powershell
kubectl apply -f .\demo.yaml
```

**Output:**

```text
pod/nginx-pod created
```

Check the Pod status:

```powershell
kubectl get pods
```

**Output:**

```text
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   0/1     Pending   0          2m23s
```

### Why is the Pod Pending?

The Pod does not contain any tolerations for the taints applied to the worker nodes.

As a result, Kubernetes cannot schedule this Pod on those tainted nodes.

To verify the reason, run:

```powershell
kubectl describe pod nginx-pod
```

Look at the Events section.

**Example output:**

```text
Warning  FailedScheduling  default-scheduler
0/3 nodes are available: 3 node(s) had untolerated taint(s).
```

This indicates that none of the available nodes can accept the Pod because of their taints.

## 5. Adding a Toleration for the First Worker Node

To allow a Pod to be scheduled on `desktop-worker`, add a toleration matching its taint:

```text
gpu=true:NoSchedule
```

Create a file named `demo1.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod1
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
```

Apply the YAML file:

```powershell
kubectl apply -f .\demo1.yaml
```

Check the Pods:

```powershell
kubectl get pods
```

**Example output:**

```text
NAME         READY   STATUS    RESTARTS   AGE
nginx-pod    0/1     Pending   0          15s
nginx-pod1   1/1     Running   0          9s
```

### Why is `nginx-pod1` Running?

The Pod has the following toleration:

```yaml
tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

This toleration matches the taint on `desktop-worker`:

```text
gpu=true:NoSchedule
```

Therefore, Kubernetes is allowed to schedule `nginx-pod1` on that node.

Verify which node is running the Pod:

```powershell
kubectl get pods -o wide
```

**Example output:**

```text
NAME         READY   STATUS    IP            NODE
nginx-pod    0/1     Pending   <none>        <none>
nginx-pod1   1/1     Running   10.244.2.10   desktop-worker
```

The Pod is running on `desktop-worker`.

**Important:** A toleration does not select a specific Node. Kubernetes can schedule the Pod on any suitable node whose taints it tolerates. In this example, `nginx-pod1` was scheduled on `desktop-worker`.

## 6. Adding a Toleration for the Second Worker Node

Now, we want to allow a Pod to run on `desktop-worker2`.

First, check the taint on the second worker node.

```powershell
kubectl describe node desktop-worker2 | Select-String -Pattern "Taints" -Context 0,2
```

On Linux, the equivalent command is:

```bash
kubectl describe node desktop-worker2 | grep -iA 2 taints
```

**Example output:**

```text
Taints:             gpu=false:NoSchedule
Unschedulable:      false
```

The taint is:

```text
gpu=false:NoSchedule
```

We must use a toleration with the matching key, value, and effect.

Create a file named `demo2.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod2
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "false"
      effect: "NoSchedule"
```

Apply the YAML file:

```powershell
kubectl apply -f .\demo2.yaml
```

Check the Pod status:

```powershell
kubectl get pods -o wide
```

The Pod is now allowed to run on `desktop-worker2`, provided that node is otherwise suitable for scheduling.

**Important:** The toleration alone does not guarantee placement on `desktop-worker2`. To select that exact node, use a `nodeSelector` as explained below.

## 7. Understanding Node Selector

A **Node Selector** is used to schedule a Pod onto a Node that has a specific label.

Unlike a toleration, which allows a Pod to tolerate a matching taint, a node selector identifies the node on which the Pod must be scheduled.

### Step 1: Add Labels to the Worker Nodes

Label the first worker node:

```powershell
kubectl label node desktop-worker gpu=true
```

Label the second worker node:

```powershell
kubectl label node desktop-worker2 gpu=false
```

If a label with the same key already exists and has a different value, use `--overwrite` to update it.

```powershell
kubectl label node desktop-worker2 gpu=false --overwrite
```

Verify the labels:

```powershell
kubectl get nodes --show-labels
```

The nodes should include these labels:

| Node | Label |
|---|---|
| `desktop-worker` | `gpu=true` |
| `desktop-worker2` | `gpu=false` |

### Step 2: Use Node Selector in the Pod YAML

To schedule a Pod specifically on `desktop-worker2`, use the following YAML.

Create a file named `demo3.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-selector-pod
  labels:
    app: nginx
spec:
  nodeSelector:
    gpu: "false"
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "false"
      effect: "NoSchedule"
```

Apply the YAML file:

```powershell
kubectl apply -f .\demo3.yaml
```

Verify the Pod:

```powershell
kubectl get pods -o wide
```

The Pod should run on `desktop-worker2` if the node is healthy and has sufficient resources.

### How Does It Work?

The following configuration:

```yaml
nodeSelector:
  gpu: "false"
```

tells Kubernetes to select a Node with the label:

```text
gpu=false
```

The toleration:

```yaml
tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "false"
    effect: "NoSchedule"
```

allows the Pod to be scheduled on the node despite its matching taint.

**Both configurations are needed in this example:**

- `nodeSelector` selects a node with the required label.
- `tolerations` allows the Pod to tolerate the node's taint.

## 8. Difference Between Taints, Tolerations, and Node Selector

| Feature | Purpose |
|---|---|
| Taint | Prevents Pods without matching tolerations from being scheduled on a node. |
| Toleration | Allows a Pod to be scheduled on a node with a matching taint. |
| Node Selector | Restricts a Pod to nodes with the specified labels. |

### Easy Way to Remember

- **Taint:** Keep unwanted Pods away from a Node.
- **Toleration:** Allow a Pod to use a tainted Node.
- **Node Selector:** Choose a Node based on its labels.

## 9. Useful Commands

Check all nodes:

```powershell
kubectl get nodes
```

Check node labels:

```powershell
kubectl get nodes --show-labels
```

Describe a node and check its taints:

```powershell
kubectl describe node desktop-worker
kubectl describe node desktop-worker2
```

Check Pods and their assigned nodes:

```powershell
kubectl get pods -o wide
```

Check Pod scheduling events:

```powershell
kubectl describe pod nginx-pod
```

Delete a Pod:

```powershell
kubectl delete pod nginx-pod
```

Remove the taint from the first worker node:

```powershell
kubectl taint node desktop-worker gpu=true:NoSchedule-
```

Remove the taint from the second worker node:

```powershell
kubectl taint node desktop-worker2 gpu=false:NoSchedule-
```

Remove a label from a node:

```powershell
kubectl label node desktop-worker gpu-
```

## 10. Conclusion

In this practical, we learned how Kubernetes Taints, Tolerations, and Node Selectors control Pod scheduling.

1. We applied different taints to two worker nodes.
2. We created a Pod without a toleration and observed that it remained Pending.
3. We added a matching toleration and allowed a Pod to run on a suitable node.
4. We learned that tolerations allow scheduling but do not select a particular node.
5. We applied labels and used a node selector to target a node with a specific label.
6. We combined a node selector and a toleration to schedule a Pod onto the intended tainted worker node.

These concepts help administrators control where applications run in a Kubernetes cluster.