# 🌐 Services

> *"Ravi, Pods are ephemeral — they die, get new IPs, move between nodes. So how do other apps (or users) find them? SERVICES! They give your pods a stable IP and DNS name that never changes. This is networking magic! 🪄"*

<div align="center">

| 📖 Reading Time | 🎯 Difficulty | 🏷️ Category |
|:---:|:---:|:---:|
| ~10 min | ⭐⭐ Intermediate | Networking |

</div>

---

## 🤔 What is a Service?

A **Service** is a stable network endpoint that exposes a set of Pods. It provides:

- A **fixed IP address** (doesn't change when pods restart)
- A **DNS name** (auto-discoverable within the cluster)
- **Load balancing** across multiple pods

```
Without Service:
  pod-1 (10.244.0.5) → dies → pod-1 (10.244.1.8) → new IP! 😱

With Service:
  Service (10.96.0.100) → always same IP → routes to current pods ✅
```

---

## 💡 Why Do We Need Services?

| Problem | Service Solution |
|---------|-----------------|
| Pod IP changes when restarted | Service has a **fixed** IP 📍 |
| Need DNS for service discovery | Service gets a **DNS name** 🌐 |
| Traffic to multiple pods | **Load balances** across all pods ⚖️ |
| External access needed | Exposes pods to the outside world 🌍 |

---

## 🔄 How It Works

```
Client sends request to Service IP
    ↓
Service checks pod selector (e.g., app=web)
    ↓
Finds all matching pods
    ↓
Load balances traffic across them
    ↓
Request reaches one of the pods
```

---

## 📚 Types of Services

### 1. ClusterIP (Default) 🔒

**Internal only.** Accessible only within the cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP        # Default — can be omitted
  selector:
    app: web
  ports:
    - port: 80           # Service port (what other pods connect to)
      targetPort: 80     # Container port (where nginx listens)
```

```
┌─────────── CLUSTER ──────────────┐
│                                   │
│  Pod-A ─┐                        │
│  Pod-B ─┼─ Service (10.96.0.100) │
│  Pod-C ─┘    :80                 │
│                                   │
│  Other pods → can reach service   │
│  Outside    → CANNOT reach        │
└───────────────────────────────────┘
```

> 💡 **Use when:** Internal microservice communication (frontend → backend)

---

### 2. NodePort 🚪

Exposes the Service on each node's IP at a **static port** (30000-32767).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80           # Service port inside cluster
      targetPort: 80     # Container port
      nodePort: 30080    # Port on each node (30000-32767)
```

```
External User → http://<any-node-ip>:30080 → Service → Pod

Node 1 (192.168.1.10:30080) ─┐
Node 2 (192.168.1.11:30080) ─┼─ Service → Pods
Node 3 (192.168.1.12:30080) ─┘
```

> 💡 **Use when:** Development/testing, quick external access. NOT recommended for production.

---

### 3. LoadBalancer ☁️

Exposes the Service externally using a **cloud provider's load balancer**.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-lb
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```

```
External User → Cloud Load Balancer → Service → Pods

  Internet
      ↓
  ┌──────────────┐
  │ Load Balancer │  ← Cloud provider creates this (AWS ELB, GCP LB)
  └──────┬───────┘
         ↓
  ┌──────────────┐
  │   Service    │
  └──────┬───────┘
         ↓
  ┌──────┴──────┐
  │  Pods       │
  └─────────────┘
```

> 💡 **Use when:** Production applications that need external access.

---

### 4. ExternalName 🌍

Maps a Service to an **external DNS name** (outside the cluster).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: db.example.com
```

```
Pod inside cluster → DNS lookup for "external-db"
    ↓
Returns CNAME: db.example.com
    ↓
Connects to external database
```

> 💡 **Use when:** Referencing external services (RDS, external APIs) with a clean DNS name.

---

### Service Types Comparison

| Type | Access | Use Case | Production? |
|------|--------|----------|------------|
| **ClusterIP** | Internal only | Microservice communication | ✅ Yes |
| **NodePort** | External via node IP:port | Dev/testing | ⚠️ Limited |
| **LoadBalancer** | External via cloud LB | External-facing apps | ✅ Yes |
| **ExternalName** | Maps to external DNS | External service reference | ✅ Yes |

---

## 🧠 Important Concepts

### DNS & Service Discovery

Every Service gets a DNS entry automatically:

```
Service: web-service in namespace: default

DNS Name: web-service.default.svc.cluster.local
Short:    web-service (within same namespace)
          web-service.default (across namespaces)
```

```bash
# Test DNS from inside a pod
kubectl exec -it test-pod -- nslookup web-service

# Access service from another pod
curl http://web-service:80
```

### Endpoints

```bash
# See which pods a Service is routing to
kubectl get endpoints web-service

# NAME            ENDPOINTS
# web-service     10.244.0.5:80,10.244.1.3:80,10.244.2.7:80
```

### Session Affinity

```yaml
spec:
  sessionAffinity: ClientIP    # Same client → same pod
  # Default: None (round-robin)
```

> 🍕 **Fun Fact / Joke:** A Kubernetes Service is like an auto-rickshaw stand — you don't care which driver picks you up, you just want to reach your destination. That's load balancing, and it works even better than the meter! 🛺

---

## 📋 Best Practices

- ✅ **Use ClusterIP** for internal services (most common)
- ✅ **Use LoadBalancer** for external-facing services
- ✅ **Use Ingress** instead of NodePort for HTTP routing (covered later)
- ✅ **Always set `targetPort`** to match your container's listening port
- ✅ **Use readiness probes** so Services only route to healthy pods
- ❌ **Don't use NodePort in production** — use Ingress or LoadBalancer
- ❌ **Don't hardcode Pod IPs** — always use Services
- ❌ **Don't use `sessionAffinity`** unless you have a specific reason

---

## ❌ Common Mistakes

1. **Port mismatch** 🔥
   ```yaml
   # ❌ Wrong — targetPort must match container's listening port
   ports:
     - port: 80
       targetPort: 8080    # But nginx listens on 80!

   # ✅ Correct
   ports:
     - port: 80
       targetPort: 80      # Matches nginx default port
   ```

2. **Selector doesn't match any pods** 🤦
   ```bash
   # Check endpoints — if empty, selector is wrong
   kubectl get endpoints my-service

   # NAME            ENDPOINTS
   # my-service      <none>    ← No pods matched!
   ```

3. **Forgetting that Service is L4 (TCP) only** 🤦
   > Services load balance at TCP/UDP level, not HTTP. For HTTP routing, use Ingress.

4. **Hardcoding ClusterIP** 🤦
   > Kubernetes assigns ClusterIP automatically. Don't hardcode it — use DNS names instead.

---

## 🎤 Interview Questions

1. **What is a Kubernetes Service?**
   > A stable network endpoint that provides a fixed IP and DNS name for a set of pods, with built-in load balancing.

2. **What are the types of Services?**
   > ClusterIP (internal), NodePort (external via node), LoadBalancer (external via cloud LB), ExternalName (maps to external DNS).

3. **What is the default Service type?**
   > ClusterIP — accessible only within the cluster.

4. **How does Service discovery work in Kubernetes?**
   > Every Service gets a DNS entry: `<service-name>.<namespace>.svc.cluster.local`. Pods can resolve services by name.

5. **What is the difference between `port` and `targetPort`?**
   > `port` is the port the Service listens on. `targetPort` is the port on the container that receives traffic.

6. **When would you use NodePort vs LoadBalancer?**
   > NodePort for dev/testing (quick access). LoadBalancer for production (managed cloud LB with SSL, health checks).

---

## 📝 Summary

| Type | Access | DNS | Use Case |
|------|--------|-----|----------|
| **ClusterIP** | Internal | ✅ | Internal microservices |
| **NodePort** | External (node:port) | ✅ | Dev/testing |
| **LoadBalancer** | External (cloud LB) | ✅ | Production external |
| **ExternalName** | External (CNAME) | ✅ | External service ref |

> *"Ravi, Services are the glue that connects everything. Pods come and go, but Services stay constant. Think of them as a phone number that always routes to whoever currently has it! 📱"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 08: Deployments](../08-Deployments/README.md)** | [📚 Home](../README.md) | **[Next: 10: Namespaces →](../10-Namespaces/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
