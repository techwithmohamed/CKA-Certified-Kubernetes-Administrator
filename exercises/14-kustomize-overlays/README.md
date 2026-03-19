# Exercise 14 — Kustomize: Base and Overlay

> **Medium** | ~15 min | Domain: Cluster Architecture (25%)
>
> Related: [README — Cluster Architecture](../../README.md#domain-4--cluster-architecture-installation--configuration-25)

Build a base deployment and apply an overlay that changes the namespace and replica count. Kustomize is built into kubectl and is now part of the CKA skill set.

## Tasks

1. Create a directory structure:
   ```
   kustomize-lab/
     base/
       deployment.yaml
       service.yaml
       kustomization.yaml
     overlays/
       staging/
         kustomization.yaml
   ```
2. In the base:
   - A Deployment named `app` with image `nginx:1.27` and 1 replica
   - A ClusterIP Service named `app-svc` on port 80
   - A `kustomization.yaml` listing both resources
3. In the staging overlay:
   - Set namespace to `exercise-14`
   - Patch replicas to 3
   - Add a common label `env: staging`
4. Create namespace `exercise-14`
5. Apply the staging overlay with `kubectl apply -k`
6. Verify 3 pods are running in namespace `exercise-14` with label `env=staging`

## Hints

- `kubectl apply -k <overlay-dir>/`
- `kubectl kustomize <overlay-dir>/` to preview the output without applying
- Overlay `kustomization.yaml` references the base with `resources: [../../base]`
- Use `replicas` field in kustomization.yaml to patch replica count

## Verify

```bash
# 3 pods running
k get pods -n exercise-14

# Label applied
k get deploy app -n exercise-14 --show-labels | grep staging

# Service created
k get svc app-svc -n exercise-14
```

## Cleanup

```bash
k delete ns exercise-14
rm -rf kustomize-lab/
```

<details>
<summary>Solution</summary>

```bash
mkdir -p kustomize-lab/base kustomize-lab/overlays/staging
```

base/deployment.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
```

base/service.yaml:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-svc
spec:
  selector:
    app: app
  ports:
  - port: 80
    targetPort: 80
```

base/kustomization.yaml:
```yaml
resources:
- deployment.yaml
- service.yaml
```

overlays/staging/kustomization.yaml:
```yaml
resources:
- ../../base

namespace: exercise-14

commonLabels:
  env: staging

replicas:
- name: app
  count: 3
```

```bash
k create ns exercise-14
k apply -k kustomize-lab/overlays/staging/

# Verify
k get pods -n exercise-14
k get deploy app -n exercise-14 --show-labels
k get svc app-svc -n exercise-14
```

</details>
