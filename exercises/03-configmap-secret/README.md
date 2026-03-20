# Exercise 03 — ConfigMap and Secret

> Related: [ConfigMap/Secret skeleton](../../skeletons/configmap-secret.yaml) | [README — Workloads & Scheduling](../../README.md#domain-3--workloads--scheduling-15)

Create ConfigMaps and Secrets, then inject them into a pod as environment variables and mounted files.

## Tasks

1. Create a namespace called `exercise-03`
2. Create a ConfigMap named `app-config` with:
   - Key `APP_MODE` = `production`
   - Key `LOG_LEVEL` = `debug`
3. Create a Secret named `db-creds` with:
   - Key `DB_USER` = `admin`
   - Key `DB_PASS` = `s3cretP@ss`
4. Create a pod named `app` that:
   - Uses image `busybox:1.36`, command `sleep 3600`
   - Loads `APP_MODE` and `LOG_LEVEL` from the ConfigMap as env vars
   - Loads `DB_USER` and `DB_PASS` from the Secret as env vars
   - Mounts the entire ConfigMap as files at `/etc/config/`
5. Verify the env vars are set inside the pod
6. Verify the mounted files exist at `/etc/config/`

## Hints

- `k create configmap app-config --from-literal=APP_MODE=production --from-literal=LOG_LEVEL=debug`
- `k create secret generic db-creds --from-literal=DB_USER=admin --from-literal=DB_PASS=s3cretP@ss`
- Use `envFrom` to load all keys from a ConfigMap or Secret
- Use `volumes` + `volumeMounts` to mount ConfigMap as files

## What tripped me up

> I referenced a ConfigMap name that didn't exist yet (`app-conf` instead of `app-config` — typo). The pod went into `CreateContainerConfigError` and I spent 3 minutes staring at the YAML before checking `k describe pod` events. The event message says exactly which ConfigMap is missing. Check events first, always.
>
> Also confused `envFrom` vs `env.valueFrom`. `envFrom` loads ALL keys from a ConfigMap/Secret as env vars. `env.valueFrom.configMapKeyRef` loads a single key. On the exam, if the question says "load all keys," use `envFrom`. If it says "load KEY_X as MY_VAR," use `valueFrom`. Getting them backwards doesn't error — you just get wrong variable names.

## Verify

```bash
# Check env vars
k exec app -n exercise-03 -- env | grep -E "APP_MODE|LOG_LEVEL|DB_USER|DB_PASS"

# Check mounted files
k exec app -n exercise-03 -- ls /etc/config/
k exec app -n exercise-03 -- cat /etc/config/APP_MODE
```

## Cleanup

```bash
k delete ns exercise-03
```

<details>
<summary>Solution</summary>

```bash
k create ns exercise-03

k create configmap app-config -n exercise-03 \
  --from-literal=APP_MODE=production \
  --from-literal=LOG_LEVEL=debug

k create secret generic db-creds -n exercise-03 \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASS=s3cretP@ss
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: exercise-03
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sleep", "3600"]
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: db-creds
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: app-config
```

```bash
k apply -f app.yaml

# Verify
k exec app -n exercise-03 -- env | grep -E "APP_MODE|LOG_LEVEL|DB_USER|DB_PASS"
k exec app -n exercise-03 -- ls /etc/config/
```

</details>
