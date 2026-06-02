# Learning Progress Tracker

**Start Date**: 2026-06-01  
**Target Completion**: 2026-07-27 (8 weeks)  
**Current Week**: 1/8

---

## Module 1: Foundations (WEEK 1) ✅

### Completed Topics

#### 1.1 What is Kubernetes?
- **Status**: ✅ Understood
- **Depth**: Deep technical
- **Key Learnings**:
  - K8s is a control plane that maintains desired state via reconciliation loops
  - Master (Control Plane) vs Worker (Node) distinction
  - Declarative vs Imperative: K8s uses declarative model
  - API Server as central gateway
  - etcd as source of truth
  - Scheduler assigns pods to nodes
  - Controller loops continuously reconcile state

#### 1.2 Containers Deep Dive
- **Status**: ✅ Understood (technical level)
- **Depth**: Very deep (kernel internals)
- **Key Learnings**:
  - **Namespaces** (PID, network, mount, user, IPC, UTS)
    - Isolate visibility of resources
    - Each container sees its own filesystem, network stack, processes
    - User namespace maps container UID 0 → host UID 165536+
  - **Cgroups** (CPU, memory, I/O, PIDs)
    - Limit resource consumption
    - Container hitting memory.limit_in_bytes = OOM killer
    - CPU shares relative weight, not hard limits
  - **Union Filesystems** (AUFS, overlay2)
    - Layers share base image (read-only)
    - Copy-on-write for each container (R/W layer)
    - 100 containers from same image = 1× base image + 100× small R/W layers
  - **Seccomp/AppArmor**
    - Restrict syscalls (mount, reboot, kexec_load blocked by default)
    - Container cannot mount filesystem or reboot host

- **Hands-on**:
  - Created simple container isolation example with `unshare`
  - Understood `/sys/fs/cgroup` structure
  - Mapped memory.limit_in_bytes to JVM heap management

#### 1.3 Container vs VM
- **Status**: ✅ Understood
- **Key Learnings**:
  - **VM**: abstracts hardware, each VM has own kernel (1-2GB, 30s boot)
  - **Container**: isolates process, shares kernel (100-500MB, <1s boot)
  - **Density**: VMs support 10-20 per host, containers support 100-1000s
  - **Isolation**: VMs strong (separate OS), containers moderate (shared kernel)
  - **Use VM when**: different OS needed, legacy workloads, compliance isolation
  - **Use container when**: modern apps, high density, fast scaling, multi-cloud

#### 1.4 Kubernetes Architecture Overview
- **Status**: ✅ Understood
- **Key Learnings**:
  - **Control Plane (Master)**: makes decisions
    - API Server: gateway, validates, stores in etcd
    - etcd: distributed KV store, source of truth
    - Scheduler: assigns pods to nodes
    - Controller Manager: runs reconciliation loops
  - **Worker Nodes**: execute decisions
    - kubelet: agent on node, talks to API Server and container runtime
    - kube-proxy: networking, load balancing between pods
    - Container runtime: Docker/containerd, actually runs containers

#### 1.5 Component Deep Dive (Partial)
- **Status**: 🟡 Partially understood
- **Completed**:
  - ✅ API Server (detailed)
  - ✅ etcd (detailed)
  - ✅ Scheduler (detailed, including filtering + scoring algorithm)
  - ✅ Taints & Tolerations (clarified with analogies)

- **Remaining**:
  - ⏳ Controller Manager (Deployment, ReplicaSet, Node, etc controllers)
  - ⏳ kubelet (pod lifecycle, health checks, container management)
  - ⏳ kube-proxy (service networking, iptables/IPVS)

---

## Module 2: Control Plane Components (WEEK 2) 🟡 IN PROGRESS

### Topics to Cover

#### 2.1 Controller Manager
- **Status**: ⏳ Pending
- **To Learn**:
  - What is a controller (infinite loop observing state)
  - Deployment controller (ensures replicas)
  - ReplicaSet controller (ensures pod count)
  - Node controller (monitors node health)
  - Endpoints controller (pod → service mapping)
  - How controllers watch etcd and take action

#### 2.2 kubelet
- **Status**: ⏳ Pending
- **To Learn**:
  - Node agent architecture
  - Pod lifecycle (pending → running → terminating → terminal)
  - Health checks (liveness, readiness, startup)
  - Container runtime interface (CRI)
  - How kubelet gets pod spec from API Server
  - Volume mounting at pod level
  - Resource limits enforcement via cgroups

#### 2.3 kube-proxy
- **Status**: ⏳ Pending
- **To Learn**:
  - Service abstraction (ClusterIP, NodePort, LoadBalancer)
  - How kube-proxy implements load balancing
  - iptables vs IPVS (two modes)
  - Endpoint slice distribution
  - Service discovery via DNS

### Comprehension Check (After completion)
- [ ] Explain how a Deployment controller ensures 3 replicas
- [ ] What happens if kubelet crashes?
- [ ] How does kube-proxy route traffic to pod IPs?

---

## Module 3: Kubernetes Objects (WEEK 3-4) 🔲 PENDING

### Topics to Cover
- [ ] **Pods**: atomic unit, multi-container, sidecar patterns
- [ ] **Services**: ClusterIP, NodePort, LoadBalancer, DNS
- [ ] **Deployments**: replicas, rolling updates, rollback
- [ ] **StatefulSets**: ordered, stable identity, headless service
- [ ] **DaemonSets**: one pod per node (logging, monitoring)
- [ ] **Jobs & CronJobs**: batch workloads, scheduled jobs

### Checkpoint Test
- [ ] Create pod manifest with 2 containers
- [ ] Create deployment with 3 replicas + service
- [ ] Understand difference between Deployment and StatefulSet

---

## Module 4: Configuration & Secrets (WEEK 4) 🔲 PENDING

### Topics to Cover
- [ ] **ConfigMaps**: decouple configuration from image
- [ ] **Secrets**: store sensitive data (encoded, not encrypted by default)
- [ ] **Environment variables**: pass config to pods
- [ ] **Volume mounts**: config as files

### Project Integration
- Create ConfigMap for app configuration (database URL, etc)
- Create Secret for credentials
- Update Deployment to use both

---

## Module 5: Storage (WEEK 5) 🔲 PENDING

### Topics to Cover
- [ ] **Persistent Volumes (PV)**: cluster-level storage resource
- [ ] **Persistent Volume Claims (PVC)**: pod request for storage
- [ ] **Storage Classes**: dynamic provisioning
- [ ] **StatefulSets with storage**: ordered, persistent identity

### Project Integration
- PostgreSQL StatefulSet with persistent volume
- Redis with ephemeral storage

---

## Module 6: Networking (WEEK 5-6) 🔲 PENDING

### Topics to Cover
- [ ] **Ingress**: external access, routing
- [ ] **NetworkPolicies**: micro-segmentation
- [ ] **Service mesh basics**: Istio/Linkerd intro

### Project Integration
- Ingress to expose frontend
- NetworkPolicy to restrict backend access

---

## Module 7: Observability (WEEK 6-7) 🔲 PENDING

### Topics to Cover
- [ ] **Metrics**: Prometheus integration, resource metrics
- [ ] **Logging**: log aggregation, ELK stack intro
- [ ] **Health checks**: liveness, readiness, startup probes
- [ ] **Tracing**: distributed tracing concepts

### Project Integration
- Prometheus scrape config for app metrics
- Grafana dashboard for monitoring
- Liveness/readiness probes on all pods

---

## Module 8: Advanced (WEEK 7-8) 🔲 PENDING

### Topics to Cover
- [ ] **Custom Resources (CRDs)**: extend K8s API
- [ ] **Operators**: automate complex deployments
- [ ] **Helm**: package management, templating
- [ ] **Multi-cluster patterns**: federation basics

---

## Hands-On Project Progress

### Phase 1: Local Setup (WEEK 2)
- [ ] Install Minikube
- [ ] Setup kubectl
- [ ] Deploy first pod to verify setup

### Phase 2: Backend (WEEK 3-4)
- [ ] Create Spring Boot app (simple REST API)
- [ ] Docker image build
- [ ] Deployment + Service manifest
- [ ] Test locally on Minikube

### Phase 3: Frontend (WEEK 4)
- [ ] nginx Deployment
- [ ] Service expose
- [ ] Ingress route
- [ ] Test in browser

### Phase 4: Database (WEEK 5)
- [ ] PostgreSQL StatefulSet
- [ ] PVC with persistence
- [ ] Backend → PostgreSQL networking
- [ ] Verify data persistence

### Phase 5: Observability (WEEK 6-7)
- [ ] Prometheus + Grafana deployment
- [ ] Metrics scraping
- [ ] Dashboard creation
- [ ] Alert rules

### Phase 6: Integration & Polish (WEEK 7-8)
- [ ] All components integrated
- [ ] Health checks on all pods
- [ ] Rolling updates tested
- [ ] Cleanup scripts
- [ ] Documentation complete

---

## Knowledge Gaps / Clarifications Needed

### Resolved
- ✅ Difference between Taints and Tolerations (repulsion vs preference)
- ✅ Why containers need multiple namespaces
- ✅ How reconciliation loops work

### Outstanding
- ⏳ How exactly Controller Manager works with multiple controllers
- ⏳ kubelet pod lifecycle state machine
- ⏳ kube-proxy iptables rules generation
- ⏳ How to test locally without cloud infrastructure

---

## Study Sessions Log

### Session 1 (2026-06-01)
- **Duration**: 1h
- **Topics**: What is K8s, containers overview, VM vs containers
- **Key Insight**: Containers are isolated processes via kernel features, not mini-VMs
- **Next**: Control plane architecture

### Session 2 (2026-06-01)
- **Duration**: 1.5h
- **Topics**: K8s architecture, API Server, etcd, Scheduler, Taints/Tolerations
- **Key Insight**: Scheduler is multi-phase (filtering + scoring), taints are node repulsion
- **Clarification**: Taint/Toleration is different from NodeAffinity (repulsion vs preference)
- **Next**: Controller Manager, kubelet, kube-proxy deep dive

---

## Testing Strategy

### Unit Comprehension Tests
After each component:
- [ ] 2-3 conceptual questions
- [ ] 1 hands-on scenario
- [ ] Feedback if gaps exist

### Integration Tests
After each module:
- [ ] Create manifest from scratch
- [ ] Deploy to local cluster
- [ ] Verify expected behavior
- [ ] Modify and test changes

### Project Tests
After project phase:
- [ ] All components running
- [ ] Services communicating
- [ ] Persistence verified
- [ ] Monitoring showing metrics

---

## Resources Used

- Course: Kubernetes for Developers (Mumshad Mannambeth, Udemy)
- Book: Kubernetes Up and Running (Hightower)
- Docs: kubernetes.io/docs
- Playground: killercoda.com/kubernetes (when needed)
- Official repos: kubernetes/kubernetes, etcd-io/etcd, kubernetes-sigs

---

## Notes

- Focus on **why** each component exists, not just **how** it works
- Always connect back to the project
- Test comprehension before moving forward
- Build mental models, not just memorize
