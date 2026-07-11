# 🏗️ Architecture

> *"Ravi, this is the BIG one! Understanding Kubernetes architecture is like understanding how a city works. Once you get this, everything else falls into place. Let's break it down piece by piece."*

<div align="center">

| 📖 Reading Time | 🎯 Difficulty | 🏷️ Category |
|:---:|:---:|:---:|
| ~15 min | ⭐⭐ Intermediate | Foundation |

</div>

---

## 🤔 What is it?

Kubernetes architecture defines **how all the components work together** to manage your containers. It's divided into two main parts:

```
┌─────────────────────────────────────────┐
│           Kubernetes Cluster            │
│                                         │
│  ┌─────────────┐  ┌─────────────────┐  │
│  │ Control Plane│  │   Worker Node   │  │
│  │ (Brain) 🧠  │  │   (Muscle) 💪   │  │
│  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────┘
```

---

## 💡 Why Do We Need This Knowledge?

> "Ravi, you wouldn't drive a car without knowing where the steering wheel and brakes are, right?"

- 🐛 **Debugging** — 90% of issues are easier to debug when you understand architecture
- 🎤 **Interviews** — Architecture questions are almost GUARANTEED in any K8s interview
- 🏗️ **Production** — You need to know what to monitor and what can fail
- 🧠 **Mental Model** — Everything we learn later (Pods, Services, etc.) makes more sense

---

## 🧠 Control Plane (Master Node)

The Control Plane is the **brain** of the cluster. It makes all the decisions: what to run, where to run it, and what happens when something fails.

```
┌─────────────────────────────────────────────┐
│            CONTROL PLANE                    │
│                                             │
│  ┌─────────────┐    ┌───────────────────┐  │
│  │ API Server  │◄──►│      etcd         │  │
│  │  (Front Door)│   │  (Database) 💾    │  │
│  └──────┬──────┘    └───────────────────┘  │
│         │                                   │
│  ┌──────┴──────┐    ┌───────────────────┐  │
│  │  Scheduler  │    │ Controller Manager │  │
│  │  (Planner) 📋│   │  (Manager) 🔄     │  │
│  └─────────────┘    └───────────────────┘  │
└─────────────────────────────────────────────┘
```

### 1. 🚪 API Server (kube-apiserver)

**The front door of the cluster.** EVERYTHING goes through the API Server.

```
You (kubectl) ──→ API Server ──→ etcd
                     │
                     ├──→ Scheduler
                     └──→ Controller Manager
```

- **Handles all REST operations** (create, read, update, delete)
- **Validates** every request before processing
- **Serves** as the gateway between components
- **Authenticates & authorizes** (who can do what)

> 💡 Think of it as the **receptionist** of a company — nothing happens without going through them first!

---

### 2. 💾 etcd

**The cluster's database.** Stores ALL cluster data.

```
What etcd stores:
├── Cluster state (what nodes exist, their health)
├── All resources (pods, services, deployments, etc.)
├── Secrets & ConfigMaps
├── RBAC policies
└── Everything!
```

- **Key-value store** (like a dictionary/hashmap)
- **Highly consistent** (uses Raft consensus algorithm)
- **Distributed** (can run multiple replicas for HA)
- **The single source of truth** for the cluster

> ⚠️ **If etcd dies, the cluster is blind!** Always back up etcd in production.

> 😄 **Joke:** etcd is like that one relative who remembers EVERY argument from the last 15 family gatherings — every secret, every config, every mistake you made. Lose etcd and you're basically starting the family from scratch. 💾😅

---

### 3. 📋 Scheduler (kube-scheduler)

**Decides WHERE to run new pods.** It watches for unassigned pods and assigns them to nodes.

```
New Pod Created → Scheduler checks:
  ├── Which nodes have enough resources?
  ├── Any taints/tolerations to consider?
  ├── Any node affinity rules?
  └── Picks the BEST node → Assigns pod to it
```

**The scheduler considers:**
- CPU & Memory requirements
- Node labels and selectors
- Taints and tolerations
- Pod affinity/anti-affinity rules
- Data locality

> 💡 The scheduler **only assigns** pods to nodes. It doesn't actually start them — that's the kubelet's job!

---

### 4. 🔄 Controller Manager (kube-controller-manager)

**The manager that watches and fixes.** It runs background loops to ensure the actual state matches your desired state.

```
You say: "I want 3 replicas of nginx"
         (Desired State)

Reality: Only 2 are running
         (Actual State)

Controller Manager: "Let me fix that!" → Creates 1 more pod
```

**Key Controllers inside:**

| Controller | What It Does |
|-----------|-------------|
| **ReplicaSet** | Ensures desired number of pod replicas exist |
| **Deployment** | Manages ReplicaSets and rolling updates |
| **Node** | Monitors node health, marks nodes as ready/not-ready |
| **Job** | Ensures jobs complete successfully |
| **ServiceAccount** | Creates default service accounts |
| **Namespace** | Handles namespace creation and finalization |

```
The Control Loop (Reconciliation Loop):

while True:
    actual_state = get_current_state()
    desired_state = get_desired_state()
    if actual_state != desired_state:
        take_action_to_fix()
```

---

## 💪 Worker Node

Worker Nodes are where your **actual applications run**. Each node runs the containers (pods) that you deploy.

```
┌─────────────────────────────────────────────┐
│              WORKER NODE                    │
│                                             │
│  ┌──────────┐  ┌───────────┐  ┌─────────┐ │
│  │ kubelet  │  │ kube-proxy │  │ Container│ │
│  │ (Agent)  │  │ (Network)  │  │ Runtime  │ │
│  └────┬─────┘  └─────┬─────┘  └────┬────┘ │
│       │               │              │      │
│  ┌────┴───────────────┴──────────────┴────┐ │
│  │           Pods running here            │ │
│  │  ┌─────┐  ┌─────┐  ┌─────┐           │ │
│  │  │Pod 1│  │Pod 2│  │Pod 3│           │ │
│  │  └─────┘  └─────┘  └─────┘           │ │
│  └────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

### 1. 🤖 kubelet

**The agent on each node.** It communicates with the Control Plane and manages pods.

- Watches the API Server for new pod assignments
- **Starts/stops containers** as instructed
- **Reports node status** back to the Control Plane
- **Runs health checks** (liveness & readiness probes)
- **Collects resource usage** metrics

```
API Server says: "Run pod X with nginx"
        ↓
   kubelet says: "Got it!"
        ↓
   kubelet tells container runtime: "Start nginx container"
        ↓
   kubelet reports back: "Pod X is running! ✅"
```

---

### 2. 🌐 kube-proxy

**Handles networking.** Makes sure pods can communicate with each other and with the outside world.

- Maintains **network rules** on each node
- Routes traffic to the correct pods
- Implements **Services** abstraction (ClusterIP, NodePort, LoadBalancer)
- Uses **iptables** or **IPVS** rules

```
User Request → Service (ClusterIP) → kube-proxy rules → Pod

Example:
Request to 10.96.0.100:80 
  → iptables rule says "forward to pod at 10.244.1.5:80"
  → Request reaches the correct pod
```

---

### 3. 🐳 Container Runtime

**Actually runs the containers.** Kubernetes doesn't run containers directly — it uses a runtime.

| Runtime | Description |
|---------|-------------|
| **containerd** | Most common, CNCF project, used by Docker |
| **CRI-O** | Lightweight, built specifically for Kubernetes |
| **Docker** (deprecated) | Was the original, now replaced by containerd |

```
kubelet → Container Runtime Interface (CRI) → containerd → Container
```

> 💡 You don't usually interact with the container runtime directly. kubelet handles it for you!

---

## 🔄 Full Flow: Deploying an Application

Let's trace what happens when you run `kubectl apply -f deployment.yaml`:

```
Step 1: You run kubectl apply
        ↓
Step 2: kubectl sends request to API Server
        ↓
Step 3: API Server validates & stores in etcd
        ↓
Step 4: Controller Manager detects new Deployment
        ↓
Step 5: Controller Manager creates a ReplicaSet
        ↓
Step 6: Controller Manager creates Pods (desired count)
        ↓
Step 7: Scheduler assigns pods to nodes
        ↓
Step 8: kubelet on each node starts the pods
        ↓
Step 9: kube-proxy configures networking
        ↓
Step 10: Application is running! 🎉
```

---

## 🖥️ Complete Architecture Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                     KUBERNETES CLUSTER                       │
│                                                              │
│  ┌───────────────────── CONTROL PLANE ─────────────────────┐ │
│  │                                                         │ │
│  │  ┌─────────────┐          ┌──────────┐                 │ │
│  │  │ API Server  │◄────────►│   etcd   │                 │ │
│  │  │  :6443      │          │  💾 DB   │                 │ │
│  │  └──────┬──────┘          └──────────┘                 │ │
│  │         │                                               │ │
│  │  ┌──────┴──────┐          ┌──────────────────────┐     │ │
│  │  │  Scheduler  │          │ Controller Manager   │     │ │
│  │  │  📋         │          │  🔄                  │     │ │
│  │  └─────────────┘          └──────────────────────┘     │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────── WORKER NODE 1 ────────────────────────┐ │
│  │  ┌──────────┐  ┌───────────┐  ┌──────────────┐        │ │
│  │  │ kubelet  │  │ kube-proxy │  │ containerd   │        │ │
│  │  │          │  │           │  │ (Runtime)     │        │ │
│  │  └────┬─────┘  └───────────┘  └──────┬───────┘        │ │
│  │       │                               │                 │ │
│  │  ┌────┴───────────────────────────────┴──────────────┐ │ │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐          │ │ │
│  │  │  │  Pod A  │  │  Pod B  │  │  Pod C  │          │ │ │
│  │  │  │ ┌─────┐ │  │ ┌─────┐ │  │ ┌─────┐ │          │ │ │
│  │  │  │ │App  │ │  │ │App  │ │  │ │App  │ │          │ │ │
│  │  │  │ └─────┘ │  │ └─────┘ │  │ └─────┘ │          │ │ │
│  │  │  └─────────┘  └─────────┘  └─────────┘          │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────── WORKER NODE 2 ────────────────────────┐ │
│  │  ┌──────────┐  ┌───────────┐  ┌──────────────┐        │ │
│  │  │ kubelet  │  │ kube-proxy │  │ containerd   │        │ │
│  │  └────┬─────┘  └───────────┘  └──────┬───────┘        │ │
│  │       │                               │                 │ │
│  │  ┌────┴───────────────────────────────┴──────────────┐ │ │
│  │  │  ┌─────────┐  ┌─────────┐                        │ │ │
│  │  │  │  Pod D  │  │  Pod E  │                        │ │ │
│  │  │  │ ┌─────┐ │  │ ┌─────┐ │                        │ │ │
│  │  │  │ │App  │ │  │ │App  │ │                        │ │ │
│  │  │  │ └─────┘ │  │ └─────┘ │                        │ │ │
│  │  │  └─────────┘  └─────────┘                        │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

---

## 🧠 Key Concepts

### Control Loop (The Heart of Kubernetes)

```
Desired State (YAML)  ──→  Compare  ──→  Actual State
                              │
                         Different? 
                           │    │
                          Yes   No
                           │    │
                     Take Action  Do nothing
```

This loop runs **constantly** for every resource. It's what makes Kubernetes "self-healing."

### Communication Flow

```
kubectl ──→ API Server ──→ etcd (store)
                │
                ├──→ Scheduler (assigns pods to nodes)
                │
                └──→ Controller Manager (ensures desired state)
                          │
                    kubelet (on each node)
                          │
                    container runtime (starts containers)
```

---

## 📋 Best Practices

- ✅ **Run Control Plane on dedicated nodes** in production
- ✅ **Use multiple etcd replicas** (3 or 5) for high availability
- ✅ **Monitor all components** — API server, etcd, kubelet
- ✅ **Keep cluster version updated** (but test first!)
- ✅ **Use at least 3 worker nodes** for production
- ❌ **Don't run applications on Control Plane nodes** in production
- ❌ **Don't use a single etcd instance** in production
- ❌ **Don't ignore Control Plane component alerts**

---

## ❌ Common Mistakes

1. **Confusing Control Plane and Worker Nodes** 🤦
   > Control Plane = brain (makes decisions)
   > Worker Node = muscle (runs your apps)

2. **Thinking kubelet runs containers directly** 🤦
   > kubelet tells the container runtime (containerd) to run containers

3. **Thinking etcd is optional** 🤦
   > etcd is the ONLY data store. Without it, the cluster is useless.

4. **Forgetting that the scheduler only assigns** 🤦
   > Scheduler says "run this pod on node X" — kubelet actually starts it

5. **Running apps on Control Plane in production** 🤦
   > In learning: fine. In production: NEVER. Control Plane should be dedicated.

---

## 🎤 Interview Questions

1. **What are the main components of the Kubernetes Control Plane?**
   > API Server, etcd, Scheduler, and Controller Manager.

2. **What does the API Server do?**
   > It's the front door for all communication. It handles REST operations, validates requests, authenticates/authorizes users, and stores state in etcd.

3. **What is etcd?**
   > A distributed key-value store that holds all cluster data. It's the single source of truth for the cluster.

4. **What does the Scheduler do?**
   > Watches for unassigned pods and assigns them to nodes based on resource requirements, affinity rules, taints/tolerations, and other constraints.

5. **What is the Controller Manager?**
   > Runs controller loops that ensure the actual state of resources matches the desired state defined in your configurations.

6. **What components run on a Worker Node?**
   > kubelet (agent), kube-proxy (networking), and container runtime (containerd).

7. **What does kubelet do?**
   > An agent that runs on each node. It watches the API Server for pod assignments, starts/stops containers, runs health checks, and reports node status.

8. **What does kube-proxy do?**
   > Maintains network rules on each node that enable Service-based networking. Routes traffic to the correct pods.

9. **What is the reconciliation loop?**
   > The core mechanism where controllers constantly compare desired state (from your YAML) with actual state (in the cluster) and take action to fix any differences.

10. **What is the difference between Control Plane and Worker Node?**
    > Control Plane runs cluster management components (API Server, etcd, Scheduler, Controller Manager). Worker Nodes run the actual application workloads (pods) via kubelet and kube-proxy.

---

## 📝 Summary

| Component | Runs On | Purpose |
|-----------|---------|---------|
| **API Server** | Control Plane | Front door — handles all requests |
| **etcd** | Control Plane | Database — stores all cluster data |
| **Scheduler** | Control Plane | Assigns pods to nodes |
| **Controller Manager** | Control Plane | Ensures desired state = actual state |
| **kubelet** | Worker Node | Manages pods on the node |
| **kube-proxy** | Worker Node | Handles networking rules |
| **Container Runtime** | Worker Node | Actually runs containers |

> *"Ravi, this is the most important chapter for interviews. Read it twice, draw the diagram yourself, and be able to explain each component. You'll thank yourself later! 🙏"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 03: kubectl](../03-kubectl/README.md)** | [📚 Home](../README.md) | **[Next: 05: Labels & Selectors →](../05-Labels-and-Selectors/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
