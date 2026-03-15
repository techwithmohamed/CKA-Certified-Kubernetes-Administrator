# Exercise 08 — Node Drain and Cordon

> Related: [README — Cluster Architecture](../../README.md#domain-4--cluster-architecture-installation--configuration-25)

Drain a worker node for maintenance, then bring it back. This tests your understanding of pod eviction, DaemonSets, and scheduling.

## Tasks

1. List all nodes and identify a worker node
2. Cordon the worker node (mark it unschedulable)
3. Verify the node shows `SchedulingDisabled`
4. Create a Deployment with 3 replicas and observe where pods are scheduled
5. Drain the worker node — handle DaemonSets and local data
6. Verify all non-DaemonSet pods have been evicted from the node
7. Uncordon the node
8. Scale the deployment to 6 replicas and verify pods get scheduled on the uncordoned node

## Hints

- `k cordon <node>` marks unschedulable but doesn't evict existing pods
- `k drain <node> --ignore-daemonsets --delete-emptydir-data` evicts pods
- Without `--ignore-daemonsets`, drain fails if DaemonSet pods exist
- Without `--delete-emptydir-data`, drain fails if pods use emptyDir volumes
- `k uncordon <node>` makes the node schedulable again

## Verify

```bash
# After cordon
k get nodes
# Worker should show SchedulingDisabled

# After drain
k get pods -o wide
# No pods on the drained node (except DaemonSets)

# After uncordon + scale
k get pods -o wide
# Pods should spread across nodes again
```

## Cleanup

```bash
k uncordon <node-name>
k delete deployment drain-test
```

<details>
<summary>Solution</summary>

```bash
# List nodes
k get nodes
# Pick a worker node, e.g., worker-1

# Cordon
k cordon worker-1
k get nodes
# worker-1 should show SchedulingDisabled

# Create deployment
k create deployment drain-test --image=nginx:1.27 --replicas=3

# Check pod placement
k get pods -o wide

# Drain
k drain worker-1 --ignore-daemonsets --delete-emptydir-data

# Verify eviction
k get pods -o wide
# All drain-test pods should be on other nodes

# Check node
k get pods -A --field-selector spec.nodeName=worker-1
# Only DaemonSet pods should remain

# Uncordon
k uncordon worker-1
k get nodes
# worker-1 should be Ready (no SchedulingDisabled)

# Scale up — new pods should land on worker-1 too
k scale deployment drain-test --replicas=6
k get pods -o wide
```

</details>
