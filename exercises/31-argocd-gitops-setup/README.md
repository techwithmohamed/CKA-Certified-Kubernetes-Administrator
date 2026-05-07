# Exercise 31 — Argo CD Setup (GitOps Deployment)

> Related: [README — Deployments](../../README.md#domain-3--workloads--scheduling-15)

Install Argo CD and configure it to deploy applications from a Git repository. Tests GitOps concepts and custom resource management.

## Tasks

1. Install Argo CD in cluster:
   - Create `argocd` namespace
   - Apply Argo CD manifests (WITHOUT CRDs if specified)
2. Generate Kubernetes manifest template:
   - Define an Application resource
   - Point to a Git repository
   - Specify deployment target namespace
   - Do NOT install CRDs — let Argo CD install them
3. Save generated Application manifest to file
4. Verify Argo CD server is Running
5. Apply the Application manifest
6. Verify application deployment syncs
7. Create a port-forward to Argo CD UI (optional testing)

## Key Learning

- GitOps: cluster state matches Git repository
- Argo CD Application resource is a CRD
- CRDs are NOT in base manifests — error if you apply before Argo CD
- Application must specify repoURL, path, destination
- Exam tests CRD understanding and GitOps workflow

## Hints

<details>
<summary>Stuck? Click to reveal hints</summary>

- Argo CD install: `k create ns argocd` + apply base manifests
- Don't install CRDs separately — Argo CD operator does it
- Wait for server to start: `k get pod -n argocd -l app.kubernetes.io/name=argocd-server`
- Application spec needs: `repoURL`, `path`, `destination.namespace`
- Sync status: `k get application -A`

</details>

## What tripped me up

> I installed Argo CD with CRDs included and then the Application CRD already existed. Then when I tried to apply the Application manifest, it didn't have the right fields. Argo CD has a specific Application spec structure that's different from regular Deployments. The exam said "generate WITHOUT CRDs" — follow instructions exactly. Also: the Git repository URL matters — wrong URL means Application stays out-of-sync. Test the repo URL is accessible BEFORE applying the Application.

## Verify

```bash
# Argo CD is running
k get pod -n argocd

# Argo CD server is accessible
k port-forward -n argocd svc/argocd-server 8080:443

# Application resource exists and is in sync
k get application -A

# Get application sync status
k get application -n <namespace> <app-name> -o jsonpath='{.status.sync.status}'
# Should show: Synced

# Check deployed resources
k get deployment,service -n <app-namespace>
```

## Cleanup

```bash
k delete ns argocd
k delete -f application.yaml
```

<details>
<summary>Solution</summary>

```bash
# 1. Create Argo CD namespace
k create namespace argocd

# 2. Install Argo CD (check https://argo-cd.readthedocs.io/en/stable/getting_started/)
# Use specific version, e.g., v2.10.3
k apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.10.3/manifests/install.yaml

# Note: Don't apply -f manifests/crds.yaml separately

# 3. Wait for Argo CD to be ready
k wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd

# 4. Create an Application manifest (GitOps)
cat <<EOF > application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF

# 5. Apply the Application
k apply -f application.yaml

# 6. Watch sync status
k get application -n argocd -w

# 7. Check deployment status
k get pod -n default

# 8. Verify resources created by Argo CD
k get all -n default

# 9. Access Argo CD UI (optional)
k port-forward -n argocd svc/argocd-server 8080:443
# Navigate to https://localhost:8080
# Default password: admin / (stored in secret)
# Get password: k get secret argocd-initial-admin-secret -n argocd -o jsonpath='{.data.password}' | base64 -d

# 10. View application sync status
k get application my-app -n argocd -o jsonpath='{.status.sync.status}'
# Output: Synced
```

References:
- Argo CD Docs: https://argo-cd.readthedocs.io/
- Application CRD: https://argo-cd.readthedocs.io/en/stable/declarative-setup/

</details>
