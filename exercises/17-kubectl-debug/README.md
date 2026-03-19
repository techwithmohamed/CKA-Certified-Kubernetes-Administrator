# Exercise 17 — kubectl debug: Pod and Node

> **Medium** | ~15 min | Domain: Troubleshooting (30%)
>
> Related: [README — Troubleshooting](../../README.md#domain-2--troubleshooting-30)

Use `kubectl debug` to troubleshoot running pods and access node-level resources. This is GA in v1.35 and directly relevant to CKA troubleshooting tasks.

## Tasks

### Part A: Debug a pod

1. Create a namespace called `exercise-17`
2. Create a pod named `broken-app` with image `nginx:1.27`
3. Attach a debug container to the running pod using `kubectl debug`:
   - Image: `busybox:1.36`
   - Target the `broken-app` container to share process namespace
4. From inside the debug container, list the processes (you should see the nginx process)
5. Check the filesystem and network from the debug container
6. Exit the debug session

### Part B: Debug a node

7. Run `kubectl debug` against a node to get a shell with host filesystem access
8. Use `chroot /host` to access the real node root
9. Check kubelet status from the debug session
10. Exit the debug session

## Hints

- Pod debug: `k debug <pod> -it --image=busybox:1.36 --target=<container>`
- Node debug: `k debug node/<node-name> -it --image=busybox:1.36`
- The `--target` flag shares process namespace with the target container
- Node debug mounts the host filesystem at `/host`
- Use `chroot /host` to run node commands directly

## Verify

```bash
# Part A: process list should show nginx
# Inside debug container:
ps aux

# Part B: kubelet check
# Inside node debug (after chroot /host):
systemctl status kubelet
```

## Cleanup

```bash
k delete ns exercise-17
```

<details>
<summary>Solution</summary>

### Part A

```bash
k create ns exercise-17

k run broken-app -n exercise-17 --image=nginx:1.27

# Wait for running
k get pod broken-app -n exercise-17 -w

# Debug with shared process namespace
k debug broken-app -n exercise-17 -it --image=busybox:1.36 --target=broken-app

# Inside the debug container:
ps aux                      # should show nginx master + worker processes
ls /proc/1/root/etc/nginx/  # access the target container filesystem
wget -qO- localhost:80      # test connectivity from inside
exit
```

### Part B

```bash
# Get a node name
k get nodes
# e.g., node-1

k debug node/node-1 -it --image=busybox:1.36

# Inside:
chroot /host

# Now you have real node access
systemctl status kubelet
journalctl -u kubelet --no-pager | tail -20
crictl ps
cat /etc/kubernetes/manifests/kube-apiserver.yaml | head -20

exit  # exit chroot
exit  # exit debug pod
```

</details>
