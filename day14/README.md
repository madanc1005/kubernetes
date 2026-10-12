# Kubernetes User Authentication Using CSR

## 📌 Project Overview

This project demonstrates how to create a user named `obito` in Kubernetes using a Certificate Signing Request (CSR).

We will generate a private key, create a CSR, submit it to the Kubernetes API, approve the CSR, and retrieve the signed certificate.

## 🛠️ Prerequisites

- A working Kubernetes cluster.
- `kubectl` configured to access the cluster.
- OpenSSL installed.
- Access to approve Kubernetes CertificateSigningRequests (CSRs).

## Step 1: Generate the Private Key

Generate a 2048-bit RSA private key for the user `obito`.

```bash
openssl genrsa -out obito.key 2048
```

**Explanation:**
- `openssl genrsa` generates an RSA private key.
- `-out obito.key` saves the private key in `obito.key`.
- `2048` specifies the key size in bits.

**Output:** `obito.key`

## Step 2: Generate the Certificate Signing Request

Use the private key to generate a CSR.

```bash
openssl req -new -key obito.key -out obito.csr -subj "/CN=obito"
```

**Explanation:**
- `-new` creates a new CSR.
- `-key obito.key` specifies the private key.
- `-out obito.csr` saves the CSR.
- `-subj "/CN=obito"` sets the Common Name (CN) to `obito`.

The Common Name will identify the username in Kubernetes client-certificate authentication.

**Output:** `obito.csr`

## Step 3: Encode the CSR in Base64

Kubernetes expects the CSR in the `request` field to be Base64-encoded.

On Linux or Git Bash, run:

```bash
cat obito.csr | base64 | tr -d '\n'
```

Copy the complete output. It is Base64-encoded data, not encrypted data.

On Windows PowerShell, you can also use:

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("obito.csr"))
```

## Step 4: Create the Kubernetes CSR YAML File

Create a file named `obito-csr.yaml`.

Paste the Base64 output from Step 3 into the `request` field.

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: obito
spec:
  request: <PASTE_BASE64_ENCODED_CSR_HERE>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 864000
  usages:
    - client auth
```

**Note:** `864000` seconds equals 10 days. The actual issued certificate duration depends on the Kubernetes cluster's configuration.

## Step 5: Submit the CSR to Kubernetes

Apply the YAML file.

```bash
kubectl apply -f obito-csr.yaml
```

Check the CSR status.

```bash
kubectl get csr
```

Example output:

```text
NAME    AGE   SIGNERNAME                            REQUESTOR          CONDITION
obito   2m    kubernetes.io/kube-apiserver-client   kubernetes-admin   Pending
```

The CSR is initially in the `Pending` state because it has not yet been approved.

## Step 6: Approve the CSR

Approve the request using:

```bash
kubectl certificate approve obito
```

Verify the status:

```bash
kubectl get csr obito
```

The CSR should show `Approved,Issued` if the Kubernetes signer successfully issues the certificate.

## Step 7: Retrieve the Signed Certificate

Export the issued certificate from the Kubernetes CSR object.

```bash
kubectl get csr obito -o jsonpath='{.status.certificate}' | base64 -d > obito.crt
```

This command saves the signed client certificate to `obito.crt`.

On Windows PowerShell, use:

```powershell
$cert = kubectl get csr obito -o jsonpath='{.status.certificate}'
[IO.File]::WriteAllBytes("obito.crt", [Convert]::FromBase64String($cert))
```

**Important:** The output is a certificate, not a private key. Keep the private key in `obito.key`; use `obito.crt` as the signed client certificate.

## Step 8: Verify the Certificate

Inspect the certificate details.

```bash
openssl x509 -in obito.crt -text -noout
```

Verify that the certificate contains the expected subject:

```text
Subject: CN = obito
```

## Step 9: Configure kubectl for the User

The following commands configure a separate kubeconfig user and context.

```bash
kubectl config set-credentials obito \
  --client-key=obito.key \
  --client-certificate=obito.crt \
  --embed-certs=true
```

Get the current cluster and context information:

```bash
kubectl config current-context
kubectl config view
```

Create a context using your actual cluster name and context settings:

```bash
kubectl config set-context obito-context \
  --cluster=<YOUR_CLUSTER_NAME> \
  --user=obito
```

Switch to the new context:

```bash
kubectl config use-context obito-context
```

**Note:** Use the appropriate cluster and namespace settings for your environment. This configures authentication but does not automatically grant permissions to access Kubernetes resources.

## Step 10: Grant Permissions (RBAC)

To allow `obito` to perform operations, configure Kubernetes Role-Based Access Control (RBAC).

For example, the following Role and RoleBinding grant read-only access to Pods in the `default` namespace.

Create `obito-rbac.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: obito-pod-reader
  namespace: default
subjects:
  - kind: User
    name: obito
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply the RBAC configuration using an administrator context:

```bash
kubectl apply -f obito-rbac.yaml
```

Test the permission:

```bash
kubectl auth can-i list pods --namespace default --as=obito
```

This checks the RBAC authorization decision. Actual access using the user's kubeconfig should also be tested.


Keep `obito.key`, `obito.crt`, and any kubeconfig containing credentials outside the public repository.



## 🎯 What I Learned

- Generating RSA private keys using OpenSSL.
- Creating Certificate Signing Requests.
- Understanding Base64 encoding.
- Creating and managing Kubernetes CSR resources.
- Approving CSRs and retrieving signed certificates.
- Configuring kubectl client-certificate authentication.
- Granting permissions using Kubernetes RBAC.

## 📚 Useful Commands

```bash
kubectl api-resources | grep certificatesigningrequests
kubectl get csr
kubectl describe csr obito
kubectl certificate approve obito
kubectl get csr obito -o yaml
kubectl config get-contexts
```

## Conclusion

This project demonstrates Kubernetes client-certificate authentication using OpenSSL and CertificateSigningRequests. A signed certificate identifies the user, while RBAC determines which Kubernetes resources and operations the user is authorized to access.
