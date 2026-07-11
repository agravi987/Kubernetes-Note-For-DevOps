# 📊 Resource Management

> *"Ravi, imagine a shared office with no assigned desks. One person takes ALL the chairs, and nobody else can sit. That's what happens without resource management! Kubernetes lets you set limits so every pod gets what it needs and nobody hogs everything! 📊"*

---

## 🤔 What is Resource Management?

Resource management in Kubernetes means setting **CPU and memory** requirements and limits for containers.

```
Two concepts:
├── Requests  → "I NEED at least this much" (guaranteed minimum)
└── Limits    → "I CAN'T use more than this" (hard ceiling)
```

---

## 💡 Why Do We Need It?

| Without Resources | With Resources |
|------------------|---------------|
| One pod uses ALL CPU 😱 | Fair distribution ⚖️ |
| Noisy neighbor problem 🤦 | Predictable performance 🎯 |
| OOM kills are random 😰 | Controlled memory limits 🛡️ |
| Can't schedule pods properly | Scheduler knows resource needs 📋 |

---

## 📝 YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          resources:
            requests:
              cpu: "100m"       # 0.1 CPU core (100 millicores)
              memory: "128Mi"   # 128 Mebibytes
            limits:
              cpu: "500m"       # Max 0.5 CPU core
              memory: "256Mi"   # Max 256 MiB
```

---

## 🧠 Important Concepts

### CPU

```
CPU is measured in "millicores" (m):
  1 CPU core = 1000m
  500m = half a core
  100m = 0.1 of a core

requests.cpu: "200m"  → Guaranteed 200 millicores
limits.cpu: "1000m"   → Can burst up to 1 full core
```

**CPU is compressible:**
- If a container exceeds its CPU limit → it gets **throttled** (slowed down), NOT killed
- If a node runs out of CPU → pods may be evicted

### Memory

```
Memory is measured in bytes:
  128Mi = 128 Mebibytes = ~134 MB
  1Gi   = 1 Gibibyte   = ~1.07 GB

requests.memory: "128Mi"  → Guaranteed 128 MiB
limits.memory: "256Mi"    → OOM killed if exceeds 256 MiB
```

**Memory is NOT compressible:**
- If a container exceeds its memory limit → **OOMKilled** (process terminated!)
- This is why setting memory limits carefully is CRITICAL

### Requests vs Limits

```
                    Requests        Limits
                    ─────────       ──────
What:               Minimum         Maximum
Guaranteed?         ✅ Yes          ❌ No
Can burst beyond?   N/A             ✅ Yes (CPU only)
Exceed limit?       N/A             CPU: throttled
                                    Memory: OOMKilled
Scheduled based on? ✅ Yes          ❌ No
```

> 🔑 **Key insight, Ravi:** Kubernetes schedules pods based on **requests**, NOT limits. If you set high limits but low requests, Kubernetes might pack too many pods on one node!

> 📱💸 **Joke:** Not setting resource limits is like letting your cousin use your phone 'for just one game' — 3 hours later, your storage is full, your battery is dead, and somehow your Jio data is finished. SET THOSE LIMITS!

---

## 📚 ResourceQuota and LimitRange

### ResourceQuota (Per Namespace)

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    requests.cpu: "4"          # Total CPU requests in namespace
    requests.memory: "8Gi"     # Total memory requests
    limits.cpu: "8"            # Total CPU limits
    limits.memory: "16Gi"      # Total memory limits
    pods: "20"                 # Max pods in namespace
```

### LimitRange (Default Values)

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: development
spec:
  limits:
    - default:                  # Default limits if not specified
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:           # Default requests if not specified
        cpu: "100m"
        memory: "128Mi"
      type: Container
```

---

## 📋 Best Practices

- ✅ **Always set requests AND limits** for every container
- ✅ **Set requests = limits** for Guaranteed QoS (best for critical pods)
- ✅ **Use LimitRange** to enforce defaults per namespace
- ✅ **Use ResourceQuota** to prevent namespace resource hogging
- ✅ **Monitor resource usage** with metrics-server
- ❌ **Don't set limits too high** — they're wasted resources
- ❌ **Don't set requests too low** — pods may be evicted under pressure
- ❌ **Don't set memory limit = memory request** for Java apps (they need headroom)

---

## ❌ Common Mistakes

1. **No resource limits set** 🔥
   > One pod can consume all node resources, starving others.

2. **Requests much lower than actual usage** 🤦
   > Scheduler thinks pod needs little, packs too many on one node, everything slows down.

3. **Memory limits too low** 🔥
   > OOMKilled! Your container is killed without warning.

4. **CPU limit too low** 🤦
   > Container gets throttled, responds slowly, users complain.

5. **Confusing MiB and MB** 🤦
   > 128Mi = ~134 MB (1 MiB = 1.048 MB). K8s uses binary units (Mi, Gi).

---

## 🎤 Interview Questions

1. **What is the difference between requests and limits?**
   > Requests are the minimum guaranteed resources. Limits are the maximum a container can use.

2. **What happens when a container exceeds its memory limit?**
   > It gets OOMKilled (terminated by the kernel).

3. **What happens when a container exceeds its CPU limit?**
   > It gets throttled (slowed down), not killed.

4. **What is QoS in Kubernetes?**
   > Quality of Service: Guaranteed (requests = limits), Burstable (requests < limits), BestEffort (no requests/limits).

5. **What is a ResourceQuota?**
   > Limits total resource consumption per namespace.

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **Requests** | Minimum guaranteed (used for scheduling) |
| **Limits** | Maximum allowed |
| **CPU** | Throttled when exceeded (compressible) |
| **Memory** | OOMKilled when exceeded (incompressible) |
| **ResourceQuota** | Per-namespace total limits |
| **LimitRange** | Default requests/limits per container |

> *"Ravi, Resource Management is what separates a toy cluster from a production one. ALWAYS set requests and limits. Your future on-call self will thank you at 3 AM! 📊"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 19: Probes](../19-Probes/README.md)** | [📚 Home](../README.md) | **[21: HPA →](../21-Horizontal-Pod-Autoscaler/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
