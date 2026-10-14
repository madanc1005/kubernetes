# Kubernetes ServiceAccount with RBAC

## 📌 Overview

A **ServiceAccount** in Kubernetes provides an identity for applications, Pods, and automated processes to interact with the Kubernetes API.

For example, Jenkins or another automation tool may use a ServiceAccount to perform Kubernetes operations such as listing Pods or deploying applications, when properly configured.

## 👥 Types of Users in Kubernetes

1. **Human Users:** People who access and manage the Kubernetes cluster.
2. **ServiceAccounts:** Identities used by applications, Pods, and automated processes to access the Kubernetes API.

**Note:** Kubernetes creates a default ServiceAccount in each namespace. A Pod uses the namespace's default ServiceAccount unless another one is specified.

## 🎯 Objective

In this practice, we will:

- Create a ServiceAccount.
- Create a Role with permission to list, get, and watch Pods.
- Create a RoleBinding to assign the Role to the ServiceAccount.
- Verify permissions using `kubectl auth can-i`.
- Create a Pod using the custom ServiceAccount.

## 🛠️ Prerequisites

- A running Kubernetes cluster.
- `kubectl` installed and configured.
- Basic knowledge of Kubernetes Pods and YAML files.

## Step 1: Create a ServiceAccount

Run the following command:

```powershell
kubectl create sa service-account1
```

Verify the ServiceAccount:

```powershell
kubectl get serviceaccounts
```

**Explanation:** This creates a ServiceAccount named `service-account1` in the current namespace, which is `default` unless configured otherwise.

## Step 2: Create a Role

Generate the Role YAML:

```powershell
kubectl create role sa-role --verb=list,get,watch --resource=pods --dry-run=client -o yaml > role.yml
```

Apply the Role:

```powershell
kubectl apply -f role.yml
```

Verify the Role:

```powershell
kubectl get roles
kubectl describe role sa-role
```

### Role YAML (`role.yml`)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: sa-role
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list", "get", "watch"]
```

**Explanation:**

- `Role`: Defines permissions within a namespace.
- `resources: ["pods"]`: Specifies Pods as the resource.
- `list`: Allows listing Pods.
- `get`: Allows retrieving an individual Pod.
- `watch`: Allows watching for changes to Pods.

## Step 3: Create a RoleBinding

Generate the RoleBinding YAML:

```powershell
kubectl create rolebinding sa-binding --role=sa-role --serviceaccount=default:service-account1 --dry-run=client -o yaml > binding.yaml
```

Apply the RoleBinding:

```powershell
kubectl apply -f binding.yaml
```

Verify the RoleBinding:

```powershell
kubectl get rolebindings
kubectl describe rolebinding sa-binding
```

### RoleBinding YAML (`binding.yaml`)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-binding
subjects:
  - kind: ServiceAccount
    name: service-account1
    namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: sa-role
```

**Explanation:** A RoleBinding connects the Role `sa-role` to the ServiceAccount `service-account1`, granting it the permissions defined in the Role.

**Important:** Use `--serviceaccount=default:service-account1`, not `--user=service-account1`, because a ServiceAccount is not a regular username.

## Step 4: Verify ServiceAccount Permissions

Before creating the Role and RoleBinding, the ServiceAccount should not have the permissions defined in `sa-role`.

Check permission to get Pods:

```powershell
kubectl auth can-i get pods --as=system:serviceaccount:default:service-account1
```

Expected result before applying the RoleBinding:

```text
no
```

After applying `role.yml` and `binding.yaml`, run the same command:

```powershell
kubectl auth can-i get pods --as=system:serviceaccount:default:service-account1
```

Expected result:

```text
yes
```

Check the other permissions:

```powershell
kubectl auth can-i list pods --as=system:serviceaccount:default:service-account1
kubectl auth can-i watch pods --as=system:serviceaccount:default:service-account1
kubectl auth can-i delete pods --as=system:serviceaccount:default:service-account1
```

Expected results:

- `list pods` → `yes`
- `watch pods` → `yes`
- `delete pods` → `no`

**Explanation:** The ServiceAccount receives only the permissions granted by the Role and any other applicable RBAC bindings. It does not receive permission to delete Pods from this Role.

## Step 5: Create a Pod Using the ServiceAccount

Create a file named `demo.yaml`.

### Pod YAML (`demo.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
  labels:
    run: sa-pod
spec:
  serviceAccountName: service-account1
  containers:
    - name: sa-pod
      image: nginx
```

Apply the YAML file:

```powershell
kubectl apply -f demo.yaml
```

Verify the Pod:

```powershell
kubectl get pods
kubectl describe pod sa-pod
```

Check which ServiceAccount the Pod uses:

```powershell
kubectl get pod sa-pod -o jsonpath="{.spec.serviceAccountName}"
```

Expected output:

```text
service-account1
```

**Explanation:** The `serviceAccountName` field tells Kubernetes to assign the custom ServiceAccount to the Pod.

The ServiceAccount provides the Pod's identity when it accesses the Kubernetes API. Creating a Pod with this ServiceAccount does not automatically make the container perform API operations.

## Step 6: Clean Up Resources

Delete the Pod:

```powershell
kubectl delete pod sa-pod
```

Delete the RoleBinding:

```powershell
kubectl delete rolebinding sa-binding
```

Delete the Role:

```powershell
kubectl delete role sa-role
```

Delete the ServiceAccount:

```powershell
kubectl delete sa service-account1
```

## 📂 Project Structure

```text
day16/
├── README.md
├── role.yml
├── binding.yaml
└── demo.yaml
```

## 🔑 Key Takeaways

- A ServiceAccount provides an identity for Pods and applications.
- A Role defines permissions within a namespace.
- A RoleBinding assigns those permissions to a user, group, or ServiceAccount.
- `kubectl auth can-i` helps verify Kubernetes authorization.
- `serviceAccountName` specifies which ServiceAccount a Pod uses.
- Always follow the principle of least privilege: grant only the permissions required.

## 📚 Commands Summary

| Command | Purpose |
|---|---|
| `kubectl create sa` | Create a ServiceAccount |
| `kubectl create role` | Define permissions |
| `kubectl create rolebinding` | Assign a Role to an identity |
| `kubectl auth can-i` | Check authorization |
| `kubectl apply -f` | Create or update resources |
| `kubectl describe` | Inspect a resource |
| `kubectl delete` | Remove a resource |

---

**Practice completed:** Created a Kubernetes ServiceAccount, configured RBAC permissions, verified access, and deployed a Pod using the custom ServiceAccount.

