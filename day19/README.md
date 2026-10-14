# Kubernetes Cluster Setup on Ubuntu (AWS EC2)

This guide documents a **learning/lab setup** for a Kubernetes cluster using `kubeadm`, `kubelet`, `kubectl`, `containerd`, and Calico on Ubuntu EC2 instances.

> **Important:** Verify all versions and commands against the official documentation before using this in production. The versions in the original notes (Kubernetes v1.37, containerd 2.4.2, runc 1.5.2, and Calico v3.33.0) must be checked for availability and compatibility before use.

## Architecture

- **Control-plane node:** runs the Kubernetes API server, scheduler, controller manager, and etcd.
- **Worker node(s):** run application Pods.
- **Container runtime:** containerd.
- **CNI network plugin:** Calico.
- **Bootstrap tools:** kubeadm, kubelet, kubectl, and crictl.

## Prerequisites

- Ubuntu EC2 instances using `amd64` for the commands below.
- A private IP address for each node; instances must communicate over required ports.
- SSH access and a user with `sudo` privileges.
- Suitable EC2 security groups. Restrict SSH (port 22) to your trusted IP instead of exposing it to `0.0.0.0/0`.
- Unique hostnames for all nodes.
- The actual private IP address of the control-plane node.

## 1. Update Ubuntu and install utilities

Run on **every node**:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt-get install -y apt-transport-https ca-certificates curl gpg ncdu net-tools
```

## 2. Disable swap

Kubernetes kubelet normally requires swap to be disabled unless swap support has been explicitly configured.

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
free -h
```

Confirm swap is disabled. The `sed` command comments out matching swap entries in `/etc/fstab`; inspect that file first if your setup differs.

## 3. Load kernel modules and configure networking

Run on **every node**:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

lsmod | grep -E 'br_netfilter|overlay'
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

The three `sysctl` values should report `1`.

## 4. Install containerd

Use the official [Docker Engine installation guide for Ubuntu](https://docs.docker.com/engine/install/ubuntu/) and the [Kubernetes kubeadm installation guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) as primary references.

### Option A — install the packaged containerd runtime

This is usually simpler than manually installing a tarball.

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
sudo apt install -y containerd.io
```

> Correction: use `/etc/apt/keyrings/docker.asc` consistently. The original notes had a typo (`docker.as`).

### Option B — install a specific containerd release manually

Use this only if you intentionally need a specific release and have checked its compatibility and checksums. **Do not mix** manual tarball installation with APT installation on the same node.

Release list: [containerd releases](https://github.com/containerd/containerd/releases)

Example from the original notes; verify that the release asset exists before running:

```bash
curl -LO https://github.com/containerd/containerd/releases/download/v2.4.2/containerd-2.4.2-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-2.4.2-linux-amd64.tar.gz
curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo mkdir -p /etc/containerd
```

Service-file details and runtime paths can vary by release. Follow the selected release's official instructions if they differ.

## 5. Configure containerd for Kubernetes

For the APT-installed package:

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
grep -n -i 'SystemdCgroup\|sandbox_image' /etc/containerd/config.toml
```

Set `SystemdCgroup = true` in the runtime options for the CRI plugin in `/etc/containerd/config.toml`. The exact table path varies between containerd major versions, so inspect the file and follow the documentation for your installed version rather than relying blindly on a text replacement.

Set the sandbox/pause image to a version compatible with your Kubernetes release if required. The original notes used `registry.k8s.io/pause:3.10`; verify the recommended value for your version.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
sudo systemctl restart containerd
systemctl status containerd --no-pager
containerd --version
```

## 6. Install runc and CNI plugins (manual-runtime route only)

An APT-installed `containerd.io` package may already provide runtime components needed for your setup. Check the package contents and official instructions before installing these separately; avoid overwriting package-managed files without a reason.

References: [runc releases](https://github.com/opencontainers/runc/releases) · [CNI plugins releases](https://github.com/containernetworking/plugins/releases)

Commands from the original notes; verify release availability and checksums first:

```bash
curl -LO https://github.com/opencontainers/runc/releases/download/v1.5.2/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc

curl -fLO https://github.com/containernetworking/plugins/releases/download/v1.9.1/cni-plugins-linux-amd64-v1.9.1.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.9.1.tgz
sudo apt-get update
```

## 7. Install kubeadm, kubelet, and kubectl

Choose one Kubernetes minor version and use it consistently across the cluster. Repository paths and package versions must exist and be supported when you install them.

Official instructions: [Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Example using the `v1.37` repository from the original notes — verify that this release and package version are available first:

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.37/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.37/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
apt-cache madison kubeadm
```

Install package versions that are actually listed by `apt-cache madison`. The versions below are from the original notes and may not be available:

```bash
sudo apt-get install -y kubelet=1.37.1-1.1 kubeadm=1.37.1-1.1 kubectl=1.37.1-1.1
sudo apt-mark hold kubelet kubeadm kubectl

kubeadm version
kubelet --version
kubectl version --client
```

If APT cannot find those exact versions, select a version shown by `apt-cache madison` and follow Kubernetes' version-skew policy. Do not force downgrades or package changes without understanding the consequences.

## 8. Configure crictl

Install a `crictl` release compatible with your Kubernetes version. The original notes use `v1.37.0`; verify that the release exists before downloading.

```bash
curl -LO https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.37.0/crictl-v1.37.0-linux-amd64.tar.gz
sudo tar zxvf crictl-v1.37.0-linux-amd64.tar.gz -C /usr/local/bin
sudo crictl config runtime-endpoint unix:///run/containerd/containerd.sock
sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock version
sudo crictl info
```

**Do not run `sudo chmod 666 /run/containerd/containerd.sock`.** This makes the runtime socket writable by every local user, which is unsafe. Use `sudo crictl ...` or a suitably restricted access policy.

## 9. Initialize the control-plane node

Run this section **only on the control-plane node**, after confirming the runtime is running and prerequisites are installed.

Find the actual private IP address:

```bash
ip -4 addr
hostname
```

Replace `CONTROL_PLANE_PRIVATE_IP` below with the real private IP of the control-plane EC2 instance. The original value `172.31.43.1` is only an example and must not be assumed to belong to your instance.

```bash
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=CONTROL_PLANE_PRIVATE_IP \
  --cri-socket unix:///run/containerd/containerd.sock \
  --node-name controlplane
```

Save the `kubeadm join` command printed after initialization for worker nodes. Do not publish its token or discovery hash in GitHub.

Configure kubectl for the current non-root user:

```bash
mkdir -p "$HOME/.kube"
sudo cp -i /etc/kubernetes/admin.conf "$HOME/.kube/config"
sudo chown "$(id -u):$(id -g)" "$HOME/.kube/config"
```

Check status:

```bash
kubectl get nodes
kubectl get pods -A
```

The node may remain `NotReady` until the CNI network plugin is installed.

## 10. Install Calico networking

The pod CIDR passed to `kubeadm init` must match the Calico IP pool configuration. The original notes use `192.168.0.0/16`; verify the selected manifest matches it.

Documentation: [Calico for Kubernetes](https://docs.tigera.io/calico/latest/getting-started/kubernetes/)

The original notes use these manifests. Verify that the paths exist and the selected release supports your Kubernetes version:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.33.0/manifests/v3_projectcalico_org.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.33.0/manifests/tigera-operator.yaml
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.33.0/manifests/custom-resources-bpf.yaml
kubectl create -f custom-resources-bpf.yaml
kubectl get tigerastatus
```

> **Important:** Do not apply multiple Calico installation methods or custom resources blindly. Follow one complete installation path from the official documentation. Confirm that `custom-resources-bpf.yaml` is appropriate for your environment and that its IP pool matches `192.168.0.0/16`. If a manifest is missing or the CRD is not installed, stop and use the current official Calico instructions rather than continuing with a partial installation.

## 11. AWS EC2 security group rules

Use these tables as a review checklist, not a universal firewall recipe. Exact ports depend on Kubernetes and Calico configuration. Allow cluster traffic only from the private CIDR/security group used by your nodes, and do not expose control-plane ports publicly.

### Control-plane node

| Protocol | Port | Suggested source | Purpose |
|---|---:|---|---|
| TCP | 22 | Your trusted public IP/CIDR | SSH administration |
| TCP | 6443 | Cluster node private CIDR or node security group | Kubernetes API server |
| TCP | 2379–2380 | Control-plane nodes only | etcd client/peer communication |
| TCP | 10250 | Cluster nodes as required | Kubelet API |
| TCP | 10257 | Control-plane nodes as required | Controller manager secure port |
| TCP | 10259 | Control-plane nodes as required | Scheduler secure port |
| TCP | 179 | Cluster nodes, if required by Calico BGP mode | Calico BGP |
| TCP | 5473 | Cluster nodes, only if required by chosen Calico mode | Calico Typha |

### Worker node

| Protocol | Port | Suggested source | Purpose |
|---|---:|---|---|
| TCP | 22 | Your trusted public IP/CIDR | SSH administration |
| TCP | 10250 | Cluster nodes | Kubelet API |
| TCP | 30000–32767 | Only required client networks | NodePort services, if used |
| TCP/UDP | 179 and other Calico ports | Cluster nodes, only as required by chosen Calico mode | Calico networking |
| TCP | 10256 | As required by kube-proxy configuration | kube-proxy health checks |

Calico ports vary by networking mode (such as BGP versus VXLAN/IP-in-IP and whether Typha is used). Check [Calico network requirements](https://docs.tigera.io/calico/latest/getting-started/kubernetes/requirements) and [Kubernetes ports and protocols](https://kubernetes.io/docs/reference/networking/ports-and-protocols/) before applying rules. NodePort does not automatically require opening both TCP and UDP; allow only the protocols your services use.

## 12. Join worker nodes

On each worker, install and configure the same compatible container runtime and Kubernetes packages. Then run the **exact** `kubeadm join` command produced by `kubeadm init`, using `sudo`.

Do not put the join token in this README or commit it to GitHub.

On the control-plane node, verify:

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl get tigerastatus
```

## Troubleshooting checklist

- `systemctl status containerd`: verify containerd is active.
- `sudo crictl info`: verify the CRI endpoint works.
- `sudo journalctl -u kubelet -xe`: inspect kubelet errors.
- `kubectl get nodes -o wide`: check node status and IP addresses.
- `kubectl get pods -A`: inspect system Pods.
- Verify swap, kernel modules, sysctl values, private IP addresses, and security-group rules.
- Check Kubernetes, containerd, and Calico version compatibility.

## Security reminders

- Never commit kubeconfig files, certificates, private keys, `admin.conf`, join tokens, or other secrets.
- Do not make the containerd socket world-writable.
- Restrict SSH and the Kubernetes API server to trusted sources.
- Use official release documentation and verify downloaded artifacts/checksums.
- This is a learning guide, not a production-hardening guide.

## References

- [Kubernetes: Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [Kubernetes: Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
- [Kubernetes: Ports and protocols](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)
- [Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [containerd releases](https://github.com/containerd/containerd/releases)
- [runc releases](https://github.com/opencontainers/runc/releases)
- [CNI plugins releases](https://github.com/containernetworking/plugins/releases)
- [Calico for Kubernetes](https://docs.tigera.io/calico/latest/getting-started/kubernetes/)
- [Calico network requirements](https://docs.tigera.io/calico/latest/getting-started/kubernetes/requirements)
