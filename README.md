[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

# ☸️ Certified Kubernetes Administrator (CKA) Exam Guide - V1.33 (2025)

<p align="center">
  <img src="assets/cka.png" alt="CKA EXAM">
</p>

> This guide is part of our blog [Pass the CKA Certification Exam: The Complete Study Guide ](https://techwithmohamed.com/blog/cka-exam-study/).

## Hit the Star! :star:
> If you are using this repo for guidance, please hit the star. Thanks A lot !

>  The [Certified Kubernetes Administrator (CKA) certification](https://www.cncf.io/certification/cka/) exam certifies that candidates have the skills, knowledge, and competency to perform the responsibilities of Kubernetes administrators.
 
## CKA Exam details (v1.33  2025 ) 

| **CKA Exam Details**                     | **Information**                                                                                     |
|-------------------------------------------|-----------------------------------------------------------------------------------------------------|
| **Exam Type**                             | Performance Based ( NOT MCQ )                                    |
| **Exam Duration**                         | 2 hours                                                                                            |
| **Pass Percentage**                       | 66%  ( One Retake )                                                                                                |
| **CKA Exam Kubernetes Version**          | [Kubernetes v1.33]((https://kubernetes.io/blog/2025/04/23/kubernetes-v1-33-release/))                                                                               |
| **CKA Validity**                         | 2 Years  |
| **Exam Cost**                            | $445 USD   |


## What’s New in Kubernetes 1.33

Kubernetes **v1.33** (April 2025) brings a mix of stability, new features, and important deprecations that are directly relevant for the CKA exam.

### 🔥 Key Features

- **In-place Pod Resize (beta, enabled by default)**  
  You can now adjust CPU and memory requests/limits of running Pods **without deleting or recreating them**.  
  ```bash
  # Increase CPU/memory for a running pod
  kubectl patch pod mypod --subresource=resize     --type=merge -p '{"spec":{"containers":[{"name":"app","resources":{"requests":{"cpu":"500m","memory":"256Mi"},"limits":{"cpu":"1","memory":"512Mi"}}}]}}'
  ```
  This is especially useful in troubleshooting or resource tuning scenarios.

- **User Namespaces**  
  Stronger pod-level isolation: Kubernetes can now map container “root” to an **unprivileged UID on the host**.  
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: userns-demo
  spec:
    hostUsers: false   # opt-in to user namespaces
    containers:
    - name: c
      image: busybox
      command: ["sh","-c","id && sleep 1d"]
  ```
  Expect to see it in security-related exam tasks.

- **Storage Enhancements**  
  - **Image Volumes (beta):** mount OCI images directly as read-only volumes (handy for configs/data).  
  - **Volume Populators GA:** more flexible ways to pre-populate PVCs.  
  - **Leak-prevention improvements:** Kubernetes cleans up dangling PVs more reliably.  

- **Gateway API Progress**  
  Gateway API (`GatewayClass`, `Gateway`, `HTTPRoute`) continues to replace classic Ingress in many clusters. Exam scenarios now reference it directly.

- **Ephemeral Containers & `kubectl debug` (stable)**  
  Debugging tools are fully supported. Example:  
  ```bash
  kubectl debug pod/mypod -it --image=busybox --target=app -- sh
  ```

### ⚠️ Deprecations & Removals

- **`kubectl get componentstatuses` removed**  
  Use API server health endpoints instead:  
  ```bash
  kubectl get --raw='/readyz?verbose'
  kubectl get --raw='/livez?verbose'
  ```

- **PersistentVolume `Recycle` policy removed**  
  Only **Retain** and **Delete** are valid reclaim policies now.  

---

👉 These changes mean the exam in 2025 will place more weight on **resizing resources, debugging live Pods, Gateway API traffic routing, and CSI-driven storage management**, while also testing that you know which commands/policies are deprecated.


## Table of Contents

- [CKA Exam Syllabus (Updated Kubernetes 1.33)](#cka-exam-syllabus-updated-kubernetes-133)
- [CKA Exam Questions And Answers](#cka-exam-questions-and-answers)
- [Additional Resources](#additional-resources)
- [Practice](#practice)


## CKA Exam Syllabus (Updated Kubernetes 1.33 - 2025)

| **Topic** | **Concepts** | **Weightage** |
|-----------|--------------|---------------|
| [**1. Cluster Architecture, Installation & Configuration - 25%**](#1-cluster-architecture-installation--configuration-25) | 1. Manage role-based access control (RBAC)<br>2. Prepare underlying infrastructure for installing a Kubernetes cluster<br>3. Create and manage Kubernetes clusters using kubeadm<br>4. Manage the lifecycle of Kubernetes clusters<br>5. Implement and configure a highly-available control plane<br>6. Use Helm and Kustomize to install cluster components<br>7. Understand extension interfaces (CNI, CSI, CRI, etc.)<br>8. Understand CRDs, install and configure operators | 25% |
| [**2. Workloads & Scheduling - 15%**](#2-workloads--scheduling-15) | 1. Understand application deployments and how to perform rolling update and rollbacks<br>2. Use ConfigMaps and Secrets to configure applications<br>3. Configure workload autoscaling<br>4. Understand the primitives used to create robust, self-healing, application deployments<br>5. Configure Pod admission and scheduling (limits, node affinity, etc.) | 15% |
| [**3. Services & Networking - 20%**](#3-services--networking-20) | 1. Understand connectivity between Pods<br>2. Define and enforce Network Policies<br>3. Use ClusterIP, NodePort, LoadBalancer service types and endpoints<br>4. Use the Gateway API to manage Ingress traffic<br>5. Know how to use Ingress controllers and Ingress resources<br>6. Understand and use CoreDNS | 20% |
| [**4. Storage - 10%**](#4-storage-10) | 1. Implement storage classes and dynamic volume provisioning<br>2. Configure volume types, access modes and reclaim policies<br>3. Manage persistent volumes and persistent volume claims | 10% |
| [**5. Troubleshooting - 30%**](#5-troubleshooting-30) | 1. Troubleshoot clusters and nodes<br>2. Troubleshoot cluster components<br>3. Monitor cluster and application resource usage<br>4. Manage and evaluate container output streams<br>5. Troubleshoot services and networking | 30% |


## 1. Cluster Architecture, Installation & Configuration (25%)

This section tests your ability to **build, configure, and manage Kubernetes clusters** using `kubeadm` and core cluster components.

### ✅ Manage Role-Based Access Control (RBAC)

```bash
kubectl create role cm-writer --verb=create --resource=configmaps -n dev
kubectl create rolebinding writer-bind --role=cm-writer --serviceaccount=dev:app-sa -n dev
kubectl auth can-i create configmap --as system:serviceaccount:dev:app-sa -n dev
```

### ✅ Prepare Underlying Infrastructure

```bash
hostnamectl set-hostname control-plane-node
echo "10.0.0.2 worker-node" >> /etc/hosts
swapoff -a
```

### ✅ Create and Manage Clusters (kubeadm)

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
```

Join worker node:
```bash
kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

### ✅ Manage Cluster Lifecycle

```bash
kubeadm upgrade plan
kubeadm upgrade apply v1.33.1
kubelet --version
```

### ✅ Configure HA Control Plane

```bash
kubeadm init --control-plane-endpoint "LOADBALANCER_DNS:6443" --upload-certs
```

Add extra control-plane node:
```bash
kubeadm join LOADBALANCER_DNS:6443 --control-plane --token <TOKEN>   --discovery-token-ca-cert-hash sha256:<HASH> --certificate-key <CERT_KEY>
```

### ✅ Helm & Kustomize

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install nginx bitnami/nginx
kubectl apply -k ./my-kustomization/
```

### ✅ CNI / CSI / CRI

```bash
crictl info
ls /etc/cni/net.d/
kubectl get csidrivers
```

### ✅ CRDs & Operators

```bash
kubectl get crds
kubectl describe crd <customresource>
kubectl apply -f https://github.com/prometheus-operator/prometheus-operator/releases/download/v0.70.0/bundle.yaml
```

### ✅ New for v1.33 — In-Place Resize & Debugging

```bash
kubectl patch pod mypod --subresource=resize --type=merge   -p '{"spec":{"containers":[{"name":"app","resources":{"limits":{"cpu":"1","memory":"512Mi"}}}]}}'

kubectl debug pod/mypod -it --image=busybox --target=app -- sh
```

---

## 2. Workloads & Scheduling (15%)

This domain evaluates your knowledge in defining and managing application workloads and controlling how they are deployed and scheduled in a Kubernetes cluster. You should be confident with rolling updates, autoscaling, resource limits, health probes, and scheduling constraints.

---

### ✅ Understand Application Deployments and Perform Rolling Updates and Rollbacks

**Deployment**: A controller that ensures the desired number of Pods are running. It supports rolling updates (gradual replacement of Pods) and rollbacks (return to the last known good state).

**You're expected to:**
- Deploy applications using `kubectl` or YAML
- Upgrade images with zero downtime
- Rollback failed updates

**Example:**
```bash
kubectl create deployment nginx --image=nginx:1.27 --replicas=2
kubectl set image deployment nginx nginx=nginx:1.28
kubectl rollout undo deployment nginx
```

---

### ✅ Use ConfigMaps and Secrets to Configure Applications

- **ConfigMap**: Stores non-sensitive configuration (e.g., app modes, URLs).  
- **Secret**: Stores sensitive values (e.g., passwords, tokens). Data is base64-encoded but not encrypted by default.  

**Example:**
```bash
kubectl create configmap app-config --from-literal=APP_MODE=prod
kubectl create secret generic app-secret --from-literal=PASSWORD=1234
```

Injected into a Pod:
```yaml
env:
- name: APP_MODE
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_MODE
- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: PASSWORD
```

---

### ✅ Configure Workload Autoscaling

**Horizontal Pod Autoscaler (HPA):** Automatically increases or decreases the number of Pods based on CPU, memory, or custom metrics. Requires `metrics-server` or an external metrics adapter.

**Example:**
```bash
kubectl autoscale deployment nginx --cpu-percent=50 --min=1 --max=5
kubectl get hpa
```

---

### ✅ Understand the Primitives Used to Create Robust, Self-Healing Deployments

Kubernetes uses several **probes and controllers** to make applications resilient:

- **Liveness Probe** → restarts containers if they hang or crash.  
- **Readiness Probe** → ensures traffic only reaches Pods ready to serve.  
- **Startup Probe** → for slow apps, prevents false liveness failures.  
- **ReplicaSet** → guarantees a certain number of Pods run at all times.  

**Example:**
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

Check in Deployment:
```bash
kubectl get deploy nginx -o yaml | grep probe -A 5
```

---

### ✅ Configure Pod Admission and Scheduling (limits, node affinity, tolerations)

Kubernetes schedules Pods onto nodes based on resources and constraints:

- **Requests/Limits**: Reserve minimum CPU/memory (requests) and enforce maximum usage (limits).  
- **NodeSelector**: Place Pods only on nodes with specific labels.  
- **Affinity/Anti-Affinity**: Fine-grained rules to group or separate Pods.  
- **Taints/Tolerations**: Prevent Pods from scheduling on nodes unless explicitly tolerated.  

**Example:**
```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"

nodeSelector:
  disktype: ssd

tolerations:
- key: "node-role.kubernetes.io/control-plane"
  operator: "Exists"
  effect: "NoSchedule"
```

---

✅ **Kubernetes v1.33 Updates to Remember:**  
- Use `node-role.kubernetes.io/control-plane` instead of `master` (deprecated).  
- `startupProbe`, `livenessProbe`, and `readinessProbe` are stable and exam-relevant.  
- HPA supports CPU and memory natively; custom metrics need an adapter (Prometheus/Stackdriver).  

---


## 3. Services & Networking (20%)

This domain focuses on the networking mechanisms in Kubernetes. You need to understand **Pod-to-Pod communication, Services, DNS, ingress traffic, the new Gateway API, and network security (NetworkPolicies).**

---

### ✅ Understand Connectivity Between Pods

**Pod networking**: Every Pod gets its own IP address. By default, all Pods in the cluster can communicate with each other without NAT.  

You should be able to:
- Troubleshoot unreachable Pods
- Test connectivity using tools like `ping`, `curl`, and `nslookup`

**Example:**
```bash
kubectl exec -it pod-a -- ping pod-b
kubectl exec -it pod-a -- curl http://<pod-ip>:<port>
```

---

### ✅ Define and Enforce Network Policies

**NetworkPolicy**: Controls Pod-to-Pod traffic. By default, all traffic is allowed; once a NetworkPolicy is applied, traffic is denied unless explicitly allowed.  

- Policies use **labels** (Pods/Namespaces)  
- Enforced only when the **CNI plugin** supports it (e.g., Calico, Cilium)  

**Example:**
```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

This allows traffic **from frontend Pods → to backend Pods**, blocking all other ingress.

---

### ✅ Use ClusterIP, NodePort, LoadBalancer Service Types and Endpoints

**Service**: Provides stable networking to Pods (since Pod IPs are ephemeral).  

Types:
- **ClusterIP (default):** Internal-only access within the cluster.  
- **NodePort:** Exposes service on each node’s IP at a static port (30000–32767).  
- **LoadBalancer:** Integrates with cloud provider to provision an external LB.  

**Example:**
```bash
kubectl expose pod nginx --port=80 --type=NodePort
kubectl get svc
```

---

### ✅ Use the Gateway API to Manage Ingress Traffic

**Gateway API**: The modern replacement for Ingress. Provides more **flexible routing**, standardization, and portability across different implementations.

Key resources:
- **GatewayClass** → defines the type of gateway (like IngressClass).  
- **Gateway** → represents a load balancer/data plane instance.  
- **HTTPRoute** → defines routing rules for traffic.  

**Example:**
```bash
kubectl get gatewayclasses
kubectl describe gateway <name>
```

👉 Expect Gateway API scenarios on the exam (Ingress still appears, but Gateway API is the future).

---

### ✅ Know How to Use Ingress Controllers and Ingress Resources

**Ingress**: Routes external HTTP(S) traffic into the cluster based on hostnames/paths. Requires an **Ingress Controller** (e.g., `nginx-ingress`, HAProxy, Traefik).  

**Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

Verify the Ingress Controller is deployed:
```bash
kubectl get pods -n ingress-nginx
```

---

### ✅ Understand and Use CoreDNS

**CoreDNS**: The DNS server inside Kubernetes clusters. Handles service discovery (resolving `service.namespace.svc.cluster.local`).  

- Pods and Services get automatic DNS entries.  
- Configurable via the `Corefile` in the `coredns` ConfigMap.  

**Example:**
```bash
kubectl exec -it <pod> -- nslookup kubernetes.default
kubectl -n kube-system get configmap coredns -o yaml
```

---

✅ **Kubernetes v1.33 Updates to Remember:**  
- **Gateway API** is exam-relevant → expect `GatewayClass`, `Gateway`, and `HTTPRoute` tasks.  
- NetworkPolicy enforcement depends on CNI → exam clusters often use Calico.  
- `kubectl get endpoints` helps verify which Pods are backing a Service.  
- CoreDNS is stable and often part of troubleshooting scenarios.  

---

## 4. Storage (10%)

This section tests your knowledge of **persistent storage in Kubernetes**. Unlike temporary storage (like `emptyDir` which disappears when the Pod dies), persistent storage ensures that data survives Pod restarts or rescheduling.

In Kubernetes, storage is managed through **Persistent Volumes (PVs)** and **Persistent Volume Claims (PVCs)**, usually provisioned dynamically via **StorageClasses**.

---

### ✅ Implement Storage Classes and Dynamic Volume Provisioning

- **StorageClass**: Think of this as a *template* that tells Kubernetes *how* to create volumes (what type, speed, reclaim policy).  
- **Dynamic provisioning**: Instead of manually creating volumes, Kubernetes automatically provisions them when a PVC requests storage.  

**Example:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Apply and check:
```bash
kubectl apply -f storageclass.yaml
kubectl get sc
```

👉 In real clusters, cloud providers (AWS, GCP, Azure) or CSI drivers (e.g., rook-ceph, longhorn) are used as the **provisioner**.

---

### ✅ Configure Volume Types, Access Modes, and Reclaim Policies

**Volume Types** define where data is stored:
- `hostPath`: A path on the node’s filesystem (for single-node testing only).  
- `emptyDir`: Temporary space that vanishes when the Pod is deleted.  
- `nfs`, `cephfs`, `glusterfs`, CSI drivers → external, shared storage backends.  

**Access Modes** (who can read/write):
- `ReadWriteOnce (RWO)` → One node can read/write.  
- `ReadOnlyMany (ROX)` → Many nodes, read-only.  
- `ReadWriteMany (RWX)` → Many nodes can read/write simultaneously.  

**Reclaim Policies** (what happens when PVC is deleted):
- `Retain` → Keep the data for manual cleanup.  
- `Delete` → Delete the volume automatically.  
- `Recycle` → Deprecated in v1.33, no longer used.  

**Example PV:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

Apply and verify:
```bash
kubectl apply -f pv.yaml
kubectl get pv
```

---

### ✅ Manage Persistent Volumes (PV) and Persistent Volume Claims (PVC)

- **PersistentVolume (PV)**: The actual piece of storage (like a disk).  
- **PersistentVolumeClaim (PVC)**: A request for storage by a Pod. Kubernetes binds a PVC to a matching PV automatically.  

**PVC Example:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

**Pod using the PVC:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: demo-pvc
```

Check:
```bash
kubectl get pvc
kubectl describe pod pvc-pod
```

---

### ✅ Kubernetes v1.33 Updates to Remember (Storage)

- **Recycle policy is removed** → only `Retain` and `Delete` remain.  
- **Image Volumes (beta)** → use container images as data sources for volumes.  
- **Volume Populators GA** → custom logic can pre-populate PVCs (e.g., with test data).  
- **CSI (Container Storage Interface)** is the standard → older in-tree plugins are deprecated.  

---

---

## 5. Troubleshooting (30%)

This is the **most important section of the CKA exam**. It tests your ability to **debug and recover a Kubernetes cluster** when things break.  
You’ll be expected to quickly identify issues with **nodes, Pods, control plane components, networking, and logs** — under time pressure.  

---

### ✅ Troubleshoot Clusters and Nodes

Nodes are the "machines" (VMs or physical) where Pods run. If a node is unhealthy, workloads may fail.  

- **Check cluster health:**
```bash
kubectl get nodes
kubectl describe node <node-name>
```

- **Put a node into maintenance mode (drain):**
```bash
kubectl drain <node-name> --ignore-daemonsets
```

- **Bring it back after maintenance:**
```bash
kubectl uncordon <node-name>
```

👉 *Think of this as telling Kubernetes “stop scheduling new Pods here, move the old ones out” and then “okay, this node is ready again.”*

---

### ✅ Troubleshoot Cluster Components

The control plane consists of components like:
- **etcd** → the cluster’s database  
- **kube-apiserver** → the main API for kubectl and clients  
- **kube-scheduler** → decides which node runs a Pod  
- **kube-controller-manager** → ensures desired state  

These are usually run as **static Pods** (YAML manifests in `/etc/kubernetes/manifests`).  

- **Check control plane Pods:**
```bash
ls /etc/kubernetes/manifests
```

- **Check kubelet logs (runs on every node):**
```bash
journalctl -u kubelet
```

👉 If the API server is down, `kubectl` won’t work — you may need to check logs directly on the node.  

---

### ✅ Monitor Cluster and Application Resource Usage

Resource bottlenecks (CPU/memory pressure) often cause issues.  

- **Use metrics-server (must be installed):**
```bash
kubectl top nodes
kubectl top pods --all-namespaces
```

- **Sort by usage:**
```bash
kubectl top pods --sort-by=cpu
```

👉 This shows if Pods are consuming too many resources or if a node is overloaded.  

---

### ✅ Manage and Evaluate Container Output Streams

Logs are your best friend in debugging.  

- **Check Pod logs:**
```bash
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>   # for multi-container Pods
kubectl logs -f <pod-name>                    # stream logs
```

- **Check node process logs (e.g., kubelet):**
```bash
journalctl -u kubelet -f
```

👉 Use `-f` (follow) to watch logs in real time as you reproduce issues.  

---

### ✅ Troubleshoot Services and Networking

Networking is one of the most common failure points.  

- **Check Services:**
```bash
kubectl get svc
kubectl describe svc <service-name>
```

- **Test DNS resolution inside a Pod:**
```bash
kubectl exec -it <pod-name> -- nslookup <service-name>
```

- **Test connectivity between Pods:**
```bash
kubectl exec -it <source-pod> -- curl <target-pod-ip>:<port>
```

👉 This helps you confirm whether the issue is with DNS, Pod IPs, or Service routing.  

---

### ✅ Kubernetes v1.33 Updates to Remember (Troubleshooting)

- `kubectl get events --sort-by=.metadata.creationTimestamp` → helpful to see why scheduling failed.  
- Gateway API troubleshooting (v1.33 exam includes Gateway instead of just Ingress).  
- Container runtimes (`containerd`, `crictl`) often used for low-level debugging.  
- `kubectl debug` (ephemeral containers) is now stable and commonly tested.  

---

### Resources to Prepare

- [Kubernetes Documentation](https://kubernetes.io/docs/)  
- [Troubleshooting Guide](https://kubernetes.io/docs/tasks/debug/)  
- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)  

---

## CKA Exam Questions and Answers

This section contains **hands-on practice scenarios** that mirror the exam.  
For each, we include: **Task → Reasoning → Solution (YAML/commands) → Exam tips**.

---

### 🟦 Schedule Pod on Master Node

**Task**  
Create a Pod `pod1` in `default` using image `httpd:2.4.41-alpine`.  
- Container name: `pod1-container`  
- Pod must **only** run on a master node (do not add new labels).  
- Write to `/opt/course/2/master_schedule_reason` why Pods are not scheduled on masters by default.

**Reasoning**  
Masters are typically tainted with `NoSchedule` to protect control-plane resources. To place a Pod there:  
1) **toleration** to ignore the taint, and 2) **nodeSelector** to target the master.

**Solution**

Inspect taints/labels:
```bash
kubectl get nodes
kubectl describe node cluster1-master1 | grep -i taint
kubectl get node cluster1-master1 --show-labels
```

Create YAML:
```bash
kubectl run pod1 --image=httpd:2.4.41-alpine -o yaml --dry-run=client > pod1.yaml
```

Edit and apply:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: pod1-container
    image: httpd:2.4.41-alpine
  tolerations:
  - key: node-role.kubernetes.io/master
    effect: NoSchedule
  nodeSelector:
    node-role.kubernetes.io/master: ""
```
```bash
kubectl apply -f pod1.yaml
kubectl get pod pod1 -o wide
echo "Master nodes are tainted to avoid scheduling regular workloads" > /opt/course/2/master_schedule_reason
```

**Exam tips**
- You need **both** toleration and nodeSelector.  
- Don’t relabel nodes; the task forbids “adding new labels”.

---

### 🟦 Storage with PV, PVC, Deployment

**Task**  
- PV `safari-pv`: capacity 2Gi, `ReadWriteOnce`, `hostPath: /Volumes/Data`, no storageClass.  
- PVC `safari-pvc` in `project-tiger`: request 2Gi, `ReadWriteOnce`, no storageClass.  
- Deployment `safari` in `project-tiger` running `httpd:2.4.41-alpine` mounting at `/tmp/safari-data`.

**Reasoning**  
PV is the “disk”, PVC is the “request”. If specs match, binding occurs. Deployment mounts the claim.

**Solution**

PV:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: safari-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/Volumes/Data"
```
PVC:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: safari-pvc
  namespace: project-tiger
spec:
  accessModes: [ "ReadWriteOnce" ]
  resources:
    requests:
      storage: 2Gi
```
Deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: safari
  namespace: project-tiger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: safari
  template:
    metadata:
      labels:
        app: safari
    spec:
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: safari-pvc
      containers:
      - name: container
        image: httpd:2.4.41-alpine
        volumeMounts:
        - name: data
          mountPath: /tmp/safari-data
```

Apply & verify:
```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deploy.yaml
kubectl -n project-tiger get pv,pvc,deploy,pods -o wide
```

**Exam tips**
- If PVC stays `Pending`, check size/access modes and that storageClassName is unset on **both** PV & PVC.  
- HostPath is node-local; for exams it’s fine.

---

### 🟦 RBAC with ServiceAccount, Role, RoleBinding

**Task**  
Create ServiceAccount `processor` in `project-hamster`. Create Role/RoleBinding (both named `processor`) to allow **only** `create` on **Secrets** and **ConfigMaps** in that namespace.

**Reasoning**  
Role defines allowed verbs/resources **in namespace**; RoleBinding attaches it to the ServiceAccount.

**Solution**
```bash
kubectl -n project-hamster create sa processor
kubectl -n project-hamster create role processor   --verb=create --resource=secrets --resource=configmaps
kubectl -n project-hamster create rolebinding processor   --role=processor --serviceaccount=project-hamster:processor
```
Verify:
```bash
kubectl -n project-hamster auth can-i create secret   --as system:serviceaccount:project-hamster:processor
kubectl -n project-hamster auth can-i create configmap   --as system:serviceaccount:project-hamster:processor
kubectl -n project-hamster auth can-i delete secret   --as system:serviceaccount:project-hamster:processor
```

**Exam tips**
- Mind pluralization: **secrets**, **configmaps**.  
- Use `kubectl -n <ns> auth can-i ...` to prove it.

---

### 🟦 DaemonSet on All Nodes

**Task**  
Namespace: `project-tiger`. Create DaemonSet `ds-important` (image `httpd:2.4-alpine`), labels `id=ds-important`, `uuid=18426a0b-5f59-4e10-923f-c0e078e82462`. Requests: `cpu:10m`, `memory:10Mi`. Must run on **all nodes including masters**.

**Reasoning**  
DaemonSet = one Pod per node. Need **toleration** for master taint.

**Solution**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-important
  namespace: project-tiger
  labels:
    id: ds-important
    uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
spec:
  selector:
    matchLabels:
      id: ds-important
  template:
    metadata:
      labels:
        id: ds-important
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: ds-important
        image: httpd:2.4-alpine
        resources:
          requests:
            cpu: 10m
            memory: 10Mi
```

Verify:
```bash
kubectl -n project-tiger get ds
kubectl -n project-tiger get pods -o wide -l id=ds-important
```

**Exam tips**
- DaemonSets don’t have `replicas`.  
- Add toleration for scheduling on masters.

---

### 🟦 Deployment with Anti-Affinity or Spread Constraints

**Task**  
Namespace `project-tiger`. Create Deployment `deploy-important` with label `id=very-important`, replicas=3.  
Two containers per Pod:  
- `container1`: `nginx:1.17.6-alpine`  
- `container2`: `kubernetes/pause`  
Ensure **max one Pod per worker node**; with 2 workers, only 2 Pods run, 1 Pending.

**Reasoning**  
Use **podAntiAffinity** or **topologySpreadConstraints** to distribute Pods across nodes and **avoid co-location**.

**Solution A — podAntiAffinity**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-important
  namespace: project-tiger
  labels:
    id: very-important
spec:
  replicas: 3
  selector:
    matchLabels:
      id: very-important
  template:
    metadata:
      labels:
        id: very-important
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: id
                operator: In
                values: ["very-important"]
            topologyKey: kubernetes.io/hostname
      containers:
      - name: container1
        image: nginx:1.17.6-alpine
      - name: container2
        image: kubernetes/pause
```

**Solution B — topologySpreadConstraints**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-important
  namespace: project-tiger
  labels:
    id: very-important
spec:
  replicas: 3
  selector:
    matchLabels:
      id: very-important
  template:
    metadata:
      labels:
        id: very-important
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            id: very-important
      containers:
      - name: container1
        image: nginx:1.17.6-alpine
      - name: container2
        image: kubernetes/pause
```

Verify behavior:
```bash
kubectl -n project-tiger get deploy,pods -o wide -l id=very-important
```
Expect 2/3 Pods running (one per worker), 1 Pending.

**Exam tips**
- `topologyKey` usually `kubernetes.io/hostname`.  
- If a Pod stays `Pending`, read the **Events** in `kubectl describe pod`.

---

### 🟦 Multi-Container Pod with Shared Volume

**Task**  
Create Pod `multi-container-playground` in `default` with three containers: `c1`, `c2`, `c3` and a shared, **non-persistent** volume.  
- `c1`: `nginx:1.17.6-alpine`, env `MY_NODE_NAME` = node name.  
- `c2`: `busybox:1.31.1`, write `date` every second to `/vol/date.log`.  
- `c3`: `busybox:1.31.1`, `tail -f /vol/date.log`.  
Confirm logs in `c3`.

**Reasoning**  
Use `emptyDir:` as in-memory/shared ephemeral storage between containers in the same Pod.

**Solution**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-playground
spec:
  volumes:
  - name: vol
    emptyDir: {}
  containers:
  - name: c1
    image: nginx:1.17.6-alpine
    env:
    - name: MY_NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    volumeMounts:
    - name: vol
      mountPath: /vol
  - name: c2
    image: busybox:1.31.1
    command: ["sh","-c","while true; do date >> /vol/date.log; sleep 1; done"]
    volumeMounts:
    - name: vol
      mountPath: /vol
  - name: c3
    image: busybox:1.31.1
    command: ["sh","-c","tail -f /vol/date.log"]
    volumeMounts:
    - name: vol
      mountPath: /vol
```

Apply & verify:
```bash
kubectl apply -f pod.yaml
kubectl get pod multi-container-playground
kubectl exec multi-container-playground -c c1 -- env | grep MY_NODE_NAME
kubectl logs multi-container-playground -c c3 | head
```

**Exam tips**
- `emptyDir` is perfect for **sharing** within a Pod, not across Pods.  
- `fieldRef: spec.nodeName` exposes the node name.

---

### 🟦 Secrets: Mount + Env Vars

**Task**  
- Create namespace `secret`.  
- There is an existing Secret manifest at `/opt/course/19/secret1.yaml` → create it in `secret` and mount read-only at `/tmp/secret1`.  
- Create Secret `secret2` in `secret` with `user=user1`, `pass=1234`. Inject as env `APP_USER` and `APP_PASS` in a long-running `busybox:1.31.1` Pod named `secret-pod` that can run on masters (add toleration). Verify.

**Reasoning**  
Secrets can be **mounted as files** or **exposed as env vars**. Toleration allows scheduling on tainted masters.

**Solution**

Create namespace & Secret #1:
```bash
kubectl create ns secret
# edit the namespace inside the provided file if needed
sed -e 's/namespace: .*/namespace: secret/' /opt/course/19/secret1.yaml > /tmp/secret1.yaml
kubectl apply -f /tmp/secret1.yaml
```

Create Secret #2:
```bash
kubectl -n secret create secret generic secret2   --from-literal=user=user1 --from-literal=pass=1234
```

Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
  namespace: secret
spec:
  tolerations:
  - key: node-role.kubernetes.io/master
    effect: NoSchedule
  containers:
  - name: secret-pod
    image: busybox:1.31.1
    command: ["sh","-c","sleep 1d"]
    env:
    - name: APP_USER
      valueFrom:
        secretKeyRef:
          name: secret2
          key: user
    - name: APP_PASS
      valueFrom:
        secretKeyRef:
          name: secret2
          key: pass
    volumeMounts:
    - name: secret1
      mountPath: /tmp/secret1
      readOnly: true
  volumes:
  - name: secret1
    secret:
      secretName: secret1
```

Verify:
```bash
kubectl -n secret apply -f pod.yaml
kubectl -n secret exec secret-pod -- env | grep APP_
kubectl -n secret exec secret-pod -- ls -l /tmp/secret1
```

**Exam tips**
- Mounting a Secret as a volume is read-only by default.  
- Ensure namespace matches in Secret manifests.

---

### 🟦 Upgrade Node and Join Cluster

**Task**  
A node (`cluster3-worker2`) is not part of the cluster and runs an older Kubernetes. Upgrade it to the **exact** version running on `cluster3-master1` and **join** it to the cluster using `kubeadm`.

**Reasoning**  
Workers must match (or be compatible with) control-plane minor version. Use `kubeadm` to join.

**Solution**

On control plane, find version & get join command:
```bash
kubectl get nodes -o wide
sudo kubeadm token create --print-join-command
# Example output:
# kubeadm join <CP_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

On the target worker (`cluster3-worker2`), install matching versions (example for Debian/Ubuntu):
```bash
# Make sure repos are configured per Kubernetes docs, then:
sudo apt-get update
# Pin to the version you saw on the control plane (example v1.33.0-00):
sudo apt-get install -y kubelet=1.33.0-00 kubeadm=1.33.0-00 kubectl=1.33.0-00
sudo systemctl enable --now kubelet
```

If the node was previously initialized, reset first:
```bash
sudo kubeadm reset -f
sudo rm -rf ~/.kube
```

Join:
```bash
sudo <PASTE_THE_JOIN_COMMAND_FROM_MASTER>
# e.g. sudo kubeadm join <CP_IP>:6443 --token ... --discovery-token-ca-cert-hash sha256:...
```

Back on control plane:
```bash
kubectl get nodes
```

**Exam tips**
- Use the **exact** version numbers from the control plane.  
- If join fails, check `journalctl -u kubelet -f` and consider `kubeadm reset -f` then re-join.

---

### 🟦 NetworkPolicy with Egress Control

**Task**  
Namespace `project-snake`. Create NetworkPolicy `np-backend` to allow **only**:  
- backend-* Pods → db1-* Pods on port **1111**  
- backend-* Pods → db2-* Pods on port **2222**  
Afterwards, backend → vault on 3333 must be **blocked**. Selector uses label `app`.

**Reasoning**  
Using **egress** rules with label selectors and ports. When any NetworkPolicy selects Pods for **egress**, **all other egress is denied** unless allowed.

**Solution**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-snake
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: db1
    ports:
    - protocol: TCP
      port: 1111
  - to:
    - podSelector:
        matchLabels:
          app: db2
    ports:
    - protocol: TCP
      port: 2222
```

Test:
```bash
kubectl -n project-snake exec backend-0 -- curl -s <db1-ip>:1111
kubectl -n project-snake exec backend-0 -- curl -s <db2-ip>:2222
kubectl -n project-snake exec backend-0 -- curl -s <vault-ip>:3333
```

**Exam tips**
- Multiple **egress** items are **OR**; within each item, `to` + `ports` are **AND**.  
- Your CNI must support NetworkPolicy (Calico, Cilium, etc.).

---

### 🟦 Etcd Backup and Restore

**Task**  
- Take an etcd **snapshot** on `cluster3-master1` to `/tmp/etcd-backup.db`.  
- Create any test Pod.  
- Restore the snapshot and verify the test Pod is gone (state rolled back).

**Reasoning**  
`etcdctl snapshot save` needs certs to auth to etcd. After restore to a new data dir, point the etcd static Pod to that dir and restart control plane.

**Solution**

On control plane, save snapshot (paths may vary with your install):
```bash
export ETCDCTL_API=3
etcdctl --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key   snapshot save /tmp/etcd-backup.db
```

Create a test Pod:
```bash
kubectl run test --image=nginx
kubectl get pods -l run=test -w
```

Stop control plane by moving static Pod manifests out temporarily:
```bash
sudo mkdir -p /root/manifests-bak
sudo mv /etc/kubernetes/manifests/* /root/manifests-bak/
# wait for components to stop
```

Restore snapshot to a fresh data dir:
```bash
etcdctl snapshot restore /tmp/etcd-backup.db --data-dir /var/lib/etcd-backup
```

Point etcd manifest to the new data dir (`/etc/kubernetes/manifests/etcd.yaml`):
```yaml
# find the hostPath volume named etcd-data and change its path:
hostPath:
  path: /var/lib/etcd-backup
  type: DirectoryOrCreate
```

Start control plane again:
```bash
sudo mv /root/manifests-bak/* /etc/kubernetes/manifests/
# wait for components to come back
kubectl get pods -A
kubectl get pods -l run=test
# expected: Not found (rolled back state)
```

**Exam tips**
- Use the **server cert/key** or **apiserver-etcd-client** pair depending on your setup.  
- Be careful editing files under `/etc/kubernetes/manifests` — kubelet auto-restarts components.

---

## Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)  
- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)  
- [CNCF CKA Page](https://www.cncf.io/certification/cka/)  

---

## 💬 Direct Candidate Feedback: 2025 CKA Exam Insights (August Update)

> “The exam has shifted. It’s less about copying manifests and more about **troubleshooting**, **Helm**, **CRDs**, and **Gateway APIs**.”

> “I left 3 questions blank — Helm packaging, Gateway API, and cluster setup using `kubeadm`. It was tougher than expected.”

> “Gateway API and HTTPRoute threw me off — the documentation wasn’t clear. Helm was tricky too, but overall manageable.”

> “Topics I saw: **HPA**, **Ingress**, **Helm**, **ArgoCD**, **cert-manager CRDs**, **container runtimes**, and setting up **NetworkPolicies with least privilege**.”

> “The new format made me think more deeply. I had to actually solve problems — not just recognize YAML.”

> “Low-resolution VM made it hard to read docs. And some questions were significantly more complex than the mock exams.”

> “Expect **lag, tab loading delays, and broken copy-paste**. Know your `kubectl`, `vim`, and command-line tools well.”

> “Mouse movement was sluggish — keyboard navigation with `:vsplit` and `kubectl explain` helped me save time.”


## 📌 Key Topic Coverage Summary (2025)

| Topic Area                  | Common Feedback Summary                                                                 |
|-----------------------------|------------------------------------------------------------------------------------------|
| **Helm & Packaging**        | Required for install and templating questions; Helm + dpkg appeared                     |
| **Gateway API / HTTPRoute** | Often noted as tricky; vague docs made questions harder                                 |
| **CRI / containerd**        | Manual installs and configuration came up in several questions                          |
| **CRDs & Admission Webhooks** | Edge-case topics that still showed up — be familiar                                     |
| **HPA, ArgoCD, NetPol**     | Popular topics; applied in realistic scenarios like scaling and access control          |
| **PVC, Volumes, Ingress**   | Included in questions requiring debug/fix/validate workflows                            |
| **Cluster Setup (kubeadm)** | Appeared unexpectedly; time-consuming if unprepared                                     |
| **PSI Exam UI**             | Reported lag, poor copy/paste support, and low-res experience                           |
| **Keyboard Proficiency**    | Critical due to interface slowness — `vim`, tabs, and shortcuts helped significantly    |


## Additional Resources
* 💬 [Kubernetes Slack Channel #certifications](https://kubernetes.slack.com/)<sup>Slack</sup>
* 📚 [Guide to Certified Kubernetes Administrator (CKA)]([https://techwithmohamed.com/cka-exam-study-guide/](https://techwithmohamed.com/blog/ckad-exam-study/))<sup>Blog</sup>
* 🎞️ [Udemy: CKA Certified Kubernetes AdministratorCrash Course](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)<sup>Video Course</sup>
* 🎞️ [Certified Kubernetes Administrator (CKA) - A Cloud Guru (formerly Linux Academy)](https://acloudguru.com/course/cloud-native-certified-kubernetes-administrator-cka/)<sup>Video Course</sup>
* 🎞️ [Kubernetes Fundamentals (LFS258) - Linux Foundation](https://training.linuxfoundation.org/training/kubernetes-fundamentals/)<sup>Official Course</sup>
* 🎞️ [Kubernetes Deep Dive - A Cloud Guru](https://acloud.guru/learn/kubernetes-deep-dive)<sup>Video Course</sup>

## Practice
Practice a lot with Kubernetes:

- [CKA Simulator - killer.sh](https://killer.sh/cka)
- [Kubernetes the Hard Way by Kelsey Hightower](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [CKA Scenarios - killercoda.com](https://killercoda.com/killer-shell-cka)
- [Learning Playground - by Docker](https://labs.play-with-k8s.com/)


## 💬 Share To Your Network
If this repo has helped you in any way, feel free to share !

