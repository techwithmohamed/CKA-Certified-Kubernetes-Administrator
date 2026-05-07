# Exercise 28 — Complex NetworkPolicy (Multi-Namespace, Traffic Control)

> Related: [NetworkPolicy skeleton](../../skeletons/networkpolicy.yaml) | [README — Networking](../../README.md#domain-5--services--networking-13)

Create multi-namespace NetworkPolicies with ingress/egress rules, requiring careful label matching and debugging network connectivity.

## Tasks

1. Create 3 namespaces: `frontend`, `backend`, `database`
2. Deploy apps:
   - Frontend (nginx): labels `tier=frontend`
   - Backend (busybox): labels `tier=backend`
   - Database (redis): labels `tier=database`
3. Create NetworkPolicies:
   - Allow frontend → backend (only port 8080)
   - Allow backend → database (only port 6379)
   - Deny all other traffic initially
   - Allow DNS egress (UDP 53) to kube-dns for all pods
4. Test connectivity:
   - Frontend CAN reach backend on 8080
   - Frontend CANNOT reach database
   - Backend CAN reach database on 6379
   - Backend CANNOT reach frontend
5. Verify DNS still works from all namespaces

## Key Learning

- NetworkPolicies are pod label selectors, not namespace selectors alone
- Ingress rules applied to target pods
- Egress rules applied to source pods
- Must explicitly allow DNS egress or service discovery breaks
- Debugging: use `k describe pod` and check labels carefully
- Exam tests label matching precision

## Hints

<details>
<summary>Stuck? Click to reveal hints</summary>

- For cross-namespace: use `namespaceSelector` with labels on namespaces
- DNS rule: `to: []` with `ports: [{protocol: UDP, port: 53}]`
- Test connectivity: `k exec <pod> -- curl <service>.<namespace>.svc.cluster.local:port`
- Debug: `k describe np <policy>` shows selector AND rules
- Remember: ingress on receiving pod, egress on sending pod

</details>

## What tripped me up

> I created correct ingress rules but forgot that the SOURCE pod also needs egress rules allowing it OUT. Had to create matching egress rules on frontend to allow traffic to backend. Also, I forgot to allow DNS egress — pods couldn't resolve service names at all. Plus: label matching is case-sensitive and whitespace matters. A typo in a label selector silently fails — no error, just traffic blocked.

## Verify

```bash
# Pods are running in correct namespaces
k get pods -A

# NetworkPolicies exist
k get networkpolicies -A

# From frontend, test connectivity
k exec -it <frontend-pod> -n frontend -- curl http://backend.<backend>.svc.cluster.local:8080
# Should succeed

# From frontend, try database (should fail)
k exec -it <frontend-pod> -n frontend -- curl http://database.<database>.svc.cluster.local:6379
# Should timeout/fail

# Test DNS works
k exec -it <any-pod> -- nslookup kubernetes.default
# Should succeed
```

## Cleanup

```bash
k delete ns frontend backend database
```

<details>
<summary>Solution</summary>

```bash
# Create namespaces
k create ns frontend backend database

# Label namespaces for cross-ns selection
k label ns frontend name=frontend
k label ns backend name=backend
k label ns database name=database

# Deploy frontend
k run frontend-web -n frontend --image=nginx:1.27 --labels=tier=frontend

# Deploy backend
k run backend-api -n backend --image=busybox:1.36 --command sleep 3600 --labels=tier=backend

# Deploy database
k run database-redis -n database --image=busybox:1.36 --command sleep 3600 --labels=tier=database

# Create services
k expose pod frontend-web -n frontend --port=80 --type=ClusterIP
k expose pod backend-api -n backend --port=8080 --type=ClusterIP
k expose pod database-redis -n database --port=6379 --type=ClusterIP

# Allow frontend → backend
cat <<EOF | k apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: backend
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    ports:
    - protocol: TCP
      port: 8080
EOF

# Allow backend → database
cat <<EOF | k apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-database
  namespace: database
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: backend
    ports:
    - protocol: TCP
      port: 6379
EOF

# Allow DNS egress from all namespaces
for ns in frontend backend database; do
cat <<EOF | k apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: $ns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: []
    ports:
    - protocol: UDP
      port: 53
EOF
done

# Test connectivity
# Frontend → Backend (should work)
k exec -it $(k get po -n frontend -o name | head -1) -n frontend -- curl -m 2 http://backend-api.backend.svc.cluster.local:8080

# Frontend → Database (should fail)
k exec -it $(k get po -n frontend -o name | head -1) -n frontend -- curl -m 2 http://database-redis.database.svc.cluster.local:6379
```

</details>
