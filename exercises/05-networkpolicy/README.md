# Exercise 05 — NetworkPolicy

> Related: [NetworkPolicy skeleton](../../skeletons/networkpolicy.yaml) | [README — Services & Networking](../../README.md#domain-5--services--networking-20)

Create NetworkPolicies to control pod-to-pod traffic. Watch out for the DNS gotcha.

## Tasks

1. Create a namespace called `exercise-05`
2. Deploy two pods:
   - `frontend` with image `nginx:1.27` and label `role=frontend`
   - `backend` with image `nginx:1.27` and label `role=backend`
3. Verify that `frontend` can reach `backend` on port 80 (should work before any policy)
4. Create a NetworkPolicy named `backend-policy` that:
   - Applies to pods with label `role=backend`
   - Allows ingress only from pods with label `role=frontend` on port 80
   - Allows egress to DNS (UDP port 53) — if you skip this, DNS breaks
5. Verify that `frontend` can still reach `backend`
6. Create a third pod `attacker` with label `role=attacker` and verify it cannot reach `backend`

## Hints

- Once you apply any NetworkPolicy to a pod, all traffic not explicitly allowed is denied
- You almost always need to allow DNS egress (UDP 53), or the pod can't resolve service names
- Test connectivity with `k exec <pod> -- wget -qO- --timeout=2 http://<target-ip>`

## What tripped me up

> I wrote a perfect ingress rule but forgot egress entirely. The backend pod could receive traffic but couldn't respond to DNS lookups — so any test using service names failed. I thought my ingress rule was wrong and spent 8 minutes rewriting it. The fix was two lines: allow UDP 53 egress. I've made this mistake three separate times during practice. Now I write the DNS egress rule FIRST, before anything else.
>
> AND vs OR tripped me up too. Two selectors in the *same* `from` block = AND (must match both). Two separate `from` blocks = OR (match either). I had `podSelector` and `namespaceSelector` in separate blocks when they should have been in the same one — it allowed traffic from any pod in the right namespace OR any pod with the right label in any namespace.

## Verify

```bash
# Get backend IP
BACKEND_IP=$(k get pod backend -n exercise-05 -o jsonpath='{.status.podIP}')

# Frontend -> Backend: should work
k exec frontend -n exercise-05 -- wget -qO- --timeout=2 http://$BACKEND_IP

# Attacker -> Backend: should time out
k exec attacker -n exercise-05 -- wget -qO- --timeout=2 http://$BACKEND_IP
```

## Cleanup

```bash
k delete ns exercise-05
```

<details>
<summary>Solution</summary>

```bash
k create ns exercise-05

# Create pods
k run frontend -n exercise-05 --image=nginx:1.27 --labels=role=frontend
k run backend -n exercise-05 --image=nginx:1.27 --labels=role=backend
k run attacker -n exercise-05 --image=nginx:1.27 --labels=role=attacker

# Wait for all pods
k get pods -n exercise-05 -w
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: exercise-05
spec:
  podSelector:
    matchLabels:
      role: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
```

```bash
k apply -f netpol.yaml

# Test
BACKEND_IP=$(k get pod backend -n exercise-05 -o jsonpath='{.status.podIP}')
k exec frontend -n exercise-05 -- wget -qO- --timeout=2 http://$BACKEND_IP
k exec attacker -n exercise-05 -- wget -qO- --timeout=2 http://$BACKEND_IP
```

</details>
