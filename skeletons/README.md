# YAML Skeletons

All YAML templates are now in [`../TEMPLATES.md`](../TEMPLATES.md) for easy browsing with collapsible sections.

Individual `.yaml` files remain here for direct reference, but the recommended approach is:

**Use collapsible templates in [`../TEMPLATES.md`](../TEMPLATES.md) for quick copy-paste.**

## Exam Strategy

**Generate with imperative commands first (faster, fewer typos):**

```bash
k run my-pod --image=nginx --dry-run=client -o yaml
k create deployment my-dep --image=nginx --replicas=3 --dry-run=client -o yaml
k expose deployment my-dep --port=80 --dry-run=client -o yaml
k create job my-job --image=busybox --dry-run=client -o yaml -- sh -c "echo done"
k create cronjob my-cron --image=busybox --schedule="*/5 * * * *" --dry-run=client -o yaml -- sh -c "date"
```

**Copy templates from [`../TEMPLATES.md`](../TEMPLATES.md) when imperative commands don't exist:**
- NetworkPolicy
- PersistentVolume / PersistentVolumeClaim
- RBAC (Role, RoleBinding, ClusterRole)
- StorageClass
- Ingress
- SecurityContext

See [`../TEMPLATES.md`](../TEMPLATES.md) for all templates in one place.
