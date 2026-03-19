# CKA Exercises

Hands-on labs covering all five CKA exam domains. Each exercise has tasks, hints, verification steps, and a full solution.

Start with the basics and work your way down. The difficulty increases as you go.

| # | Exercise | Domain | Difficulty | Time |
|---|---|---|---|---|
| 01 | [Pod Basics](01-pod-basics/) | Workloads & Scheduling | Easy | 10 min |
| 02 | [Multi-Container Pod](02-multi-container-pod/) | Workloads & Scheduling | Medium | 15 min |
| 03 | [ConfigMap & Secret](03-configmap-secret/) | Workloads & Scheduling | Easy | 10 min |
| 04 | [RBAC](04-rbac/) | Cluster Architecture | Medium | 15 min |
| 05 | [NetworkPolicy](05-networkpolicy/) | Services & Networking | Medium | 20 min |
| 06 | [Deployment Rollout](06-deployment-rollout/) | Workloads & Scheduling | Easy | 10 min |
| 07 | [etcd Backup & Restore](07-etcd-backup-restore/) | Cluster Architecture | Hard | 20 min |
| 08 | [Node Drain & Cordon](08-node-drain-cordon/) | Cluster Architecture | Easy | 10 min |
| 09 | [kubeadm Upgrade](09-kubeadm-upgrade/) | Cluster Architecture | Hard | 25 min |
| 10 | [Static Pod](10-static-pod/) | Workloads & Scheduling | Easy | 10 min |
| 11 | [Troubleshoot Cluster](11-troubleshoot-cluster/) | Troubleshooting | Hard | 25 min |
| 12 | [Storage — PV & PVC](12-storage-pv-pvc/) | Storage | Medium | 15 min |
| 13 | [Helm Install & Upgrade](13-helm-install-upgrade/) | Cluster Architecture | Medium | 15 min |
| 14 | [Kustomize Overlays](14-kustomize-overlays/) | Cluster Architecture | Medium | 15 min |
| 15 | [Gateway API](15-gateway-api/) | Services & Networking | Medium | 20 min |
| 16 | [Horizontal Pod Autoscaler](16-hpa/) | Workloads & Scheduling | Medium | 15 min |
| 17 | [kubectl debug](17-kubectl-debug/) | Troubleshooting | Medium | 15 min |

## Domain Coverage

| Domain | Weight | Exercises |
|---|---|---|
| Troubleshooting | 30% | 11, 17 |
| Cluster Architecture | 25% | 04, 07, 08, 09, 13, 14 |
| Services & Networking | 20% | 05, 15 |
| Workloads & Scheduling | 15% | 01, 02, 03, 06, 10, 16 |
| Storage | 10% | 12 |

## How to Use

1. Read the exercise description
2. Try the tasks without looking at the solution
3. Use the hints if you're stuck
4. Check your work with the verification steps
5. Compare against the solution
6. Run cleanup before moving to the next exercise

Every exercise assumes you have a running cluster (kind, minikube, or kubeadm) and the aliases from [`scripts/exam-setup.sh`](../scripts/exam-setup.sh).
