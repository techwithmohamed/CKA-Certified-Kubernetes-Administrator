# Exercise 06 — Deployment Rolling Update and Rollback

> Related: [Deployment skeleton](../../skeletons/deployment.yaml) | [README — Workloads & Scheduling](../../README.md#domain-3--workloads--scheduling-15)

Create a Deployment, perform a rolling update, check rollout history, and rollback.

## Tasks

1. Create a namespace called `exercise-06`
2. Create a Deployment named `webapp` with:
   - 3 replicas
   - Image: `nginx:1.26`
   - Strategy: RollingUpdate with maxSurge=1, maxUnavailable=0
   - Record the change cause
3. Verify all 3 replicas are running
4. Update the image to `nginx:1.27` and record the change
5. Watch the rollout status
6. Check rollout history — you should see 2 revisions
7. Rollback to the previous version (nginx:1.26)
8. Verify the rollback worked by checking the image

## Hints

- `k create deployment webapp --image=nginx:1.26 --replicas=3`
- `k set image deployment/webapp` to update
- `k rollout status`, `k rollout history`, `k rollout undo`
- Use `--record` or annotate with `kubernetes.io/change-cause`

## Verify

```bash
# After initial deploy
k get deploy webapp -n exercise-06
k get rs -n exercise-06

# After rollback — image should be nginx:1.26
k get deploy webapp -n exercise-06 -o jsonpath='{.spec.template.spec.containers[0].image}'
```

## Cleanup

```bash
k delete ns exercise-06
```

<details>
<summary>Solution</summary>

```bash
k create ns exercise-06

# Create deployment
k create deployment webapp -n exercise-06 --image=nginx:1.26 --replicas=3 $do > deploy.yaml
```

Edit `deploy.yaml` to add rolling update strategy, then apply:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: exercise-06
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: nginx
        image: nginx:1.26
```

```bash
k apply -f deploy.yaml
k annotate deployment webapp -n exercise-06 kubernetes.io/change-cause="initial deploy nginx:1.26"

# Verify
k get deploy webapp -n exercise-06
k get pods -n exercise-06

# Update image
k set image deployment/webapp nginx=nginx:1.27 -n exercise-06
k annotate deployment webapp -n exercise-06 kubernetes.io/change-cause="update to nginx:1.27" --overwrite

# Watch rollout
k rollout status deployment/webapp -n exercise-06

# Check history
k rollout history deployment/webapp -n exercise-06

# Rollback
k rollout undo deployment/webapp -n exercise-06

# Verify rollback
k get deploy webapp -n exercise-06 -o jsonpath='{.spec.template.spec.containers[0].image}'
# Should output: nginx:1.26
```

</details>
