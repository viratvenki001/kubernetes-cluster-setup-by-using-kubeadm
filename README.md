************************************************************************************************

✅ KUBERNETES CLUSTER SETUP 

--Minimum Requirements--

Master Node: At least 2 CPU and 4GB RAM (t2 medium )
Worker Node: At Least 2 CPU and 2GM RAM (t2 micro )
Ubuntu: Version 22.04

--ports to be enable on master and worker nodes--

Master Node
- 6443 (TCP): Kubernetes API Server
- 2379-2380 (TCP): etcd Server
- 10250 (TCP): Kubelet API
- 10251 (TCP): Kube-Scheduler
- 10252 (TCP): Kube-Controller-Manager

Worker Node
- 10250 (TCP): Kubelet API
- 30000-32767 (TCP): NodePort Services (if using NodePort services)

  

✅ PART 1 — Steps and Commands to run on BOTH Master and Worker Nodes
🔹 1. Update your system
sudo apt update && sudo apt upgrade -y

Updates your Linux system to the latest packages so nothing breaks later.

🔹 2. Install some required tools
sudo apt install -y apt-transport-https ca-certificates curl gpg

These tools help your system download Kubernetes packages securely.

🔹 3. Disable SWAP

Kubernetes does NOT work with swap turned on.

sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

✅ Turns off swap now
✅ Makes sure swap stays disabled even after reboot

🔹 4. Enable required kernel modules

These modules help Kubernetes manage network traffic inside the cluster.

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

✅ overlay = helps container filesystem
✅ br_netfilter = helps Kubernetes networking / firewalls

🔹 5. Apply system network settings

These settings allow Kubernetes to route traffic properly.

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system

✅ Allows packets to flow between pods
✅ Enables IP forwarding
✅ Needed for cluster networking

🔹 6. Install Container Runtime (Containerd)

Kubernetes needs a container runtime to run containers (pods).

sudo apt install -y containerd

🔹 7. Configure containerd to use systemd

Kubernetes recommends using systemd for better resource management.

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

✅ Ensures Kubernetes works smoothly with containerd

🔹 8. Add Kubernetes official repository

This tells your system where to download Kubernetes packages from.

sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list

🔹 9. Install Kubernetes tools
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet

✅ kubeadm → Used to create the cluster
✅ kubelet → Runs on every node, manages pods
✅ kubectl → Used to run Kubernetes commands





✅ PART 2 — Steps to run ONLY on MASTER NODE
🔹 1. Initialize the Kubernetes Control Plane
sudo kubeadm init

This sets up the Kubernetes master.
At the end, it will show a kubeadm join command → copy it.
You will run it later on worker nodes.

🔹 2. Set up and install conntarck.
sudo apt update
sudo apt install -y conntrack


🔹 2. Set up kubectl for your user

So you can run commands without using sudo.

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

🔹 3. Check node status
kubectl get nodes

It will show NotReady — this is normal because the pod network is not installed yet.

🔹 4. Install Pod Network (Calico)

Calico allows pods to talk to each other.

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml

✅ After 1–2 minutes:

kubectl get nodes

Nodes will show Ready

✅ Check all pods

kubectl get pods -A





✅ PART 3 — Steps to run ONLY on Worker Nodes
🔹 Join workers to the cluster

Use the command shown earlier during master initialization.

Example:
sudo kubeadm join 192.168.56.10:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>

✅ After this, worker node becomes part of the cluster





✅ PART 4 — Verify everything (On Master)
kubectl get nodes

✅ You should see:

1 Master → Ready

All Worker Nodes → Ready

✅ ✅ Cluster is Successfully Setup 🎉 

*****************************************************************************************
