# Exercise 10 — Static Pod

> Related: [Pod skeleton](../../skeletons/pod.yaml) | [README — Workloads & Scheduling](../../README.md#domain-3--workloads--scheduling-15)

Create a static pod by placing a manifest in the kubelet's static pod directory. Static pods are managed directly by the kubelet, not the API server.

## Tasks

1. Find the static pod manifest directory on the node
2. Create a static pod manifest file for a pod named `static-web` with:
   - Image: `nginx:1.27`
   - Port: 80
3. Verify the static pod appears in `k get pods` (it will have the node name appended)
4. Try to delete the static pod with `kubectl` — observe what happens
5. Delete the static pod by removing the manifest file
6. Verify the pod is gone

## Hints

<details>
<summary>Stuck? Click to reveal hints</summary>

- The static pod path is configured in the kubelet config, usually `/etc/kubernetes/manifests/`
- Check kubelet config: `cat /var/lib/kubelet/config.yaml | grep staticPodPath`
- Static pods show up in `kubectl` but can't be deleted through the API — the kubelet recreates them
- The pod name will be `static-web-<node-name>`

</details>

## Verify

```bash
# Should show static-web-<node>
k get pods -A | grep static-web

# Describe should show the static pod
k describe pod static-web-<node> -n default
```

## Cleanup

```bash
sudo rm /etc/kubernetes/manifests/static-web.yaml
```

## What tripped me up

> I kept running `kubectl delete pod static-web-node01` and it kept coming back 5 seconds later. I did this four times before I realized: kubelet manages static pods, not the API server. You CANNOT delete a static pod through kubectl. You have to SSH into the node and delete the manifest file from `/etc/kubernetes/manifests/`. This is one of those things that feels wrong but is correct.
>
> Also forgot to check the actual `staticPodPath` on the node. Most guides say `/etc/kubernetes/manifests/` but it's configurable in the kubelet config. On the exam, always verify: `cat /var/lib/kubelet/config.yaml | grep staticPodPath`. I once put the file in the wrong directory and the pod never appeared.

<details>
<summary>Solution</summary>

```bash
# Step 1: Find static pod path
cat /var/lib/kubelet/config.yaml | grep staticPodPath
# Usually: staticPodPath: /etc/kubernetes/manifests

# Step 2: Create the manifest
sudo tee /etc/kubernetes/manifests/static-web.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    app: static-web
spec:
  containers:
  - name: web
    image: nginx:1.27
    ports:
    - containerPort: 80
EOF

# Step 3: Verify (wait a few seconds for kubelet to pick it up)
k get pods | grep static-web
# Output: static-web-<node-name>   1/1   Running

# Step 4: Try to delete via kubectl
k delete pod static-web-<node-name>
# The pod will reappear — kubelet recreates it

# Step 5: Actually delete it
sudo rm /etc/kubernetes/manifests/static-web.yaml

# Step 6: Verify it's gone
k get pods | grep static-web
# No output — pod is gone
```

</details>
