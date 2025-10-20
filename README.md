# 🧩 Kubernetes Architecture and Core Concepts

## 📘 Overview

Kubernetes (K8s) is an **open-source container orchestration platform** that automates the deployment, scaling, and management of containerized applications.  
It abstracts the underlying infrastructure and provides a consistent environment to run applications across clusters.

---
## ⚙️ 1. Kubernetes Architecture

<img width="688" height="391" alt="Screenshot 2025-10-20 130227" src="https://github.com/user-attachments/assets/842eeef7-3956-41f7-bdfa-1a51512c004a" />


Kubernetes follows a **Master-Worker (Control Plane–Node)** architecture.

### **Control Plane Components**

These manage the cluster and make global decisions.

|Component|Description|
|---|---|
|**API Server**|Acts as the central management hub for Kubernetes. All commands (CLI or REST) go through the API server.|
|**etcd**|A distributed key-value store that holds all cluster state and configuration data.|
|**Controller Manager**|Runs controllers that regulate the desired state (e.g., ReplicaSet controller ensures pods match the desired count).|
|**Scheduler**|Assigns newly created pods to suitable nodes based on resource requirements and constraints.|
|**Cloud Controller Manager**|Integrates with cloud providers for managing load balancers, volumes, and networking.|

### **Node Components**

Nodes are worker machines that run containerized workloads.

|Component|Description|
|---|---|
|**Kubelet**|Agent running on each node that ensures pods are running as expected.|
|**Kube Proxy**|Manages network rules and handles pod-to-pod or pod-to-service communication.|
|**Container Runtime**|Software responsible for running containers (e.g., Docker, containerd, CRI-O).|

---

## 🧱 2. Kubernetes Objects

Kubernetes uses declarative objects to represent the desired state of workloads.

|Object|Description|Use Case|
|---|---|---|
|**Pod**|Smallest deployable unit in K8s; can contain one or more containers.|Running a single instance of an app.|
|**ReplicaSet**|Ensures a specific number of identical pods are running.|Scaling stateless apps.|
|**Deployment**|Manages ReplicaSets; provides version control and rolling updates.|Updating web services without downtime.|
|**StatefulSet**|Manages stateful apps with stable network IDs and storage.|Databases like MongoDB, MySQL.|
|**DaemonSet**|Ensures a copy of a pod runs on all (or some) nodes.|Monitoring agents, log collectors.|
|**Job**|Runs a pod until completion (batch jobs).|Data processing tasks, backup jobs.|

**Hierarchy:**  
Deployment → ReplicaSet → Pods

## 🧱 1. **Pod Definition**

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

**Description:** Runs a single NGINX container inside a Pod.

## 🔁 2. **ReplicaSet**

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
```

**Description:** Ensures three NGINX pods are always running.

## 🚀 3. **Deployment**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.25
          ports:
            - containerPort: 80
```

**Description:** Manages ReplicaSets and rolling updates for stateless apps.

## 💾 4. **StatefulSet**

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  serviceName: "mysql"
  replicas: 2
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "rootpassword"
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-storage
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

**Description:** Provides stable network IDs and persistent storage for MySQL pods.

<img width="666" height="379" alt="Screenshot 2025-10-20 140836" src="https://github.com/user-attachments/assets/d79cbb03-433a-411f-8afe-f1efe3d989bc" />



## ⚙️ 5. **DaemonSet**

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  labels:
    app: monitoring
spec:
  selector:
    matchLabels:
      app: monitoring
  template:
    metadata:
      labels:
        app: monitoring
    spec:
      containers:
        - name: node-exporter
          image: prom/node-exporter:latest
          ports:
            - containerPort: 9100
```

**Description:** Ensures a monitoring agent runs on every node.

## ⏳ 6. **Job**

```
apiVersion: batch/v1
kind: Job
metadata:
  name: data-backup-job
spec:
  template:
    metadata:
      name: data-backup-pod
    spec:
      containers:
        - name: backup
          image: busybox
          command: ["sh", "-c", "echo Backing up data && sleep 10"]
      restartPolicy: OnFailure
```

**Description:** Executes a one-time data backup task.

---
## 🌐 3. Networking Components

### **Service Types**

|Type|Description|Use Case|
|---|---|---|
|**ClusterIP**|Exposes the service within the cluster only.|Internal microservice communication.|
|**NodePort**|Exposes the service on a static port on each node’s IP.|Simple external access for testing.|
|**LoadBalancer**|Integrates with cloud providers to expose externally via a load balancer.|Production workloads.|

## 🌐 7. **Service (ClusterIP)**

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

**Description:** Exposes the NGINX app internally within the cluster.

## 🌍 8. **Service (NodePort)**

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

**Description:** Makes the NGINX app accessible externally via Node IP:30080.

## 🌍 8. **Service (LoadBalancer)**

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
  labels:
    app: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - name: http
      port: 80
      targetPort: 80
```

You’ll get an **EXTERNAL-IP** or **DNS name** from your cloud provider — access your app via that.

---

### **CNI Plugins (Container Network Interface)**

Plugins provide pod networking and routing between nodes:

- **Flannel** – Simple overlay network.
    
- **Calico** – Advanced networking with network policies.
    
- **Weave Net** – Automatic mesh networking.
    

### **Pod Communication Flow**

- **Pod-to-Pod (Same Node):** Direct via virtual Ethernet pair.
    
- **Pod-to-Pod (Different Nodes):** Routed through CNI overlay.
    
- **Pod-to-External:** Through NAT and kube-proxy rules.


---

## 💾 4. Storage Components

|Component|Description|
|---|---|
|**Persistent Volume (PV)**|A piece of storage provisioned by an admin or dynamically via StorageClass.|
|**Persistent Volume Claim (PVC)**|A request by a user for storage with specific size and access mode.|
|**StorageClass**|Defines storage types and dynamic provisioning behavior.|
|**ConfigMap**|Stores configuration data as key-value pairs for pods.|

**Data Persistence Flow:**  
Pods → PVC → PV → Physical Storage

## 🧩 10. **Persistent Volume

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

## 🧩 11. **Persistent Volume Claim

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

**Description:** Provides persistent storage for pods using hostPath.

## ⚙️ 11. **ConfigMap**

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"
```

**Description:** Stores configuration for containers decoupled from image.

<img width="605" height="352" alt="Screenshot 2025-10-20 140340" src="https://github.com/user-attachments/assets/b1f95817-1e5a-4553-8e12-4f3bc78139c7" />



---

## 🧭 5. Namespaces

Namespaces logically separate cluster resources.

|Benefit|Description|
|---|---|
|**Isolation**|Prevents resource conflicts between teams/environments.|
|**Resource Quotas**|Enables resource allocation per namespace.|
|**Access Control**|Integrates with RBAC for security.|

Examples:

- `dev`, `staging`, `prod` namespaces for environment isolation.


```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

**Description:** Creates a dedicated namespace for development workloads.

<img width="660" height="334" alt="Screenshot 2025-10-20 140553" src="https://github.com/user-attachments/assets/e54d4796-e30f-491a-a1e6-4e2539aa90b0" />


---

## 🚀 6. Deployments

Deployment automates application updates and management.

### **Deployment Strategies**

|Strategy|Description|
|---|---|
|**Recreate**|Terminates old pods before starting new ones.|
|**Rolling Update**|Gradually replaces old pods with new ones (default).|
|**Blue/Green**|Runs new version in parallel and switches traffic upon success.|
|**Canary**|Gradually routes a percentage of traffic to new version.|

**Advantages:**

- Easy rollback
    
- Zero downtime updates
    
- Declarative versioning
    
- Auto-scaling support

---

## 🌍 7. Services & Ingress

### **Services**

Expose applications to other pods or external users.

Example:
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```
### **Ingress Controller & Ingress Resource**

Ingress provides **HTTP/HTTPS routing** into the cluster.

- **Ingress Controller:** Manages routing rules (e.g., NGINX Ingress Controller).
    
- **Ingress Resource:** Defines rules for routing.
    

Example:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
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

---

## ❤️ 8. Health Probes

|Probe|Description|Example|
|---|---|---|
|**Liveness Probe**|Checks if the container is running.|Restart if unhealthy.|
|**Readiness Probe**|Checks if pod is ready to serve traffic.|Remove from load balancer if not ready.|
|**Startup Probe**|Used for slow-start applications.|Delay other probes until started.|

## 🩺 **1. Liveness Probe**

### 🔍 Purpose:

- Checks **if the container is still running properly**.
    
- If the probe fails, Kubernetes **restarts the container** automatically.
    
- Used to detect **application deadlocks or hangs**.
    

### ✅ Example — Liveness Probe (HTTP Check)

```
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
    - name: myapp
      image: nginx:latest
      ports:
        - containerPort: 80
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 3
```

### 🧩 Explanation:

|Field|Description|
|---|---|
|**httpGet**|Sends HTTP GET request to `/` on port 80.|
|**initialDelaySeconds**|Wait 10s before first check (app startup time).|
|**periodSeconds**|Run probe every 5s.|
|**failureThreshold**|After 3 consecutive failures, restart the container.|

## 🧠 **2. Readiness Probe**

### 🔍 Purpose:

- Checks **if the container is ready to serve traffic**.
    
- If it fails, **the pod is removed from Service endpoints**.
    
- Prevents sending traffic to unready pods (e.g., during startup or heavy load).
    

### ✅ Example — Readiness Probe (TCP Check)

```
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
    - name: myapp
      image: nginx:latest
      ports:
        - containerPort: 80
      readinessProbe:
        tcpSocket:
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
```

### 🧩 Explanation:

|Field|Description|
|---|---|
|**tcpSocket**|Tries to open a TCP connection on port 80.|
|**initialDelaySeconds**|Waits 5s before starting probe checks.|
|**periodSeconds**|Checks every 10s.|
|**Effect**|If fails, pod is marked “NotReady” → temporarily removed from Service load balancing.|

## 🚀 **3. Startup Probe**

### 🔍 Purpose:

- Designed for **applications with long startup times** (like Java or large services).
    
- Disables **liveness** and **readiness** checks until the startup probe succeeds.
    
- Prevents Kubernetes from killing the container too early.
    

### ✅ Example — Startup Probe (Command Check)

```
apiVersion: v1
kind: Pod
metadata:
  name: startup-pod
spec:
  containers:
    - name: slow-app
      image: busybox
      command: ["sh", "-c", "sleep 30 && echo App started && httpd -f -h /usr/share/nginx/html"]
      startupProbe:
        exec:
          command:
            - cat
            - /usr/share/nginx/html/index.html
        initialDelaySeconds: 5
        periodSeconds: 5
        failureThreshold: 10
      livenessProbe:
        httpGet:
          path: /
          port: 80
        periodSeconds: 5
      readinessProbe:
        httpGet:
          path: /
          port: 80
        periodSeconds: 10
```

### 🧩 Explanation:

|Field|Description|
|---|---|
|**startupProbe**|Runs first to ensure the app starts before other probes run.|
|**failureThreshold**|Allows up to 10 failed attempts (50s total) before restart.|
|**Once startupProbe succeeds**|Liveness and readiness probes begin checking.|
## 🧩 **Comparison Summary**

|Probe Type|Purpose|Failure Action|Common Use Case|
|---|---|---|---|
|**Liveness**|Checks if the app is alive|Container restart|Detecting hung apps|
|**Readiness**|Checks if app can serve traffic|Pod removed from service|Delayed app startup or DB connection needed|
|**Startup**|Waits until app fully starts|Prevents premature restart|Slow-boot apps (e.g., Java, ML models)|


---

## 🔄 9. Sidecar Containers

Sidecars are helper containers that run alongside main containers in a pod.

|Use Case|Description|
|---|---|
|**Logging**|Collect and ship logs to external system (e.g., Fluentd).|
|**Proxy**|Handle network traffic (e.g., Envoy for service mesh).|
|**Monitoring**|Collect metrics and forward to Prometheus.|

---

## 📊 10. Resource Quotas & Limits

|Concept|Description|
|---|---|
|**Requests**|Minimum resources guaranteed to a pod.|
|**Limits**|Maximum resources a pod can use.|
|**ResourceQuota**|Sets constraints per namespace.|

## ⚙️ **1. Resource Requests & Limits**

### 📘 Example — `resource-limits.yml`

```
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo-pod
  labels:
    app: demo
spec:
  containers:
    - name: demo-container
      image: nginx
      resources:
        requests:
          cpu: "250m"       # 0.25 of a CPU core
          memory: "128Mi"   # Minimum memory guaranteed
        limits:
          cpu: "500m"       # Max CPU (half a core)
          memory: "256Mi"   # Max Memory
```

### 🧩 Explanation

|Field|Description|
|---|---|
|**requests.cpu**|Minimum CPU Kubernetes guarantees to the container.|
|**requests.memory**|Minimum memory reserved for the container.|
|**limits.cpu**|Maximum CPU container can use. If exceeded → throttled.|
|**limits.memory**|Maximum memory container can use. If exceeded → container is killed (OOMKilled).|

## 🧭 **2. Resource Quota**

A **ResourceQuota** object defines the total allowed CPU, memory, and object count (pods, services, PVCs).  

### 📘 Example — `resource-quota.yml`

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev-team
spec:
  hard:
    pods: "10"                  # Max 10 pods in namespace
    requests.cpu: "2"           # Total minimum requested CPU ≤ 2 cores
    requests.memory: "4Gi"      # Total minimum requested memory ≤ 4GB
    limits.cpu: "4"             # Total CPU limit ≤ 4 cores
    limits.memory: "8Gi"        # Total memory limit ≤ 8GB
    persistentvolumeclaims: "5" # Max number of PVCs
```

### 🧩 Explanation

|Field|Description|
|---|---|
|**pods**|Restricts total number of pods.|
|**requests.cpu / memory**|Caps total resource requests in that namespace.|
|**limits.cpu / memory**|Caps total resource limits for all pods in that namespace.|
|**persistentvolumeclaims**|Limits number of PVCs.|

<img width="649" height="359" alt="Screenshot 2025-10-20 140635" src="https://github.com/user-attachments/assets/f6cb494b-314f-4e05-b56b-613365540af3" />


## 🧮 **3. LimitRange (Default Values per Namespace)**

A **LimitRange** sets default **requests and limits** for containers that don’t explicitly define them.  
This helps prevent “unbounded” pods in shared environments.

### 📘 Example — `limitrange.yml`

```
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limitrange
  namespace: dev-team
spec:
  limits:
    - default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "250m"
        memory: "256Mi"
      type: Container
```

### 🧩 Explanation

| Field               | Description                                        |
| ------------------- | -------------------------------------------------- |
| **default**         | Sets default _limit_ for containers without one.   |
| **defaultRequest**  | Sets default _request_ for containers without one. |
| **type: Container** | Applies limits to container-level resources.       |

---

## 🧠 11. Advanced Scheduling Concepts

|Concept|Description|Example Use|
|---|---|---|
|**Node Selector**|Simple label-based selection.|Schedule pod on a specific node.|
|**Node Affinity**|Advanced rules to match node labels.|Deploy web pods on SSD nodes.|
|**Pod Affinity / Anti-Affinity**|Schedule pods together or apart.|Spread replicas across zones.|
|**Taints and Tolerations**|Control which pods can be scheduled on specific nodes.|Reserve nodes for specific workloads.|

## 🎯 **1. Node Selector**

### 🧩 Overview:

- Simplest way to schedule a Pod on a specific node (or group of nodes).
    
- Uses **key-value labels** on nodes.
    
### 📘 Example — `nodeselector.yml`

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-node-selector
spec:
  nodeSelector:
    disktype: ssd
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

## 🧭 **2. Node Affinity**

### 🧩 Overview:

- Advanced version of `nodeSelector`.
    
- Supports **logical operators** and **soft/hard rules**:
    
    - `requiredDuringSchedulingIgnoredDuringExecution` → **Hard rule** (must match).
        
    - `preferredDuringSchedulingIgnoredDuringExecution` → **Soft rule** (best-effort).
        
### 📘 Example — `nodeaffinity.yml`

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
              - key: region
                operator: In
                values:
                  - us-east
  containers:
    - name: nginx
      image: nginx
```

### 🧩 Explanation:

|Type|Description|
|---|---|
|**requiredDuringSchedulingIgnoredDuringExecution**|Pod **must** run only on nodes matching label `disktype=ssd`.|
|**preferredDuringSchedulingIgnoredDuringExecution**|Pod **prefers** nodes labeled `region=us-east`, but not mandatory.|

## 🤝 **3. Pod Affinity / Anti-Affinity**

### 🧩 Overview:

- Controls pod placement **based on other pods’ labels**.
    
- **Pod Affinity:** Schedule pods _together_ (same zone/node).
    
- **Pod Anti-Affinity:** Schedule pods _apart_ (different zones/nodes).
### 📘 Example — `podaffinity.yml`

```
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - backend
          topologyKey: "kubernetes.io/hostname"
  containers:
    - name: frontend
      image: nginx
```

✅ **Meaning:**  
This pod (frontend) will be scheduled **on the same node** as a pod labeled `app=backend`.

### 📘 Example — `podantiaffinity.yml`

```
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - backend
          topologyKey: "kubernetes.io/hostname"
  containers:
    - name: backend
      image: nginx
```

✅ **Meaning:**  
This ensures **no two `backend` pods** run on the same node.

## 🚫 **4. Taints and Tolerations**

### 🧩 Overview:

- **Taints** applied to nodes: repel pods unless they have a matching **toleration**.
    
- Used to **dedicate nodes** for special workloads (e.g., GPU, critical apps).
### 📘 Example — `taint-toleration.yml`

```
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
  containers:
    - name: gpu-app
      image: nvidia/cuda:12.0-base
      command: ["nvidia-smi"]
```

✅ **Result:**

- The `gpu-pod` will be **scheduled only on GPU nodes**.
    
- Pods without matching toleration → **will not run** on that node.

---
## ✅ Summary

| Category         | Key Takeaway                                                  |
| ---------------- | ------------------------------------------------------------- |
| **Architecture** | Control Plane manages the cluster; Nodes run workloads.       |
| **Objects**      | Pods, Deployments, and ReplicaSets define workloads.          |
| **Networking**   | Services and Ingress manage communication.                    |
| **Storage**      | PVs and PVCs handle persistence.                              |
| **Namespaces**   | Isolate and manage multi-team environments.                   |
| **Scheduling**   | Affinity, Taints, and Node Selectors fine-tune pod placement. |

---
