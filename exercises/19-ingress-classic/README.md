# Exercise 19 — Classic Ingress

> Related: [Ingress skeleton](../../skeletons/ingress.yaml) | [Service skeleton](../../skeletons/service.yaml) | [README — Services and Networking](../../README.md#domain-3--services--networking-20)

Configure Kubernetes Ingress resources for HTTP/HTTPS routing. CKA exam topics include classic Ingress alongside newer Gateway API. This exercise covers path-based routing, TLS termination, and IngressClass configuration.

## Context

Ingress provides external access to services inside a cluster. While Gateway API is the future direction, classic Ingress (networking.k8s.io/v1) appears on most CKA exams. You need to understand:
- How to expose services via Ingress
- Path-based and host-based routing
- TLS certificate setup
- Ingress controller requirements

This differs from Exercise 15 (Gateway API) because it tests the older API that's still in widespread production use.

## Tasks

1. Create a namespace called `exercise-19`
2. Create two Deployments:
   - `web-app` with image `nginx:1.28` and 2 replicas, serving on port 80
   - `api-app` with image `nginx:1.28` and 2 replicas, serving on port 8080
3. Expose `web-app` Deployment with a ClusterIP service named `web-service` on port 80
4. Expose `api-app` Deployment with a ClusterIP service named `api-service` on port 8080
5. Create an Ingress resource named `app-ingress` that routes:
   - `/` traffic to `web-service:80`
   - `/api` traffic to `api-service:8080`
   - Default backend (no matching path) should point to `web-service`
6. Verify the Ingress resource was created and has an address assigned
7. Test path-based routing by describing the Ingress (check rules)

## Hints

- `k create service clusterip` to create services imperatively
- Ingress requires an Ingress Controller (usually nginx-ingress or similar) to function
- Path-based routing uses `path` with `pathType: Prefix` in Ingress rules
- `k get ingress` shows ingress resources; `k describe ingress` shows routing rules
- Without an Ingress Controller, the Ingress creates successfully but no ADDRESS is assigned
- Exam scenario: You may need to install/verify nginx-ingress controller first

## What tripped me up

> I created the Ingress but forgot the services. The Ingress created successfully and showed a backend error. The error message "no endpoints available for backend" took me 8 minutes to debug. Now I always create services first, then verify they have endpoints, then create the Ingress.
>
> Path types matter: `Prefix` means any path starting with the rule path matches. `Exact` requires exact match. I used `Exact` and couldn't reach `/api/v1/users` path. Switched to `Prefix` and it worked. Read the pathType documentation carefully.
>
> The Ingress controller needs a LoadBalancer service or NodePort to expose the Ingress externally. In a lab environment without a LoadBalancer provider, the Ingress works but you can't test from outside the cluster. During CKA exam scenarios, the cluster infrastructure is already set up.

## Verify

```bash
# Create deployments and services first
k create deployment web-app -n exercise-19 --image=nginx:1.28 --replicas=2
k expose deployment web-app -n exercise-19 --name=web-service --type=ClusterIP --port=80 --target-port=80

k create deployment api-app -n exercise-19 --image=nginx:1.28 --replicas=2
k expose deployment api-app -n exercise-19 --name=api-service --type=ClusterIP --port=8080 --target-port=8080

# Verify services have endpoints
k get endpoints -n exercise-19

# Create Ingress
k create ingress app-ingress -n exercise-19 \
  --rule="/*=web-service:80" \
  --rule="/api/*=api-service:8080" \
  --class=nginx

# Check Ingress status
k get ingress -n exercise-19
k describe ingress app-ingress -n exercise-19

# Export as YAML to verify structure
k get ingress app-ingress -n exercise-19 -o yaml
```

## Cleanup

```bash
k delete ns exercise-19
```

<details>
<summary>Solution</summary>

```bash
# Namespace
k create ns exercise-19

# Deployments
k create deployment web-app -n exercise-19 --image=nginx:1.28 --replicas=2 $do | k apply -f -
k create deployment api-app -n exercise-19 --image=nginx:1.28 --replicas=2 $do | k apply -f -

# Wait for pods
k wait --for=condition=ready pod -l app=web-app -n exercise-19 --timeout=30s

# Services
k expose deployment web-app -n exercise-19 --name=web-service --type=ClusterIP --port=80 --target-port=80
k expose deployment api-app -n exercise-19 --name=api-service --type=ClusterIP --port=8080 --target-port=8080

# Verify endpoints exist
k get endpoints -n exercise-19

# Ingress YAML definition
cat <<EOF | k apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: exercise-19
spec:
  ingressClassName: nginx
  defaultBackend:
    service:
      name: web-service
      port:
        number: 80
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
EOF

# Verify Ingress was created
k get ingress -n exercise-19
k describe ingress app-ingress -n exercise-19

# Note: ADDRESS will remain <pending> if no ingress controller is running
# In a real cluster with nginx-ingress installed, it would show the external IP/hostname
```

</details>
