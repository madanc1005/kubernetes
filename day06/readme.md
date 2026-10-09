# Kubernetes Multi-Container Pod

## Overview
This project demonstrates a Multi-Container Pod in Kubernetes using Nginx and BusyBox containers.

## Containers
- **Nginx:** Serves a webpage.
- **BusyBox:** Writes a message to a file every 5 seconds.

Both containers share data using an `emptyDir` volume.

## Technologies
- Kubernetes
- YAML
- Nginx
- BusyBox

## Deploy the Pod

```bash
kubectl apply -f multi-container-pod.yaml
```

Check the Pod status:

```bash
kubectl get pods
```

Access the application:

```bash
kubectl port-forward pod/multi-container-pod 8080:80
```

Open another terminal and run:

```bash
curl.exe http://localhost:8080
```

## Key Concepts
- Multi-Container Pods
- Shared Volumes
- `emptyDir`
- `kubectl` commands

## Conclusion
This project demonstrates how two containers work together inside a Kubernetes Pod using shared storage.