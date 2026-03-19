# Exercise 13 — Helm Install, Upgrade, and Rollback

> **Medium** | ~15 min | Domain: Cluster Architecture (25%)
>
> Related: [README — Cluster Architecture](../../README.md#domain-4--cluster-architecture-installation--configuration-25)

Install a chart, override values, upgrade to a new version, and roll back when something breaks. Helm is now a standard operational skill on the CKA.

## Tasks

1. Add the Bitnami chart repository and update the repo cache
2. Install NGINX using Helm in a namespace called `exercise-13`:
   - Release name: `web`
   - Set `replicaCount=2`
3. Verify the release is deployed and 2 pods are running
4. Upgrade the release to `replicaCount=3` and record the change
5. Check the release history and confirm there are 2 revisions
6. Roll back to revision 1
7. Verify the replica count is back to 2

## Hints

- `helm repo add bitnami https://charts.bitnami.com/bitnami`
- `helm install <name> <chart> -n <ns> --create-namespace --set key=value`
- `helm upgrade <name> <chart> -n <ns> --set key=value`
- `helm rollback <name> <revision> -n <ns>`
- `helm history <name> -n <ns>`

## Verify

```bash
# Release deployed
helm list -n exercise-13

# Pods running
k get pods -n exercise-13

# After rollback
k get deploy -n exercise-13 -o jsonpath='{.items[0].spec.replicas}'
# Should output 2
```

## Cleanup

```bash
helm uninstall web -n exercise-13
k delete ns exercise-13
```

<details>
<summary>Solution</summary>

```bash
# Add repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Install
helm install web bitnami/nginx -n exercise-13 --create-namespace --set replicaCount=2

# Verify
helm list -n exercise-13
k get pods -n exercise-13

# Upgrade
helm upgrade web bitnami/nginx -n exercise-13 --set replicaCount=3

# Check history
helm history web -n exercise-13

# Rollback
helm rollback web 1 -n exercise-13

# Verify rollback
k get deploy -n exercise-13 -o jsonpath='{.items[0].spec.replicas}'
helm history web -n exercise-13
```

</details>
