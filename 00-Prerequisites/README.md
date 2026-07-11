# 🧰 Prerequisites

> *"Hey Ravi, before we jump into Kubernetes, let's make sure our basics are solid. Think of this as stretching before a workout — skip it and you'll pull a muscle later!"*

---

## 🖥️ Linux / Terminal Basics

You don't need to be a Linux wizard, but you MUST be comfortable with:

| Command | What It Does |
|---------|-------------|
| `ls` | List files in a directory |
| `cd` | Change directory |
| `pwd` | Print current working directory |
| `cat` | Display file contents |
| `grep` | Search text in files |
| `curl` | Make HTTP requests from terminal |
| `ssh` | Remote login to a server |
| `sudo` | Run commands as superuser |
| `chmod` | Change file permissions |

```bash
# Quick practice, Ravi:
pwd
ls -la
cat /etc/os-release
```

> 💡 **Tip:** Most of your Kubernetes work happens in the terminal. Get comfy with it!

---

## 📝 YAML

Kubernetes uses YAML to define everything — pods, services, deployments, all of it.

### Basic YAML Rules

```yaml
# This is a comment
name: my-app          # String value
replicas: 3           # Integer value
enabled: true         # Boolean value

# Lists
containers:
  - nginx
  - redis

# Nested objects
metadata:
  name: my-pod
  labels:
    app: web
```

### YAML Gotchas ⚠️

- **Indentation matters!** Use spaces, NOT tabs
- **2 spaces** is the standard indent (not 4, not tabs)
- Watch out for trailing whitespace — it can break things silently

```yaml
# ✅ Correct
metadata:
  name: my-pod

# ❌ Wrong (used tab)
metadata:
	name: my-pod

# ❌ Wrong (inconsistent spacing)
metadata:
  name: my-pod
   labels:       # extra space = BROKEN
    app: web
```

> 🔥 **Ravi, remember:** 90% of Kubernetes bugs are YAML indentation issues. Always double-check!

---

## 🌐 Basic Networking Concepts

You don't need a networking degree, but know these:

### IP Addresses
```
Every device on a network gets an IP address.
Think of it like a home address — how computers find each other.

Example: 192.168.1.10
```

### Ports
```
A port is like a door to a specific service on a machine.

Example: 
  - Web server runs on port 80 (HTTP) or 443 (HTTPS)
  - SSH runs on port 22
  - Database (MySQL) runs on port 3306
```

### DNS (Domain Name System)
```
Translates domain names to IP addresses.

google.com → 142.250.74.110

In Kubernetes, services get DNS names automatically!
```

### HTTP Basics
```
GET    → Fetch data
POST   → Create something
PUT    → Update something
DELETE → Remove something
```

---

## 🐳 Docker Basics

Kubernetes orchestrates containers, so you should know what containers are:

```bash
# Run a container
docker run nginx

# List running containers
docker ps

# Stop a container
docker stop <container_id>

# Build an image from Dockerfile
docker build -t my-app:1.0 .
```

### Key Docker Concepts

| Concept | What It Is | Analogy |
|---------|-----------|---------|
| **Image** | Blueprint to create a container | A recipe |
| **Container** | Running instance of an image | The dish made from the recipe |
| **Dockerfile** | Instructions to build an image | Written recipe |
| **Registry** | Where images are stored | A cookbook library (like Docker Hub) |

```dockerfile
# Simple Dockerfile example
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]
```

> 💡 **Ravi, don't worry!** You don't need to master Docker before learning Kubernetes. Just understand the basics — images, containers, and Dockerfiles.

---

## 🔧 Tools You Need Installed

Before starting Kubernetes, install these on your machine:

### 1. kubectl (Kubernetes CLI)
```bash
# macOS
brew install kubectl

# Windows
choco install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

### 2. A Kubernetes Cluster (for practice)
```bash
# Option 1: Minikube (local single-node cluster)
# macOS
brew install minikube
minikube start

# Option 2: kind (Kubernetes in Docker) — lightweight!
brew install kind
kind create cluster

# Option 3: Docker Desktop (has built-in Kubernetes)
# Just enable it in Settings → Kubernetes → Enable Kubernetes
```

### 3. A Text Editor
- **VS Code** (recommended) with the YAML extension
- Or any editor you like

---

## 💡 Conceptual Prerequisites

### What is a Server?
```
A computer that runs services (apps) for other computers to use.
Can be physical or virtual (cloud VM).
```

### What is a VM (Virtual Machine)?
```
A software-based computer running inside another computer.
Like a computer within a computer.
Cloud providers (AWS, Azure, GCP) give you VMs.
```

### What is a Container?
```
A lightweight, isolated way to run an application.
Shares the host OS kernel (unlike VMs).
Starts in seconds (unlike VMs which take minutes).
This is what Kubernetes manages!
```

### Containers vs VMs

```
┌─────────────────────────────┐
│         VMs                 │
│  ┌───────┐  ┌───────┐      │
│  │App 1  │  │App 2  │      │
│  │Bloat │  │Bloat │      │
│  │OS    │  │OS    │      │
│  └───────┘  └───────┘      │
│     Hypervisor              │
│     Host OS                 │
└─────────────────────────────┘

┌─────────────────────────────┐
│      Containers             │
│  ┌───────┐  ┌───────┐      │
│  │App 1  │  │App 2  │      │
│  └───────┘  └───────┘      │
│     Container Runtime       │
│     Host OS (shared)        │
└─────────────────────────────┘
```

> 🔑 **Key takeaway:** Containers are lighter, faster, and that's why Kubernetes exists — to manage thousands of them!

---

## ✅ Checklist Before Proceeding

Ravi, make sure you can do all of these before moving on:

- [ ] Open a terminal and run basic Linux commands
- [ ] Write a simple YAML file without errors
- [ ] Explain the difference between a VM and a container
- [ ] Run a Docker container (`docker run hello-world`)
- [ ] Install `kubectl` and verify (`kubectl version --client`)
- [ ] Have a Kubernetes cluster running (Minikube / kind / Docker Desktop)

---

## 🎯 Summary

| Prerequisite | Why It Matters |
|-------------|---------------|
| **Linux Basics** | You'll live in the terminal |
| **YAML** | Every K8s resource is defined in YAML |
| **Networking** | Pods talk to each other via networking |
| **Docker** | K8s orchestrates containers |
| **Tools** | kubectl + a cluster = hands-on practice |

> *"Ravi, if you've got these basics down, you're ready to dive into Kubernetes. Let's go! 🚀"*
