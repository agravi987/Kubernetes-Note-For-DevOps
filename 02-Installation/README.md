# ⚙️ Installation

> *"Alright Ravi, time to get our hands dirty! Let's set up Kubernetes on your machine. Don't worry — we'll start simple and work our way up."*

---

## 🤔 What is it?

Installation means setting up a **Kubernetes cluster** — a group of machines (nodes) that work together to run your containerized applications. For learning, we'll set up a **local cluster** on your own machine.

---

## 💡 Why Do We Need It?

You can't learn Kubernetes without a cluster to play with! It's like trying to learn swimming without water 🏊. You need a working cluster to:

- Run `kubectl` commands
- Deploy pods and services
- Practice debugging
- Experiment safely without affecting production

---

## 🔧 How It Works

Kubernetes can run in many environments:

```
Your Laptop (Learning)
    ↓
Single Node Cluster (Minikube/kind)
    ↓
Multi-Node Cluster (kubeadm)
    ↓
Managed Cluster (EKS/GKE/AKS)
    ↓
Production (Thousands of nodes!)
```

---

## 🛠️ Installation Options

### Option 1: Minikube (⭐ Recommended for Beginners)

Minikube creates a **single-node cluster** inside a VM on your machine.

```bash
# --- macOS ---
brew install minikube

# --- Windows (PowerShell as Admin) ---
choco install minikube

# --- Linux ---
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start the cluster!
minikube start

# Verify it's running
minikube status
```

**Use Minikube when:**
- ✅ You're just starting out
- ✅ You want a full cluster experience
- ✅ You want to try different Kubernetes features

---

### Option 2: kind (Kubernetes in Docker) ⚡

kind runs Kubernetes **inside Docker containers**. Super lightweight!

```bash
# --- macOS ---
brew install kind

# --- Windows ---
choco install kind

# --- Linux ---
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create a cluster
kind create cluster

# Verify
kubectl cluster-info
```

**Use kind when:**
- ✅ You want something fast and lightweight
- ✅ You're running Docker already
- ✅ You want to test multi-node clusters easily

```bash
# Multi-node cluster with kind (advanced)
kind create cluster --name multi-node --config - <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF
```

---

### Option 3: Docker Desktop 🐳

The easiest option if you already use Docker Desktop!

```
1. Open Docker Desktop
2. Go to Settings ⚙️
3. Click "Kubernetes" in the left sidebar
4. Check "Enable Kubernetes"
5. Click "Apply & Restart"
6. Wait 2-3 minutes ☕
7. Done! ✅
```

Verify:
```bash
kubectl get nodes
kubectl cluster-info
```

**Use Docker Desktop when:**
- ✅ You already have Docker Desktop installed
- ✅ You want the simplest setup possible
- ✅ You don't want to manage a VM

---

### Option 4: kubeadm (Production-like) 🏭

kubeadm sets up a **real multi-node cluster**. This is closer to production.

```bash
# On the master node:
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Set up kubectl access:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install a network plugin (e.g., Flannel):
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# Join worker nodes (use the token from kubeadm init output):
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

**Use kubeadm when:**
- ✅ You want to understand how production clusters are built
- ✅ You're practicing for CKA/CKS exams
- ✅ You have multiple machines (VMs) available

---

## 📊 Comparison Table

| Feature | Minikube | kind | Docker Desktop | kubeadm |
|---------|----------|------|---------------|---------|
| **Difficulty** | Easy | Easy | Easiest | Hard |
| **Speed** | Medium | Fast ⚡ | Medium | Slow |
| **Multi-node** | No | Yes ✅ | No | Yes ✅ |
| **Realistic** | Medium | Medium | Low | High 🏭 |
| **Best for** | Learning | Testing | Quick start | Production-like |
| **Requires** | VM/Hypervisor | Docker | Docker Desktop | Multiple VMs |

---

## ✅ Verify Your Installation

After installing, run these commands to make sure everything works:

```bash
# Check cluster info
kubectl cluster-info

# Check nodes (should show 1 node in "Ready" state)
kubectl get nodes

# Check system pods (should all be Running)
kubectl get pods -n kube-system

# Run a test deployment
kubectl create deployment hello-nginx --image=nginx

# Check the pod
kubectl get pods

# Clean up
kubectl delete deployment hello-nginx
```

Expected output for `kubectl get nodes`:
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   5m    v1.28.3
```

> 💡 **Ravi, if you see "Ready" status, you're good to go!** 🎉

---

## 🔄 How It Works (Behind the Scenes)

```
┌─────────────────────────────────────────┐
│           Your Machine                  │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │      Minikube / kind VM         │   │
│  │                                 │   │
│  │  ┌───────────────────────────┐  │   │
│  │  │    Kubernetes Cluster     │  │   │
│  │  │                           │  │   │
│  │  │  Control Plane Components │  │   │
│  │  │  ├── API Server           │  │   │
│  │  │  ├── etcd                 │  │   │
│  │  │  ├── Scheduler            │  │   │
│  │  │  └── Controller Manager   │  │   │
│  │  │                           │  │   │
│  │  │  Worker Node              │  │   │
│  │  │  ├── kubelet              │  │   │
│  │  │  ├── kube-proxy           │  │   │
│  │  │  └── Container Runtime    │  │   │
│  │  └───────────────────────────┘  │   │
│  └─────────────────────────────────┘   │
│                                         │
│  kubectl ←── talks to ──→ API Server   │
└─────────────────────────────────────────┘
```

> 🎉 **Fun Fact / Joke:** Installing Kubernetes for the first time is like making momos at home — you watch YouTube tutorials, the first batch looks terrible, the filling spills everywhere, but somehow by the 3rd try, you've got something that works! 🥟

---

## 🧠 Key Components

When you install Kubernetes, these components get set up automatically:

| Component | Role |
|-----------|------|
| **API Server** | The front door — all communication goes through it |
| **etcd** | Database that stores all cluster data |
| **Scheduler** | Decides which node runs a new pod |
| **Controller Manager** | Makes sure actual state matches desired state |
| **kubelet** | Agent on each node that manages pods |
| **kube-proxy** | Handles networking between pods |

> 🔍 Don't worry about memorizing these now — we'll deep-dive in the Architecture chapter!

---

## 📋 Best Practices

- ✅ **Start with Minikube or kind** — don't jump to kubeadm
- ✅ **Use Docker Desktop** if you already have it installed
- ✅ **Delete clusters you're not using** (`minikube delete`, `kind delete cluster`)
- ✅ **Allocate enough resources** — at least 2GB RAM and 2 CPUs for Minikube
- ✅ **Keep kubectl version close to cluster version** (within 1 minor version)
- ❌ **Don't skip verification** — always test with a test deployment
- ❌ **Don't install multiple clusters** on the same port — they'll conflict

---

## ❌ Common Mistakes

1. **Not enough resources allocated** 💻
   ```bash
   # Minikube with more resources:
   minikube start --cpus 4 --memory 8192
   ```

2. **Forgetting to start the cluster** 😅
   ```bash
   # Always check first:
   minikube status
   kubectl get nodes
   ```

3. **kubectl version mismatch** ⚠️
   ```bash
   # Check versions:
   kubectl version --client
   minikube version
   ```

4. **Not cleaning up** 🧹
   ```bash
   # When done practicing:
   minikube delete
   kind delete cluster
   ```

5. **Using sudo with kubectl unnecessarily** 🙅
   > If you set up kubeconfig correctly, you don't need sudo!

---

## 🎤 Interview Questions

1. **What are the different ways to set up a Kubernetes cluster?**
   > Minikube, kind, kubeadm, managed services (EKS, GKE, AKS), and Docker Desktop.

2. **What is Minikube?**
   > A tool that runs a single-node Kubernetes cluster locally inside a VM or container for development and learning purposes.

3. **What is kind?**
   > "Kubernetes in Docker" — runs Kubernetes nodes as Docker containers. Great for testing and CI/CD.

4. **What is kubeadm?**
   > A Kubernetes community tool for bootstrapping a production-grade cluster. It sets up the control plane and joins worker nodes.

5. **What is a managed Kubernetes service?**
   > Cloud provider services like AWS EKS, Google GKE, or Azure AKS where the control plane is managed for you. You only manage worker nodes.

6. **How do you verify a Kubernetes cluster is working?**
   > `kubectl get nodes` — if all nodes show "Ready" status, the cluster is healthy.

---

## 📝 Summary

| Option | When to Use |
|--------|------------|
| **Minikube** | Best for learning, full-featured single node |
| **kind** | Fast, lightweight, great for testing |
| **Docker Desktop** | Easiest setup if you already use Docker |
| **kubeadm** | Production-like, multi-node, CKA practice |
| **Managed (EKS/GKE/AKS)** | Real production workloads |

> *"Ravi, for now just pick Minikube or Docker Desktop and get a cluster running. We can always move to more advanced setups later. The important thing is to START! 🚀"*
