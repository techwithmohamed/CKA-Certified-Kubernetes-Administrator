# Exercise 18 — CRI-dockerd Installation & Configuration

> Related: [Cluster Architecture](../../README.md#domain-4--cluster-architecture-installation--configuration-25) | Container runtime setup for kubeadm clusters

Install and configure cri-dockerd as a container runtime for a Kubernetes node. This is common when upgrading clusters that need Docker support or when preparing a mixed-runtime cluster.

## Context

Kubernetes deprecated dockershim in v1.20 and removed it in v1.24. To continue using Docker as a container runtime, you must install CRI-dockerd explicitly. The exam may ask you to prepare a node to join a cluster using Docker via CRI-dockerd, or troubleshoot why a node can't join because the runtime isn't configured.

## Tasks

1. Load the required kernel modules: `overlay` and `br_netfilter`
2. Configure kernel networking parameters via sysctl
3. Install Docker (required dependency for CRI-dockerd)
4. Download and install CRI-dockerd binary from GitHub releases
5. Create systemd service files for cri-docker.service and cri-docker.socket
6. Enable and start the cri-docker services
7. Verify the CRI-dockerd socket is listening at `/run/cri-dockerd.sock`

## Hints

- Download the latest amd64 release from https://github.com/Mirantis/cri-dockerd/releases
- Use `curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest` to get the latest version automatically
- The socket path must match what you pass to kubelet: `--cri-socket=unix:///run/cri-dockerd.sock`
- Kernel modules persist only if added to `/etc/modules-load.d/`
- Sysctl configuration in `/etc/sysctl.d/` survives reboots
- Check service status with: `sudo systemctl status cri-docker.service`
- Verify socket exists: `ls -la /run/cri-dockerd.sock`
- Check logs if service fails: `sudo journalctl -u cri-docker.service -n 50`

## What tripped me up

> I downloaded the wrong architecture (arm64 instead of amd64) because I didn't check my system with `uname -m` first. The binary gave a permission denied error, making me think it was a service issue when it was just the wrong file.
>
> I also tried to manually create the socket file instead of letting the systemd socket unit create it. That doesn't work — the socket file is created automatically when you enable and start the socket unit, not beforehand.
>
> One more thing: I forgot that Docker needs to be running separately. The CRI-dockerd service has `After=docker.service` in the systemd file, which means it starts after Docker but doesn't require it to be running. I needed to `systemctl start docker` first or the CRI-dockerd service would fail immediately with "cannot connect to docker daemon".

## Verify

```bash
# Check kernel modules are loaded
lsmod | grep overlay
lsmod | grep br_netfilter

# Check sysctl parameters are set
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.ipv4.ip_forward

# Check CRI-dockerd is running
sudo systemctl status cri-docker.service
sudo systemctl status cri-docker.socket

# Verify the socket exists and is listening
ls -la /run/cri-dockerd.sock

# Test CRI socket connectivity
sudo crictl version
```

## Cleanup

If moving to another exercise, stop and disable CRI-dockerd:

```bash
sudo systemctl stop cri-docker.service cri-docker.socket
sudo systemctl disable cri-docker.service cri-docker.socket
```

<details>
<summary>Solution</summary>

```bash
# Step 1: Load kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Verify modules are loaded
lsmod | grep overlay
lsmod | grep br_netfilter

# Step 2: Configure sysctl parameters
sudo tee /etc/sysctl.d/99-kubernetes-cri.conf > /dev/null <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
EOF

# Apply the changes
sudo sysctl --system

# Verify parameters are applied
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.ipv4.ip_forward
sysctl net.ipv6.conf.all.forwarding

# Step 3: Install Docker (prerequisite for CRI-dockerd)
sudo apt-get update
sudo apt-get install -y docker.io

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Step 4: Download and install CRI-dockerd
VER=$(curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest | grep tag_name | cut -d '"' -f 4 | sed 's/v//')
echo "Installing CRI-dockerd version: $VER"

wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.amd64.tgz
tar xzf cri-dockerd-${VER}.amd64.tgz
sudo mv cri-dockerd/cri-dockerd /usr/local/bin/
cri-dockerd --version

# Step 5: Create systemd service file
sudo tee /etc/systemd/system/cri-docker.service > /dev/null <<'EOF'
[Unit]
Description=CRI Docker daemon
After=docker.service network-online.target firewalld.service
Wants=network-online.target
Documentation=https://github.com/Mirantis/cri-dockerd

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/cri-docker
ExecStart=/usr/local/bin/cri-dockerd --network-plugin=cni
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutStopSec=120
RestartSec=5
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Step 6: Create systemd socket file
sudo tee /etc/systemd/system/cri-docker.socket > /dev/null <<'EOF'
[Unit]
Description=CRI Docker socket
Documentation=https://github.com/Mirantis/cri-dockerd
PartOf=cri-docker.service

[Socket]
ListenStream=%t/cri-dockerd.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
EOF

# Step 7: Enable and start CRI-dockerd services
sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service cri-docker.socket
sudo systemctl start cri-docker.socket
sudo systemctl start cri-docker.service

# Step 8: Verify services are running
sudo systemctl status cri-docker.service
sudo systemctl status cri-docker.socket

# Step 9: Verify socket is accessible
ls -la /run/cri-dockerd.sock
sudo crictl version

# Step 10: For kubeadm, use this when initializing or joining:
# sudo kubeadm init --cri-socket unix:///run/cri-dockerd.sock
# Or for joining a node:
# sudo kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --cri-socket unix:///run/cri-dockerd.sock
```

</details>
