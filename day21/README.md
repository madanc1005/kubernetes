# DevOps Hands-On Learning

A personal learning repository documenting my DevOps practice, commands, troubleshooting, and key concepts. I am building these notes step by step while practicing in Linux, Git, Docker, AWS, Jenkins, and Kubernetes.

> **Note:** Some sections describe hands-on practice; others are learning notes and topics to explore further. Commands and paths are examples from my lab environment and may need to be adjusted for another system.

## Table of Contents

- [Learning Goals](#learning-goals)
- [Tools and Technologies](#tools-and-technologies)
- [1. Linux and Shell](#1-linux-and-shell)
- [2. Git and GitHub](#2-git-and-github)
- [3. Cloud and AWS](#3-cloud-and-aws)
- [4. Docker](#4-docker)
- [5. CI/CD and Jenkins](#5-cicd-and-jenkins)
- [6. Kubernetes](#6-kubernetes)
- [7. etcd Backup and Restore Lab](#7-etcd-backup-and-restore-lab)
- [Troubleshooting Lessons](#troubleshooting-lessons)
- [Next Steps](#next-steps)

## Learning Goals

- Understand DevOps fundamentals through practical exercises.
- Become comfortable using Linux commands and troubleshooting from the terminal.
- Track changes with Git and publish learning notes to GitHub.
- Practice containerization with Docker and Docker Compose.
- Understand CI/CD concepts and Jenkins pipelines.
- Learn Kubernetes components and practice cluster troubleshooting.
- Understand how etcd snapshots can be backed up and restored.

## Tools and Technologies

- Linux / Ubuntu
- Git and GitHub
- AWS EC2 and SSH
- Docker and Docker Compose
- Jenkins and CI/CD concepts
- Kubernetes, `kubectl`, `kubelet`, `crictl`, and `etcdctl`
- Terraform concepts
- PowerShell and Windows command-line tools

## 1. Linux and Shell

### Topics covered

- Navigating directories and files.
- Working with file and directory permissions.
- Understanding processes and services.
- Using shell commands and beginning shell scripting.
- Troubleshooting commands that fail because of permissions or incorrect syntax.

### Commands to practice

```bash
pwd
ls -la
cd /path/to/directory
mkdir practice
cp source.txt destination.txt
mv old-name.txt new-name.txt
rm filename.txt
cat filename.txt
less filename.txt
grep "text" filename.txt
find . -name "*.yaml"
ps aux
```

### Permissions and services

```bash
ls -l
chmod 644 file.txt
chmod 755 script.sh
sudo systemctl status docker
sudo systemctl start docker
sudo systemctl stop docker
```

**Learning note:** Be careful with `chmod 777`. It gives every user read, write, and execute permissions. Use the narrowest permissions required instead of making files or directories broadly writable.

## 2. Git and GitHub

### Topics covered

- Initializing a repository and checking repository status.
- Configuring Git username and email.
- Staging, unstaging, and committing changes.
- Working with branches, including renaming `master` to `main`.
- Reviewing differences and commit history.
- Understanding reset, revert, amend, rebase, reflog, merge, remotes, and tags.
- Recognizing and resolving merge/revert conflicts.

### Common commands

```bash
git init
git status
git config user.name "Your Name"
git config user.email "you@example.com"
git branch -M main
git add README.md
git add .
git diff
git diff --staged
git restore --staged README.md
git commit -m "Add learning notes"
git log --oneline
git branch
git remote -v
git push -u origin main
```

### Important concepts

- **Working tree:** Files currently being edited.
- **Staging area:** Changes selected for the next commit.
- **Commit:** A saved point in the repository history.
- **Branch:** A separate line of development.
- **Merge conflict:** Git cannot automatically combine some changes.
- **`git revert`:** Creates a new commit that reverses an earlier commit.
- **`git reset`:** Moves the current branch reference and may change the staging area or working tree, depending on the option.
- **`git reflog`:** Shows recent movements of local Git references and can help recover work.

Before using destructive commands such as `git reset --hard` or `git clean`, check `git status` and make sure any important work is backed up.

## 3. Cloud and AWS

### Topics covered

- Understanding cloud computing and over-provisioning.
- Launching EC2 virtual machines.
- Connecting to an instance using SSH.
- Practicing on Amazon Linux and Ubuntu.
- Understanding that the default SSH username depends on the machine image.

Example SSH pattern:

```bash
ssh -i /path/to/key.pem ec2-user@PUBLIC_IP
```

For many Ubuntu EC2 images, the username is `ubuntu`:

```bash
ssh -i /path/to/key.pem ubuntu@PUBLIC_IP
```

Use the actual key path and public IP/DNS name for your instance. Keep private keys secure and never commit them to GitHub.

## 4. Docker

### Topics covered

- Understanding containers and Docker basics.
- Starting and stopping the Docker service.
- Practicing Docker Compose.
- Understanding multi-stage Docker builds.
- Troubleshooting directory and command syntax issues.

### Commands to practice

```bash
docker --version
docker ps
docker ps -a
docker images
docker logs CONTAINER_ID
docker compose up -d
docker compose down
sudo systemctl status docker
```

- **Image:** The packaged template used to create containers.
- **Container:** A running or stopped instance created from an image.
- **Docker Compose:** A way to define and run an application made of multiple containers.
- **Multi-stage build:** Uses multiple build stages so the final image can contain only the files needed to run the application.

## 5. CI/CD and Jenkins

### Topics covered

- Continuous Integration (CI) fundamentals.
- The purpose of Continuous Delivery/Deployment (CD).
- Jenkins and build automation.
- Freestyle jobs and Pipeline jobs.
- Jenkinsfiles.
- Declarative versus Scripted Pipeline syntax.
- Troubleshooting a Jenkins service on Windows.

### Key concepts

- **CI:** Frequently integrates code changes and runs automated checks.
- **Continuous Delivery:** Keeps software ready for release; production release may require approval.
- **Continuous Deployment:** Automatically releases changes that pass the required pipeline checks.
- **Jenkinsfile:** A text file that defines a Jenkins Pipeline as code.
- **Freestyle job:** A job configured through Jenkins job settings.
- **Pipeline job:** A job that runs a defined sequence of build, test, and delivery stages.

Example Declarative Pipeline structure:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Build step goes here'
            }
        }

        stage('Test') {
            steps {
                echo 'Test step goes here'
            }
        }
    }
}
```

This is a learning example, not a complete application build pipeline.

## 6. Kubernetes

### Topics covered

- Kubernetes cluster basics.
- Control-plane components and worker-node concepts.
- Pods and Deployments.
- Static Pods managed by the kubelet.
- Kubernetes manifests under `/etc/kubernetes/manifests/`.
- Using `kubectl` to inspect cluster resources.
- Using `crictl` to inspect containers managed by the container runtime.
- Troubleshooting API-server and etcd behavior.

### Useful commands

```bash
kubectl get nodes
kubectl get pods -A
kubectl get deployments -A
kubectl get deployment nginx -n default
kubectl get replicasets -n default
kubectl describe pod POD_NAME -n NAMESPACE
kubectl logs POD_NAME -n NAMESPACE
```

If an alias such as `k` is configured, it can be used instead of `kubectl`.

### Control-plane components

- **kube-apiserver:** Exposes the Kubernetes API.
- **etcd:** Stores Kubernetes cluster state.
- **kube-scheduler:** Selects suitable nodes for Pods.
- **kube-controller-manager:** Runs controllers that help move actual state toward desired state.
- **kubelet:** Ensures Pods assigned to a node are running; it also manages static Pods from local manifest files.

### Static Pods and container IDs

A control-plane Pod may be managed directly by the kubelet from a manifest file. For example, the API-server manifest is commonly located at:

```bash
/etc/kubernetes/manifests/kube-apiserver.yaml
```

To find the running API-server container:

```bash
sudo crictl ps --name kube-apiserver
```

To include stopped containers:

```bash
sudo crictl ps -a --name kube-apiserver
```

The container ID can change after a restart. Use the current ID displayed by `crictl`, rather than relying on an old ID.

## 7. etcd Backup and Restore Lab

### Objective

Practice creating an etcd snapshot, restoring it into a separate data directory, and understanding how a static-Pod manifest determines which data directory etcd uses.

### Environment and paths used in the lab

- Kubernetes control-plane node: `controlplane`
- etcd endpoint: `https://127.0.0.1:2379`
- Snapshot file: `/opt/chaithra/madan/backup.db`
- Test restore directory: `/tmp/backup_file`
- Static-Pod manifest: `/etc/kubernetes/manifests/etcd.yaml`

These paths describe this lab environment; they are not universal defaults.

### Step 1: Create a snapshot

The following command was used to save a snapshot:

```bash
sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/chaithra/madan/backup.db
```

The CA certificate verifies the server certificate. The client certificate and matching private key authenticate the request. Keep private keys protected.

### Step 2: Restore to a separate directory

```bash
sudo etcdctl snapshot restore \
  /opt/chaithra/madan/backup.db \
  --data-dir=/tmp/backup_file
```

In this lab, `etcdutl` was not available in the environment, so `etcdctl snapshot restore` was used and printed a deprecation notice recommending `etcdutl`. For a new environment, check the installed etcd version and its supported restore command.

Restoring a snapshot creates data files in the specified directory. It does **not**, by itself, make a running etcd process use that directory.

### Step 3: Understand the etcd static-Pod manifest

The etcd manifest is:

```bash
sudo nano /etc/kubernetes/manifests/etcd.yaml
```

For this lab, the manifest was edited so the etcd container's `--data-dir`, its volume mount, and the corresponding host path pointed to `/tmp/backup_file`.

The relevant configuration must be consistent:

- The etcd `--data-dir` argument points to the restored directory as seen inside the container.
- The container's `volumeMounts` mount the host directory at that same container path.
- The matching `volumes.hostPath.path` points to the intended host directory.

**Important:** The lab intentionally used `/tmp/backup_file` for practice. Do not copy this setting into a real cluster without understanding the persistence and recovery implications. `/tmp` may be cleared on reboot, so it is not a suitable long-term production data location.

### Step 4: Verify etcd health

```bash
sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health
```

Inspect endpoint status:

```bash
sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint status --write-out=table
```

During the exercise, etcd reported healthy and the status output showed a leader.

### Step 5: Check Kubernetes after restore

```bash
kubectl get nodes
kubectl get pods -A
kubectl get deployments -A
kubectl get deployment nginx -n default
kubectl get replicasets -n default
```

Check the API-server container and logs if needed:

```bash
sudo crictl ps --name kube-apiserver
sudo crictl logs CONTAINER_ID
```

Replace `CONTAINER_ID` with the current ID shown by `crictl`.

### What I learned

1. A snapshot is a backup of etcd data at a point in time.
2. A restore writes snapshot data into a data directory; it does not automatically reconfigure a running etcd instance.
3. A static-Pod manifest controls how the kubelet runs etcd on a kubeadm-style control plane.
4. The data directory, volume mount, and host path need to agree.
5. `kubectl`, `crictl`, and `etcdctl` answer different questions:
   - `kubectl` inspects Kubernetes API resources.
   - `crictl` inspects containers through the container runtime.
   - `etcdctl` interacts directly with etcd.
6. Container IDs can change when containers are recreated.
7. After restoring older etcd data, Kubernetes components may show resource-version or cache-consistency errors. Investigate logs and resource state rather than assuming a restart alone fixes the underlying issue.
8. Keep the original etcd data directory and snapshot safe until recovery is verified. Do not delete or overwrite data directories casually.

### Safety reminders

- Take and verify a snapshot before attempting a restore.
- Record the original manifest and data-directory configuration before editing.
- Avoid manually stopping control-plane containers unless you understand the effect and have a recovery plan.
- Do not use `chmod 777` as a general permissions fix.
- Do not delete the original etcd directory as part of an experiment.
- In production, follow the cluster's documented recovery procedure and use persistent storage.

## Troubleshooting Lessons

- **`permission denied`:** Check which user runs the command and whether it can read the file. Use `sudo` only when required.
- **TLS private key does not match certificate:** Ensure the certificate and private key belong together. Do not assume `peer.crt` matches `server.key`; use the correct pair.
- **`unknown command` or argument errors:** Check spaces, command spelling, etcd version, and command syntax.
- **`etcdutl: command not found`:** The utility may not be installed or available in `PATH`; check installed tools and version-specific instructions.
- **`kubectl` connection refused:** The API server may be unavailable. Check the control-plane Pod/container and kubelet before changing cluster state.
- **Unexpected Kubernetes resources or resource-version errors after restore:** Inspect etcd health, API-server logs, and resource lists. Restoring an older snapshot changes the cluster state and can expose consistency issues.
- **Container ID not found:** Container IDs may change after restart. Run `sudo crictl ps` again to find the current ID.

## Next Steps

- [ ] Practice Git branching, merging, revert, and conflict resolution in a test repository.
- [ ] Write small shell scripts and learn exit codes, variables, conditions, loops, and functions.
- [ ] Build a small Docker image and run it locally.
- [ ] Create a multi-container application with Docker Compose.
- [ ] Build a Jenkins Pipeline that checks out code, builds, and tests an application.
- [ ] Learn Terraform by creating and safely destroying a small test resource.
- [ ] Practice Kubernetes Deployments, Services, ConfigMaps, Secrets, and namespaces.
- [ ] Repeat the etcd backup-and-restore exercise in a disposable lab, documenting verification before and after recovery.
- [ ] Add screenshots or sanitized command output to the repository where useful; never commit credentials, private keys, tokens, or sensitive infrastructure details.

---

## Repository Update Workflow

After saving this file as `README.md`:

```bash
git status
git add README.md
git commit -m "Add DevOps learning README"
git push
```

If this is a new local repository with a GitHub remote not yet configured, add the remote URL provided by GitHub and push the `main` branch:

```bash
git remote add origin YOUR_GITHUB_REPOSITORY_URL
git branch -M main
git push -u origin main
```

Replace `YOUR_GITHUB_REPOSITORY_URL` with your own repository URL. Do not publish credentials, private keys, IP addresses you need to keep private, or unredacted logs.
