# Exercise 20 — Pod Security Standards

> Related: [Security Context skeleton](../../skeletons/securitycontext.yaml) | [README — Cluster Architecture](../../README.md#domain-4--cluster-architecture-installation--configuration-25)

Implement Pod Security Standards (PSS) at the namespace level to enforce security policies. This exercise covers enforcing restricted, baseline, and restricted security contexts on pods.

## Context

Pod Security Standards replaced Pod Security Policies in Kubernetes 1.25+. They enforce security constraints at the namespace level through labels. The CKA tests your ability to:
- Apply PSS labels to namespaces
- Understand the three levels: restricted, baseline, and privileged
- Diagnose when pods violate security policies
- Use audit and warn modes for testing before enforcement

In Kubernetes 1.35, Pod Security Standards are part of the standard admission control process.

## Tasks

1. Create a namespace called `exercise-20`
2. Label it with Pod Security Standards at `restricted` level in `enforce` mode
3. Attempt to run a pod with a privileged container in `exercise-20`—it should be rejected
4. Create a second namespace `exercise-20-baseline` with `baseline` PSS level
5. Run a pod with `runAsNonRoot: true` in the baseline namespace—it should succeed
6. Run a pod with `privileged: true` in the baseline namespace—it should be rejected
7. Create a third namespace `exercise-20-audit` with `restricted` level but in `audit` mode
8. Run a privileged pod in audit mode—it should succeed but be logged as a violation
9. Check the audit annotations on the pod that ran in audit mode

## Hints

- Namespace labels for PSS use format: `pod-security.kubernetes.io/enforce=restricted`
- Three modes: `enforce` (deny), `audit` (allow but log), `warn` (allow but show warning)
- Pod security levels: `privileged` (no restrictions), `baseline` (minimal restrictions), `restricted` (modern hardening)
- `k label ns <namespace> pod-security.kubernetes.io/enforce=restricted` to add PSS labels
- `k get pods -o yaml` shows audit annotations in `metadata.annotations` when violations occur
- A "restricted" level pod must have: `runAsNonRoot=true`, `allowPrivilegeEscalation=false`, `readOnlyRootFilesystem=true`

## What tripped me up

> I labeled a namespace and then ran a privileged pod expecting it to fail. It failed immediately with "Pod rejected by Pod Security Policy" error. But my pod was already created! Actually, Kubernetes prevents creation but still leaves a cache artifact. The pod appears in `k get pods` as "Pending" with a reason "Pod Security Policy violation". I learned to check pod status, not just creation response.
>
> The PSS label format was wrong the first time. I wrote `pod-security/enforce=restricted` but it should be `pod-security.kubernetes.io/enforce=restricted` (full path with `.io`). Labels with wrong paths are silently ignored, so the namespace never got PSS applied. Always double-check label syntax.
>
> Audit mode is subtle. The pod runs successfully, but you have to describe the namespace or check the API server audit logs to see violations were recorded. In exam conditions, I almost thought audit mode wasn't working because the pod started fine. It was working correctly—that's the point of audit mode.

## Verify

```bash
# Create restricted namespace
k create ns exercise-20
k label ns exercise-20 pod-security.kubernetes.io/enforce=restricted

# Try to run privileged pod (should fail)
k run rogue --image=nginx:1.28 --privileged -n exercise-20
# Expected: Pod is rejected with error

# Create baseline namespace
k create ns exercise-20-baseline
k label ns exercise-20-baseline pod-security.kubernetes.io/enforce=baseline

# Try baseline pod with non-root (should succeed)
k run safe-pod --image=nginx:1.28 -n exercise-20-baseline
k get pods -n exercise-20-baseline

# Try privileged in baseline (should fail)
k run priv-pod --image=nginx:1.28 --privileged -n exercise-20-baseline
# Expected: Pod rejected

# Create audit namespace
k create ns exercise-20-audit
k label ns exercise-20-audit pod-security.kubernetes.io/enforce=restricted pod-security.kubernetes.io/audit=restricted

# Run pod in audit mode (should succeed but log violation)
k run audit-pod --image=nginx:1.28 -n exercise-20-audit
k describe pod audit-pod -n exercise-20-audit
# Check annotations for security violations
```

## Cleanup

```bash
k delete ns exercise-20 exercise-20-baseline exercise-20-audit
```

<details>
<summary>Solution</summary>

```bash
# Create and label namespaces with PSS
k create ns exercise-20
k label ns exercise-20 pod-security.kubernetes.io/enforce=restricted

k create ns exercise-20-baseline
k label ns exercise-20-baseline pod-security.kubernetes.io/enforce=baseline

k create ns exercise-20-audit
k label ns exercise-20-audit \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted

# Test restricted enforcement
# This command will fail:
kubectl run restricted-test --image=nginx:1.28 -n exercise-20 --dry-run=server
# Error: pods "restricted-test" is forbidden: violates PodSecurityPolicy: ...

# Create compliant pod for restricted namespace
cat <<EOF | k apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
  namespace: exercise-20
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: nginx
    image: nginx:1.28
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /var/cache/nginx
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
EOF

# Test baseline
kubectl run baseline-pod --image=nginx:1.28 -n exercise-20-baseline
k get pods -n exercise-20-baseline

# Test audit mode
kubectl run audit-test --image=nginx:1.28 -n exercise-20-audit
k describe pod audit-test -n exercise-20-audit
# Look for pod-security.kubernetes.io/restricted annotation showing violations

# Verify labels were applied correctly
k get ns exercise-20 exercise-20-baseline exercise-20-audit -o json | jq '.items[].metadata.labels'
```

</details>
