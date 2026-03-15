# Exercise 07 — etcd Backup and Restore

> Related: [README — Cluster Architecture](../../README.md#domain-4--cluster-architecture-installation--configuration-25)

Back up and restore etcd. This shows up on almost every CKA exam. Know the etcdctl flags by heart.

## Tasks

1. Find the etcd pod and identify the cert/key/CA paths from its manifest
2. Take a snapshot of etcd and save it to `/tmp/etcd-backup.db`
3. Verify the snapshot is valid
4. Simulate data loss by creating a namespace `before-restore`, then deleting it won't be needed — you'll restore from the snapshot
5. Restore the snapshot to a new data directory `/var/lib/etcd-restored`
6. Update the etcd pod manifest to point to the restored data directory
7. Verify etcd comes back up and cluster is functional

## Hints

- The etcd pod manifest is at `/etc/kubernetes/manifests/etcd.yaml`
- You need three TLS flags: `--cacert`, `--cert`, `--key`
- The endpoint is usually `https://127.0.0.1:2379`
- After restore, update `--data-dir` and the corresponding `hostPath` volume in the etcd manifest
- etcd restarts automatically because it's a static pod

## Verify

```bash
# Snapshot should exist and be valid
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup.db --write-table

# After restore, cluster should work
k get nodes
k get pods -A
```

## Cleanup

```bash
# If you want to revert, restore original etcd manifest and data dir
```

<details>
<summary>Solution</summary>

```bash
# Step 1: Find cert paths from etcd manifest
cat /etc/kubernetes/manifests/etcd.yaml | grep -E "cert|key|cacert|data-dir"

# Typical paths:
# --cert-file=/etc/kubernetes/pki/etcd/server.crt
# --key-file=/etc/kubernetes/pki/etcd/server.key
# --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
# --data-dir=/var/lib/etcd

# Step 2: Take snapshot
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Step 3: Verify
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup.db --write-table

# Step 4: Restore to new directory
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
  --data-dir=/var/lib/etcd-restored

# Step 5: Update etcd manifest
# Edit /etc/kubernetes/manifests/etcd.yaml:
# 1. Change --data-dir=/var/lib/etcd to --data-dir=/var/lib/etcd-restored
# 2. Change the hostPath volume from /var/lib/etcd to /var/lib/etcd-restored

sudo vi /etc/kubernetes/manifests/etcd.yaml
# Find:   --data-dir=/var/lib/etcd
# Replace: --data-dir=/var/lib/etcd-restored
# Find:   path: /var/lib/etcd
# Replace: path: /var/lib/etcd-restored

# Step 6: Wait for etcd to restart (static pod auto-restarts)
# It may take 30-60 seconds
k get pods -n kube-system | grep etcd

# Step 7: Verify cluster
k get nodes
k get pods -A
```

Key things to remember:
- Always use `ETCDCTL_API=3`
- The three cert flags are listed in the etcd pod manifest
- After restore, update BOTH `--data-dir` AND the `hostPath` volume path
- Don't panic if kubectl stops responding for a minute — etcd is restarting

</details>
