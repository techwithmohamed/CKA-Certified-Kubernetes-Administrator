# Exercise 16 — Horizontal Pod Autoscaler

> **Medium** | ~15 min | Domain: Workloads & Scheduling (15%)
>
> Related: [HPA skeleton](../../skeletons/hpa.yaml) | [README — Workloads & Scheduling](../../README.md#domain-3--workloads--scheduling-15)

Configure a Horizontal Pod Autoscaler to scale a deployment based on CPU usage. Also understand Vertical Pod Autoscaler for resource optimization. This requires the metrics-server to be installed in the cluster.

## Context: HPA vs VPA

## Context: HPA vs VPA

HPA (Horizontal Pod Autoscaler) scales the number of replicas based on metrics like CPU or memory. When CPU exceeds target, HPA creates more pods. When load drops, it removes pods.

VPA (Vertical Pod Autoscaler) adjusts the CPU and memory requests/limits for existing pods based on actual usage. If a pod requests 256Mi memory but only uses 64Mi, VPA recommends lowering the request. This doesn't scale replicas; it optimizes resource allocation to improve bin-packing and reduce waste.

The CKA focuses on HPA, but understanding VPA helps with cluster resource efficiency.

## Tasks (HPA)

1. Create a namespace called `exercise-16`
2. Verify the metrics-server is running in the cluster
3. Create a Deployment named `load-app` with:
   - Image: `registry.k8s.io/hpa-example` (or `nginx:1.27` with resource requests)
   - 1 replica
   - CPU request: 50m
   - CPU limit: 100m
4. Expose it with a ClusterIP Service named `load-svc` on port 80
5. Create an HPA targeting `load-app`:
   - Min replicas: 1
   - Max replicas: 5
   - Target CPU utilization: 50%
6. Generate load against the service and observe the HPA scaling up
7. Stop the load and observe the HPA scaling back down

## Hints

- `k top pods -n exercise-16` to check if metrics-server is working
- `k autoscale deployment load-app --cpu-percent=50 --min=1 --max=5`
- Load generation: `k run load-gen --image=busybox:1.36 --rm -it -- sh -c "while true; do wget -q -O- http://load-svc.exercise-16; done"`
- Scale down takes a few minutes after load stops

## What tripped me up

> HPA showed `<unknown>/50%` for CPU and never scaled. I stared at the HPA for 5 minutes thinking the metrics-server was broken. The actual problem: my Deployment didn't have `resources.requests.cpu` set. HPA calculates percentage based on *requested* CPU. No request = no baseline = `<unknown>`. Always set resource requests on pods that need autoscaling.
>
> Scale-down is slow by default — it takes 5 minutes after load stops. During practice I thought it was broken and kept restarting the HPA. It's working, it's just conservative. Don't panic if replicas stay high for a few minutes after you stop the load generator.

## Verify

```bash
# HPA created
k get hpa -n exercise-16

# Under load: replicas should increase
k get hpa -n exercise-16 -w

# Pods scaling
k get pods -n exercise-16 -w
```

## Cleanup

```bash
k delete ns exercise-16
```

<details>
<summary>Solution</summary>

```bash
k create ns exercise-16

# Check metrics-server
k get pods -n kube-system | grep metrics-server
k top nodes   # should return data
```

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: load-app
  namespace: exercise-16
spec:
  replicas: 1
  selector:
    matchLabels:
      app: load-app
  template:
    metadata:
      labels:
        app: load-app
    spec:
      containers:
      - name: app
        image: nginx:1.27
        resources:
          requests:
            cpu: "50m"
          limits:
            cpu: "100m"
        ports:
        - containerPort: 80
```

```bash
k apply -f deployment.yaml

# Create service
k expose deployment load-app -n exercise-16 --port=80 --target-port=80 --name=load-svc

# Create HPA
k autoscale deployment load-app -n exercise-16 --cpu-percent=50 --min=1 --max=5

# Check HPA
k get hpa -n exercise-16

# Generate load (run in a separate terminal)
k run load-gen -n exercise-16 --image=busybox:1.36 --rm -it -- sh -c \
  "while true; do wget -q -O- http://load-svc.exercise-16; done"

# Watch scaling in another terminal
k get hpa -n exercise-16 -w
k get pods -n exercise-16 -w

# Stop load (Ctrl+C on the load-gen terminal)
# Wait a few minutes for scale-down
```

</details>

## Optional: Vertical Pod Autoscaler (VPA) Understanding

While VPA is not always tested on the CKA, understanding it helps optimize workloads and appears in cluster efficiency questions. VPA recommends resource adjustments but doesn't enforce them automatically—you review recommendations and apply them.

VPA workflow:

```bash
# Install VPA (if not already installed)
# VPA components: Recommender, Updater, Admission Controller
# Usually installed via: git clone https://github.com/kubernetes/autoscaler.git
# ./autoscaler/vertical-pod-autoscaler/hack/vpa-up.sh

# After VPA is running, create a VPA policy
kubectl apply -f - <<EOF
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: vpa-example
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: load-app
  updatePolicy:
    updateMode: "Off"  # "Off" = recommendations only, no changes
EOF

# Check VPA recommendations (after metrics are collected)
k get vpa vpa-example --watch
k describe vpa vpa-example

# View recommended resource changes
k get vpa vpa-example -o jsonpath='{.status.recommendation}'
```

Key VPA concepts:

- updateMode: "Off" (recommendations only), "Initial" (apply on pod creation), "Recreate" (apply and restart pods), "Auto" (smart choice)
- VPA may recreate pods to apply new resource requests—plan for disruption
- VPA works best with PodDisruptionBudgets to avoid cascading disruptions
- Unlike HPA, VPA requires the Recommender to collect metrics first (takes 1-5 minutes)
