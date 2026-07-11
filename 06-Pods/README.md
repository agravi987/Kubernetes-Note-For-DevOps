# 📦 Pods

> *"Ravi, this is where the magic starts! Pods are the SMALLEST thing you can create in Kubernetes. Everything else — Deployments, Services, StatefulSets — they all manage Pods. Let's understand them inside out."*

---

## 🤔 What is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes. It's a wrapper around one or more containers that share:

- **Network** — same IP address, same ports
- **Storage** — same volumes
- **Namespace** — same Linux namespace
- **Lifecycle** — they start and stop together

```
┌─────────────────── Pod ───────────────────┐
│                                           │
│  ┌─────────────┐    ┌─────────────┐      │
│  │ Container 1 │    │ Container 2 │      │
│  │   (nginx)   │    │  (sidecar)  │      │
│  └─────────────┘    └─────────────┘      │
│                                           │
│  Shared:                                  │
│  ├── IP: 10.244.0.5                       │
│  ├── Volume: /data                        │
│  └── Port: 80                             │
└───────────────────────────────────────────┘
```

---

## 💡 Why Do We Need Pods?

"Why not just run containers directly?"

1. **Multi-container patterns** — Some apps need tightly coupled containers working together (sidecar pattern, ambassador pattern)
2. **Shared context** — Containers in a pod can share volumes and network
3. **Atomic scheduling** — Kubernetes schedules pods, not individual containers
4. **Co-location** — Related containers run on the same node

> 💡 **Real-world example:** A web server (nginx) + a log collector (Fluentd) in the same pod. The log collector reads nginx logs and sends them to a central system.

---

## 🔄 How It Works

```
1. You create a Pod YAML → kubectl apply
2. API Server stores it in etcd
3. Scheduler assigns it to a node
4. kubelet on that node tells container runtime to start containers
5. Pod gets an IP address
6. kubelet monitors pod health
7. When all containers stop → Pod is done
```

### Pod Lifecycle

```
Pending → Running → Succeeded/Failed
   │        │            │
   │        │            └── All containers exited
   │        └── At least one container running
   └── Waiting to be scheduled (or pulling images)
```

---

## 📝 YAML Example

### Simple Pod (Single Container)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  labels:
    app: web
    env: dev
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "250m"
          memory: "256Mi"
```

### Multi-Container Pod (Sidecar Pattern)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-logger
spec:
  containers:
    # Main container — serves the web app
    - name: web-app
      image: nginx:1.25
      volumeMounts:
        - name: logs
          mountPath: /var/log/nginx

    # Sidecar container — collects logs
    - name: log-collector
      image: busybox
      command: ["sh", "-c", "tail -f /var/log/nginx/access.log"]
      volumeMounts:
        - name: logs
          mountPath: /var/log/nginx

  volumes:
    - name: logs
      emptyDir: {}
```

---

## 🧠 Important Concepts

### Pod IP Address

```
Each Pod gets a unique IP address
├── Pod A → 10.244.0.5
├── Pod B → 10.244.1.3
└── Pod C → 10.244.0.8

Pods can communicate with each other using their IPs
BUT — don't rely on pod IPs! They're ephemeral (can change).
Use Services for stable networking (covered later).
```

### Ephemeral Nature

```
⚠️ Pods are TEMPORARY!

Pod dies → New pod is created (with NEW IP)
Pod moves → New pod on different node (NEW IP)
Pod rescheduled → Completely new pod

This is why we use:
  ├── Deployments (to manage pod replicas)
  ├── Services (for stable networking)
  └── Volumes (for persistent data)
```

> 🎉 **Fun Fact / Joke:** Pods are like pani puri — gone the moment you create them, you can never catch the same one twice, and you need a whole system (Deployment) to keep making new ones before your guests (users) run out! 🥙

### Init Containers

```yaml
# Runs BEFORE the main containers
spec:
  initContainers:
    - name: wait-for-db
      image: busybox
      command: ['sh', '-c', 'until nc -z my-db 3306; do sleep 2; done']

  containers:
    - name: web-app
      image: my-app:1.0
      # Won't start until init container completes
```

> 💡 Init containers are useful for: waiting for dependencies, running setup scripts, checking prerequisites.

### Pod Status

```bash
kubectl get pods
# NAME       READY   STATUS    RESTARTS   AGE
# my-nginx   1/1     Running   0          5m
```

| Status | What It Means |
|--------|--------------|
| **Pending** | Waiting to be scheduled or pulling images |
| **Running** | At least one container is running |
| **Succeeded** | All containers exited successfully (Jobs) |
| **Failed** | At least one container failed |
| **CrashLoopBackOff** | Container keeps crashing and restarting |
| **ImagePullBackOff** | Can't pull the container image |

---

## 📋 Best Practices

- ✅ **Don't create bare Pods** — always use Deployments/ReplicaSets
- ✅ **Set resource requests and limits** on every container
- ✅ **Use liveness and readiness probes** (covered in Probes chapter)
- ✅ **Use meaningful labels** for easy filtering
- ✅ **Keep pods small** — one primary process per container
- ❌ **Don't store data in pods** — they're ephemeral, data will be lost
- ❌ **Don't run multiple unrelated containers** in one pod
- ❌ **Don't rely on Pod IPs** — they change when pods restart
- ❌ **Don't manually create pods in production** — use Deployments

---

## ❌ Common Mistakes

1. **Creating bare Pods (without Deployment)** 🤦
   ```bash
   # ❌ Don't do this in production
   kubectl run my-pod --image=nginx

   # ✅ Do this instead
   kubectl create deployment my-app --image=nginx
   ```

2. **Not setting resource limits** 💸
   > Without limits, one pod can consume ALL node resources, starving other pods.

3. **Using `docker exec` instead of `kubectl exec`** 🤦
   ```bash
   # ❌ Wrong
   docker exec -it <container-id> bash

   # ✅ Right
   kubectl exec -it <pod-name> -- bash
   ```

4. **Ignoring CrashLoopBackOff** 😱
   ```bash
   # Always check WHY it's crashing:
   kubectl describe pod <pod-name>    # Look at Events
   kubectl logs <pod-name>            # Check application logs
   kubectl logs <pod-name> --previous # Logs from crashed container
   ```

5. **Putting more than one unrelated app in a pod** 🤦
   > Pods should contain tightly coupled containers. Don't put nginx and MySQL in the same pod!

---

## 🎤 Interview Questions

1. **What is a Pod?**
   > The smallest deployable unit in Kubernetes. A wrapper around one or more containers that share network, storage, and lifecycle.

2. **Why do we need Pods instead of just running containers?**
   > Pods enable multi-container patterns (sidecar, ambassador), shared context, atomic scheduling, and co-location of tightly coupled containers.

3. **What is a sidecar container?**
   > A helper container that runs alongside the main container in a pod. Common use cases: log collection, monitoring agents, proxies.

4. **What happens to a Pod's IP when it dies?**
   > It's lost forever. New pod gets a new IP. Use Services for stable networking.

5. **Why shouldn't you create bare Pods in production?**
   > Bare Pods don't self-heal. If the node dies, the pod is gone forever. Deployments ensure pod replacement and desired replica count.

6. **What are Init Containers?**
   > Containers that run to completion BEFORE the main containers start. Used for setup tasks like waiting for dependencies.

7. **What does CrashLoopBackOff mean?**
   > The container is repeatedly crashing and being restarted by kubelet. The backoff period increases with each crash (10s, 20s, 40s, up to 5min).

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **Pod** | Smallest K8s unit, wraps 1+ containers |
| **Multi-container** | Sidecar, Ambassador, Adapter patterns |
| **Ephemeral** | Pods die → new pod, new IP |
| **Init Containers** | Run before main containers |
| **Status** | Pending → Running → Succeeded/Failed |
| **Never** | Create bare Pods in production |

> *"Ravi, Pods are the atoms of Kubernetes. Everything is built on top of them. You don't manage atoms directly though — you use Deployments and ReplicaSets to manage Pods for you. That's coming up next! 🔜"*
