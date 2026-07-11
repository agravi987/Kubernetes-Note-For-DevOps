# 👥 DaemonSets

> *"Ravi, imagine you need a log collector running on EVERY single node in your cluster. One per node. No more, no less. If a new node joins, automatically deploy there. If a node leaves, clean up. That's EXACTLY what DaemonSets do!"*

---

## 🤔 What is a DaemonSet?

A **DaemonSet** ensures that exactly **one copy** of a pod runs on every (or selected) node in the cluster.

```
Node 1:  ┌──────────┐
         │ DaemonSet │  ← One pod
         └──────────┘

Node 2:  ┌──────────┐
         │ DaemonSet │  ← One pod
         └──────────┘

Node 3:  ┌──────────┐
         │ DaemonSet │  ← One pod
         └──────────┘

New Node joins → DaemonSet automatically deploys a pod there! 🚀
```

---

## 💡 Why Do We Need DaemonSets?

| Use Case | Why DaemonSet? |
|----------|---------------|
| **Log collection** | Fluentd/FluentBit on every node |
| **Monitoring** | Node exporter on every node |
| **Networking** | kube-proxy, CNI plugins |
| **Storage** | Storage drivers on every node |
| **Security** | File integrity monitoring |

---

## 📝 YAML Example

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  namespace: kube-system
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
        - name: fluentd
          image: fluent/fluentd-kubernetes-daemonset:v1.16
          resources:
            requests:
              cpu: "100m"
              memory: "200Mi"
            limits:
              cpu: "200m"
              memory: "400Mi"
          volumeMounts:
            - name: varlog
              mountPath: /var/log
              readOnly: true

      volumes:
        - name: varlog
          hostPath:
            path: /var/log
```

---

## 🧠 Important Concepts

### Node Selection

```yaml
spec:
  template:
    spec:
      # Only run on specific nodes
      nodeSelector:
        disktype: ssd

      # Or use tolerations to run on tainted nodes
      tolerations:
        - key: "node-role.kubernetes.io/control-plane"
          operator: "Exists"
          effect: "NoSchedule"
```

### Scaling Behavior

```
DaemonSet behavior:
├── New node added → Pod automatically created on it
├── Node removed → Pod is garbage collected
├── Scale down → Pods are removed from nodes
└── Scale up → N/A (already runs on all nodes)
```

> ☕🏢 **Fun Fact / Joke:** A DaemonSet is like a chai wallah at an IT park — you need one at EVERY floor (node), they're always running, and the day they're NOT there, productivity drops to zero!

---

## 📋 Best Practices

- ✅ **Use DaemonSets** for node-level system services (logging, monitoring, networking)
- ✅ **Set resource requests/limits** — these pods run permanently
- ✅ **Use nodeSelector/tolerations** to control which nodes get the pod
- ❌ **Don't use DaemonSets** for application workloads (use Deployments)
- ❌ **Don't run more than one pod per node** (unless specifically needed)
- ❌ **Don't forget to monitor DaemonSet pods** — they're critical infrastructure

---

## ❌ Common Mistakes

1. **Using DaemonSets for applications** 🤦
   > Applications should use Deployments. DaemonSets are for node-level infrastructure.

2. **Forgetting to set resource limits** 🤦
   > DaemonSet pods run permanently on every node. Without limits, they consume all node resources.

3. **Running on control-plane nodes** 🤦
   > Unless you specifically need to, add a toleration or nodeSelector to skip control-plane nodes.

---

## 🎤 Interview Questions

1. **What is a DaemonSet?**
   > A Kubernetes workload that ensures exactly one pod runs on every (or selected) node in the cluster.

2. **What are common use cases for DaemonSets?**
   > Log collection (Fluentd), monitoring (node-exporter), networking (kube-proxy, CNI plugins), storage drivers.

3. **What happens when a new node is added to the cluster?**
   > The DaemonSet automatically creates a pod on the new node.

4. **How is DaemonSet different from Deployment?**
   > Deployment ensures a desired number of replicas across the cluster. DaemonSet ensures exactly one pod per node.

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **DaemonSet** | One pod per node |
| **Use Cases** | Logging, monitoring, networking, storage drivers |
| **Scaling** | Automatic — follows node lifecycle |
| **Selection** | nodeSelector, tolerations |
| **Not for** | Application workloads |

> *"Ravi, DaemonSets are the unsung heroes of Kubernetes. They run the infrastructure that makes everything else work — logging, monitoring, networking. You rarely create them as a developer, but you'll see them everywhere! 👥"*
