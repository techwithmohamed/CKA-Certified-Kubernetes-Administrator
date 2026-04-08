# Exercise 22 — PriorityClass

**Domain:** Workloads & Scheduling (15%)

**Difficulty:** Medium | **Time:** 15 min

---

## Context

PriorityClass defines the relative importance of pods. When nodes have limited resources, the scheduler prioritizes high-priority pods first. If a high-priority pod can't fit on any node, the scheduler can evict lower-priority pods to make room (preemption).

On the exam, you'll create PriorityClasses and assign them to pods. You need to know the difference between priority values, how preemption works, and how to debug which pods get scheduled when resources are tight.

---

## Tasks

1. Create a PriorityClass named `high-priority` with value 1000 and preemptionPolicy set to `PreemptLowerPriority`.

2. Create a PriorityClass named `low-priority` with value 10 and preemptionPolicy set to `Never`.

3. Create a Pod named `critical-app` using `nginx:1.28` and assign it to the `high-priority` class.

4. Create a Pod named `background-job` using `busybox:1.37` to run `sleep 3600` and assign it to the `low-priority` class.

5. Verify both pods are running. Check their priority values with `kubectl get pod -o json` and filter for `priority` and `priorityClassName`.

6. If possible, drain a node to trigger preemption and observe which pod gets evicted.

---

## Hints

<details>
<summary>Stuck? Click to reveal hints</summary>

- **Priority values**: Higher number = higher priority. System pods use 2000000000 (reserved by Kubernetes). Use 1-1000 for application priorities to stay safe.
- **preemptionPolicy: PreemptLowerPriority**: Allow this priority class to evict lower-priority pods if resources are scarce.
- **preemptionPolicy: Never**: This priority class will not evict or be evicted, even if it can't fit. It'll just wait.
- Default priority is 0. Pods without an explicit PriorityClass use priority 0.
- You must create a PriorityClass with `kind: PriorityClass` (not a built-in). PriorityClasses are cluster-wide, not namespaced.
- Preemption only happens when the scheduler can't fit a pod and has eviction candidates. It's not automatic.

</details>

---

## What tripped me up

I thought higher priority pods always run first. They do — but only when the scheduler evaluates new pods. If low-priority pods are already running and using resources, the high-priority pod still has to wait unless preemption kicks in.

I also confused priority (user-defined) with QoS (Quality of Service: Guaranteed, Burstable, BestEffort), which is automatic based on resource limits. They're different and work together.

Creating a Pod with a nonexistent PriorityClass doesn't fail — it just fails to schedule (pending forever). No error message. You have to check the events: `kubectl describe pod <name>`.

---

## Verify

```bash
# Check PriorityClasses exist
k get priorityclass -o wide

# Check pod priority
k get pod -o wide
k get pod critical-app -o json | jq '.spec.priorityClassName, .spec.priority'
k get pod background-job -o json | jq '.spec.priorityClassName, .spec.priority'

# Check pod scheduling
k describe pod critical-app
k describe pod background-job

# Monitor preemption (if draining a node)
k get events --sort-by='.lastTimestamp'
```

---

## Cleanup

```bash
k delete pod critical-app
k delete pod background-job
k delete priorityclass high-priority
k delete priorityclass low-priority
```

---

<details>
<summary><b>Solution</b></summary>

**Step 1: Create PriorityClasses**

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: "High priority for critical applications"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 10
preemptionPolicy: Never
globalDefault: false
description: "Low priority for background jobs"
```

```bash
k apply -f priorityclasses.yaml
```

Verify:
```bash
k get priorityclass
# NAME              VALUE        GLOBAL-DEFAULT   AGE
# high-priority     1000         false            5s
# low-priority      10           false            5s
```

**Step 2: Create Pods with PriorityClasses**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-app
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: nginx:1.28
---
apiVersion: v1
kind: Pod
metadata:
  name: background-job
spec:
  priorityClassName: low-priority
  containers:
  - name: worker
    image: busybox:1.37
    command: ["sleep", "3600"]
```

```bash
k apply -f pods.yaml
```

**Step 3: Verify Priority**

```bash
k get pod -o wide
k get pod critical-app -o json | jq '.spec.priorityClassName'
# "high-priority"

k get pod critical-app -o json | jq '.spec.priority'
# 1000

k get pod background-job -o json | jq '.spec.priority'
# 10
```

**Step 4: Trigger Preemption (optional)**

If you have a multi-node cluster, drain a node:

```bash
k drain <node> --ignore-daemonsets --delete-emptydir-data
```

Monitor with:
```bash
k get events --sort-by='.lastTimestamp'
```

The `background-job` pod should be evicted (assuming resources force it), and `critical-app` should remain running because it has higher priority.

**Step 5: Cleanup**

```bash
k delete pod critical-app background-job
k delete priorityclass high-priority low-priority
```

</details>

---

## Resources

- [Pod Priority and Preemption](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/)
- [PriorityClass API](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/priority-class-v1/)
- [Preemption and Priority](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#preemption)
