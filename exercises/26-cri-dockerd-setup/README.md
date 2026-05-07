# Exercise 26 — CRI-dockerd Installation and Configuration

> Related: [README — Cluster Maintenance](../../README.md#domain-7--cluster-maintenance-11)

Install and configure cri-dockerd as a container runtime alternative to containerd. Essential for clusters using Docker as the container runtime.

## Tasks

1. SSH into a worker node
2. Install cri-dockerd using the `.deb` package:
   - Download from GitHub releases
   - Install the package
3. Configure system for cri-dockerd:
   - Enable IP forwarding using `sysctl`
   - Add required kernel modules (`overlay`, `br_netfilter`)
4. Enable and start cri-dockerd service:
   - `systemctl enable cri-dockerd`
   - `systemctl start cri-dockerd`
5. Verify cri-dockerd socket is available
6. Configure kubelet to use cri-dockerd
7. Restart kubelet and verify node is Ready

## Key Learning

- cri-dockerd is a CRI adapter for Docker
- Kubernetes v1.35 dropped built-in dockershim — must use cri-dockerd
- IP forwarding and kernel modules are prerequisites
- Socket location: `/run/cri-dockerd.sock`
- Exam tests installation procedure and troubleshooting

## Hints

<details>
<summary>Stuck? Click to reveal hints</summary>

- Download URL: `https://github.com/Mirantis/cri-dockerd/releases`
- Install: `sudo dpkg -i cri-dockerd_*_amd64.deb`
- Enable IP forwarding: `sudo sysctl -w net.ipv4.ip_forward=1`
- Load modules: `sudo modprobe overlay` and `sudo modprobe br_netfilter`
- Check socket: `ls -la /run/cri-dockerd.sock`
- Kubelet config: `--container-runtime=remote --container-runtime-endpoint=unix:///run/cri-dockerd.sock`

</details>

## What tripped me up

> I installed cri-dockerd but didn't enable IP forwarding. Node stayed NotReady because networking wasn't configured. Then I forgot to load `overlay` and `br_netfilter` modules — pods wouldn't start. Always do the full network setup BEFORE starting the service. Once cri-dockerd is running, kubelet needs a full restart (`systemctl restart kubelet`) to recognize it, not just a reload.

## Verify

```bash
# Socket exists and is accessible
sudo ls -la /run/cri-dockerd.sock

# Service is running
sudo systemctl status cri-dockerd

# Kernel modules loaded
lsmod | grep overlay
lsmod | grep br_netfilter

# IP forwarding enabled
cat /proc/sys/net/ipv4/ip_forward

# Kubelet configured to use it
grep container-runtime /etc/kubernetes/kubelet.conf

# Node is Ready
k get nodes
```

## Cleanup

```bash
sudo systemctl stop cri-dockerd
sudo dpkg -r cri-dockerd
```

<details>
<summary>Solution</summary>

```bash
# SSH into worker node
ssh <node-ip>

# 1. Download cri-dockerd (example version)
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.14/cri-dockerd_0.3.14.3-0~ubuntu-focal_amd64.deb

# 2. Install
sudo dpkg -i cri-dockerd_0.3.14.3-0~ubuntu-focal_amd64.deb

# 3. Load required kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# 4. Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
# Persist across reboots
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sudo sysctl -p

# 5. Enable and start cri-dockerd
sudo systemctl enable cri-dockerd
sudo systemctl start cri-dockerd

# 6. Verify socket exists
sudo ls -la /run/cri-dockerd.sock

# 7. Check cri-dockerd is running
sudo systemctl status cri-dockerd

# 8. Update kubelet to use cri-dockerd
# Edit /etc/kubernetes/kubelet.conf or /etc/sysconfig/kubelet
sudo sed -i 's|--container-runtime=containerd|--container-runtime=remote|' /etc/kubernetes/kubelet.conf
sudo sed -i '/--container-runtime=remote/a --container-runtime-endpoint=unix:///run/cri-dockerd.sock' /etc/kubernetes/kubelet.conf

# 9. Restart kubelet (full restart, not reload)
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# 10. Check node status (may take 30 seconds)
k get nodes
# Should show Ready status
```

Reference: https://kubernetes.io/docs/setup/production-environment/container-runtimes/

</details>
