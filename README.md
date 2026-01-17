# Kustomize Complete Guide (Base, Overlay, Patch, Component, Generator)

This repository is a **complete, practical Kustomize reference**.
It explains **everything step by step**, with **real directory structures**, **clear examples**, and **why each concept exists**.

---

## Repository Structure

```
kustomize-complete-repo/
├── base/
│   ├── deployment.yaml
│   └── kustomization.yaml
├── components/
│   └── scale/
│       ├── patch.yaml
│       └── kustomization.yaml
├── overlays/
│   └── prod/
│       └── kustomization.yaml
└── generator-example/
    ├── deployment.yaml
    └── kustomization.yaml
```

---

# 1. BASE (The Application Itself)

**Base is the original application.**
It must work alone, with no environments, no features, no conditions.

### `base/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: app
          image: nginx
```

### `base/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
```

**Key idea:**

* `kubectl apply -k base` → replicas = **1**

---

# 2. PATCH (How Kustomize Modifies Resources)

A **patch** changes existing resources **without copying them**.

Example patch (change replicas):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
```

* Dictionaries → merge
* Lists → merge using `name`

---

# 3. COMPONENT (Optional Feature)

A **Component is an optional patch**.
It is **not an environment**.
It is a **feature you can turn ON or OFF**.

### Component Example: Scaling Feature

### `components/scale/patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
```

### `components/scale/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

patchesStrategicMerge:
  - patch.yaml
```

**Key rules:**

* Components contain **patches only**
* Components do **not** work alone

---

# 4. OVERLAY (Environment)

An **Overlay represents an environment** (dev, prod, staging).
It decides:

* Which base to use
* Which components to enable

### `overlays/prod/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

components:
  - ../../components/scale
```

### Result

* `kubectl apply -k base` → replicas = **1**
* `kubectl apply -k overlays/prod` → replicas = **3**

**Overlay = where you run from**

---

# 5. GENERATORS (ConfigMap Generator)

Generators **create Kubernetes resources automatically**.

## Why generators exist

Kubernetes **does not restart Pods** when ConfigMaps change.
Kustomize fixes this by:

* Generating a **new ConfigMap name with a hash**
* Updating all references automatically

---

## Generator Example

### `generator-example/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
        - name: app
          image: nginx
          envFrom:
            - configMapRef:
                name: app-config
```

### `generator-example/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml

configMapGenerator:
  - name: app-config
    literals:
      - ENV=prod
      - LOG_LEVEL=debug
```

### Generated Output (Automatically)

```yaml
kind: ConfigMap
metadata:
  name: app-config-x7a9k2
```

Deployment becomes:

```yaml
configMapRef:
  name: app-config-x7a9k2
```

**No patch required.**

---

# 6. HOW KUSTOMIZE KNOWS WHERE TO UPDATE

Kustomize has **built-in rules** for known references:

* `envFrom.configMapRef.name`
* `env.valueFrom.configMapKeyRef.name`
* `volumes.configMap.name`
* `secretRef`

This mechanism is called:

> **Name Reference Transformer**

---

# 7. COMMANDS YOU MUST KNOW

```bash
kustomize build .
kubectl apply -k .
kubectl delete -k .
```

---

# FINAL MENTAL MODEL

| Concept   | Meaning               |
| --------- | --------------------- |
| Base      | The application       |
| Patch     | How you modify        |
| Component | Optional feature      |
| Overlay   | Environment           |
| Generator | Auto-create resources |

---

## If you understand this repo

You **fully understand Kustomize**.
This is exactly how it is used in:

* Real DevOps projects
* GitOps (ArgoCD / Flux)
* Production Kubernetes

---

✅ You are done with Kustomize.
