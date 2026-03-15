# YAML Skeletons

Ready-to-use Kubernetes YAML templates for the CKA exam. These cover every resource type you'll encounter.

During the exam, I wrote most of these from memory instead of copying from docs. It was faster. Practice writing them until you don't need to look anything up.

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

## Quick Reference

Generate YAML on the fly instead of copying from here:

```bash
# Pod
k run my-pod --image=nginx:1.27 $do > pod.yaml

# Deployment
k create deployment my-dep --image=nginx:1.27 --replicas=3 $do > dep.yaml

# Service
k expose deployment my-dep --port=80 --target-port=80 $do > svc.yaml

# Job
k create job my-job --image=busybox:1.36 -- sh -c "echo done" $do > job.yaml

# CronJob
k create cronjob my-cron --image=busybox:1.36 --schedule="*/5 * * * *" -- sh -c "date" $do > cron.yaml
```

For anything more complex (NetworkPolicy, PV, StatefulSet, DaemonSet), use the skeletons in this directory as your starting point.
