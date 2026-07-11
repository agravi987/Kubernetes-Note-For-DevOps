# ⚙️ ConfigMaps

> *"Ravi, never hardcode configuration in your Docker images! That's a rookie mistake. ConfigMaps let you separate config from code — just like .env files do in regular development. Clean, safe, and flexible! 🧹"*

---

## 🤔 What is a ConfigMap?

A **ConfigMap** stores **non-sensitive configuration data** as key-value pairs. Pods can consume this data as environment variables, command-line arguments, or mounted files.

```
ConfigMap stores:
├── Database URLs
├── Feature flags
├── API endpoints
├── Configuration files
├── Any non-secret data
```

---

## 💡 Why Do We Need ConfigMaps?

| Without ConfigMap | With ConfigMap |
|------------------|---------------|
| Config baked into Docker image 🔒 | Config lives outside the image 🔓 |
| Change config → rebuild image 😰 | Change config → update ConfigMap 🎉 |
| Same image for dev/staging/prod? 😱 | Same image, different configs per environment ✅ |
| Secrets mixed with config 🤦 | Separation of concerns 🎯 |

---

## 📝 YAML Example

### Create from YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Simple key-value pairs
  DATABASE_HOST: "mysql.default.svc.cluster.local"
  DATABASE_PORT: "3306"
  LOG_LEVEL: "info"

  # File-like configuration
  nginx.conf: |
    server {
      listen 80;
      server_name localhost;
      location / {
        proxy_pass http://localhost:3000;
      }
    }
```

### Create from Command Line

```bash
# Create from literal values
kubectl create configmap app-config \
  --from-literal=DATABASE_HOST=mysql.default.svc.cluster.local \
  --from-literal=DATABASE_PORT=3306

# Create from a file
kubectl create configmap nginx-config --from-file=nginx.conf

# Create from a directory (each file becomes a key)
kubectl create configmap all-configs --from-file=configs/
```

---

## 🧠 Important Concepts

### Ways to Use ConfigMap in Pods

#### 1. Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: my-app:1.0
      env:
        # Single key
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DATABASE_HOST

        # All keys as env vars
      envFrom:
        - configMapRef:
            name: app-config
```

#### 2. Volume Mount (as files)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: my-app:1.0
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config    # Files appear here

  volumes:
    - name: config-volume
      configMap:
        name: app-config
        # Optional: map specific keys to filenames
        items:
          - key: nginx.conf
            path: nginx.conf
```

```
/etc/config/
├── nginx.conf     ← from the ConfigMap
├── DATABASE_HOST  ← one file per key
├── DATABASE_PORT
└── LOG_LEVEL
```

> 📝 **Fun Fact / Joke:** Hardcoding config in your Docker image is like writing your WiFi password on the main gate of your society — sure, convenient, but now the entire colony is on your network! ConfigMaps exist for a reason! 📝📶

---

## 📋 Best Practices

- ✅ **Use ConfigMaps** for non-sensitive configuration only
- ✅ **Use Secrets** for passwords, API keys, certificates
- ✅ **Mount as files** when you need hot-reload (app re-reads files)
- ✅ **Use envFrom** for simpler configuration injection
- ✅ **Version your ConfigMaps** — create new versions for changes
- ❌ **Don't store secrets** in ConfigMaps (use Secrets instead)
- ❌ **Don't hardcode config in Docker images**
- ❌ **Don't put large files** in ConfigMaps (1MB limit)

---

## ❌ Common Mistakes

1. **Storing secrets in ConfigMaps** 🔥
   > ConfigMaps are NOT encrypted! Use Secrets for sensitive data.

2. **Forgetting to restart pods after ConfigMap change** 🤦
   > ConfigMap changes don't auto-restart pods. You need to:
   > - Restart pods manually, OR
   > - Use a deployment rollout, OR
   > - Mount as files (auto-updates after ~1min)

3. **Key name conflicts** 🤦
   > ConfigMap keys must be valid DNS subdomains (no special characters except `-`, `_`, `.`).

---

## 🎤 Interview Questions

1. **What is a ConfigMap?**
   > A Kubernetes object that stores non-sensitive configuration data as key-value pairs, injectable as env vars or mounted as files.

2. **What is the difference between ConfigMap and Secret?**
   > ConfigMap stores non-sensitive data in plain text. Secret stores sensitive data (base64 encoded, can be encrypted at rest).

3. **How do you update a ConfigMap without restarting pods?**
   > Mount as a volume — files auto-update within ~1 minute. For env vars, pods must be restarted.

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **Purpose** | Store non-sensitive configuration |
| **Usage** | Env vars or mounted files |
| **Limit** | 1MB max size |
| **Security** | NOT encrypted — use Secrets for sensitive data |
| **Update** | Volume mounts auto-update; env vars need restart |

> *"Ravi, the golden rule: separate config from code. ConfigMaps make this easy. Your future self (and your team) will thank you! 🙏"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 10: Namespaces](../10-Namespaces/README.md)** | [📚 Home](../README.md) | **[12: Secrets →](../12-Secrets/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
