# Exercise 30 — TLS Configuration Update (Cipher Support)

> Related: [README — Security](../../README.md#domain-4--security-12)

Update TLS configuration to support additional protocol versions for backward compatibility. Tests ConfigMap modification and service restart.

## Tasks

1. A service currently supports only TLS 1.3
2. Add support for TLS 1.2 (make both available)
3. Update configuration file or ConfigMap:
   - Add TLS 1.2 to supported protocols
   - Keep TLS 1.3 enabled
4. ConfigMap is mounted in Deployment as env var or config file
5. Restart Deployment to apply changes
6. Verify service accepts TLS 1.2 connections:
   - Use `openssl s_client -connect <service> -tls1_2`
   - Or curl with explicit TLS 1.2
7. Verify TLS 1.3 still works

## Key Learning

- TLS configuration often in ConfigMaps
- Adding vs replacing: exam specifies "ADD support" (don't remove existing)
- Service restart may be needed after config change
- TLS negotiation must support minimum version
- Testing TLS versions requires understanding openssl commands

## Hints

<details>
<summary>Stuck? Click to reveal hints</summary>

- Edit ConfigMap: `k edit cm <name>`
- Find TLS setting (could be `tlsVersion`, `tls1.3Min`, etc.)
- After edit, restart pod: `k rollout restart deployment/<name>`
- Test TLS 1.2: `openssl s_client -connect <ip>:<port> -tls1_2`
- Verify version in output: "Protocol  : TLSv1.2"
- Test TLS 1.3: `openssl s_client -connect <ip>:<port> -tls1_3`

</details>

## What tripped me up

> The ConfigMap had `tls_min_version=1.3`. I thought I had to REPLACE it with `tls_min_version=1.2`, but the exam asked to ADD 1.2 support (keep 1.3). I ended up breaking 1.3 connections. Should have changed it to support a RANGE or list: `supported_versions=[1.2, 1.3]` or similar. Read the requirement twice: UPDATE to support BOTH, not replace.
>
> Also, ConfigMap changes don't auto-reload in pods. Had to explicitly restart the deployment with `k rollout restart`. And testing with openssl requires specifying the TLS version explicitly with `-tls1_2` flag.

## Verify

```bash
# ConfigMap has both TLS 1.2 and 1.3
k get cm <name> -o yaml | grep -i tls

# Deployment is running after restart
k get deployment <name>

# Service endpoint is responding
k get svc <name> -o wide

# Test TLS 1.2
echo | openssl s_client -connect <ip>:<port> -tls1_2 2>/dev/null | grep Protocol
# Output: Protocol  : TLSv1.2

# Test TLS 1.3
echo | openssl s_client -connect <ip>:<port> -tls1_3 2>/dev/null | grep Protocol
# Output: Protocol  : TLSv1.3
```

## Cleanup

```bash
k delete svc <service>
k delete deployment <deployment>
k delete cm <config>
```

<details>
<summary>Solution</summary>

```bash
# 1. Find the service and ConfigMap
k get svc --all-namespaces
k get cm --all-namespaces

# 2. Check current TLS config
k get cm <name> -o yaml

# 3. Edit ConfigMap to add TLS 1.2
k edit cm <name>
# Find line like: tls_min_version: "1.3"
# Change to: supported_versions: ["1.2", "1.3"]
# Or if format is different, add TLS 1.2

# 4. Check if there's an env var that sets this
k describe deployment <deployment>
# Look for "Environment:" section

# 5. If config is in file mount, edit ConfigMap
# Then restart deployment

k rollout restart deployment/<deployment>

# 6. Wait for pod to be ready
k rollout status deployment/<deployment>

# 7. Portforward to test (if service is internal)
k port-forward svc/<service> <local-port>:<remote-port> &

# 8. Test TLS 1.2 connection
echo | openssl s_client -connect localhost:<port> -tls1_2 2>/dev/null | grep "Protocol"
# Should show: Protocol  : TLSv1.2

# 9. Test TLS 1.3 still works
echo | openssl s_client -connect localhost:<port> -tls1_3 2>/dev/null | grep "Protocol"
# Should show: Protocol  : TLSv1.3

# 10. Or use curl
curl -k --tlsv1.2 https://localhost:<port>
curl -k --tlsv1.3 https://localhost:<port>
```

</details>
