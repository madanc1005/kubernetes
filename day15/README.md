# Kubernetes RBAC - Role and RoleBinding

## 📌 Overview

This project demonstrates how to configure **Role-Based Access Control (RBAC)** in Kubernetes.

RBAC allows us to control what actions a user can perform on Kubernetes resources. In this example, we create a Role that allows a user named `obito` to get, list, and watch pods in the `default` namespace.

## 🛠️ Prerequisites

- Kubernetes cluster running (Docker Desktop Kubernetes).
- `kubectl` installed and configured.
- OpenSSL installed to generate user certificates.
- A Kubernetes user private key and signed client certificate.

## 📂 Project Structure

```text
Kubernetes/
└── day14/
    ├── role.yml
    ├── role-binding.yml
    ├── obito.key
    ├── obito.csr
    ├── obito-certificate.crt
    └── README.md
```

> Note: Keep private keys out of GitHub. Add `obito.key` and any other private key files to `.gitignore`.

## Step 1: Create the Role

First, check the available Kubernetes API resources:

```powershell
kubectl api-resources
```

Create a file named `role.yml` with the following configuration:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: obito-role
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
```

### Explanation

- `apiVersion`: Specifies the Kubernetes RBAC API version.
- `kind: Role`: Creates a Role within a namespace.
- `metadata.name`: Names the Role `obito-role`.
- `apiGroups: [""]`: Refers to the core Kubernetes API group.
- `resources: ["pods"]`: Allows permissions on pods.
- `verbs`: Allows the user to get, watch, and list pods.

Apply the Role:

```powershell
kubectl apply -f ./day14/role.yml
```

Verify the Role:

```powershell
kubectl get roles
kubectl describe role obito-role
```

## Step 2: Create the RoleBinding

Create a file named `role-binding.yml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: obito-role-binding
  namespace: default
subjects:
  - kind: User
    name: obito
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: obito-role
  apiGroup: rbac.authorization.k8s.io
```

### Explanation

- `kind: RoleBinding`: Connects a Role to a user or another supported subject.
- `subjects`: Specifies the Kubernetes username `obito`.
- `roleRef`: Refers to the Role named `obito-role`.
- `namespace: default`: Limits the binding and Role permissions to the `default` namespace.

Apply the RoleBinding:

```powershell
kubectl apply -f ./day14/role-binding.yml
```

Verify the RoleBinding:

```powershell
kubectl get rolebindings
kubectl describe rolebinding obito-role-binding
```

## Step 3: Configure the User Credentials

The following steps assume that you have already generated a private key and obtained a **CA-signed client certificate** for the user `obito`.

Set the credentials in your kubeconfig:

```powershell
kubectl config set-credentials obito `
  --client-key=./day14/obito.key `
  --client-certificate=./day14/obito-certificate.crt `
  --embed-certs=true
```

**Important:** Use the actual path and filename of your signed certificate. A `.csr` file is a certificate signing request, not a client certificate.

Verify the configured clusters, current context, and available contexts:

```powershell
kubectl config get-clusters
kubectl config current-context
kubectl config get-contexts
```

## Step 4: Create a New Context

Create a context for the user `obito`:

```powershell
kubectl config set-context obito-context `
  --cluster=docker-desktop `
  --user=obito `
  --namespace=default
```

Verify the context:

```powershell
kubectl config get-contexts
```

Switch to the new context:

```powershell
kubectl config use-context obito-context
```

Verify the active context:

```powershell
kubectl config current-context
```

Expected output:

```text
obito-context
```

> Make sure `docker-desktop` is the actual cluster name shown by `kubectl config get-clusters`.

## Step 5: Verify RBAC Permissions

Check whether the current authenticated user can get pods:

```powershell
kubectl auth can-i get pods
```

Expected output:

```text
yes
```

You can also check the permissions assigned to `obito` explicitly:

```powershell
kubectl auth can-i get pods --as=obito
kubectl auth can-i list pods --as=obito
kubectl auth can-i watch pods --as=obito
```

Expected output for all three commands:

```text
yes
```

Check an action that was not granted:

```powershell
kubectl auth can-i delete pods --as=obito
```

Expected output:

```text
no
```

**Note:** The `--as=obito` option tests impersonated permissions and may require additional impersonation privileges. The command does not switch your actual identity to `obito`.

## Step 6: Test Pod Access

List the pods in the default namespace:

```powershell
kubectl get pods
```

The configured user's real access depends on successful client-certificate authentication and the RoleBinding.

## 🔐 Security Best Practices

- Never commit private keys such as `obito.key` to a public repository.
- Do not upload credentials or sensitive certificates to GitHub.
- Use `.gitignore` to exclude private key files.
- Use CA-signed certificates for Kubernetes client authentication.
- Follow the principle of least privilege by granting only the permissions required.
- Remember that a Kubernetes Role and RoleBinding grant permissions; they do not create a user account or issue a certificate.

## 🎯 What I Learned

- Kubernetes RBAC fundamentals.
- Creating a Role and RoleBinding using YAML.
- Configuring Kubernetes client credentials.
- Creating and switching kubeconfig contexts.
- Checking permissions with `kubectl auth can-i`.
- Applying the principle of least privilege.

## 📚 Useful Commands

| Command | Purpose |
|---|---|
| `kubectl api-resources` | Lists API resources |
| `kubectl apply -f file.yml` | Applies a YAML configuration |
| `kubectl get roles` | Lists Roles |
| `kubectl get rolebindings` | Lists RoleBindings |
| `kubectl config get-contexts` | Lists kubeconfig contexts |
| `kubectl config use-context NAME` | Switches the active context |
| `kubectl auth can-i get pods` | Checks a permission |

---

**Project:** Kubernetes RBAC - Role and RoleBinding  
**Tools:** Kubernetes, kubectl, YAML, PowerShell, Docker Desktop
