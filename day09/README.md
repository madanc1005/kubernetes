# Kubernetes Node Affinity

## 1. What Is Node Affinity?

**Node Affinity** is a Kubernetes scheduling feature that allows us to control which nodes a Pod can run on based on the labels assigned to those nodes.

For example, if an application requires a node with an SSD disk or a GPU, we can use Node Affinity to schedule the Pod on a node that has the required label.

Without Node Affinity, Kubernetes may schedule the Pod on any suitable node in the cluster.

### Example

Suppose we have two worker nodes:

- `desktop-worker`
- `desktop-worker2`

We want to run an Nginx Pod only on the node labeled `disk=ssd`.

To achieve this, we will use Node Affinity.

---

## 2. Types of Node Affinity

Kubernetes provides two main types of Node Affinity.

### 1. `requiredDuringSchedulingIgnoredDuringExecution`

- This is a mandatory rule during Pod scheduling.
- Kubernetes schedules the Pod only on a node that matches the specified conditions.
- If no node matches the rule, the Pod remains in the `Pending` state.
- If the node's labels change after the Pod is scheduled, Kubernetes does not automatically move the running Pod to another node.

### 2. `preferredDuringSchedulingIgnoredDuringExecution`

- This is a preference rather than a mandatory rule.
- Kubernetes tries to schedule the Pod on a matching node.
- If no matching node is available, Kubernetes may schedule the Pod on another suitable node.

---

## 3. Step 1: Check the Existing Nodes

First, list all nodes in the Kubernetes cluster and check their labels.

**Command:**

```powershell
kubectl get nodes --show-labels
```

**Output:**

```text
NAME               STATUS   ROLES    AGE   VERSION   LABELS
desktop-worker     Ready    <none>   25h   v1.34.3   kubernetes.io/hostname=desktop-worker,...
desktop-worker2    Ready    <none>   25h   v1.34.3   kubernetes.io/hostname=desktop-worker2,...
```

From the output, we can see that the cluster contains two worker nodes.

Neither node has the required `disk=ssd` label yet.

---

## 4. Step 2: Create the Pod YAML File

Create a file named `demo.yaml` and add the following configuration.

**File: `demo.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disk
                operator: In
                values:
                  - ssd
  containers:
    - name: nginx
      image: nginx
```

### Explanation

- `apiVersion: v1` specifies the Kubernetes API version.
- `kind: Pod` creates a Pod.
- `metadata.name: nginx-pod` specifies the Pod name.
- `affinity.nodeAffinity` defines the Node Affinity rules.
- `requiredDuringSchedulingIgnoredDuringExecution` makes the rule mandatory during scheduling.
- `key: disk` specifies the node label key.
- `operator: In` means the label value must match one of the specified values.
- `values: - ssd` specifies the required label value.
- `image: nginx` specifies the Nginx container image.

**Important:** At least one suitable node must have the label `disk=ssd` for this Pod to be scheduled successfully.

---

## 5. Step 3: Label the Required Node

We want the Pod to run on `desktop-worker2`.

Add the label `disk=ssd` to that node.

**Command:**

```powershell
kubectl label node desktop-worker2 disk=ssd
```

**Output:**

```text
node/desktop-worker2 labeled
```

The node now has the required label.

### Verify the Node Label

Run the following command:

```powershell
kubectl get nodes --show-labels -l disk=ssd
```

**Output:**

```text
NAME              STATUS   ROLES    AGE   VERSION   LABELS
desktop-worker2   Ready    <none>   25h   v1.34.3   disk=ssd,...
```

The output confirms that `desktop-worker2` has the label `disk=ssd`.

The `-l disk=ssd` option filters the nodes and displays only those with the specified label.

---

## 6. Step 4: Create the Pod

Apply the YAML file to create the Nginx Pod.

**Command:**

```powershell
kubectl apply -f .\demo.yaml
```

**Output:**

```text
pod/nginx-pod created
```

The Pod has been created successfully.

---

## 7. Step 5: Check the Pod Status

Run the following command:

```powershell
kubectl get pods
```

**Output:**

```text
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          3m25s
```

The output shows that the Nginx Pod is running successfully.

However, this command does not show which node is running the Pod.

To verify the node placement, we need to use the next command.

---

## 8. Step 6: Verify Which Node Is Running the Pod

Run the following command:

```powershell
kubectl get pods -o wide
```

**Output:**

```text
NAME        READY   STATUS    RESTARTS   AGE     IP           NODE
nginx-pod   1/1     Running   0          4m37s   10.244.1.6   desktop-worker2
```

### Explanation

The important column is `NODE`.

```text
NODE
desktop-worker2
```

This confirms that `nginx-pod` is running on `desktop-worker2`, as expected.

Kubernetes selected this node because it has the required label:

```text
disk=ssd
```

The Node Affinity rule prevented Kubernetes from scheduling this Pod on `desktop-worker`, which does not have the required label.

---

## 9. Complete Workflow

The following steps summarize the practical:

1. Check the available Kubernetes nodes.
2. Create a Pod YAML file with Node Affinity.
3. Label `desktop-worker2` with `disk=ssd`.
4. Verify the node label.
5. Create the Pod using `kubectl apply`.
6. Check the Pod status using `kubectl get pods`.
7. Verify the assigned node using `kubectl get pods -o wide`.

### Workflow Diagram

```text
Kubernetes Cluster
        |
        v
Check Worker Nodes
        |
        v
Label desktop-worker2
with disk=ssd
        |
        v
Apply demo.yaml
        |
        v
Kubernetes Evaluates
Node Affinity Rules
        |
        v
Select desktop-worker2
        |
        v
nginx-pod Running
on desktop-worker2
```

---

## 10. Conclusion

In this practical, we learned how to use **Node Affinity in Kubernetes** to control Pod placement based on node labels.

We created an Nginx Pod using the `requiredDuringSchedulingIgnoredDuringExecution` rule and specified that the Pod must run on a node with the label `disk=ssd`.

After labeling `desktop-worker2`, we created the Pod and verified its placement using:

```powershell
kubectl get pods -o wide
```

The output confirmed that `nginx-pod` was running on `desktop-worker2`.

**Result:** Node Affinity successfully scheduled the Pod on the node that matched the specified label.

---

## Key Takeaways

- Node Affinity controls Pod placement using node labels.
- `requiredDuringSchedulingIgnoredDuringExecution` enforces a mandatory scheduling condition.
- `preferredDuringSchedulingIgnoredDuringExecution` expresses a scheduling preference.
- `kubectl label node` adds a label to a Kubernetes node.
- `kubectl get nodes --show-labels` displays node labels.
- `kubectl get pods -o wide` shows which node is running each Pod.
