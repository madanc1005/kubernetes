# Kubernetes Secrets

## 1. What is a Secret in Kubernetes?

In Kubernetes, a **Secret** is an object used to store sensitive information, such as usernames, passwords, API tokens, database credentials, and TLS certificates.

Secrets help us manage sensitive data separately from application code and Pod configuration files.

## 2. Why Do We Need Secrets?

Imagine we have a web application running inside a Kubernetes Pod. The application needs to connect to a MySQL database, which requires a username and password.

Instead of directly writing these credentials into the application code or Pod manifest, we can store them in a Kubernetes Secret.

### Example

```text
       Web Application
       Running inside a Pod
               |
               v
      Kubernetes Secret
      Username and Password
               |
               v
         MySQL Database
       Authenticates the App
```

**Benefits of Secrets:**
- Store sensitive information separately from application configuration.
- Avoid hardcoding credentials in application code.
- Make sensitive configuration easier to manage.
- Reuse the same Secret across multiple Pods when appropriate.

**Note:** Kubernetes Secrets are Base64-encoded by default, not automatically encrypted just because they are Secrets. Use encryption at rest, appropriate RBAC permissions, and other security best practices.

## 3. What Information Can a Secret Store?

Examples include:

- Database usernames and passwords.
- Docker registry credentials.
- API keys.
- Authentication tokens.
- TLS certificates and private keys.

## 4. Creating a Kubernetes Secret

### Step 1: Create an Environment File

Create a file named `text.txt` with the following sample contents:

```text
color=pink
key=value
madan=gowda
my-life=obito-life
obito=uchiha
```

This file contains sample values for learning purposes. Do not commit real passwords, API keys, or production credentials to GitHub.

### Step 2: Generate the Secret YAML

Run the following command in PowerShell:

```powershell
kubectl create secret generic my-secret --from-env-file=./text.txt --dry-run=client -o yaml
```

**Explanation:**
- `kubectl create secret`: Creates a Kubernetes Secret.
- `generic`: Creates a generic Secret.
- `my-secret`: Specifies the Secret name.
- `--from-env-file=./text.txt`: Reads key-value pairs from the environment file.
- `--dry-run=client`: Generates the configuration without creating the resource.
- `-o yaml`: Displays the generated YAML.

Example output:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
data:
  color: cGluaw==
  key: dmFsdWU=
  madan: Z293ZGE=
  my-life: b2JpdG8tbGlmZQ==
  obito: dWNoaWhh
```

The values under `data` are Base64-encoded.

### Step 3: Create the Secret

Run:

```powershell
kubectl create secret generic my-secret --from-env-file=./text.txt
```

Expected output:

```text
secret/my-secret created
```

### Step 4: Verify the Secret

```powershell
kubectl get secrets
```

To inspect the Secret metadata and keys:

```powershell
kubectl describe secret my-secret
```

To view the Secret YAML:

```powershell
kubectl get secret my-secret -o yaml
```

Be careful: displaying Secret YAML can expose encoded sensitive values.

## 5. Creating a Pod and Mounting the Secret

Create a file named `pod.yaml` with the following configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/db-secret
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: my-secret
```

### Explanation

- `kind: Pod`: Creates a Kubernetes Pod.
- `name: my-app`: Specifies the Pod name.
- `image: nginx`: Uses the NGINX container image.
- `volumeMounts`: Mounts the Secret into the container.
- `mountPath: /etc/db-secret`: Specifies where the Secret files appear.
- `readOnly: true`: Prevents the container from modifying the mounted files.
- `secretName: my-secret`: Specifies the Secret to mount.

Each key in the Secret becomes a file inside the mounted directory.

## 6. Create the Pod

Run:

```powershell
kubectl apply -f pod.yaml
```

Check the Pod status:

```powershell
kubectl get pods
```

Wait until the Pod is running before executing commands inside it.

## 7. Access the Pod and Verify the Secret

### Step 1: Execute a Shell Inside the Pod

```powershell
kubectl exec -it my-app -- /bin/bash
```

### Step 2: List the Mounted Secret Files

Inside the container, run:

```bash
ls /etc/db-secret/
```

Expected output:

```text
color
key
madan
my-life
obito
```

Each filename corresponds to a key in the Secret.

### Step 3: Read a Secret Value

For example:

```bash
cat /etc/db-secret/color
```

Expected output:

```text
pink
```

Kubernetes presents the decoded value inside the mounted file.

Exit the container shell:

```bash
exit
```

## 8. Restarting a Deployment

If a Secret is consumed through environment variables, existing containers do not automatically receive updated environment values when the Secret changes. Restart the Deployment to recreate its Pods:

```bash
kubectl rollout restart deployment/deploy-name
```

Replace `deploy-name` with your actual Deployment name.

**Important:** Secret volumes are updated eventually when the Secret changes, subject to Kubernetes propagation behavior. A `subPath` mount does not receive automatic updates.

## 9. Important Kubernetes Commands

| Command | Purpose |
|---|---|
| `kubectl get secrets` | Lists Secrets |
| `kubectl describe secret my-secret` | Displays Secret details without showing values |
| `kubectl get secret my-secret -o yaml` | Displays Secret YAML |
| `kubectl get pods` | Lists Pods |
| `kubectl apply -f pod.yaml` | Creates or updates resources |
| `kubectl exec -it my-app -- /bin/bash` | Opens a shell inside the Pod |
| `kubectl delete pod my-app` | Deletes the Pod |
| `kubectl delete secret my-secret` | Deletes the Secret |

## 10. What I Learned

- What Kubernetes Secrets are and why they are needed.
- How to create a Secret from an environment file.
- How to generate Secret YAML using `--dry-run=client -o yaml`.
- How to mount a Secret as a volume inside a Pod.
- How to access and verify Secret files inside a running container.
- How to use essential `kubectl` commands for Secret management.

## Conclusion

Kubernetes Secrets provide a convenient way to manage sensitive application configuration. By creating a Secret and mounting it into a Pod, applications can access the required values without hardcoding them into their Pod manifest.

For production environments, configure encryption at rest, restrict access using RBAC, and never commit real credentials or unprotected Secret manifests to public GitHub repositories.
