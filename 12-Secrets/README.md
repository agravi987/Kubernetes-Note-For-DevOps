# 🔐 Secrets

> *"Ravi, passwords, API keys, TLS certificates — you NEVER want these hardcoded in your images or lying around in plain text. Kubernetes Secrets keep your sensitive data... well, secret! But there are important caveats you need to know. Let's dive in!"*

---

## 🤔 What is a Secret?

A **Secret** is a Kubernetes object that stores **sensitive data** like passwords, tokens, keys, and certificates. They're similar to ConfigMaps but designed for confidential information.

```
Secret stores:
├── Database passwords
├── API keys
├── TLS certificates
├── SSH keys
├── OAuth tokens
└── Any sensitive data
```

---

## 💡 Why Do We Need Secrets?

| Without Secrets | With Secrets |
|----------------|-------------|
| Passwords in YAML files 😱 | Encrypted storage 🔐 |
| Passwords in Docker images 🔓 | Separate from images ✅ |
| Passwords in Git repo 🤦 | Can be excluded from Git 🎯 |
| All devs see prod secrets 😰 | RBAC controls access 🔑 |

---

## ⚠️ Important Caveat

> **Secrets are ONLY base64 encoded, NOT encrypted by default!**

```
Secret "security levels":

1. Base64 encoded (default)    → Anyone with access can decode
2. Encrypted at rest           → Requires etcd encryption config
3. External secret managers    → Vault, AWS Secrets Manager (best)

Always combine with RBAC to limit who can read Secrets!
```

---

## 📝 YAML Example

### Create from YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  # Values must be base64 encoded!
  DB_USER: YWRtaW4=                    # "admin"
  DB_PASSWORD: UEA3c3cwcmQ=            # "@P7sw0rd"
stringData:
  # Plain text — Kubernetes will encode for you
  DB_HOST: mysql.default.svc.cluster.local
```

### Create from Command Line

```bash
# From literal values (auto base64 encodes)
kubectl create secret generic db-secret \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD='@P7sw0rd'

# From a file
kubectl create secret tls my-tls \
  --cert=tls.crt \
  --key=tls.key

# View secret (base64 encoded)
kubectl get secret db-secret -o yaml

# Decode a secret value
kubectl get secret db-secret -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode
```

---

## 📚 Types of Secrets

| Type | Purpose |
|------|---------|
| **Opaque** | Generic key-value (default) |
| **kubernetes.io/tls** | TLS certificates |
| **kubernetes.io/dockerconfigjson** | Docker registry credentials |
| **kubernetes.io/basic-auth** | Basic authentication |
| **kubernetes.io/ssh-auth** | SSH private keys |
| **kubernetes.io/service-account-token** | Service account tokens |

> 😂 **Fun Fact / Joke:** Base64 encoding is like hiding your diary under your mattress and telling everyone 'it's locked!' — your little brother found it in 2 seconds. Use real encryption, Ravi! This isn't a Bollywood spy movie! 🕵️😂

---

## 🧠 Important Concepts

### Using Secrets in Pods

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
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD

      # All keys as env vars
      envFrom:
        - secretRef:
            name: db-secret
```

#### 2. Volume Mount (as files)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: my-app:1.0
          volumeMounts:
            - name: secret-volume
              mountPath: /etc/secrets
              readOnly: true

      volumes:
        - name: secret-volume
          secret:
            secretName: db-secret
```

```
/etc/secrets/
├── DB_USER       ← file containing the decoded value
├── DB_PASSWORD
└── DB_HOST
```

---

## 📋 Best Practices

- ✅ **Always use RBAC** to restrict who can read Secrets
- ✅ **Enable encryption at rest** for etcd in production
- ✅ **Use external secret managers** (HashiCorp Vault, AWS SM) for production
- ✅ **Don't commit secrets to Git** — use sealed-secrets or external SM
- ✅ **Use `stringData`** in YAML for readability (K8s encodes it for you)
- ✅ **Mount as files** for better security (not env vars)
- ❌ **Don't rely on base64 as encryption** — it's just encoding!
- ❌ **Don't store production secrets in Git**
- ❌ **Don't forget to rotate secrets** periodically

---

## ❌ Common Mistakes

1. **Using base64 as "encryption"** 🤦
   ```bash
   # Anyone can decode base64!
   echo "VEA3c3cwcmQ=" | base64 --decode
   # @P7sw0rd

   # Base64 is NOT security!
   ```

2. **Committing secrets to Git** 🔥
   > Even base64 encoded secrets can be trivially decoded. Use sealed-secrets or external secret managers.

3. **Using env vars instead of file mounts** 🤦
   > Env vars are visible in `kubectl describe pod`. File mounts are slightly more secure.

4. **Not enabling etcd encryption** 🔥
   ```yaml
   # In kube-apiserver config:
   --encryption-provider-config=/path/to/encryption.yaml
   ```

---

## 🎤 Interview Questions

1. **What is a Kubernetes Secret?**
   > A Kubernetes object that stores sensitive data like passwords, tokens, and certificates. Data is base64 encoded.

2. **Is a Secret encrypted by default?**
   > No. Secrets are only base64 encoded. Encryption requires enabling encryption at rest for etcd.

3. **What is the difference between ConfigMap and Secret?**
   > ConfigMap is for non-sensitive data. Secret is for sensitive data and supports encryption at rest.

4. **How do you decode a Secret?**
   > `kubectl get secret <name> -o jsonpath='{.data.<key>}' | base64 --decode`

5. **How would you secure Secrets in production?**
   > Enable etcd encryption at rest, use RBAC to restrict access, and ideally use external secret managers like HashiCorp Vault.

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **Purpose** | Store sensitive data (passwords, keys, certs) |
| **Encoding** | Base64 (NOT encryption!) |
| **Types** | Opaque, TLS, docker-registry, etc. |
| **Usage** | Env vars or volume mounts |
| **Security** | RBAC + encryption at rest + external SM |
| **Danger** | Don't commit to Git, don't trust base64 |

> *"Ravi, Secrets are a good start, but they're not bulletproof. In real production, use a dedicated secret manager like HashiCorp Vault. For now, remember: base64 ≠ encryption! 🔐"*
