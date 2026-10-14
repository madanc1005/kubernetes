# Kubernetes Network Policies

## 📌 Overview

Kubernetes Network Policies are used to control network communication between Pods. They help define which Pods can communicate with other Pods and which connections should be allowed or blocked.

**In this hands-on project, I learned:**
- What CNI (Container Network Interface) is.
- How Pods communicate through Kubernetes Services.
- How to test connectivity between frontend, backend, and database Pods.
- How to configure Network Policies to restrict traffic.
- How to allow backend access to the database while blocking frontend access to the database.
- How to create a Network Policy that allows all ingress and egress traffic.

## 🎯 Project Architecture

The project contains three main components:

1. **Frontend Pod:** Runs an NGINX web server.
2. **Backend Pod:** Runs an NGINX web server and represents the backend application.
3. **Database Pod:** Runs a MySQL database.

### Expected Network Communication

| Source | Destination | Expected Result |
|---|---|---|
| Frontend | Backend | Allowed |
| Frontend | Database | Blocked |
| Backend | Frontend | Allowed |
| Backend | Database | Allowed |

## 🛠️ Technologies Used

- Kubernetes
- Kind (Kubernetes in Docker)
- YAML
- kubectl
- NGINX
- MySQL
- NetworkPolicy
- CNI

## 📁 Project Structure

```text
kubernetes-network-policies/
│
├── deploy.yml
├── db-network-policy.yml
├── allow-all-policy.yml
└── README.md
```

## 1. Deploy the Kubernetes Resources

Create the frontend Pod and Service, backend Pod and Service, and database Pod and Service in `deploy.yml`.

The frontend and backend use the NGINX image, while the database uses the MySQL image.

Apply the configuration:

```bash
kubectl apply -f deploy.yml
```

Check the Pods:

```bash
kubectl get pods
```

Check the Services:

```bash
kubectl get svc
```

All three Pods should reach the `Running` state before you test connectivity.

## 2. Test Communication Before Applying Network Policies

### Test frontend to backend

Enter the frontend Pod:

```bash
kubectl exec -it frontend-pod -- /bin/bash
```

Test the backend Service:

```bash
curl backend-service:80
```

Expected result: the NGINX welcome page.

### Test frontend to database

Install Telnet inside the frontend container if needed:

```bash
apt-get update
apt-get install -y telnet
```

Test the database connection:

```bash
telnet db-service 3306
```

Before applying the restrictive Network Policy, the connection should be possible if the cluster networking is working.

**Note:** Telnet is used here to test TCP connectivity. MySQL may display protocol data or close the connection because Telnet does not perform a MySQL login.

### Test backend connectivity

Enter the backend Pod:

```bash
kubectl exec -it backend-pod -- /bin/bash
```

Test frontend connectivity:

```bash
curl frontend-service:80
```

Test database connectivity:

```bash
telnet db-service 3306
```

These tests help establish the initial connectivity between the Pods.

## 3. Configure a Network Policy for the Database

Create a file named `db-network-policy.yml`.

The following policy allows incoming TCP traffic to the database Pod on port `3306` only from Pods with the label `role: backend-role`.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-test
spec:
  podSelector:
    matchLabels:
      role: db-role
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: backend-role
      ports:
        - protocol: TCP
          port: 3306
```

Apply the policy:

```bash
kubectl apply -f db-network-policy.yml
```

Check the policy:

```bash
kubectl get networkpolicy
```

### How Does This Policy Work?

- `podSelector` identifies the database Pod using `role: db-role`.
- `policyTypes: Ingress` specifies that the policy controls incoming connections.
- `from` identifies the Pods allowed to connect to the database.
- `role: backend-role` matches the label on `backend-pod`.
- `port: 3306` allows MySQL TCP traffic.

**Important:** The labels in the policy must match the labels in `deploy.yml`. The backend Pod uses `role: backend-role`, so the policy must use the same label.

## 4. Test Connectivity After Applying the Policy

### Backend to database

Enter the backend Pod:

```bash
kubectl exec -it backend-pod -- /bin/bash
```

Test the database:

```bash
telnet db-service 3306
```

Expected result: the TCP connection should be allowed.

### Frontend to database

Enter the frontend Pod:

```bash
kubectl exec -it frontend-pod -- /bin/bash
```

Test the database:

```bash
telnet db-service 3306
```

Expected result: the connection should time out because the frontend Pod is not permitted by the database's ingress policy.

### Frontend to backend

From the frontend Pod:

```bash
curl backend-service:80
```

Expected result: the NGINX welcome page.

The database ingress policy does not block this connection because it targets incoming traffic to the database Pod, not the backend Pod.

### Backend to frontend

From the backend Pod:

```bash
curl frontend-service:80
```

Expected result: the NGINX welcome page.

## 5. Allow All Ingress and Egress Traffic

Create a file named `allow-all-policy.yml`.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - {}
  egress:
    - {}
```

Apply the policy:

```bash
kubectl apply -f allow-all-policy.yml
```

This policy allows all incoming and outgoing traffic for the selected Pods in the namespace.

**Important:** This is a broad rule and is not recommended for production environments. Also, Network Policies are additive: an allow-all policy can override the practical restrictions imposed by another policy on the same Pods and traffic directions.

## 6. Understanding CNI

CNI stands for **Container Network Interface**. It provides networking for Kubernetes Pods.

A CNI plugin that supports NetworkPolicy enforcement is required for Network Policies to work correctly.

For a Kind cluster, you can configure a cluster without the default CNI by using a configuration such as:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  podSubnet: 192.168.0.0/16
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

**Important:** Disabling the default CNI does not install a replacement or enable Network Policy enforcement by itself. Install a compatible CNI plugin before relying on Pod networking or Network Policies. Creating a new cluster is generally safer than changing the networking configuration of a cluster used for other work.

## 🧠 Key Learnings

- Kubernetes Services provide stable endpoints for accessing Pods.
- Network Policies control allowed ingress and egress traffic.
- Pod labels are important when selecting traffic sources and destinations.
- A database ingress policy can restrict access to selected backend Pods.
- A policy allowing all traffic is useful for testing but weakens network isolation.
- Network Policies require support from the installed networking plugin.

## 🚀 Conclusion

This hands-on exercise demonstrates Kubernetes Pod-to-Pod communication and how Network Policies can restrict access to a MySQL database.

The main objective is to allow the backend to communicate with both the frontend and database while preventing the frontend from accessing the database. The database ingress policy implements the database access restriction; additional egress policies may be needed to enforce a complete frontend-only-to-backend rule.

#Kubernetes #DevOps #NetworkPolicy #CNI #Docker #CloudComputing #LearningJourney
