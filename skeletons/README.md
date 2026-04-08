# YAML Skeletons

Ready-to-use Kubernetes YAML templates for the CKA exam. These cover every resource type you'll encounter.

During the exam, don't write YAML from memory. Instead, use imperative kubectl commands to generate YAML — it's faster and has fewer typos. These skeletons are for reference and for resources where no imperative command exists. For everything else, use `kubectl create`, `kubectl run`, or `kubectl expose`, then pipe to `--dry-run=client -o yaml`.

| Skeleton | Kind(s) | Domain | When You Need It |
|---|---|---|---|
| [pod.yaml](pod.yaml) | Pod | Workloads | Basic pod with resources |
| [deployment.yaml](deployment.yaml) | Deployment | Workloads | Rolling updates, scaling |
| [service.yaml](service.yaml) | Service (ClusterIP) | Networking | Exposing pods internally |
| [networkpolicy.yaml](networkpolicy.yaml) | NetworkPolicy | Networking | Ingress/egress traffic control |
| [ingress.yaml](ingress.yaml) | Ingress | Networking | HTTP routing with TLS |
| [gateway-api.yaml](gateway-api.yaml) | Gateway + HTTPRoute | Networking | Gateway API (v1.35 GA) |
| [rbac.yaml](rbac.yaml) | Role + RoleBinding | Cluster Arch | Namespace-scoped permissions |
| [clusterrole.yaml](clusterrole.yaml) | ClusterRole + ClusterRoleBinding | Cluster Arch | Cluster-wide permissions |
| [pv.yaml](pv.yaml) | PersistentVolume | Storage | Cluster-scoped storage |
| [pvc.yaml](pvc.yaml) | PersistentVolumeClaim | Storage | Namespace-scoped storage request |
| [storageclass.yaml](storageclass.yaml) | StorageClass | Storage | Dynamic provisioning config |
| [daemonset.yaml](daemonset.yaml) | DaemonSet | Workloads | One pod per node (logging, monitoring) |
| [statefulset.yaml](statefulset.yaml) | StatefulSet | Workloads | Stateful apps with stable identity |
| [job.yaml](job.yaml) | Job | Workloads | Run-to-completion tasks |
| [cronjob.yaml](cronjob.yaml) | CronJob | Workloads | Scheduled tasks |
| [configmap-secret.yaml](configmap-secret.yaml) | ConfigMap + Secret | Workloads | App configuration |
| [securitycontext.yaml](securitycontext.yaml) | Pod (SecurityContext) | Workloads | Non-root, read-only, drop caps |
| [resourcequota.yaml](resourcequota.yaml) | ResourceQuota + LimitRange | Cluster Arch | Namespace resource limits |
| [hpa.yaml](hpa.yaml) | HorizontalPodAutoscaler | Workloads | CPU-based autoscaling |
| [limitrange.yaml](limitrange.yaml) | LimitRange | Cluster Arch | Default container resource constraints |
| [serviceaccount.yaml](serviceaccount.yaml) | ServiceAccount | Cluster Arch | Identity for pods and RBAC bindings |
| [sidecar-init-container.yaml](sidecar-init-container.yaml) | Pod (native sidecar) | Workloads | Init container with restartPolicy: Always (v1.35 GA) |
| [validatingadmissionpolicy.yaml](validatingadmissionpolicy.yaml) | ValidatingAdmissionPolicy + Binding | Cluster Arch | CEL-based admission control (v1.35 GA) |

## Exam Strategy: Generate vs Write

**Generate YAML with imperative commands (faster, fewer typos):**

```bash
# Pod
k run my-pod --image=nginx:1.28 --dry-run=client -o yaml

# Deployment
k create deployment my-dep --image=nginx:1.28 --replicas=3 --dry-run=client -o yaml

# Service
k expose deployment my-dep --port=80 --target-port=8080 --type=ClusterIP --dry-run=client -o yaml

# Job
k create job my-job --image=busybox:1.37 --dry-run=client -o yaml -- sh -c "echo done"

# CronJob
k create cronjob my-cron --image=busybox:1.37 --schedule="*/5 * * * *" --dry-run=client -o yaml -- sh -c "date"

# DaemonSet (no imperative command exists; use skeleton or kubectl explain)
k explain daemonset.spec
```

**When to write from YAML skeletons:**
- NetworkPolicy (no `kubectl create networkpolicy`)
- PersistentVolume / PersistentVolumeClaim (static provisioning)
- RBAC (Role, RoleBinding, ClusterRole)
- StorageClass (custom provisioner logic)
- Ingress / Gateway API (complex routing rules)
- SecurityContext (fine-grained pod security)

**Always have these available in the exam:**
- `kubectl explain <resource>` — Check valid field names and structure
- `kubectl explain pod.spec.containers` — Drill into nested fields
- kubernetes.io documentation (searchable in exam browser)

Use the skeletons in this directory as starting points when you need to write YAML. Copy, paste, modify — don't type from memory.
