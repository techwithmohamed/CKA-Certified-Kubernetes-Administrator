# Exercise 11 — Troubleshoot Cluster Components

> Related: [README — Troubleshooting](../../README.md#domain-2--troubleshooting-30)

Fix a broken cluster. This exercise simulates common failures you'll see on the CKA: kubelet down, kube-proxy misconfigured, CoreDNS not resolving.

## Tasks

### Scenario A: Broken kubelet

1. SSH to a worker node
2. Stop the kubelet service
3. From the control plane, observe the node status change to `NotReady`
4. SSH back to the worker and check kubelet logs
5. Restart kubelet and verify the node comes back to `Ready`

### Scenario B: CoreDNS troubleshooting

1. Create a pod and try to resolve a service name — should fail if CoreDNS is broken
2. Check CoreDNS pods in `kube-system`
3. Check CoreDNS logs for errors
4. Verify the CoreDNS ConfigMap for misconfigurations
5. Check that the `kube-dns` service has endpoints
6. Fix any issues and verify DNS resolution works

### Scenario C: kube-proxy

1. Check if kube-proxy pods are running on all nodes
2. Check kube-proxy logs
3. Verify kube-proxy ConfigMap
4. Check iptables rules on a node

## Hints

- `systemctl status kubelet` and `journalctl -u kubelet` for kubelet issues
- CoreDNS pods run as a Deployment in `kube-system`
- `k exec <pod> -- nslookup kubernetes` to test DNS
- kube-proxy runs as a DaemonSet in `kube-system`
- `iptables-save | grep <service-name>` to check kube-proxy rules

## Verify

```bash
# All nodes Ready
k get nodes

# DNS works
k run test-dns --image=busybox:1.36 --rm -it -- nslookup kubernetes

# Services are routable
k get endpoints -n kube-system
```

## Cleanup

```bash
k delete pod test-dns --force 2>/dev/null
```

<details>
<summary>Solution</summary>

### Scenario A: kubelet

```bash
# SSH to worker
ssh worker-1

# Stop kubelet
sudo systemctl stop kubelet

# From control plane (different terminal)
k get nodes
# worker-1 shows NotReady after ~40 seconds

# Back on worker — check why
sudo systemctl status kubelet
sudo journalctl -u kubelet --no-pager -l | tail -30

# Fix: restart kubelet
sudo systemctl start kubelet

# Back on control plane
k get nodes
# worker-1 should be Ready again
```

### Scenario B: CoreDNS

```bash
# Check CoreDNS pods
k get pods -n kube-system -l k8s-app=kube-dns

# Check logs
k logs -n kube-system -l k8s-app=kube-dns

# Check ConfigMap
k get configmap coredns -n kube-system -o yaml

# Check endpoints
k get endpoints kube-dns -n kube-system
# Should show IPs of CoreDNS pods

# Test DNS
k run test-dns --image=busybox:1.36 --rm -it -- nslookup kubernetes.default.svc.cluster.local
```

### Scenario C: kube-proxy

```bash
# Check DaemonSet
k get ds kube-proxy -n kube-system

# Check pods
k get pods -n kube-system -l k8s-app=kube-proxy -o wide

# Check logs
k logs -n kube-system -l k8s-app=kube-proxy | tail -20

# Check ConfigMap
k get configmap kube-proxy -n kube-system -o yaml

# On a node, check iptables
sudo iptables-save | grep -c KUBE
```

</details>
