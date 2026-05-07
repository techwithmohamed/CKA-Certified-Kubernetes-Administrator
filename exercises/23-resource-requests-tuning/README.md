# Exercise 23 — Resource Requests Tuning (Pods Won't Schedule)

> Related: [Deployment skeleton](../../skeletons/deployment.yaml) | [README — Scheduling](../../README.md#domain-3--workloads--scheduling-15)

Debug and fix a Deployment where pods won't schedule due to insufficient resource requests. Learn to calculate proper requests based on node capacity.

## Tasks

1. Create a namespace called `exercise-23`
2. Create a Deployment with:
   - 3 replicas
   - Image: `nginx:1.27`
   - Resource requests: 512Mi memory, 500m CPU (intentionally too high)
   - Cluster has 2 nodes with limited resources
3. Observe pods pending — they won't schedule
4. Check node capacity and available resources
5. Calculate correct resource requests to fit 3 replicas
6. Update Deployment with new (lower) requests
7. Verify all 3 pods schedule and run
8. Document the calculation

## Key Learning

The exam trick: **Do NOT use fixed rules like "10% overhead"** — calculate based on actual node allocatable resources and what's already running.

Formula:
```
Per-pod request = (Node Allocatable - System Reserved - Already Running) / Number of Replicas
Add buffer of 5-10%
```

## Hints

<details>
<summary>Stuck? Click to reveal hints</summary>

- `k get nodes` → see available nodes
- `k describe node <node>` → see Allocatable and Allocated resources
- `k top nodes` → see actual usage
- If pods are Pending: `k describe pod <pod>` → check Events for "Insufficient memory"
- Start with 100m CPU and 64Mi memory per replica, then adjust based on errors
- Reduce requests incrementally until all 3 pods schedule

</details>

## What tripped me up

> I used the standard "reserve 10% per node" rule and pods still didn't schedule. Turns out on the lab cluster, nodes had way more system pods running. I wasted 8 minutes before checking `k describe node` to see actual allocatable vs allocated. The real exam has variable node sizes and random system pods — you MUST verify actual capacity, not assume. Then divide evenly across replicas, not just apply a formula.
>
> Also: resource requests are hard limits for scheduling. If a pod requests 512Mi and node has only 256Mi free, it will NEVER schedule, even if the pod only uses 10Mi at runtime. Requests are for the scheduler, not for actual usage. On the exam, if pods won't schedule, first check if requests are too high via `k describe node`.

## Verify

```bash
# Should see Pending pods with reason "Insufficient memory/cpu"
k get pods -n exercise-23

# Check why pods won't schedule
k describe pod <pod-name> -n exercise-23

# Check node capacity
k describe node <node-name>

# After fix — all 3 should be Running
k get pods -n exercise-23

# Check actual resource usage (much lower than requests)
k top pods -n exercise-23
```

## Cleanup

```bash
k delete ns exercise-23
```

<details>
<summary>Solution</summary>

```bash
k create ns exercise-23
kn exercise-23

# Create deployment with too-high requests (will fail to schedule)
k create deployment web --image=nginx:1.27 --replicas=3 $do > deploy.yaml
```

Edit `deploy.yaml` to add initial (intentionally high) requests:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: exercise-23
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
```

```bash
k apply -f deploy.yaml

# Check pods — they are Pending
k get pods

# Describe to see error
k describe pod <pod-name>
# Output: "Insufficient memory"

# Check node capacity
k describe node <node-name>
# Allocatable:  2000m (CPU)
#              2Gi (Memory)
# With 3 replicas, need to divide available resources

# Calculate: If node has 2Gi available and 2 cores (2000m)
# Per pod: ~600Mi memory and 600m CPU (with overhead)
# But if already running system pods, reduce further

# Edit deploy.yaml with lower values
```

Fix the requests to something realistic:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
```

```bash
k apply -f deploy.yaml

# Verify all 3 pods now Running
k get pods -n exercise-23

# Check actual usage (will be much lower than requests)
k top pods -n exercise-23

# Cleanup
k delete ns exercise-23
```

</details>
