# Kubernetes ConfigMap Demo

## 📌 Overview

A **ConfigMap** in Kubernetes is used to store configuration data separately from application code and container images.

For example, an application may need environment variables such as the application environment, port number, log level, or database server address. Instead of hardcoding these values in the application, we can store them in a ConfigMap and inject them into a Pod.

In this project, we create a ConfigMap and inject its configuration values into a Kubernetes Pod using `envFrom` and `configMapRef`.

## 🎯 Objectives

- Understand what a ConfigMap is in Kubernetes.
- Create a ConfigMap using a YAML file.
- Inject ConfigMap values into a Pod as environment variables.
- Verify the environment variables inside a running container.

## 🛠️ Prerequisites

- Kubernetes cluster
- `kubectl` installed and configured
- Basic knowledge of Kubernetes Pods and YAML files

## 📁 Project Structure

```text
Kubernetes/
├── config.yml
├── demo.yml
└── README.md
```

## 1. Create a ConfigMap

Create a file named `config.yml` with the following configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config1
data:
  APP_ENV: "development"
  APP_PORT: "8080"
  LOG_LEVEL: "info"
  ENV: "testing"
```

### Explanation

- `apiVersion: v1` — Specifies the Kubernetes API version.
- `kind: ConfigMap` — Defines the resource as a ConfigMap.
- `metadata.name` — Sets the ConfigMap name to `config1`.
- `data` — Contains the configuration key-value pairs.

The ConfigMap stores the following values:

| Key | Value |
|---|---|
| `APP_ENV` | `development` |
| `APP_PORT` | `8080` |
| `LOG_LEVEL` | `info` |
| `ENV` | `testing` |

## 2. Create a Pod and Inject the ConfigMap

Create a file named `demo.yml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
    - name: pod1
      image: nginx
      envFrom:
        - configMapRef:
            name: config1
```

### Explanation

- `kind: Pod` — Creates a Kubernetes Pod.
- `image: nginx` — Uses the Nginx container image.
- `envFrom` — Imports configuration values as environment variables.
- `configMapRef.name: config1` — References the ConfigMap created earlier.

**Important:** The ConfigMap must exist before creating the Pod that references it.

## 3. Apply the Configuration

Apply the ConfigMap:

```powershell
kubectl apply -f .\config.yml
```

Expected output:

```text
configmap/config1 created
```

Apply the Pod configuration:

```powershell
kubectl apply -f .\demo.yml
```

Expected output:

```text
pod/pod1 created
```

## 4. Verify the ConfigMap

Run the following command:

```powershell
kubectl get cm
```

Expected output:

```text
NAME               DATA   AGE
config1            4      20s
kube-root-ca.crt   1      47h
```

The `DATA` column shows that `config1` contains four configuration entries.

## 5. Verify the Pod

Run:

```powershell
kubectl get pods
```

Expected output:

```text
NAME   READY   STATUS    RESTARTS   AGE
pod1   1/1     Running   0          17s
```

The `Running` status indicates that the Pod is running successfully.

## 6. Access the Container

Execute the following command to open a Bash shell inside the Pod:

```powershell
kubectl exec -it pod1 -- /bin/bash
```

You should see a prompt similar to:

```text
root@pod1:/#
```

## 7. Verify the Environment Variables

Check the application environment:

```bash
echo $APP_ENV
```

Output:

```text
development
```

Check the application port:

```bash
echo $APP_PORT
```

Output:

```text
8080
```

Check the log level:

```bash
echo $LOG_LEVEL
```

Output:

```text
info
```

Check the environment value:

```bash
echo $ENV
```

Output:

```text
testing
```

These results confirm that Kubernetes successfully injected the ConfigMap values into the container as environment variables.

## 🔄 How It Works

```text
config.yml
    |
    v
ConfigMap: config1
    |
    v
demo.yml
    |
    v
Pod: pod1
    |
    v
envFrom + configMapRef
    |
    v
Environment Variables
    |
    v
APP_ENV=development
APP_PORT=8080
LOG_LEVEL=info
ENV=testing
```

## 🧹 Cleanup

To delete the Pod:

```powershell
kubectl delete -f .\demo.yml
```

To delete the ConfigMap:

```powershell
kubectl delete -f .\config.yml
```

## ⚠️ Important Notes

- ConfigMaps are intended for non-sensitive configuration data.
- Do not store passwords, API keys, or other secrets in a ConfigMap. Use Kubernetes Secrets and appropriate access controls instead.
- ConfigMap values injected as environment variables are set when the container starts. Updating the ConfigMap does not automatically update existing environment variables in a running container; the container generally needs to be restarted.
- Environment variables are available to processes inside the container. They do not automatically configure Nginx unless the application or its startup configuration uses them.

## ✅ Conclusion

In this hands-on project, we created a Kubernetes ConfigMap, connected it to a Pod using `envFrom` and `configMapRef`, and verified the configuration values inside the running container.

This demonstrates how Kubernetes allows us to manage application configuration separately from application code and container images.

**Key takeaway:** ConfigMaps help make applications more flexible by keeping configuration data separate from the application itself.