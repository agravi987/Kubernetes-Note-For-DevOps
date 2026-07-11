# 🗄️ Persistent Volumes and PVC

> *"Ravi, emptyDir is great for temp data, but what about your database? User uploads? You need storage that LIVES beyond pod restarts. PersistentVolumes (PV) and PersistentVolumeClaims (PVC) are how Kubernetes manages durable storage. This is critical for stateful apps!"*

---

## 🤔 What are PV and PVC?

### PersistentVolume (PV)
A **piece of storage** in the cluster provisioned by an admin or dynamically.

### PersistentVolumeClaim (PVC)
A **request for storage** by a user/pod. It's like a ticket that claims a PV.

```
Think of it like parking spots:

PV = A parking spot in the garage (storage)
PVC = A ticket claiming a parking spot (request)

You don't park directly — you get a ticket (PVC) and it points you to an available spot (PV).
```

---

## 💡 Why Do We Need Them?

| Without PV/PVC | With PV/PVC |
|---------------|------------|
| Data dies with pod 💀 | Data survives pod restarts ✅ |
| Hardcoded storage details 🤦 | Abstraction layer 🎯 |
| Can't move between nodes 😰 | Storage is cluster-level ⚡ |
| Manual provisioning 🤷 | Dynamic provisioning 🚀 |

---

## 📝 YAML Examples

### PersistentVolume (PV)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
  labels:
    type: local
spec:
  capacity:
    storage: 5Gi               # Size of the volume
  accessModes:
    - ReadWriteOnce            # Mount as read-write by one node
  persistentVolumeReclaimPolicy: Retain   # What happens when PVC is deleted
  storageClassName: standard    # Used for dynamic provisioning
  hostPath:
    path: /mnt/data            # Local path (for learning)
```

### PersistentVolumeClaim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce            # Must match PV's access mode
  resources:
    requests:
      storage: 2Gi             # Request 2Gi (PV must be >= this)
  storageClassName: standard   # Must match PV's storage class
```

### Using PVC in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-database
spec:
  containers:
    - name: mysql
      image: mysql:8.0
      env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
      volumeMounts:
        - name: db-storage
          mountPath: /var/lib/mysql    # MySQL data directory

  volumes:
    - name: db-storage
      persistentVolumeClaim:
        claimName: my-pvc              # Reference the PVC
```

---

## 🔄 How It Works

```
Admin creates PV ──→ Available in cluster
                         │
User creates PVC ──→ Kubernetes finds matching PV
                         │
PVC bound to PV ──→ Pod uses PVC ──→ Data written to PV
                         │
Pod deleted ──→ PVC and PV survive ──→ Data intact! ✅
```

### Dynamic Provisioning (Automated)

```
Without dynamic provisioning:
  Admin creates PV manually → PVC binds to it

With dynamic provisioning:
  PVC requests storage → Provisioner automatically creates PV!

  PVC: "I need 5Gi of storage"
       ↓
  Provisioner: "Here's a new 5Gi PV!" (creates it on the fly)
```

> 🚂 **Joke:** PV and PVC are like Indian train reservations — the PV is the confirmed seat, the PVC is your ticket. You don't need to know which coach or seat number, you just need to know you have a confirmed berth! Now please don't ask for side-lower.

---

## 🧠 Important Concepts

### Access Modes

| Mode | Description | One Node | Multiple Nodes |
|------|-------------|----------|---------------|
| **ReadWriteOnce (RWO)** | Read-write by one node | ✅ | ❌ |
| **ReadOnlyMany (ROX)** | Read-only by many nodes | ✅ | ✅ |
| **ReadWriteMany (RWX)** | Read-write by many nodes | ✅ | ✅ |
| **ReadWriteOncePod (RWOP)** | Read-write by one pod | ✅ | ❌ |

```
ReadWriteOnce  → Databases, single-writer apps
ReadOnlyMany   → Shared static content
ReadWriteMany  → Shared file systems (NFS, EFS)
```

### Reclaim Policies

| Policy | When PVC is deleted... |
|--------|----------------------|
| **Retain** | PV and data preserved (manual cleanup) |
| **Delete** | PV and underlying storage deleted |
| **Recycle** | Data erased, PV becomes Available (deprecated) |

### StorageClasses

```yaml
# StorageClass defines "type" of storage
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iopsPerGB: "10"
reclaimPolicy: Delete
allowVolumeExpansion: true
```

```bash
# Available StorageClasses
kubectl get storageclass

# NAME                 PROVISIONER                    RECLAIMPOLICY
# standard (default)   kubernetes.io/hostpath         Delete
# fast-ssd             kubernetes.io/aws-ebs          Delete
```

---

## 📋 Best Practices

- ✅ **Use Dynamic Provisioning** with StorageClasses (avoid manual PV creation)
- ✅ **Set reclaimPolicy: Retain** for important data
- ✅ **Use ReadWriteOnce** for databases (single node access)
- ✅ **Enable volume expansion** when possible (`allowVolumeExpansion: true`)
- ✅ **Back up PersistentVolumes** regularly
- ❌ **Don't use hostPath PVs** in production
- ❌ **Don't forget to set resource requests** for storage
- ❌ **Don't use Delete reclaimPolicy** for production data

---

## ❌ Common Mistakes

1. **PVC size > PV size** 🤦
   > PVC requests 10Gi but PV is only 5Gi → binding fails!

2. **StorageClass mismatch** 🤦
   > PVC and PV must have matching storageClassName (or both omit it).

3. **Access mode mismatch** 🤦
   > PVC requests ReadWriteMany but PV only supports ReadWriteOnce → binding fails.

4. **Forgetting reclaimPolicy** 🔥
   > Default is often Delete. If you delete the PVC, your data is gone!

---

## 🎤 Interview Questions

1. **What is a PersistentVolume (PV)?**
   > A piece of cluster-level storage that has been provisioned by an admin or dynamically. It exists independently of pods.

2. **What is a PersistentVolumeClaim (PVC)?**
   > A request for storage by a user/pod. It requests size, access mode, and storage class. Kubernetes binds it to a matching PV.

3. **What is Dynamic Provisioning?**
   > Automatic PV creation when a PVC is submitted. A provisioner (like AWS EBS driver) creates the storage on-demand.

4. **What are the Access Modes?**
   > ReadWriteOnce (one node R/W), ReadOnlyMany (many nodes read), ReadWriteMany (many nodes R/W), ReadWriteOncePod (one pod R/W).

5. **What happens to a PV when its PVC is deleted?**
   > Depends on reclaimPolicy: Retain (data preserved), Delete (everything deleted).

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **PV** | Cluster storage resource |
| **PVC** | Request for storage |
| **StorageClass** | Defines storage type/tier |
| **Dynamic Provisioning** | Auto-creates PVs from PVCs |
| **ReclaimPolicy** | What happens when PVC is deleted |
| **RWO/ROX/RWX** | Access modes |

> *"Ravi, PV and PVC are the yin and yang of Kubernetes storage. PVC asks, PV provides. For production, always use dynamic provisioning with StorageClasses and set reclaimPolicy: Retain for anything important! 🗄️"*
