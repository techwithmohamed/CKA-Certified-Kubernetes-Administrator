# CKA Troubleshooting Playbook

Symptom-based lookup. Find the problem, follow the steps.

---

## Pod is Pending

The scheduler can't place the pod on any node.

```bash
k describe pod <pod> -n <ns> | grep -A10 Events
```

| Cause | Fix |
|---|---|
| Insufficient CPU/memory | Scale down other pods, add nodes, or reduce resource requests |
| No matching node (nodeSelector/affinity) | Fix nodeSelector labels or add labels to nodes |
| Taint without toleration | Add toleration to pod spec or remove taint from node |
| PVC not bound | Check PVC — storageClassName, accessModes, capacity must match a PV |
| Too many pods (ResourceQuota) | Delete pods or increase quota |

```bash
# Check node resources
k describe nodes | grep -A5 "Allocated resources"

# Check taints
k describe nodes | grep Taints

# Check PVC
k get pvc -n <ns>
```

---

## Pod is CrashLoopBackOff

The container starts, crashes, restarts, crashes again.

```bash
k logs <pod> -n <ns> --previous    # Logs from the crashed container
k describe pod <pod> -n <ns>       # Exit code in Last State
```

| Exit Code | Meaning | Fix |
|---|---|---|
| 0 | Completed (but restartPolicy says restart) | Change restartPolicy or fix command to keep running |
| 1 | Application error | Check logs — wrong config, missing file, bad command |
| 127 | Command not found | Fix `command` or `args` in pod spec |
| 137 | OOMKilled (128 + 9) | Increase memory limit |
| 139 | Segfault (128 + 11) | Application bug or wrong image |

```bash
# Common fixes
k edit pod <pod> -n <ns>       # Fix command/args/env
k set env deployment/<dep> KEY=value  # Fix env vars
k get cm,secret -n <ns>       # Check if ConfigMap/Secret exists
```

---

## Pod is ImagePullBackOff

Kubernetes can't pull the container image.

```bash
k describe pod <pod> -n <ns> | grep -A3 "Warning.*Failed"
```

| Cause | Fix |
|---|---|
| Wrong image name or tag | Fix the image field — check for typos |
| Private registry, no credentials | Add `imagePullSecrets` to pod spec |
| Network issues | Check node internet access, DNS resolution |
| Image doesn't exist | Verify image exists in the registry |

```bash
# Check image
k get pod <pod> -n <ns> -o jsonpath='{.spec.containers[*].image}'
```

---

## Node is NotReady

```bash
k describe node <node> | grep -A5 Conditions
```

| Cause | Fix |
|---|---|
| kubelet not running | `ssh <node>`, then `sudo systemctl start kubelet && sudo systemctl enable kubelet` |
| kubelet misconfigured | `sudo journalctl -u kubelet --no-pager | tail -30` — fix config |
| Expired certificates | `sudo kubeadm certs check-expiration`, then `sudo kubeadm certs renew all` |
| Swap enabled | `sudo swapoff -a` |
| CNI plugin missing | Install CNI (Calico, Flannel) |
| Disk pressure | Free up disk space on the node |

```bash
# On the node
sudo systemctl status kubelet
sudo journalctl -u kubelet --no-pager | tail -50
sudo cat /var/lib/kubelet/config.yaml
```

---

## Service Has No Endpoints

Traffic to the service goes nowhere.

```bash
k get endpoints <svc> -n <ns>
# If empty: selector doesn't match any running pod
```

| Cause | Fix |
|---|---|
| Selector doesn't match pod labels | Fix service selector or pod labels |
| Pod not running | Fix the pod first |
| Wrong namespace | Service and pods must be in the same namespace |

```bash
# Compare
k describe svc <svc> -n <ns> | grep Selector
k get pods -n <ns> --show-labels

# Quick fix
k patch svc <svc> -n <ns> -p '{"spec":{"selector":{"app":"correct-label"}}}'
```

---

## Service Reachable but Wrong Response

Endpoints exist but the response is wrong.

| Cause | Fix |
|---|---|
| targetPort doesn't match container port | Fix `targetPort` in service spec |
| Wrong pod behind service (label collision) | Make labels more specific |
| Pod serving wrong content | Check pod logs and config |

```bash
k get svc <svc> -n <ns> -o yaml | grep -A5 ports
k exec <pod> -n <ns> -- wget -qO- localhost:<port>
```

---

## DNS Not Resolving

Pods can't reach services by name.

```bash
k run test-dns --image=busybox:1.36 --rm -it -- nslookup kubernetes.default.svc.cluster.local
```

| Check | Command |
|---|---|
| CoreDNS running? | `k get pods -n kube-system -l k8s-app=kube-dns` |
| CoreDNS has endpoints? | `k get endpoints kube-dns -n kube-system` |
| Corefile correct? | `k get cm coredns -n kube-system -o yaml` |
| Pod DNS config? | `k exec <pod> -- cat /etc/resolv.conf` |

Common fix: restart CoreDNS — `k rollout restart deployment coredns -n kube-system`

---

## NetworkPolicy Blocking Traffic

Traffic was working, now it's not (or vice versa).

```bash
k get netpol -n <ns>
k describe netpol <name> -n <ns>
```

Remember: **any** NetworkPolicy applied to a pod = default deny for that pod. Only explicitly allowed traffic gets through.

| Symptom | Cause |
|---|---|
| Pod can't reach anything | Egress policy missing DNS allow (UDP 53) |
| Pod can't be reached | Ingress policy doesn't match source pod labels |
| Works in one namespace, not another | namespaceSelector missing or wrong |

```bash
# Test connectivity
k exec <source-pod> -- wget -qO- --timeout=2 http://<target-svc>

# Check if policy matches the pod
k get pods -n <ns> --show-labels
# Compare labels with netpol podSelector
```

---

## etcd Issues

```bash
# Check etcd pod
k get pods -n kube-system | grep etcd
k logs etcd-<node> -n kube-system

# Check etcd health
ETCDCTL_API=3 etcdctl endpoint health \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

| Symptom | Fix |
|---|---|
| etcd pod not running | Check `/etc/kubernetes/manifests/etcd.yaml` for errors |
| Data loss after restore | Did you update both `--data-dir` AND `hostPath.path`? |
| kubectl hangs after restore | Wait 30-60 seconds — etcd is restarting |
| Wrong cert paths | Find them: `cat /etc/kubernetes/manifests/etcd.yaml | grep -E "cert|key|ca"` |

---

## Control Plane Down

kube-apiserver, kube-scheduler, or kube-controller-manager not working.

```bash
# Check static pod manifests
ls /etc/kubernetes/manifests/

# Check pods
k get pods -n kube-system | grep -E "apiserver|scheduler|controller"

# If kubectl doesn't work, check container runtime directly
sudo crictl pods | grep kube
sudo crictl logs <container-id>
```

| Component Down | Impact |
|---|---|
| kube-apiserver | kubectl doesn't work, no API access |
| kube-scheduler | New pods stay Pending |
| kube-controller-manager | Deployments don't create ReplicaSets, no self-healing |
| etcd | Everything breaks — API server can't store/retrieve data |

Fix: check the manifest YAML in `/etc/kubernetes/manifests/` for typos, wrong image, wrong flags, bad volume mounts.

---

## Quick Diagnostic Commands

```bash
# Everything at a glance
k get nodes
k get pods -A -o wide
k get events -A --sort-by=.metadata.creationTimestamp | tail -20

# Node-level
k describe node <node> | grep -E "Conditions|Taints|Allocatable" -A5
k top nodes

# Pod-level
k describe pod <pod> -n <ns>
k logs <pod> -n <ns> --all-containers
k exec <pod> -n <ns> -- env
k exec <pod> -n <ns> -- cat /etc/resolv.conf
```
