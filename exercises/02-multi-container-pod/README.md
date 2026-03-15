# Exercise 02 — Multi-Container Pod (Sidecar Logging)

> Related: [Pod skeleton](../../skeletons/pod.yaml) | [README — Workloads & Scheduling](../../README.md#domain-3--workloads--scheduling-15)

Create a pod with a main container and a sidecar container that tails the main container's log file. This uses the v1.35 native sidecar container feature (`restartPolicy: Always` on an init container).

## Tasks

1. Create a namespace called `exercise-02`
2. Create a pod named `logger` in namespace `exercise-02` with:
   - An init container (sidecar) named `log-agent` using `busybox:1.36`
     - Set `restartPolicy: Always` on the init container to make it a native sidecar
     - Command: `tail -f /var/log/app/app.log`
     - Mount a shared volume at `/var/log/app`
   - A main container named `app` using `busybox:1.36`
     - Command: write a line to `/var/log/app/app.log` every 3 seconds
     - Mount the same shared volume at `/var/log/app`
   - Use an `emptyDir` volume named `log-volume`
3. Verify both containers are running
4. Check the sidecar's logs to see the streamed output

## Hints

- Native sidecars in v1.35: put the sidecar in `initContainers` with `restartPolicy: Always`
- The sidecar starts before the main container and keeps running alongside it
- Shared volume: both containers mount the same `emptyDir`

## Verify

```bash
# Both containers should be Running (READY 2/2)
k get pod logger -n exercise-02

# Should see the log lines from the app container
k logs logger -n exercise-02 -c log-agent
```

## Cleanup

```bash
k delete ns exercise-02
```

<details>
<summary>Solution</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logger
  namespace: exercise-02
spec:
  initContainers:
  - name: log-agent
    image: busybox:1.36
    restartPolicy: Always
    command: ["sh", "-c", "tail -f /var/log/app/app.log"]
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "while true; do echo \"$(date) - app running\" >> /var/log/app/app.log; sleep 3; done"]
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app
  volumes:
  - name: log-volume
    emptyDir: {}
```

```bash
k create ns exercise-02
k apply -f logger.yaml

# Wait for pod to be ready
k get pod logger -n exercise-02 -w

# Check sidecar logs
k logs logger -n exercise-02 -c log-agent
```

</details>
