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

- **Check if components are reporting healthy:**
```bash
kubectl get componentstatuses
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

## CKA Exam Questions And Answers

## Schedule Pod on Master Node
> Create a single Pod of image httpd:2.4.41-alpine in Namespace default. The Pod should be named pod1 and the container should be named pod1-container. This Pod should only be scheduled on a master node, do not add new labels any nodes.
Shortly write the reason on why Pods are by default not scheduled on master nodes into /opt/course/2/master_schedule_reason .

<details><summary>Show Answer</summary>
<p>

First we find the master node(s) and their taints:

``` bash
k get node # find master node

k describe node cluster1-master1 | grep Taint # get master node taints

k describe node cluster1-master1 | grep Labels -A 10 # get master node labels

k get node cluster1-master1 --show-labels # OR: get master node labels
```

Next we create the Pod template:

``` bash
# check the export on the very top of this document so we can use $do
k run pod1 --image=httpd:2.4.41-alpine $do > 2.yaml

vim 2.yaml
```

Perform the necessary changes manually. Use the Kubernetes docs and search for example for tolerations and nodeSelector to find examples:


``` yaml
# 2.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: httpd:2.4.41-alpine
    name: pod1-container                  # change
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  tolerations:                            # add
  - effect: NoSchedule                    # add
    key: node-role.kubernetes.io/master   # add
  nodeSelector:                           # add
    node-role.kubernetes.io/master: ""    # add
status: {}
```

Important here to add the toleration for running on master nodes, but also the nodeSelector to make sure it only runs on master nodes. If we only specify a toleration the Pod can be scheduled on master or worker nodes.

Now we create it:
``` bash
k -f 2.yaml create
```

Let's check if the pod is scheduled:

``` bash
➜ k get pod pod1 -o wide
NAME   READY   STATUS    RESTARTS   ...    NODE               NOMINATED NODE
pod1   1/1     Running   0          ...    cluster1-master1   <none>     

```


Finally the short reason why Pods are not scheduled on master nodes by default:

``` bash
# /opt/course/2/master_schedule_reason
master nodes usually have a taint defined
```

</p>
</details>


## Storage, PV, PVC, Pod volume

> Create a new PersistentVolume named safari-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

> Next create a new PersistentVolumeClaim in Namespace project-tiger named safari-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

> Finally create a new Deployment safari in Namespace project-tiger which mounts that volume at /tmp/safari-data. The Pods of that Deployment should be of image httpd:2.4.41-alpine.

<details><summary>Show Answer</summary>
<p>
``` bash
vim 6_pv.yaml
```
Find an example from https://kubernetes.io/docs and alter it:

``` yaml
# 6_pv.yaml
kind: PersistentVolume
apiVersion: v1
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
Then create it:
``` bash
k -f 6_pv.yaml create

```
Next the PersistentVolumeClaim:

``` bash
vim 6_pvc.yaml

```
Find an example from https://kubernetes.io/docs and alter it:

``` yaml
# 6_pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: safari-pvc
  namespace: project-tiger
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
     storage: 2Gi

```
Then create:

``` bash
k -f 6_pvc.yaml create

```
And check that both have the status Bound:

``` bash 
k -n project-tiger get pv,pvc
NAME                         CAPACITY  ... STATUS   CLAIM                    ...
persistentvolume/safari-pv   2Gi       ... Bound    project-tiger/safari-pvc ...

NAME                               STATUS   VOLUME      CAPACITY ...
persistentvolumeclaim/safari-pvc   Bound    safari-pv   2Gi      ...
Next we create a Deployment and mount that volume:

k -n project-tiger create deploy safari \
  --image=httpd:2.4.41-alpine $do > 6_dep.yaml

vim 6_dep.yaml
```
Alter the yaml to mount the volume:

``` yaml 
# 6_dep.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: safari
  name: safari
  namespace: project-tiger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: safari
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: safari
    spec:
      volumes:                                      # add
      - name: data                                  # add
        persistentVolumeClaim:                      # add
          claimName: safari-pvc                     # add
      containers:
      - image: httpd:2.4.41-alpine
        name: container
        volumeMounts:                               # add
        - name: data                                # add
          mountPath: /tmp/safari-data               # add
```
``` bash 
k -f 6_dep.yaml create

```
We can confirm its mounting correctly:

``` bash
k -n project-tiger describe pod safari-5cbf46d6d-mjhsb  | grep -A2 Mounts:   
    Mounts:
      /tmp/safari-data from data (rw) # there it is
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-n2sjj (ro)
```
</p>
</details>


## RBAC ServiceAccount Role RoleBinding
> Create a new ServiceAccount processor in Namespace project-hamster. Create a Role and RoleBinding, both named processor as well. These should allow the new SA to only create Secrets and ConfigMaps in that Namespace.

<details><summary>Show Answer</summary>
<p>

``` bash
k -n project-tiger describe pod safari-5cbf46d6d-mjhsb  | grep -A2 Mounts:   
    Mounts:
      /tmp/safari-data from data (rw) # there it is
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-n2sjj (ro)
```

Let's talk a little about RBAC resources
A ClusterRole|Role defines a set of permissions and where it is available, in the whole cluster or just a single Namespace.

A ClusterRoleBinding|RoleBinding connects a set of permissions with an account and defines where it is applied, in the whole cluster or just a single Namespace.

Because of this there are 4 different RBAC combinations and 3 valid ones:

Role + RoleBinding (available in single Namespace, applied in single Namespace)
ClusterRole + ClusterRoleBinding (available cluster-wide, applied cluster-wide)
ClusterRole + RoleBinding (available cluster-wide, applied in single Namespace)
Role + ClusterRoleBinding (NOT POSSIBLE: available in single Namespace, applied cluster-wide)

To the solution
We first create the ServiceAccount:

``` bash
k -n project-hamster create sa processor
serviceaccount/processor created
```
Then for the Role:

``` bash
k -n project-hamster create role -h # examples

```
So we execute:
``` bash
k -n project-hamster create role processor \
  --verb=create \
  --resource=secret \
  --resource=configmap
```
Which will create a Role like:

``` yaml 
# kubectl -n project-hamster create role accessor --verb=create --resource=secret --resource=configmap
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: processor
  namespace: project-hamster
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  - configmaps
  verbs:
  - create

```
Now we bind the Role to the ServiceAccount:

``` bash
k -n project-hamster create rolebinding -h # examples

```

So we create it:

``` bash
k -n project-hamster create rolebinding processor \
  --role processor \
  --serviceaccount project-hamster:processor
```
This will create a RoleBinding like:

``` yaml 
# kubectl -n project-hamster create rolebinding processor --role processor --serviceaccount project-hamster:processor
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: processor
  namespace: project-hamster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: processor
subjects:
- kind: ServiceAccount
  name: processor
  namespace: project-hamster
```

To test our RBAC setup we can use kubectl auth can-i:

```bash
k auth can-i -h # examples
```

Like this:

``` bash
k -n project-hamster auth can-i create secret \
  --as system:serviceaccount:project-hamster:processor
yes

k -n project-hamster auth can-i create configmap \
  --as system:serviceaccount:project-hamster:processor
yes

k -n project-hamster auth can-i create pod \
  --as system:serviceaccount:project-hamster:processor
no

k -n project-hamster auth can-i delete secret \
  --as system:serviceaccount:project-hamster:processor
no

k -n project-hamster auth can-i get configmap \
  --as system:serviceaccount:project-hamster:processor
no
```

</p>
</details>
 
##  DaemonSet on all Nodes
> Use Namespace project-tiger for the following. Create a DaemonSet named ds-important with image httpd:2.4-alpine and labels id=ds-important and uuid=18426a0b-5f59-4e10-923f-c0e078e82462.

> The Pods it creates should request 10 millicore cpu and 10 mebibyte memory. The Pods of that DaemonSet should run on all nodes, master and worker.

<details><summary>Show Answer</summary>
<p>

As of now we aren't able to create a DaemonSet directly using kubectl, so we create a Deployment and just change it up:

``` bash
k -n project-tiger create deployment --image=httpd:2.4-alpine ds-important $do > 11.yaml
vim 11.yaml

```
(Sure yuu could also search for a DaemonSet example yaml in the Kubernetes docs and alter it.)
Then we adjust the yaml to:

``` yaml 
# 11.yaml
apiVersion: apps/v1
kind: DaemonSet                                     # change from Deployment to Daemonset
metadata:
  creationTimestamp: null
  labels:                                           # add
    id: ds-important                                # add
    uuid: 18426a0b-5f59-4e10-923f-c0e078e82462      # add
  name: ds-important
  namespace: project-tiger                          # important
spec:
  #replicas: 1                                      # remove
  selector:
    matchLabels:
      id: ds-important                              # add
      uuid: 18426a0b-5f59-4e10-923f-c0e078e82462    # add
  #strategy: {}                                     # remove
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: ds-important                            # add
        uuid: 18426a0b-5f59-4e10-923f-c0e078e82462  # add
    spec:
      containers:
      - image: httpd:2.4-alpine
        name: ds-important
        resources:
          requests:                                 # add
            cpu: 10m                                # add
            memory: 10Mi                            # add
      tolerations:                                  # add
      - effect: NoSchedule                          # add
        key: node-role.kubernetes.io/master         # add
#status: {}                                         # remove
```
It was requested that the DaemonSet runs on all nodes, so we need to specify the toleration for this.
Let's confirm:

``` bash
k -f 11.yaml create
k -n project-tiger get ds
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ds-important   3         3         3       3            3           <none>          8s
k -n project-tiger get pod -l id=ds-important -o wide
NAME                      READY   STATUS          NODE
ds-important-6pvgm        1/1     Running   ...   cluster1-worker1
ds-important-lh5ts        1/1     Running   ...   cluster1-master1
ds-important-qhjcq        1/1     Running   ...   cluster1-worker2
```
</p>
</details>

##  Deployment on all Nodes
> Use Namespace project-tiger for the following. Create a Deployment named deploy-important with label id=very-important (the Pods should also have this label) and 3 replicas. It should contain two containers, the first named container1 with image nginx:1.17.6-alpine and the second one named container2 with image kubernetes/pause.

> There should be only ever one Pod of that Deployment running on one worker node. We have two worker nodes: cluster1-worker1 and cluster1-worker2. Because the Deployment has three replicas the result should be that on both nodes one Pod is running. The third Pod won't be scheduled, unless a new worker node will be added.

> In a way we kind of simulate the behaviour of a DaemonSet here, but using a Deployment and a fixed number of replicas.

<details><summary>Show Answer</summary>
<p>

There are two possible ways, one using podAntiAffinity and one using topologySpreadConstraint.

### PodAntiAffinity
The idea here is that we create a "Inter-pod anti-affinity" which allows us to say a Pod should only be scheduled on a node where another Pod of a specific label (here the same label) is not already running.
Let's begin by creating the Deployment template:

``` bash
k -n project-tiger create deployment \
  --image=nginx:1.17.6-alpine deploy-important $do > 12.yaml

vim 12.yaml
```

Then change the yaml to:

``` yaml 
# 12.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    id: very-important                  # change
  name: deploy-important
  namespace: project-tiger              # important
spec:
  replicas: 3                           # change
  selector:
    matchLabels:
      id: very-important                # change
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: very-important              # change
    spec:
      containers:
      - image: nginx:1.17.6-alpine
        name: container1                # change
        resources: {}
      - image: kubernetes/pause         # add
        name: container2                # add
      affinity:                                             # add
        podAntiAffinity:                                    # add
          requiredDuringSchedulingIgnoredDuringExecution:   # add
          - labelSelector:                                  # add
              matchExpressions:                             # add
              - key: id                                     # add
                operator: In                                # add
                values:                                     # add
                - very-important                            # add
            topologyKey: kubernetes.io/hostname             # add
status: {}

```

Specify a topologyKey, which is a pre-populated Kubernetes label, you can find this by describing a node.

### TopologySpreadConstraints
We can achieve the same with topologySpreadConstraints. Best to try out and play with both.

``` yaml 
# 12.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    id: very-important                  # change
  name: deploy-important
  namespace: project-tiger              # important
spec:
  replicas: 3                           # change
  selector:
    matchLabels:
      id: very-important                # change
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: very-important              # change
    spec:
      containers:
      - image: nginx:1.17.6-alpine
        name: container1                # change
        resources: {}
      - image: kubernetes/pause         # add
        name: container2                # add
      topologySpreadConstraints:                 # add
      - maxSkew: 1                               # add
        topologyKey: kubernetes.io/hostname      # add
        whenUnsatisfiable: DoNotSchedule         # add
        labelSelector:                           # add
          matchLabels:                           # add
            id: very-important                   # add
status: {}
 
```

Apply and Run
Let's run it:

``` bash
k -f 12.yaml create
```

Then we check the Deployment status where it shows 2/3 ready count:

``` bash
k -n project-tiger get deploy -l id=very-important
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
deploy-important   2/3     3            2           2m35s
```

And running the following we see one Pod on each worker node and one not scheduled.

``` bash
k -n project-tiger get pod -o wide -l id=very-important
NAME                                READY   STATUS    ...   NODE             
deploy-important-58db9db6fc-9ljpw   2/2     Running   ...   cluster1-worker1
deploy-important-58db9db6fc-lnxdb   0/2     Pending   ...   <none>          
deploy-important-58db9db6fc-p2rz8   2/2     Running   ...   cluster1-worker2

```
If we kubectl describe the Pod deploy-important-58db9db6fc-lnxdb it will show us the reason for not scheduling is our implemented podAntiAffinity ruling:
Warning  FailedScheduling  63s (x3 over 65s)  default-scheduler  0/3 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 2 node(s) didn't match pod affinity/anti-affinity, 2 node(s) didn't satisfy existing pods anti-affinity rules.
Or our topologySpreadConstraints:
Warning  FailedScheduling  16s   default-scheduler  0/3 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 2 node(s) didn't match pod topology spread constraints.
 
</p>
</details>

## Multi Containers and Pod shared Volume
> Create a Pod named multi-container-playground in Namespace default with three containers, named c1, c2 and c3. There should be a volume attached to that Pod and mounted into every container, but the volume shouldn't be persisted or shared with other Pods.

> Container c1 should be of image nginx:1.17.6-alpine and have the name of the node where its Pod is running available as environment variable MY_NODE_NAME.

> Container c2 should be of image busybox:1.31.1 and write the output of the date command every second in the shared volume into file date.log. You can use while true; do date >> /your/vol/path/date.log; sleep 1; done for this.

> Container c3 should be of image busybox:1.31.1 and constantly send the content of file date.log from the shared volume to stdout. You can use tail -f /your/vol/path/date.log for this.

> Check the logs of container c3 to confirm correct setup.

<details><summary>Show Answer</summary>
<p>

First we create the Pod template:
``` bash
k run multi-container-playground --image=nginx:1.17.6-alpine $do > 13.yaml
vim 13.yaml
``` 
And add the other containers and the commands they should execute:

``` yaml
# 13.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-container-playground
  name: multi-container-playground
spec:
  containers:
  - image: nginx:1.17.6-alpine
    name: c1                                                                      # change
    resources: {}
    env:                                                                          # add
    - name: MY_NODE_NAME                                                          # add
      valueFrom:                                                                  # add
        fieldRef:                                                                 # add
          fieldPath: spec.nodeName                                                # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  - image: busybox:1.31.1                                                         # add
    name: c2                                                                      # add
    command: ["sh", "-c", "while true; do date >> /vol/date.log; sleep 1; done"]  # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  - image: busybox:1.31.1                                                         # add
    name: c3                                                                      # add
    command: ["sh", "-c", "tail -f /vol/date.log"]                                # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:                                                                        # add
    - name: vol                                                                   # add
      emptyDir: {}                                                                # add
status: {}
```
``` bash
k -f 13.yaml create
```
Oh boy, lot's of requested things. We check if everything is good with the Pod:

``` bash
k get pod multi-container-playground
NAME                         READY   STATUS    RESTARTS   AGE
multi-container-playground   3/3     Running   0          95s
```

Good, then we check if container c1 has the requested node name as env variable:

``` bash
k exec multi-container-playground -c c1 -- env | grep MY
MY_NODE_NAME=cluster1-worker2
```
And finally we check the logging:

``` bash
k logs multi-container-playground -c c3
Sat Dec  7 16:05:10 UTC 2077
Sat Dec  7 16:05:11 UTC 2077
Sat Dec  7 16:05:12 UTC 2077
Sat Dec  7 16:05:13 UTC 2077
Sat Dec  7 16:05:14 UTC 2077
Sat Dec  7 16:05:15 UTC 2077
Sat Dec  7 16:05:16 UTC 2077

```

</p>
</details>
 


## Create Secret and mount into Pod

> Do the following in a new Namespace secret. Create a Pod named secret-pod of image busybox:1.31.1 which should keep running for some time. It should be able to run on master nodes as well, create the proper toleration.

> There is an existing Secret located at /opt/course/19/secret1.yaml, create it in the secret Namespace and mount it readonly into the Pod at /tmp/secret1.

> Create a new Secret in Namespace secret called secret2 which should contain user=user1 and pass=1234. These entries should be available inside the Pod's container as environment variables APP_USER and APP_PASS.

> Confirm everything is working.

<details><summary>Show Answer</summary>
<p>

First we create the Namespace and the requested Secrets in it:

``` bash
k create ns secret
cp /opt/course/19/secret1.yaml 19_secret1.yaml
vim 19_secret1.yaml
```

We need to adjust the Namespace for that Secret:

``` yaml
# 19_secret1.yaml
apiVersion: v1
data:
  halt: IyEgL2Jpbi9zaAo...
kind: Secret
metadata:
  creationTimestamp: null
  name: secret1
  namespace: secret           # change

```
``` bash
k -f 19_secret1.yaml create
```

Next we create the second Secret:
``` bash
k -n secret create secret generic secret2 --from-literal=user=user1 --from-literal=pass=1234
```


Now we create the Pod template:
``` bash
k -n secret run secret-pod --image=busybox:1.31.1 $do -- sh -c "sleep 5d" > 19.yaml
vim 19.yaml

```

Then make the necessary changes:

``` yaml
# 19.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-pod
  name: secret-pod
  namespace: secret                       # add
spec:
  tolerations:                            # add
  - effect: NoSchedule                    # add
    key: node-role.kubernetes.io/master   # add
  containers:
  - args:
    - sh
    - -c
    - sleep 1d
    image: busybox:1.31.1
    name: secret-pod
    resources: {}
    env:                                  # add
    - name: APP_USER                      # add
      valueFrom:                          # add
        secretKeyRef:                     # add
          name: secret2                   # add
          key: user                       # add
    - name: APP_PASS                      # add
      valueFrom:                          # add
        secretKeyRef:                     # add
          name: secret2                   # add
          key: pass                       # add
    volumeMounts:                         # add
    - name: secret1                       # add
      mountPath: /tmp/secret1             # add
      readOnly: true                      # add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:                                # add
  - name: secret1                         # add
    secret:                               # add
      secretName: secret1                 # add
status: {}

```

It might not be necessary in current K8s versions to specify the readOnly: true because it's the default setting anyways.
And execute:
```  bash
k -f 19.yaml create
```
Finally we check if all is correct:

``` bash
k -n secret exec secret-pod -- env | grep APP
APP_PASS=1234
APP_USER=user1
k -n secret exec secret-pod -- find /tmp/secret1
/tmp/secret1
/tmp/secret1/..data
/tmp/secret1/halt
/tmp/secret1/..2019_12_08_12_15_39.463036797
/tmp/secret1/..2019_12_08_12_15_39.463036797/halt
k -n secret exec secret-pod -- cat /tmp/secret1/halt
#! /bin/sh
### BEGIN INIT INFO
# Provides:          halt
# Required-Start:
# Required-Stop:
# Default-Start:
# Default-Stop:      0
# Short-Description: Execute the halt command.
# Description:
...
```
</p>
</details>


## Update Kubernetes Version and join cluster

> Your coworker said node cluster3-worker2 is running an older Kubernetes version and is not even part of the cluster. Update Kubernetes on that node to the exact version that's running on cluster3-master1. Then add this node to the cluster. Use kubeadm for this

<details><summary>Show Answer</summary>
<p>

Upgrade Kubernetes to cluster3-master1 version
Search in the docs for kubeadm upgrade: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade

``` bash
➜ k get node
NAME               STATUS     ROLES                  AGE    VERSION
cluster3-master1   Ready      control-plane,master   116m   v1.23.1
cluster3-worker1   NotReady   <none>                 112m   v1.23.1
Master node seems to be running Kubernetes 1.23.1 and cluster3-worker2 is not yet part of the cluster.

➜ ssh cluster3-worker2

➜ root@cluster3-worker2:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.1", GitCommit:"86ec240af8cbd1b60bcc4c03c20da9b98005b92e", GitTreeState:"clean", BuildDate:"2021-12-16T11:39:51Z", GoVersion:"go1.17.5", Compiler:"gc", Platform:"linux/amd64"}

➜ root@cluster3-worker2:~# kubectl version
Client Version: version.Info{Major:"1", Minor:"22", GitVersion:"v1.22.4", GitCommit:"b695d79d4f967c403a96986f1750a35eb75e75f1", GitTreeState:"clean", BuildDate:"2021-11-17T15:48:33Z", GoVersion:"go1.16.10", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?

➜ root@cluster3-worker2:~# kubelet --version
Kubernetes v1.22.4
Here kubeadm is already installed in the wanted version, so we can run:

➜ root@cluster3-worker2:~# kubeadm upgrade node
couldn't create a Kubernetes client from file "/etc/kubernetes/kubelet.conf": failed to load admin kubeconfig: open /etc/kubernetes/kubelet.conf: no such file or directory
To see the stack trace of this error execute with --v=5 or higher
This is usually the proper command to upgrade a node. But this error means that this node was never even initialised, so nothing to update here. This will be done later using kubeadm join. For now we can continue with kubelet and kubectl:

➜ root@cluster3-worker2:~# apt update
...
Fetched 5,775 kB in 2s (2,313 kB/s)                               
Reading package lists... Done
Building dependency tree       
Reading state information... Done
90 packages can be upgraded. Run 'apt list --upgradable' to see them.

➜ root@cluster3-worker2:~# apt show kubectl -a | grep 1.23
WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
Version: 1.23.1-00
Version: 1.23.0-00

➜ root@cluster3-worker2:~# apt install kubectl=1.23.1-00 kubelet=1.23.1-00
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages will be upgraded:
  kubectl kubelet
2 upgraded, 0 newly installed, 0 to remove and 88 not upgraded.
Need to get 28.4 MB of archives.
After this operation, 2,976 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.23.1-00 [8,928 kB]
Get:2 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.23.1-00 [19.5 MB]
Fetched 28.4 MB in 2s (17.9 MB/s)  
(Reading database ... 111951 files and directories currently installed.)
Preparing to unpack .../kubectl_1.23.1-00_amd64.deb ...
Unpacking kubectl (1.23.1-00) over (1.22.4-00) ...
Preparing to unpack .../kubelet_1.23.1-00_amd64.deb ...
Unpacking kubelet (1.23.1-00) over (1.22.4-00) ...
Setting up kubectl (1.23.1-00) ...
Setting up kubelet (1.23.1-00) ...

➜ root@cluster3-worker2:~# kubelet --version
Kubernetes v1.23.1
Now we're up to date with kubeadm, kubectl and kubelet. Restart the kubelet:

➜ root@cluster3-worker2:~# systemctl restart kubelet

➜ root@cluster3-worker2:~# service kubelet status
XXX
```

We can ignore the errors and move into next step to generate the join command.

Add cluster3-master2 to cluster
First we log into the master1 and generate a new TLS bootstrap token, also printing out the join command:

``` bash
➜ ssh cluster3-master1

➜ root@cluster3-master1:~# kubeadm token create --print-join-command
kubeadm join 192.168.100.31:6443 --token leqq1l.1hlg4rw8mu7brv73 --discovery-token-ca-cert-hash sha256:2e2c3407a256fc768f0d8e70974a8e24d7b9976149a79bd08858c4d7aa2ff79a

➜ root@cluster3-master1:~# kubeadm token list
TOKEN                     TTL         EXPIRES                ...
mnkpfu.d2lpu8zypbyumr3i   23h         2020-05-01T22:43:45Z   ...
poa13f.hnrs6i6ifetwii75   <forever>   <never>                ...
``` 

We see the expiration of 23h for our token, we could adjust this by passing the ttl argument.
Next we connect again to worker2 and simply execute the join command:

``` bash
➜ ssh cluster3-worker2

➜ root@cluster3-worker2:~# kubeadm join 192.168.100.31:6443 --token leqq1l.1hlg4rw8mu7brv73 --discovery-token-3c9cf14535ebfac8a23a91132b75436b36df2c087aa99c433f79d531
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W0107 13:37:31.116994   37798 utils.go:69] The recommended value for "resolvConf" in "KubeletConfiguration" is: /run/systemd/resolve/resolv.conf; the provided value is: /run/systemd/resolve/resolv.conf
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
```

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

``` bash
➜ root@cluster3-worker2:~# service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2021-09-15 17:12:32 UTC; 42s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 24771 (kubelet)
      Tasks: 13 (limit: 467)
     Memory: 68.0M
     CGroup: /system.slice/kubelet.service
             └─24771 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kuber>
If you have troubles with kubeadm join you might need to run kubeadm reset.

```

This looks great though for us. Finally we head back to the main terminal and check the node status:

``` bash
➜ k get node
NAME               STATUS    ROLES                   AGE    VERSION
cluster3-master1   Ready      control-plane,master   24h   v1.23.1
cluster3-worker1   Ready      <none>                 24h   v1.23.1
cluster3-worker2   NotReady   <none>                 32s   v1.23.1
Give it a bit of time till the node is ready.

➜ k get node
NAME               STATUS   ROLES                  AGE    VERSION
cluster3-master1   Ready    control-plane,master   24h    v1.23.1
cluster3-worker1   Ready    <none>                 24h    v1.23.1
cluster3-worker2   Ready    <none>                 107s   v1.23.1
```
We see cluster3-worker2 is now available and up to date.

</p>
</details>
 
## NetworkPolicy

> There was a security incident where an intruder was able to access the whole cluster from a single hacked backend Pod.

> To prevent this create a NetworkPolicy called np-backend in Namespace project-snake. It should allow the backend-* Pods only to:

> connect to db1-* Pods on port 1111

> connect to db2-* Pods on port 2222

> Use the app label of Pods in your policy.

> After implementation, connections from backend-* Pods to vault-* Pods on port 3333 should for example no longer work.

<details><summary>Show Answer</summary>
<p>

First we look at the existing Pods and their labels:

``` bash
➜ k -n project-snake get pod
NAME        READY   STATUS    RESTARTS   AGE
backend-0   1/1     Running   0          8s
db1-0       1/1     Running   0          8s
db2-0       1/1     Running   0          10s
vault-0     1/1     Running   0          10s

➜ k -n project-snake get pod -L app
NAME        READY   STATUS    RESTARTS   AGE     APP
backend-0   1/1     Running   0          3m15s   backend
db1-0       1/1     Running   0          3m15s   db1
db2-0       1/1     Running   0          3m17s   db2
vault-0     1/1     Running   0          3m17s   vault
``` 
We test the current connection situation and see nothing is restricted:

```  bash
➜ k -n project-snake get pod -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP          ...
backend-0   1/1     Running   0          4m14s   10.44.0.24  ...
db1-0       1/1     Running   0          4m14s   10.44.0.25  ...
db2-0       1/1     Running   0          4m16s   10.44.0.23  ...
vault-0     1/1     Running   0          4m16s   10.44.0.22  ...

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.25:1111
database one

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.23:2222
database two

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.22:3333
vault secret storage
``` 

Now we create the NP by copying and chaning an example from the k8s docs:

```  bash
vim 24_np.yaml
``` 
```  yaml
# 24_np.yaml
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
    - Egress                    # policy is only about Egress
  egress:
    -                           # first rule
      to:                           # first condition "to"
      - podSelector:
          matchLabels:
            app: db1
      ports:                        # second condition "port"
      - protocol: TCP
        port: 1111
    -                           # second rule
      to:                           # first condition "to"
      - podSelector:
          matchLabels:
            app: db2
      ports:                        # second condition "port"
      - protocol: TCP
        port: 2222

``` 

The NP above has two rules with two conditions each, it can be read as:

allow outgoing traffic if:
  (destination pod has label app=db1 AND port is 1111)
  OR
  (destination pod has label app=db2 AND port is 2222)
 

Wrong example
Now let's shortly look at a wrong example:

``` yaml
# WRONG
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
    -                           # first rule
      to:                           # first condition "to"
      - podSelector:                    # first "to" possibility
          matchLabels:
            app: db1
      - podSelector:                    # second "to" possibility
          matchLabels:
            app: db2
      ports:                        # second condition "ports"
      - protocol: TCP                   # first "ports" possibility
        port: 1111
      - protocol: TCP                   # second "ports" possibility
        port: 2222
``` 

The NP above has one rule with two conditions and two condition-entries each, it can be read as:

allow outgoing traffic if:
  (destination pod has label app=db1 OR destination pod has label app=db2)
  AND
  (destination port is 1111 OR destination port is 2222)
Using this NP it would still be possible for backend-* Pods to connect to db2-* Pods on port 1111 for example which should be forbidden.


Create NetworkPolicy
We create the correct NP:

```  bash
k -f 24_np.yaml create
``` 

And test again:

``` bash
➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.25:1111
database one

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.23:2222
database two

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.22:3333
^C
``` 

Also helpful to use kubectl describe on the NP to see how k8s has interpreted the policy.

</p>
</details>


##  Etcd Snapshot Save and Restore

> Make a backup of etcd running on cluster3-master1 and save it on the master node at /tmp/etcd-backup.db.

> Then create a Pod of your kind in the cluster.

> Finally restore the backup, confirm the cluster is still working and that the created Pod is no longer with us.

<details><summary>Show Answer</summary>
<p>

### Etcd Backup
First we log into the master and try to create a snapshop of etcd:

``` bash
➜ ssh cluster3-master1

➜ root@cluster3-master1:~# ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db
Error:  rpc error: code = Unavailable desc = transport is closing
``` 

But it fails because we need to authenticate ourselves. For the necessary information we can check the etc manifest:

```  bash
➜ root@cluster3-master1:~# vim /etc/kubernetes/manifests/etcd.yaml
``` 

We only check the etcd.yaml for necessary information we don't change it.

```  yaml
# /etc/kubernetes/manifests/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.100.31:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt                           # use
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://192.168.100.31:2380
    - --initial-cluster=cluster3-master1=https://192.168.100.31:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key                            # use
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.100.31:2379   # use
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.168.100.31:2380
    - --name=cluster3-master1
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt                    # use
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: k8s.gcr.io/etcd:3.3.15-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: etcd
    resources: {}
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd                                                     # important
      type: DirectoryOrCreate
    name: etcd-data
status: {}
``` 

But we also know that the api-server is connecting to etcd, so we can check how its manifest is configured:

```  bash
➜ root@cluster3-master1:~# cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
``` 

We use the authentication information and pass it to etcdctl:

```  bash
➜ root@cluster3-master1:~# ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key

Snapshot saved at /tmp/etcd-backup.db
``` 

NOTE: Dont use snapshot status because it can alter the snapshot file and render it invalid

### Etcd restore
Now create a Pod in the cluster and wait for it to be running:

``` bash
root@cluster3-master1:~# kubectl run test --image=nginx
pod/test created

root@cluster3-master1:~# kubectl get pod -l run=test -w
NAME   READY   STATUS    RESTARTS   AGE
test   1/1     Running   0          60s
``` 

NOTE: If you didn't solve questions 18 or 20 and cluster3 doesn't have a ready worker node then the created pod might stay in a Pending state. This is still ok for this task.

Next we stop all controlplane components:

``` bash
root@cluster3-master1:~# cd /etc/kubernetes/manifests/

root@cluster3-master1:/etc/kubernetes/manifests# mv * ..

root@cluster3-master1:/etc/kubernetes/manifests# watch crictl ps
``` 

Now we restore the snapshot into a specific directory:

``` bash
➜ root@cluster3-master1:~# ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
--data-dir /var/lib/etcd-backup \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key

2020-09-04 16:50:19.650804 I | mvcc: restore compact to 9935
2020-09-04 16:50:19.659095 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32
We could specify another host to make the backup from by using etcdctl --endpoints http://IP, but here we just use the default value which is: http://127.0.0.1:2379,http://127.0.0.1:4001.
``` 

The restored files are located at the new folder /var/lib/etcd-backup, now we have to tell etcd to use that directory:

``` bash
➜ root@cluster3-master1:~# vim /etc/kubernetes/etcd.yaml
# /etc/kubernetes/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
...
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-backup                # change
      type: DirectoryOrCreate
    name: etcd-data
status: {}
``` 

Now we move all controlplane yaml again into the manifest directory. Give it some time (up to several minutes) for etcd to restart and for the api-server to be reachable again:

``` bash
root@cluster3-master1:/etc/kubernetes/manifests# mv ../*.yaml .

root@cluster3-master1:/etc/kubernetes/manifests# watch crictl ps
Then we check again for the Pod:

➜ root@cluster3-master1:~# kubectl get pod -l run=test
No resources found in default namespace.
``` 
</p>
</details>


## 💬 Direct Candidate Feedback: 2025 CKA Exam Insights (Reddit Source) 

> “The exam has shifted. It’s less about copying manifests and more about troubleshooting, Helm, CRDs, and Gateway APIs.”

> “I left 3 questions blank — Helm packaging, Gateway API, and cluster setup using `kubeadm`. It was tougher than expected.”

> “Gateway API and HTTPRoute threw me off — the documentation wasn’t clear. Helm was tricky too, but overall manageable.”

> “Topics I saw: HPA, Ingress, Helm, ArgoCD, cert-manager CRDs, container runtimes, and setting up NetPol with least privileges.”

> “The new format made me think more deeply. I had to actually solve problems — not just recognize YAML.”

> “Low-resolution VM made it hard to read docs. And some questions were significantly more complex than the mock exams.”

> “Expect lag, tab loading delays, and broken copy-paste. Know your `kubectl`, `vim`, and command-line tools well.”

> “Mouse movement was sluggish — keyboard navigation with `:vsplit` and `kubectl explain` helped me save time.”


## Key Topic Coverage Summary (2025)

| Topic Area                  | Common Feedback Summary                                                                 |
|----------------------------|------------------------------------------------------------------------------------------|
| **Helm & Packaging**       | Required for install and templating questions; Helm + dpkg appeared                     |
| **Gateway API / HTTPRoute**| Often noted as tricky; vague docs made questions harder                                 |
| **CRI / containerd**       | Manual installs and configuration came up in several questions                          |
| **CRDs & Admission Webhooks** | Edge-case topics that still showed up — be familiar                                     |
| **HPA, ArgoCD, NetPol**    | Popular topics; applied in realistic scenarios like scaling and access control          |
| **PVC, Volumes, Ingress**  | Included in questions requiring debug/fix/validate workflows                           |
| **Cluster Setup (kubeadm)**| Appeared unexpectedly; time-consuming if unprepared                                     |
| **PSI Exam UI**            | Reported lag, poor copy/paste support, and low-res experience                           |
| **Keyboard Proficiency**   | Critical due to interface slowness — `vim`, tabs, and shortcuts helped significantly    |


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

