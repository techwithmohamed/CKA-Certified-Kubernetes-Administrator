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

### Scenario D: API Server audit logs (Kubernetes 1.35 troubleshooting)

1. Find where audit logs are stored on the control plane node
2. Check the audit policy configuration in the apiserver manifest
3. Look for failed authentication attempts in the audit log
4. Find requests that were denied by RBAC
5. Correlate user identity to request details

## Hints

<details>
<summary>Stuck? Click to reveal hints</summary>

- `systemctl status kubelet` and `journalctl -u kubelet` for kubelet issues
- CoreDNS pods run as a Deployment in `kube-system`
- `k exec <pod> -- nslookup kubernetes` to test DNS
- kube-proxy runs as a DaemonSet in `kube-system`
- `iptables-save | grep <service-name>` to check kube-proxy rules
- Audit logs typically at `/var/log/audit/audit.log` or `/var/log/kubernetes/audit/audit.log`
- Check `/etc/kubernetes/manifests/kube-apiserver.yaml` for `--audit-log-path` and `--audit-policy-file`
- Look for `"verb":"delete"` or `"user":{"username":"system:admin"}` in audit logs to find specific operations

</details>

## What tripped me up

> My first instinct when a node is NotReady was to run `kubectl describe node` from the control plane. That gives you conditions but not the root cause. The real answer is almost always in `journalctl -u kubelet` on the broken node itself. SSH first, check kubelet logs, then work backwards. I wasted 7 minutes on practice exam questions by troubleshooting from the wrong machine.
>
> For CoreDNS: `nslookup` failing doesn't always mean CoreDNS is broken. Check `resolv.conf` inside the pod first — I had a case where the pod's DNS config was pointing to the wrong IP because of a network policy blocking UDP 53 to the kube-dns service. The CoreDNS pods were fine; the traffic just couldn't reach them.
>
> For audit logs: I initially searched `grep "error"` which gave me nothing useful. The audit log format is JSON — you need to grep for specific verb patterns like `grep "verb.*delete"` or look for status codes. Also, not all clusters enable audit logging, so check if the apiserver was even started with audit flags before spending time looking for logs that don't exist.

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
k run test-dns --image=busybox:1.37 --rm -it -- nslookup kubernetes.default.svc.cluster.local
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

### Scenario D: API Server audit logs

```bash
# SSH to control plane node
ssh control-plane

# Check apiserver manifest for audit configuration
sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -i audit

# Find audit log file (look for --audit-log-path flag)
sudo cat /var/log/audit/audit.log | jq . | head -50

# Find failed authentication attempts
sudo cat /var/log/audit/audit.log | jq 'select(.user.username=="unknown" or .status.code>=400)' | head -20

# Find specific user actions (e.g., resource deletion)
sudo tail -100 /var/log/audit/audit.log | jq 'select(.verb=="delete")'

# Check request count by verb
sudo cat /var/log/audit/audit.log | jq -r '.verb' | sort | uniq -c | sort -rn

# Search for RBAC denials
sudo cat /var/log/audit/audit.log | jq 'select(.stage=="ResponseComplete" and .status.code>=400)'
```

</details>
