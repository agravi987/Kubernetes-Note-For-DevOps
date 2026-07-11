# 💾 Volumes

> *"Ravi, remember how we said Pods are ephemeral? When a pod dies, EVERYTHING inside it is gone — data, logs, temp files. That's fine for stateless apps, but what about databases? That's where Volumes come in! They give your pods persistent or shared storage. 💾"*

---

## 🤔 What is a Volume?

A **Volume** is a directory accessible to containers in a Pod. It provides storage that can:

- **Persist** beyond a container restart (ephemeral volumes)
- **Persist** beyond pod lifecycle (persistent volumes)
- **Share data** between containers in the same pod
- **Mount external storage** (cloud disks, NFS, etc.)

---

## 💡 Why Do We Need Volumes?

```
Without Volume:
  Pod dies → Container dies → ALL DATA GONE 💀

With Volume:
  Pod dies → Volume survives → New pod mounts it → DATA INTACT ✅
```

---

## 📚 Types of Volumes

### Ephemeral Volumes (Lost when Pod is deleted)

#### 1. emptyDir ✨

**Empty when pod starts, shared between containers.** Lost when pod is deleted.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-pod
spec:
  containers:
    - name: writer
      image: busybox
      command: ["sh", "-c", "echo 'hello' > /data/message.txt; sleep 3600"]
      volumeMounts:
        - name: shared-data
          mountPath: /data

    - name: reader
      image: busybox
      command: ["sh", "-c", "cat /data/message.txt; sleep 3600"]
      volumeMounts:
        - name: shared-data
          mountPath: /data

  volumes:
    - name: shared-data
      emptyDir: {}
```

#### 2. hostPath 📂

**Mounts a file/directory from the node's filesystem.** Risky in production!

```yaml
volumes:
  - name: node-logs
    hostPath:
      path: /var/log/nginx    # From the node's filesystem
      type: Directory
```

> ⚠️ **hostPath is dangerous!** Containers can access the host filesystem. Use only for specific system-level use cases.

#### 3. configMap / secret 📁

We already covered these! They mount ConfigMaps and Secrets as volumes.

---

### Persistent Volumes (Survive Pod deletion)

#### 4. PersistentVolumeClaim (PVC) 🗄️

**Requests storage from a PersistentVolume.** We'll deep-dive in the next chapter!

```yaml
volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

---

### Common Volume Types

| Volume Type | Persistent? | Use Case |
|-------------|------------|----------|
| **emptyDir** | ❌ No | Sharing data between containers |
| **hostPath** | ⚠️ Node only | System-level access (avoid in prod) |
| **configMap** | ❌ No | Mount config files |
| **secret** | ❌ No | Mount secret files |
| **PersistentVolumeClaim** | ✅ Yes | Databases, user data |
| **NFS** | ✅ Yes | Shared network storage |
| **awsElasticBlockStore** | ✅ Yes | AWS EBS volumes |
| **gcePersistentDisk** | ✅ Yes | Google Persistent Disk |

---

## 📝 YAML Example

### emptyDir — Shared Between Containers

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-cache
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      volumeMounts:
        - name: cache
          mountPath: /var/cache/nginx

    - name: cache-cleaner
      image: busybox
      command: ["sh", "-c", "while true; do rm -rf /cache/*; sleep 3600; done"]
      volumeMounts:
        - name: cache
          mountPath: /cache

  volumes:
    - name: cache
      emptyDir:
        sizeLimit: 100Mi     # Optional: limit the size
```

### emptyDir with Memory Backend

```yaml
volumes:
  - name: tmp-volume
    emptyDir:
      medium: Memory         # Uses RAM instead of disk!
      sizeLimit: 64Mi
```

> 💡 **Use `medium: Memory`** for high-speed temp storage. Data is lost when pod restarts AND counts against memory limits.

---

## 🧠 Important Concepts

### Volume Lifecycle

```
emptyDir:
  Pod created → Volume created (empty) → Container mounts → Pod deleted → Volume GONE

hostPath:
  Pod created → Mounts host path → Pod deleted → File remains on host

PVC:
  Pod created → PVC binds to PV → Container mounts → Pod deleted → PVC and PV survive
```

### Mount Paths

```yaml
volumeMounts:
  - name: my-volume
    mountPath: /app/data       # Where the volume appears inside the container
    readOnly: true             # Optional: make it read-only
    subPath: subdir            # Optional: mount a subdirectory
```

---

## 📋 Best Practices

- ✅ **Use emptyDir** for shared data between containers
- ✅ **Use PVC** for any data that needs to survive pod restarts
- ✅ **Use `medium: Memory`** for high-speed temporary storage
- ✅ **Set `sizeLimit`** on emptyDir to prevent disk exhaustion
- ❌ **Don't use hostPath** in production (security risk!)
- ❌ **Don't store important data in emptyDir** (it's ephemeral!)
- ❌ **Don't use volumes as a substitute for databases**

---

## ❌ Common Mistakes

1. **Using emptyDir for persistent data** 🤦
   > emptyDir = temporary! Use PVC for anything that must survive.

2. **Using hostPath in production** 🔥
   > Security risk — containers can access the entire host filesystem.

3. **Forgetting to set sizeLimit** 🤦
   > An emptyDir can fill up the node's disk, affecting all pods.

4. **Confusing volumeMounts with volumes** 🤦
   > `volumes` defines the volume. `volumeMounts` makes it available inside a container. You need both!

---

## 🎤 Interview Questions

1. **What is a Kubernetes Volume?**
   > A directory accessible to containers in a Pod, providing shared or persistent storage.

2. **What is the difference between emptyDir and a PersistentVolume?**
   > emptyDir is ephemeral — deleted when the pod is deleted. PersistentVolumes survive beyond pod lifecycle.

3. **What is emptyDir used for?**
   > Sharing data between containers in the same pod, caching, or temporary storage.

4. **Why is hostPath risky in production?**
   > It exposes the host filesystem to containers, creating security vulnerabilities.

---

## 📝 Summary

| Type | Persistent | Use Case |
|------|-----------|----------|
| **emptyDir** | ❌ | Shared/temporary data |
| **hostPath** | ⚠️ | Node-level access (avoid!) |
| **configMap/secret** | ❌ | Mount config/secrets |
| **PVC** | ✅ | Persistent data |

> *"Ravi, Volumes are the bridge between ephemeral pods and persistent data. For quick sharing between containers, use emptyDir. For anything that must survive, use PVC. That's coming up next! 🗄️"*
