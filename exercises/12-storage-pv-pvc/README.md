# Exercise 12 — Storage: PV, PVC, and StorageClass

> Related: [PV skeleton](../../skeletons/pv.yaml) | [PVC skeleton](../../skeletons/pvc.yaml) | [StorageClass skeleton](../../skeletons/storageclass.yaml) | [README — Storage](../../README.md#domain-1--storage-10)

Create PersistentVolumes, PersistentVolumeClaims, and mount them into pods. This covers static provisioning and StorageClass basics.

## Tasks

1. Create a namespace called `exercise-12`
2. Create a PersistentVolume named `my-pv` with:
   - Capacity: 1Gi
   - AccessMode: ReadWriteOnce
   - StorageClass: `manual`
   - hostPath: `/data/exercise-12`
   - Reclaim policy: Retain
3. Create a PersistentVolumeClaim named `my-pvc` in namespace `exercise-12`:
   - Request: 500Mi
   - AccessMode: ReadWriteOnce
   - StorageClass: `manual`
4. Verify the PVC is Bound to the PV
5. Create a pod named `storage-pod` that mounts `my-pvc` at `/usr/share/nginx/html`
6. Write a file inside the mounted volume
7. Delete the pod, create a new pod with the same PVC, and verify the file persists
8. Delete the PVC and check what happens to the PV (should be Released, not Available, because policy is Retain)

## Hints

<details>
<summary>Stuck? Click to reveal hints</summary>

- PV is cluster-scoped (no namespace). PVC is namespace-scoped.
- The PVC binds to a PV if capacity >= request AND accessMode and storageClass match
- With `Retain`, deleting the PVC releases the PV but doesn't delete the data
- With `Delete`, deleting the PVC deletes the PV and its data
- `k get pv` and `k get pvc -n <ns>` to check status

</details>

## What tripped me up

> PVC stuck in Pending for 10 minutes. I checked accessModes, checked capacity, everything matched. Turns out: my PV had `storageClassName: manual` and my PVC had `storageClassName: standard`. One word difference, no error message — just Pending forever. Always triple-check `storageClassName` matches exactly between PV and PVC.
>
> Also didn't realize PV is cluster-scoped (no namespace) but PVC is namespace-scoped. I created the PVC in `default` when the pod was in `exercise-12`. The PVC bound to the PV just fine, but the pod couldn't reference it because it was in a different namespace. PVC and pod MUST be in the same namespace.

## Verify + Cleanup

```bash
# PV should exist
k get pv my-pv

# PVC should be Bound
k get pvc my-pvc -n exercise-12

# File should persist across pod recreation
k exec storage-pod -n exercise-12 -- cat /usr/share/nginx/html/test.txt

# After deleting PVC, PV status should be Released
k get pv my-pv

# Cleanup
k delete ns exercise-12
k delete pv my-pv
sudo rm -rf /data/exercise-12
```

<details>
<summary>Solution</summary>

```bash
k create ns exercise-12

# Create the host directory (on the node)
sudo mkdir -p /data/exercise-12
```

```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /data/exercise-12
```

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  namespace: exercise-12
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: manual
```

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
  namespace: exercise-12
spec:
  containers:
  - name: nginx
    image: nginx:1.27
    volumeMounts:
    - name: data
      mountPath: /usr/share/nginx/html
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-pvc
```

```bash
k apply -f pv.yaml
k apply -f pvc.yaml

# Check binding
k get pv my-pv
k get pvc my-pvc -n exercise-12

k apply -f pod.yaml

# Write a file
k exec storage-pod -n exercise-12 -- sh -c 'echo "data persists" > /usr/share/nginx/html/test.txt'

# Delete and recreate pod
k delete pod storage-pod -n exercise-12
k apply -f pod.yaml

# Verify file persists
k exec storage-pod -n exercise-12 -- cat /usr/share/nginx/html/test.txt
# Output: data persists

# Test reclaim policy
k delete pod storage-pod -n exercise-12
k delete pvc my-pvc -n exercise-12
k get pv my-pv
# STATUS should be Released (not Available) because policy is Retain
```

</details>
