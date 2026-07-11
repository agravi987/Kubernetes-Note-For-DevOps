# 🐘 StatefulSets

> *"Ravi, Deployments are great for stateless apps like web servers. But databases? They need stable network names, persistent storage, and ordered deployment. That's what StatefulSets are for! Think of them as Deployments for stateful apps! 🐘"*

---

## 🤔 What is a StatefulSet?

A **StatefulSet** is a workload API object used to manage stateful applications. It provides:

- **Stable, unique network identities** (pod-0, pod-1, pod-2)
- **Stable, persistent storage** (each pod gets its own PVC)
- **Ordered deployment and scaling** (sequential, not parallel)
- **Ordered, graceful deletion** (reverse order)

---

## 💡 Why Do We Need StatefulSets?

| Deployment (Stateless) | StatefulSet (Stateful) |
|----------------------|----------------------|
| Pod names are random (web-abc123) | Pod names are ordered (web-0, web-1) |
| Shared storage possible | Each pod gets its own storage |
| Parallel scale up/down | Sequential scale up/down |
| No network identity | Stable DNS names |
| Great for: web servers | Great for: databases, caches, queues |

---

## 📝 YAML Example

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql          # Headless service name
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: root-password
          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql

  volumeClaimTemplates:       # Each pod gets its own PVC
    - metadata:
        name: mysql-data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

### Required Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql                 # Must match serviceName in StatefulSet
spec:
  clusterIP: None             # Headless service!
  selector:
    app: mysql
  ports:
    - port: 3306
```

---

## 🧠 Important Concepts

### Stable Network Identity

```
StatefulSet Pods get predictable names:
  mysql-0, mysql-1, mysql-2

DNS names:
  mysql-0.mysql.default.svc.cluster.local
  mysql-1.mysql.default.svc.cluster.local
  mysql-2.mysql.default.svc.cluster.local

Short form (within same namespace):
  mysql-0.mysql
  mysql-1.mysql
  mysql-2.mysql
```

### Ordered Deployment

```
Scale UP (0 → 3): Sequential, one at a time
  Create mysql-0 → Wait ready → Create mysql-1 → Wait ready → Create mysql-2

Scale DOWN (3 → 0): Reverse order
  Delete mysql-2 → Wait terminated → Delete mysql-1 → Wait terminated → Delete mysql-0

Why? Databases need to sync data before new nodes join!
```

> 🐴🥁 **Joke:** StatefulSets are like an Indian wedding baraat procession — everyone moves in ORDER, the groom (pod-0) goes first, nobody skips the line, and each person has their own designated spot. Chaos is NOT an option!

### volumeClaimTemplates

```
Each pod gets a UNIQUE PersistentVolumeClaim:
  mysql-data-mysql-0  → 10Gi PV
  mysql-data-mysql-1  → 10Gi PV
  mysql-data-mysql-2  → 10Gi PV

If mysql-0 dies and is recreated → same PVC → same data! ✅
```

---

## 📋 Best Practices

- ✅ **Use StatefulSets** for databases, message queues, distributed caches
- ✅ **Always use a Headless Service** with StatefulSets
- ✅ **Use volumeClaimTemplates** for persistent storage
- ✅ **Set resource requests/limits** on database containers
- ✅ **Backup PVCs regularly** — don't rely solely on StatefulSet guarantees
- ❌ **Don't use StatefulSets** for stateless apps (use Deployments)
- ❌ **Don't force-delete pods** in production (can cause data inconsistency)
- ❌ **Don't scale down a database** without understanding replication

---

## ❌ Common Mistakes

1. **Forgetting the Headless Service** 🤦
   > StatefulSet requires `serviceName` pointing to a Headless Service (clusterIP: None).

2. **Not using volumeClaimTemplates** 🤦
   > Without it, pods share storage or have no persistent storage at all.

3. **Force-deleting pods** 🔥
   > `kubectl delete pod mysql-0 --force` can corrupt data. Let ordered deletion handle it.

4. **Using StatefulSets for stateless apps** 🤦
   > The overhead isn't worth it for stateless apps. Use Deployments.

---

## 🎤 Interview Questions

1. **What is a StatefulSet?**
   > A Kubernetes workload for stateful applications that provides stable network identities, persistent storage, and ordered deployment/scaling.

2. **What is the difference between Deployment and StatefulSet?**
   > Deployment: random pod names, parallel scaling, no network identity. StatefulSet: ordered names, sequential scaling, stable network identities, per-pod storage.

3. **What is a Headless Service?**
   > A Service with `clusterIP: None`. It doesn't load balance; instead, it returns individual pod IPs. Used by StatefulSets for DNS-based discovery.

4. **What are volumeClaimTemplates?**
   > Templates that automatically create a unique PVC for each pod in the StatefulSet. Each pod gets its own persistent storage.

---

## 📝 Summary

| Feature | StatefulSet |
|---------|-------------|
| **Pod Names** | Ordered: pod-0, pod-1, pod-2 |
| **DNS** | Stable: pod-0.service.ns.svc |
| **Storage** | Per-pod via volumeClaimTemplates |
| **Scaling** | Sequential (ordered) |
| **Use Cases** | Databases, queues, caches |
| **Prerequisite** | Headless Service required |

> *"Ravi, StatefulSets bring order to chaos. When your database pods need to know who they are, keep their data, and join/leave in order — StatefulSets handle all of it. 🐘"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 15: Ingress](../15-Ingress/README.md)** | [📚 Home](../README.md) | **[17: DaemonSets →](../17-DaemonSets/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
