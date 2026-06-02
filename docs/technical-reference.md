# Kubernetes Technical Reference

Quick reference for concepts learned.

---

## Containers (Kernel Features)

### Namespaces (Isolation)

| Namespace | Isolates |
|-----------|----------|
| **PID** | Process IDs (container sees PID 1 for init) |
| **Network** | Network stack (IP, ports, routing) |
| **Mount** | Filesystem (own /, bind mounts) |
| **User** | UIDs/GIDs (container UID 0 → host UID 165536+) |
| **IPC** | Shared memory, semaphores, message queues |
| **UTS** | Hostname and domain name |

### Cgroups (Resource Limits)

| Resource | Location | Example |
|----------|----------|---------|
| **Memory** | `/sys/fs/cgroup/memory/docker/abc123/memory.limit_in_bytes` | `1073741824` (1GB) |
| **CPU** | `/sys/fs/cgroup/cpu/docker/abc123/cpu.shares` | `1024` (relative weight) |
| **PIDs** | `/sys/fs/cgroup/pids/docker/abc123/pids.max` | `256` (max processes) |

### Union Filesystems

```
Layer 1 (base image): read-only, shared
Layer 2 (dockerfile): read-only, shared
Layer 3 (app code): read-only, shared
└── Container R/W layer: copy-on-write, unique per container
```

**Benefit**: 100 containers from same image = 1× shared base + 100× small R/W layers

### Seccomp

Blocks syscalls:
- `mount()` → no mounting filesystems
- `reboot()` → no rebooting host
- `kexec_load()` → no kernel loading
- `acct()` → no process accounting

---

## Kubernetes Architecture

### Control Plane (Master) Components

#### API Server
- **Port**: 6443 (HTTPS)
- **Function**: 
  - Accepts REST requests (kubectl apply, etc)
  - Validates YAML schemas
  - Authorizes via RBAC
  - Persists to etcd
  - Notifies watchers (watch streams)

#### etcd
- **Type**: Distributed KV store
- **Key path pattern**: `/registry/kind/namespace/name`
- **Replication**: Raft consensus (quorum)
- **Durability**: All writes replicated before confirmation
- **Failure**: If etcd down, cluster cannot accept new requests (existing pods continue running)

#### Scheduler
- **Algorithm**: Two-phase
  1. **Filtering**: eliminate nodes that don't fit
  2. **Scoring**: rank remaining nodes, pick best
- **Considers**:
  - Resource requests (not limits)
  - Node affinity (soft/hard)
  - Pod anti-affinity (e.g., spread replicas)
  - Taints/Tolerations (hard constraint)
  - Pod priority
  - Custom plugins

#### Controller Manager
- **Contains**: Multiple loop controllers
- **Common**:
  - **Deployment Controller**: ensures replica count, handles rolling updates
  - **ReplicaSet Controller**: ensures pod count (created by Deployment)
  - **Node Controller**: monitors node health, drains pods if NotReady
  - **Endpoints Controller**: maps pod IPs to service selectors
  - **StatefulSet Controller**: ordered pod creation, stable identity

### Worker Node Components

#### kubelet
- **Role**: Agent on each node
- **Responsibilities**:
  - Watch API Server for pods assigned to this node
  - Instruct container runtime (Docker/containerd) to create containers
  - Monitor pod health (liveness, readiness probes)
  - Report pod status back to API Server
  - Enforce resource limits via cgroups
  - Mount volumes

#### kube-proxy
- **Role**: Networking on each node
- **Responsibilities**:
  - Implement Service abstraction
  - Load balance traffic to pod IPs
  - Two modes: iptables (default) or IPVS
  - Watch API Server for Service/Endpoint changes
  - Update routing rules when endpoints change

#### Container Runtime
- **Options**: Docker, containerd, CRI-O, etc
- **Interface**: CRI (Container Runtime Interface)
- **Function**: Actually creates/runs/stops containers

---

## Kubernetes Objects (Key Concepts)

### API Resources & Versions

```
APIVersion patterns:
- v1 (core): Pod, Service, ConfigMap, Secret, Namespace
- apps/v1: Deployment, StatefulSet, DaemonSet, ReplicaSet
- batch/v1: Job, CronJob
- networking.k8s.io/v1: Ingress, NetworkPolicy
- storage.k8s.io/v1: StorageClass, PersistentVolume
```

### Scheduling Constraints

#### NodeAffinity (Pod Preference)
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringScheduling:
        nodeSelectorTerms:
        - matchExpressions:
          - key: gpu
            operator: In
            values: ["true"]
```
**Effect**: Pod MUST be on node with label gpu=true

#### PodAffinity/AntiAffinity (Pod-to-Pod Preference)
```yaml
spec:
  affinity:
    podAntiAffinity:
      required:
        podAffinityTerms:
        - labelSelector:
            matchLabels:
              app: myapp
          topologyKey: kubernetes.io/hostname
```
**Effect**: Pod CANNOT be on same node as another pod with label app=myapp

#### Taints (Node Repulsion)
```bash
# Node tainted
kubectl taint nodes worker-gpu gpu=required:NoSchedule

# effect: NoSchedule = don't schedule here
# effect: PreferNoSchedule = prefer not to
# effect: NoExecute = evict existing pods (unless tolerate)
```

#### Tolerations (Pod Acceptance)
```yaml
spec:
  tolerations:
  - key: gpu
    operator: Equal
    value: required
    effect: NoSchedule
```
**Effect**: Pod tolerates (can run on) nodes with taint gpu=required

**Key Difference**:
- **Affinity**: "Pod wants to be here" (soft/hard preference)
- **Taint/Toleration**: "Node rejects unless tolerated" (hard rejection)

---

## Reconciliation Loop Pattern

Every controller follows:

```
while (true) {
  desired_state = read_from_etcd()
  actual_state = observe_reality()
  
  if desired_state != actual_state {
    take_action_to_reconcile()
  }
  
  sleep(resync_period)  // 15s default
}
```

**Examples**:
- Deployment desired=3, actual=2 → create 1 pod
- Node status=NotReady → drain pods (cordon)
- Service endpoints changed → update load balancer rules

---

## YAML Manifest Structure

```yaml
apiVersion: apps/v1              # Kind of API, version
kind: Deployment                 # Resource type
metadata:
  name: myapp                    # Name in namespace
  namespace: default             # Namespace (default if omitted)
  labels:
    app: myapp                   # Labels for selection
spec:
  replicas: 3                    # Desired state
  selector:
    matchLabels:
      app: myapp                 # Select pods to manage
  template:
    metadata:
      labels:
        app: myapp               # Pod labels must match selector
    spec:
      containers:
      - name: myapp
        image: myapp:1.2.3
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: 512Mi        # Scheduler uses this
            cpu: 250m
          limits:
            memory: 1024Mi       # kubelet enforces this
            cpu: 500m
```

---

## Common Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide

# Resources
kubectl get pods
kubectl get pods -o yaml
kubectl get pods --all-namespaces
kubectl describe pod myapp-1

# Create/Update
kubectl apply -f deployment.yaml
kubectl create -f pod.yaml

# Watch
kubectl get pods --watch
kubectl get pods -w

# Debug
kubectl logs pod-name
kubectl logs pod-name -c container-name
kubectl exec -it pod-name -- /bin/bash
kubectl port-forward pod-name 8080:8080

# Delete
kubectl delete pod pod-name
kubectl delete -f deployment.yaml

# Scale
kubectl scale deployment myapp --replicas=5

# Update
kubectl set image deployment/myapp myapp=myapp:1.2.4
kubectl rollout status deployment/myapp
kubectl rollout undo deployment/myapp
```

---

## API Server Request Flow

```
1. kubectl apply -f deployment.yaml
   ↓
2. POST https://master:6443/apis/apps/v1/namespaces/default/deployments
   ↓
3. API Server: validate schema
   ↓
4. API Server: check RBAC authorization
   ↓
5. API Server: write to etcd (/registry/deployments/default/myapp)
   ↓
6. API Server: notify watchers (Controller Manager, Scheduler, kubectl)
   ↓
7. kubectl: "201 Created"
   ↓
8. Deployment Controller observes: "new deployment"
   ↓
9. Deployment Controller: create ReplicaSet
   ↓
10. ReplicaSet Controller: create 3 Pods
   ↓
11. Scheduler: assign pods to nodes
   ↓
12. kubelet: create containers on assigned nodes
   ↓
13. Pod status → Running
```

---

## Useful Mental Models

### Kubernetes as a Computer
```
Master (CPU/Brain)
├── API Server = memory bus
├── etcd = RAM (state storage)
├── Scheduler = task dispatcher
└── Controllers = background processes

Workers (CPU cores)
├── kubelet = executor
└── Containers = actual work
```

### Desired State Machine
```
Your desired state (YAML) → etcd → Controllers observe → Actions → Reality converges
```

### Watchers as Event Streams
```
API Server maintains open connections to all controllers
Controller: "tell me everything about deployments"
API Server: ADDED, MODIFIED, DELETED events stream
Controller: reconcile based on events
```

---

## Next Topics to Learn

### Immediate (Control Plane Completion)
1. Controller Manager deep dive
2. kubelet pod lifecycle
3. kube-proxy networking

### Short Term (Kubernetes Objects)
1. Pods (multi-container, init containers, sidecar)
2. Services (ClusterIP, NodePort, LoadBalancer)
3. Deployments (rolling updates, strategy)

### Medium Term (Configuration)
1. ConfigMaps (decouple config)
2. Secrets (sensitive data)
3. Environment variables

### Later (Advanced)
1. Storage (PV, PVC, StatefulSet)
2. Networking (Ingress, NetworkPolicy)
3. Observability (Prometheus, logging)
