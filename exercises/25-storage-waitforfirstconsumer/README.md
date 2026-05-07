# Exercise 25 — Storage with WaitForFirstConsumer Binding

> Related: [Storage skeleton](../../skeletons/storageclass.yaml) | [README — Storage](../../README.md#domain-6--storage-10)

Create a StorageClass with `WaitForFirstConsumer` binding mode, and understand why PVCs remain Pending until a pod actually uses them.

## Tasks

1. Create a namespace called `exercise-25`
2. Create a StorageClass named `local-storage` with:
   - Provisioner: `kubernetes.io/no-provisioner` (local volumes)
   - Binding mode: `WaitForFirstConsumer`
3. Create a PersistentVolume (PV) with:
   - Name: `local-pv-1`
   - Size: 1Gi
   - AccessMode: ReadWriteOnce
   - StorageClassName: `local-storage`
   - Local path: `/tmp/k8s-pv` (or any valid node path)
4. Create a PersistentVolumeClaim (PVC) with:
   - Name: `local-pvc`
   - Size: 1Gi
   - StorageClassName: `local-storage`
5. Observe: PVC remains Pending (not Bound) — WHY?
6. Create a Pod that uses the PVC:
   - Image: `busybox:1.36`
   - Mount PVC at `/data`
7. Observe: Now PVC transitions to Bound
   - Pod schedules on the node where the PV exists
8. Verify data persistence by writing to the volume

## Key Learning

- `WaitForFirstConsumer` = PVC stays Pending until a pod needs it
- Binding is deferred to allow node affinity scheduling
- Useful for local storage where PV is tied to a specific node
- Exam tests understanding of binding modes and their tradeoffs

## Hints

<details>
<summary>Stuck? Click to reveal hints</summary>

- PVC should show `Pending` initially, not `Bound`
- Once pod is created using PVC, PVC transitions to `Bound`
- Pod scheduling respects the PV node affinity
- `k get pvc -w` to watch the status change

</details>

## What tripped me up

> I created the PVC and expected it to be Bound immediately. It stayed Pending and I thought something was wrong. Turns out `WaitForFirstConsumer` is by design — it doesn't bind until a consumer appears. On the exam, if you see a PVC stuck in Pending, check the StorageClass binding mode. It might be intentional, not a bug. The pod has to request the PVC first, then binding happens. If the pod and PV are on different nodes, the pod won't schedule.
>
> Also: don't forget the PV needs a valid `local` path on the node, or uses a real provisioner (cloud). Without a provisioner, `WaitForFirstConsumer` + local storage is commonly tested.

## Verify

```bash
# Initially: PVC should be Pending
k get pvc -n exercise-25

# After pod creation: PVC should be Bound
k get pvc -n exercise-25

# Pod should be Running
k get pod -n exercise-25

# Verify mount inside pod
k exec -it <pod> -n exercise-25 -- ls /data

# Write test data
k exec -it <pod> -n exercise-25 -- sh -c 'echo "test data" > /data/test.txt'

# Read it back
k exec -it <pod> -n exercise-25 -- cat /data/test.txt
```

## Cleanup

```bash
k delete ns exercise-25
k delete pv local-pv-1
k delete storageclass local-storage
```

<details>
<summary>Solution</summary>

```bash
k create ns exercise-25
kn exercise-25

# Create StorageClass with WaitForFirstConsumer
cat <<EOF | k apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF

# Create PersistentVolume
cat <<EOF | k apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  local:
    path: /tmp/k8s-pv
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - <node-name>  # Replace with actual node name
EOF

# Get node name if needed
k get nodes -o jsonpath='{.items[0].metadata.name}'

# Verify PV created
k get pv

# Create PVC — will be Pending
cat <<EOF | k apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
  namespace: exercise-25
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 1Gi
EOF

# Check status — should be Pending
k get pvc

# Create Pod using PVC
cat <<EOF | k apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  namespace: exercise-25
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sleep", "3600"]
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: local-pvc
EOF

# Now PVC should be Bound
k get pvc -w
# Watch it change from Pending to Bound

# Verify pod is running
k get pod

# Test write
k exec app-pod -n exercise-25 -- sh -c 'echo "hello" > /data/test.txt'
k exec app-pod -n exercise-25 -- cat /data/test.txt

# Cleanup
k delete ns exercise-25
k delete pv local-pv-1
k delete storageclass local-storage
```

</details>
