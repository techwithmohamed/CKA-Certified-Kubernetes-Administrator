[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

# ☸️ Certified Kubernetes Administrator (CKA) Exam Guide - V1.32 (2025)

<p align="center">
  <img src="assets/cka.png" alt="CKA EXAM">
</p>

> This guide is part of our blog [Pass the CKA Certification Exam: The Complete Study Guide ](https://techwithmohamed.com/cka-exam-study-guide/).

## Hit the Star! :star:
> If you are using this repo for guidance, please hit the star. Thanks A lot !

>  The [Certified Kubernetes Administrator (CKA) certification](https://www.cncf.io/certification/cka/) exam certifies that candidates have the skills, knowledge, and competency to perform the responsibilities of Kubernetes administrators.
 
## CKA Exam details (v1.32  2025 ) 

| **CKA Exam Details**                     | **Information**                                                                                     |
|-------------------------------------------|-----------------------------------------------------------------------------------------------------|
| **Exam Type**                             | Performance Based NOT MCQ )                                    |
| **Exam Duration**                         | 2 hours                                                                                            |
| **Pass Percentage**                       | 66%  ( One Retake )                                                                                                |
| **CKA Exam Kubernetes Version**          | [Kubernetes v1.32](#cka-exam-syllabus-updated-kubernetes-132-2025)                                                                               |
| **CKA Validity**                         | 2 Years  |
| **Exam Cost**                            | $445 USD   |


## Table of Contents

- [CKA Exam Syllabus (Updated Kubernetes 1.32)](#cka-exam-syllabus-updated-kubernetes-132)
- [CKA Exam Questions And Answers](#cka-exam-questions-and-answers)
- [Additional Resources](#additional-resources)
- [Practice](#practice)

## CKA Exam Syllabus (Updated Kubernetes 1.32)

## CKA Exam Syllabus (Updated Kubernetes 1.32 - 2025)

## CKA Exam Syllabus (Updated Kubernetes 1.32 - 2025)

| **Topic** | **Concepts** | **Weightage** |
|-----------|--------------|---------------|
| [**1. Cluster Architecture, Installation & Configuration - 25%**](#1-cluster-architecture-installation--configuration-25) | 1. Manage role-based access control (RBAC)<br>2. Prepare underlying infrastructure for installing a Kubernetes cluster<br>3. Create and manage Kubernetes clusters using kubeadm<br>4. Manage the lifecycle of Kubernetes clusters<br>5. Implement and configure a highly-available control plane<br>6. Use Helm and Kustomize to install cluster components<br>7. Understand extension interfaces (CNI, CSI, CRI, etc.)<br>8. Understand CRDs, install and configure operators | 25% |
| [**2. Workloads & Scheduling - 15%**](#2-workloads--scheduling-15) | 1. Understand application deployments and how to perform rolling update and rollbacks<br>2. Use ConfigMaps and Secrets to configure applications<br>3. Configure workload autoscaling<br>4. Understand the primitives used to create robust, self-healing, application deployments<br>5. Configure Pod admission and scheduling (limits, node affinity, etc.) | 15% |
| [**3. Services & Networking - 20%**](#3-services--networking-20) | 1. Understand connectivity between Pods<br>2. Define and enforce Network Policies<br>3. Use ClusterIP, NodePort, LoadBalancer service types and endpoints<br>4. Use the Gateway API to manage Ingress traffic<br>5. Know how to use Ingress controllers and Ingress resources<br>6. Understand and use CoreDNS | 20% |
| [**4. Storage - 10%**](#4-storage-10) | 1. Implement storage classes and dynamic volume provisioning<br>2. Configure volume types, access modes and reclaim policies<br>3. Manage persistent volumes and persistent volume claims | 10% |
| [**5. Troubleshooting - 30%**](#5-troubleshooting-30) | 1. Troubleshoot clusters and nodes<br>2. Troubleshoot cluster components<br>3. Monitor cluster and application resource usage<br>4. Manage and evaluate container output streams<br>5. Troubleshoot services and networking | 30% |



## 1. Cluster Architecture, Installation & Configuration (25%)

This section tests your ability to build, configure, and manage Kubernetes clusters using `kubeadm` and core cluster components. It covers the full cluster lifecycle—from provisioning infrastructure, enforcing access, performing upgrades, to integrating components like Helm, CRDs, and networking plugins.

### ✅ Manage Role-Based Access Control (RBAC)

RBAC lets you define *who* can do *what* on *which* Kubernetes resources.

**Example:**

```bash
kubectl create role cm-writer --verb=create --resource=configmaps -n dev
kubectl create rolebinding writer-bind --role=cm-writer --serviceaccount=dev:app-sa -n dev
```

### ✅ Prepare Underlying Infrastructure for a Kubernetes Cluster

Prepare nodes with:

- Proper hostname resolution
- Disabled swap: `swapoff -a`
- Opened required ports (6443, 2379–2380, 10250)

**Example:**

```bash
hostnamectl set-hostname master-node
echo "10.0.0.2 worker-node" >> /etc/hosts
swapoff -a
```

### ✅ Create and Manage Kubernetes Clusters Using kubeadm

Bootstrap clusters using `kubeadm`.

**Example:**

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Join node:

```bash
kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

### ✅ Manage the Lifecycle of Kubernetes Clusters

Upgrade, reset, or reconfigure.

**Example:**

```bash
kubeadm upgrade plan
kubeadm upgrade apply v1.32.1
```

### ✅ Implement and Configure a Highly-Available Control Plane

Use multiple control-plane nodes with a shared load balancer.

**Example:**

```bash
kubeadm init --control-plane-endpoint "LOAD_BALANCER_DNS:6443" ...
```

### ✅ Use Helm and Kustomize to Install Cluster Components

Deploy apps using Helm or Kustomize.

**Helm:**

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install nginx bitnami/nginx
```

**Kustomize:**

```bash
kubectl apply -k ./my-kustomization/
```

### ✅ Understand Extension Interfaces (CNI, CSI, CRI)

Know how to validate and troubleshoot:

- **CNI**: Pod networking (e.g., Calico)
- **CSI**: Storage drivers
- **CRI**: Runtime (e.g., containerd)

**Example:**

```bash
crictl info
ls /etc/cni/net.d/
kubectl get csidrivers
```

### ✅ Understand CRDs, Install and Configure Operators

Understand how CRDs extend Kubernetes and how to manage Operators.

**Example:**

```bash
kubectl get crds
kubectl describe crd <customresource>
```

Install a Prometheus operator or similar using manifest or operatorhub.io.

---
# 2. Workloads & Scheduling (15%)

This section focuses on managing workloads and scheduling in Kubernetes, making up 15% of the CKA Exam. Below are the key topics explained with `kubectl` examples:

### Understand Deployments and How to Perform Rolling Updates and Rollbacks
> Deployments manage the lifecycle of applications. Rolling updates allow you to update applications without downtime, while rollbacks revert to a previous version if needed.

#### Example:
> **Create a deployment:**
```bash
kubectl create deployment nginx --image=nginx:1.21
```

> **Perform a rolling update:**
```bash
kubectl set image deployment/nginx nginx=nginx:1.22
```

> **Rollback to a previous version:**
```bash
kubectl rollout undo deployment/nginx
```

> - [Learn more about deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

### Use ConfigMaps and Secrets to Configure Applications
> ConfigMaps and Secrets store configuration data and sensitive information, allowing you to decouple configuration from application code.

#### Example:
> **Create a ConfigMap:**
```bash
kubectl create configmap app-config --from-literal=key1=value1
```

> **Create a Secret:**
```bash
kubectl create secret generic app-secret --from-literal=password=mysecret
```

> **Use them in a Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: myapp:latest
    env:
    - name: CONFIG_KEY
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: key1
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
```
```bash
kubectl apply -f pod.yaml
```

> - [Learn more about ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)

> - [Learn more about Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

### Know How to Scale Applications
> Scaling ensures your application can handle varying levels of traffic by adjusting the number of replicas.

#### Example:
> **Scale a deployment:**
```bash
kubectl scale deployment nginx --replicas=5
```

> **Autoscale based on CPU usage:**
```bash
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=50
```

> - [Learn more about scaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

### Understand the Primitives Used to Create Robust, Self-Healing, Application Deployments
> Kubernetes primitives like Deployments, ReplicaSets, and Probes ensure applications are highly available and self-healing.

#### Example:
> **Add readiness and liveness probes:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: myapp:latest
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
```
```bash
kubectl apply -f deployment.yaml
```

> - [Learn more about probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

### Understand How Resource Limits Can Affect Pod Scheduling
> Resource requests and limits ensure fair allocation of cluster resources and influence pod scheduling.

#### Example:
> **Set resource requests and limits:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: demo-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
```bash
kubectl apply -f resource-demo.yaml
```

> - [Learn more about resource management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

### Awareness of Manifest Management and Common Templating Tools
> Manifest management tools like Helm and Kustomize simplify deploying and managing Kubernetes applications.

#### Example:
> **Using Kustomize:**
```bash
kubectl apply -k ./my-kustomization/
```

> **Using Helm:**
```bash
helm install my-app ./my-chart
```

> - [Learn more about Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)

> - [Learn more about Helm](https://helm.sh/docs/)

---

### Resources to Prepare
> - [Kubernetes Documentation](https://kubernetes.io/docs/)

> - [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

> - [Kubernetes Workloads](https://kubernetes.io/docs/concepts/workloads/)

## 3. Services & Networking (20%)

This section focuses on Kubernetes services and networking concepts, which make up 20% of the CKA Exam. Below are the key topics explained with `kubectl` examples:

### 1. Understand Host Networking Configuration on the Cluster Nodes
> Host networking involves understanding how network interfaces, IPs, and routes are configured on cluster nodes to ensure communication.

#### Key Concepts:
> - Check node network interfaces and routes using tools like `ip` or `ifconfig`.

> - Ensure proper setup of firewalls and ports for Kubernetes components.

> **Example:**
```bash
# List network interfaces on a node
kubectl get nodes -o wide
ssh <node-name> "ip a"
```

> - [Learn more about cluster networking](https://kubernetes.io/docs/concepts/architecture/nodes/)

### 2. Understand Connectivity Between Pods
> Pods communicate with each other via the cluster network. All Pods are assigned a unique IP and can communicate without NAT.

#### Example:
> **Test Pod-to-Pod connectivity:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: nginx
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```
```bash
kubectl apply -f pods.yaml
kubectl exec pod2 -- ping pod1
```

> - [Learn more about Pod networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

### 3. Understand ClusterIP, NodePort, LoadBalancer Service Types and Endpoints
> Services expose Kubernetes applications for internal and external access. Common service types include:

> - **ClusterIP:** Default; exposes the service inside the cluster.

> - **NodePort:** Exposes the service on a static port on each node.

> - **LoadBalancer:** Uses cloud provider load balancers to expose services externally.

#### Example:
> **Create a ClusterIP service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```
```bash
kubectl apply -f service.yaml
```

> - [Learn more about services](https://kubernetes.io/docs/concepts/services-networking/service/)

### 4. Know How to Use Ingress Controllers and Ingress Resources
> Ingress provides HTTP and HTTPS routing to applications inside the cluster using hostnames and paths.

#### Example:
> **Deploy an Ingress resource:**
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
            name: clusterip-service
            port:
              number: 80
```
```bash
kubectl apply -f ingress.yaml
```

> - [Learn more about Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

### 5. Know How to Configure and Use CoreDNS
> CoreDNS is a DNS server that provides name resolution within the Kubernetes cluster.

#### Example:
> **Check CoreDNS ConfigMap:**
```bash
kubectl -n kube-system get configmap coredns -o yaml
```
> **Test DNS resolution:**
```bash
kubectl exec -it <pod-name> -- nslookup kubernetes.default
```

> - [Learn more about CoreDNS](https://kubernetes.io/docs/tasks/administer-cluster/coredns/)

### 6. Choose an Appropriate Container Network Interface (CNI) Plugin
> CNIs enable networking in Kubernetes by providing Pod-to-Pod connectivity.

#### Popular CNI Plugins:
> - **Flannel:** Simple and easy-to-configure.

> - **Calico:** Offers advanced networking and network policy capabilities.

> - **Weave Net:** Supports encrypted communication.

#### Example:
> **Install Calico as a CNI:**
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

> - [Learn more about CNIs](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)

---

### Resources to Prepare
> - [Kubernetes Documentation](https://kubernetes.io/docs/)

> - [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

> - [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)


## 4. Storage (10%)

This section focuses on Kubernetes storage concepts, which make up 10% of the CKA Exam. Below are the key topics explained with `kubectl` examples:

### 1. Understand Storage Classes and Persistent Volumes
> Storage Classes define the types of storage available, while Persistent Volumes (PVs) represent storage in the cluster.

#### Example:
> **Create a Storage Class:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
```
```bash
kubectl apply -f storageclass.yaml
```

> **Create a Persistent Volume:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-storage
  hostPath:
    path: /mnt/data
```
```bash
kubectl apply -f pv.yaml
```

> - [Learn more about Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)

> - [Learn more about Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

### 2. Understand Volume Mode, Access Modes, and Reclaim Policies for Volumes
> Volumes can have different modes and access settings to fit application needs. Reclaim policies determine what happens to a volume after it is released.

#### Key Modes and Policies:
> - **Volume Modes:** Filesystem, Block.

> - **Access Modes:** ReadWriteOnce, ReadOnlyMany, ReadWriteMany.

> - **Reclaim Policies:** Retain, Delete, Recycle.

**Example:**
> Check PV details:
```bash
kubectl describe pv pv-demo
```

> - [Learn more about Volume Modes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#volume-modes)

> - [Learn more about Access Modes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)

### 3. Understand Persistent Volume Claims (PVC) Primitive
> PVCs allow Pods to request specific storage resources from PVs.

#### Example:
> **Create a Persistent Volume Claim:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: fast-storage
```
```bash
kubectl apply -f pvc.yaml
```

> **Use a PVC in a Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pvc
spec:
  containers:
  - name: app-container
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-demo
```
```bash
kubectl apply -f pod.yaml
```

> - [Learn more about PVCs](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

### 4. Know How to Configure Applications with Persistent Storage
> Applications often need persistent storage to retain data. Configure storage by mounting volumes to application Pods.

#### Example:
> **Deployment with Persistent Storage:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-with-storage
spec:
  replicas: 2
  selector:
    matchLabels:
      app: storage-app
  template:
    metadata:
      labels:
        app: storage-app
    spec:
      containers:
      - name: app-container
        image: nginx
        volumeMounts:
        - name: app-storage
          mountPath: /usr/share/nginx/html
      volumes:
      - name: app-storage
        persistentVolumeClaim:
          claimName: pvc-demo
```
```bash
kubectl apply -f deployment.yaml
```

> - [Learn more about configuring applications with storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

---

### Resources to Prepare
> - [Kubernetes Documentation](https://kubernetes.io/docs/)

> - [Persistent Volumes Guide](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

> - [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)


## 5. Troubleshooting (30%)

This section focuses on troubleshooting skills, which constitute 30% of the CKA Exam. Below are the key topics explained with `kubectl` examples:

### 1. Evaluate Cluster and Node Logging
> Logs from the cluster and nodes help identify issues with Kubernetes components and nodes.

#### Example:
> **View cluster events:**
```bash
kubectl get events --sort-by='.metadata.creationTimestamp'
```

> **Check kubelet logs on a node:**
```bash
ssh <node-name>
sudo journalctl -u kubelet
```

> - [Learn more about Kubernetes logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/)

### 2. Understand How to Monitor Applications
> Application monitoring involves tracking application performance and resource usage using tools like `kubectl top` or external monitoring solutions.

#### Example:
> **Check resource usage of Pods:**
```bash
kubectl top pod
```

> **Check resource usage of nodes:**
```bash
kubectl top node
```

> - [Learn more about monitoring](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-usage-monitoring/)

### 3. Manage Container stdout & stderr Logs
> Logs from containerized applications help diagnose issues in application behavior.

#### Example:
> **View logs from a Pod:**
```bash
kubectl logs <pod-name>
```

> **View logs from a specific container in a Pod:**
```bash
kubectl logs <pod-name> -c <container-name>
```

> - [Learn more about container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/#logging-at-the-node-level)

### 4. Troubleshoot Application Failure
> Application failures often arise from misconfigurations, missing dependencies, or resource constraints.

#### Example:
> **Describe a failing Pod to identify issues:**
```bash
kubectl describe pod <pod-name>
```

> **Debug a failing Pod:**
```bash
kubectl exec -it <pod-name> -- /bin/sh
```

> - [Learn more about troubleshooting applications](https://kubernetes.io/docs/tasks/debug/debug-application/)

### 5. Troubleshoot Cluster Component Failure
> Cluster component failures can disrupt the entire cluster. Key components include the API server, scheduler, controller manager, and etcd.

#### Example:
> **Check the status of cluster components:**
```bash
kubectl get componentstatuses
```

> **Check the logs of a specific component:**
```bash
kubectl logs -n kube-system <component-pod-name>
```

> - [Learn more about troubleshooting cluster components](https://kubernetes.io/docs/tasks/debug/debug-cluster/)

### 6. Troubleshoot Networking
> Networking issues can affect Pod communication, service access, or external connectivity.

#### Example:
> **Test Pod connectivity:**
```bash
kubectl exec -it <pod-name> -- ping <target-pod-ip>
```

> **Debug Service issues:**
```bash
kubectl describe service <service-name>
```

> **Test DNS resolution:**
```bash
kubectl exec -it <pod-name> -- nslookup <service-name>
```

> - [Learn more about troubleshooting networking](https://kubernetes.io/docs/tasks/debug/debug-cluster/dns-debugging-resolution/)

---

### Resources to Prepare
> - [Kubernetes Documentation](https://kubernetes.io/docs/)

> - [Troubleshooting Guide](https://kubernetes.io/docs/tasks/debug/)

> - [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)



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

 
## Additional Resources
* 💬 [Kubernetes Slack Channel #certifications](https://kubernetes.slack.com/)<sup>Slack</sup>
* 📚 [Guide to Certified Kubernetes Administrator (CKA)](https://techwithmohamed.com/cka-exam-study-guide/)<sup>Blog</sup>
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

