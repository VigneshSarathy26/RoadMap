# ☸️ Complete Kubernetes Engineering Roadmap (v2.0 — Production-Grade)

A structured, industrial roadmap to mastering Kubernetes—from core architecture to production-grade platform engineering, GitOps, and emerging trends.

---

## 📌 Roadmap Overview

This roadmap is designed for DevOps and Platform Engineers aiming to build scalable, resilient, and secure internal developer platforms (IDP).

| Focus | Key Outcomes |
| :--- | :--- |
| **Core Architecture** | Mastery of Control Plane, Worker Nodes, and Objects. |
| **Workloads & Networking** | Advanced scheduling, Services, and Gateway API. |
| **Ecosystem & Ops** | Helm, GitOps (ArgoCD), and eBPF Observability. |
| **Hardening & Scale** | RBAC, Cluster API, and Multi-cluster mesh. |

---

## 🔰 Phase 1: Foundations (Core Concepts)

### 1. Container & Orchestration Basics
- **Containers vs Virtual Machines**: Understanding isolation, resource utilization, and lifecycle.
- **Docker Fundamentals**: Images, containers, volumes, networks, and Dockerfiles.
- **Why Orchestration is Needed**: Solving problems like scheduling, healing, and scaling.
- **Kubernetes Architecture Overview**: The relationship between master and worker nodes.
- **Container Runtime Security**: Hardening containerd/CRI-O, using seccomp profiles, and applying AppArmor/SELinux policies.
- **Immutable Container Filesystems**: Leveraging `readOnlyRootFilesystem` and ephemeral storage patterns.

### 2. Kubernetes Architecture

| Component | Responsibility |
| :--- | :--- |
| **API Server** | The gateway for all REST commands used to control the cluster. |
| **Scheduler** | Assigns Pods to Nodes based on resource availability. |
| **Controller Manager** | Runs control loops to maintain the desired state of the cluster. |
| **etcd** | Distributed key-value store for all cluster data. |
| **Kubelet** | Agent that runs on each node in the cluster. |
| **Kube Proxy** | Maintains network rules on nodes. |
| **Container Runtime** | Software responsible for running containers (containerd, CRI-O). |

#### **kubectl Advanced Patterns**
- `kubectl debug`: Using ephemeral containers for live troubleshooting.
- `kubectl apply --server-side`: Handling field ownership conflicts gracefully.
- `kubectl port-forward`: Understanding security risks and implementing production alternatives.

### 3. Core Kubernetes Objects
- **Pods**: Lifecycle management and multi-container patterns (sidecar, init, ambassador).
- **ReplicaSets**: Ensuring a specific number of pod replicas are always running.
- **Deployments**: Managing rolling updates, rollbacks, and scaling strategies.
- **Namespaces**: Logical resource isolation and quota management within a cluster.
- **Labels & Selectors**: Organizing, categorizing, and selecting groups of objects.

---

### 📂 Repository Blueprint: Kubernetes Platform Engineering

> [!NOTE]
> This repository structure signals to recruiters: *"This person understands how to manage infrastructure-as-code and platform configurations at scale."*

```text
k8s-platform-engineering/
├── clusters/               # Multi-cluster definitions
│   ├── dev/
│   └── prod/
├── base/                   # Shared base manifests (Kustomize)
│   ├── networking/
│   └── storage/
├── charts/                 # Internal Helm charts
│   └── custom-app/
├── terraform/              # Infrastructure provisioning (EKS/GKE)
├── scripts/                # Helper scripts (backup, upgrade)
└── .github/workflows/      # CI/CD pipelines
```

| Signal | Industry Equivalent |
| :--- | :--- |
| `clusters/` | Multi-environment mindset. |
| `base/` | DRY (Don't Repeat Yourself) manifests. |
| `terraform/` | Provisioning vs. Configuration separation. |

---

## ⚙️ Phase 2: Workloads & Scheduling

### 4. Workload Types
- **Deployments**: For stateless applications and web services.
- **StatefulSets**: For stateful applications (databases, message queues) requiring stable network IDs.
- **DaemonSets**: Running background pods on every node (logs, monitoring, CNI).
- **Jobs & CronJobs**: For batch processing and scheduled execution tasks.

### 5. Scheduling
- **NodeSelector**: Simple node selection based on labels.
- **Affinity / Anti-affinity**: Complex scheduling rules for both Pods and Nodes.
- **Taints & Tolerations**: Repelling pods from specific nodes unless they tolerate the taint.
- **Resource Requests & Limits**: Controlling CPU and Memory usage guarantees and maximums.
- **Topology Spread Constraints**: Distributing pods evenly across zones/nodes for High Availability.
  - *maxSkew tuning patterns*: `maxSkew: 1` for strict balance, `maxSkew: 2` for relaxed zone distribution.
  - *whenUnsatisfiable*: `DoNotSchedule` vs `ScheduleAnyway` for critical vs best-effort workloads.
- **Node Resource Fragmentation**: Avoiding CPU/memory fragmentation with TopologyManager and CPUManager policies.
  - *TopologyManager policies*: `none`, `best-effort`, `restricted`, `single-numa-node`.
  - *CPUManager policies*: `none` (default) vs `static` (for guaranteed QoS pods with integer CPU requests).
- **Resource Quotas & LimitRanges**: Preventing noisy-neighbor problems in multi-tenant clusters.
- **PriorityClasses & Preemption**: Ensuring critical workloads get scheduled first.

> [!IMPORTANT]
> Set `preemptionPolicy: Never` for critical control plane pods to prevent cascading evictions during cluster resource constraints.

### 6. Health & Lifecycle
- **Probes**: Implementing `Liveness`, `Readiness`, and `Startup` probes effectively.
- **Lifecycle Hooks**: Using `postStart` and `preStop` for initialization and clean-up.
- **Graceful Shutdown Patterns**: Ensuring zero-downtime deployments.
- **Pod Disruption Budgets (PDBs)**: Controlling how many pods can be disrupted during node maintenance or cluster upgrades.
- **Eviction Policies**: Soft vs hard eviction thresholds, `evictionPressureTransitionPeriod`.
  - *Soft eviction*: `memory.available<500Mi` with `eviction-soft-grace-period=30s`.
  - *Hard eviction*: `memory.available<100Mi` for immediate termination.

---

## 🌐 Phase 3: Networking

### 7. Services & Ingress
- **Service Types**: `ClusterIP` (internal), `NodePort` (external exposure), `LoadBalancer` (cloud integration).
- **Headless Services**: Bypassing proxy for direct discovery in stateful applications (`clusterIP: None`).
- **Ingress Resources**: Managing HTTP/HTTPS routing rules for external access.
- **Gateway API**: The modern evolution of Ingress (`HTTPRoute`, `TCPRoute`, `TLSRoute`).

### 8. DNS & Service Discovery
- **CoreDNS Configuration**: Customizing resolution, stub domains, and forwarding.
- **Debugging Tools**: Using `nslookup` and `dig` inside ephemeral containers.
- **ExternalDNS**: Automating DNS record management in external providers (Route53, Cloudflare).
- **Node Local DNS Cache**: Reducing CoreDNS latency and cluster-wide DNS failure blast radius.
  - Runs as a DaemonSet (`node-local-dns`) on each node with `hostNetwork: true`.
  - Fallback to cluster CoreDNS on cache miss.

### 9. CNI Deep Dive & Network Security

#### **CNI Comparison: When to choose what**

| CNI | Best For | Key Features |
| :--- | :--- | :--- |
| **Cilium** | eBPF, observability, service mesh | Hubble, WireGuard, L7 policies, ambient mesh |
| **Calico** | Enterprise policy, BGP | Network policies, BGP peering, Windows support |
| **Flannel** | Simple overlay, small clusters | VXLAN, easy setup, limited policy |
| **Antrea** | VMware ecosystems, NSX integration | OVS-based, Windows support |

- **Dual Stack Networking**: Supporting IPv4 + IPv6 clusters and dual-stack allocation.
  - `IPv4AndIPv6` service IP family policy.
  - `--cluster-cidr` with comma-separated IPv4/IPv6 ranges.
- **WireGuard / IPsec Encryption**: Enabling Pod-to-Node and Pod-to-Pod encryption without a full service mesh.
  - Cilium WireGuard integration: `encryption.type=wireguard` with node-to-node key management.
- **Network Policies**: Enforcing default-deny ingress/egress, namespace isolation, and pod-to-pod rules.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

- **Egress Control**: Restricting outbound traffic to prevent data exfiltration.
- **Service Mesh**:
  - **Istio Ambient Mode**: ztunnel + waypoint proxies, operating without sidecars.
  - **Cilium Service Mesh**: eBPF-based, kernel-level L7 policies.
  - **Linkerd**: Lightweight, low-overhead Rust-based alternative.
  - **mTLS**: Zero-trust pod-to-pod encryption standard.

| Feature | Istio Ambient | Cilium Mesh | Linkerd |
| :--- | :--- | :--- | :--- |
| **Architecture** | ztunnel + waypoint | eBPF in kernel | Sidecar proxy |
| **P99 Latency** | 3-5ms | 0.5-1ms | 1-2ms |
| **Memory/Proxy** | 50-100MB | 10-15MB | 20-30MB |
| **L7 Features** | Full Envoy | Growing | Moderate |

---

## 💾 Phase 4: Storage

### 10. Persistence Basics
- **Volumes**: Understanding pod-level ephemeral and persistent storage.
- **Persistent Volumes (PV) & Claims (PVC)**: Decoupling storage provisioning from consumption.
- **Storage Classes**: Abstracting different backends for dynamic provisioning (e.g., `gp3`, `io2`).

### 11. CSI & Snapshots
- **CSI (Container Storage Interface)**: Standardizing out-of-tree storage interactions.
- **Volume Snapshots & Cloning**: Creating point-in-time copies for backup and cloning data.

### 12. Storage Security & Advanced Patterns
- **Volume Expansion Policies**: `AllowVolumeExpansion` and `ReclaimPolicy` considerations for StatefulSets.

> [!WARNING]
> Use `ReclaimPolicy: Retain` instead of `Delete` for critical database PVCs to prevent accidental data loss when PVCs are deleted.

- **Raw Block Volumes**: Useful when databases need raw block access rather than filesystem PVs.
  - `volumeMode: Block` for Oracle, Ceph RBD direct access.
- **Local PV vs HostPath**: `LocalPersistentVolume` with node affinity vs `HostPath` security implications.
  - *Local PV*: Bound to specific node, supports scheduling constraints.
  - *HostPath*: High security risk (host filesystem access), requires `DirectoryOrCreate` validation.
- **Volume Encryption**: Ensuring at-rest and in-transit encryption using cloud KMS.
- **Backup & Disaster Recovery**: Leveraging Velero for cluster-wide backups and `etcd` snapshots.
- **Ephemeral Volumes Security**: Using `emptyDir` with `medium: Memory` for sensitive temporary data.

---

## 🔐 Phase 5: Security

### 13. RBAC & Service Accounts
- **RBAC**: Scoping permissions via `Roles`, `ClusterRoles`, `RoleBindings`, and `ClusterRoleBindings`.
- **Service Accounts**: Providing distinct identities for individual workloads.

### 14. Policy & Secret Management
- **Pod Security Standards (PSS)**: Implementing `Privileged`, `Baseline`, and `Restricted` profiles.
- **Admission Controllers**: Utilizing Mutating and Validating webhooks for governance.
- **Secret Management**: Adopting `External Secrets Operator` or `Sealed Secrets` for GitOps-friendly secrets.

### 15. Workload Identity & Registry Security
- **Pod Identity / Workload Identity**: Using OIDC for pods to replace long-lived IAM keys.
  - **AWS**: EKS Pod Identity (successor to IRSA).
  - **Azure**: Azure Workload Identity.
  - **GCP**: Workload Identity Federation.
- **ImagePullSecrets & Private Registries**: Seamless integration with ECR, GAR, ACR using IRSA/Workload Identity.
  - Leveraging `imagePullSecrets` with short-lived tokens from OIDC providers.

### 16. Admission Controller Breakdown
- **MutatingAdmissionWebhook**: Modifies requests before persistence (e.g., Istio sidecar injection).
- **ValidatingAdmissionWebhook**: Rejects non-compliant requests (e.g., OPA Gatekeeper, Kyverno).
- **Built-in Controllers**: `NamespaceLifecycle`, `LimitRanger`, `ServiceAccount`, `ResourceQuota`, `PodSecurity`.

### 17. Runtime Security & Hardening
- **Security Contexts**: Enforcing `runAsNonRoot`, `readOnlyRootFilesystem`, `allowPrivilegeEscalation: false`, and dropping ALL capabilities.

> [!CAUTION]
> Avoid `procMount: Unmasked` in production. While required for some specialized debugging tools, it bypasses default proc filesystem restrictions and presents a severe security risk.

- **Security Profiles Operator**: Managing seccomp, SELinux, and AppArmor profiles as CRDs.
  - Install via OLM, define profiles as Kubernetes resources.
- **eBPF Runtime Security**:
  - **Tetragon**: Kernel-level threat detection and process killing.

```yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: detect-privilege-escalation
spec:
  kprobes:
    - call: "__x64_sys_setuid"
      message: "Privilege escalation detected"
      selectors:
        - matchNamespaces:
            - namespace: production
```

  - **Falco**: Runtime threat detection with custom behavioral rules.
- **Image Security**:
  - **Image Scanning**: Integrating Trivy or Grype for CVE detection in pipelines.
  - **Image Signing**: Using Cosign and Notation for supply chain verification.
  - **Admission Control**: Enforcing signed images at deployment time.
- **cert-manager**: Automated TLS certificate provisioning and renewal.
- **Secrets Encryption at Rest**: Encrypting `etcd` backend with external KMS providers.

---

## 📦 Phase 6: Configuration & Packaging

### 18. Helm
- **Charts & Templating**: Managing application definitions and `values.yaml` injections.
- **Helm Hooks and Subcharts**: Handling pre/post install execution logic.
- **Helm Dependency Management**: Utilizing `Chart.lock`, `helm dependency update`, and subchart version pinning.

```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: "12.1.0"
    repository: "https://charts.bitnami.com/bitnami"
```

- **Helm Rollback Safety**: Using `--history-max`, restoring previous releases, and inspecting via `helm get values --revision`.
- **Helm Chart Testing**: Writing `helm test` hooks for CI validation.
- **OCI Artifacts**: Storing and distributing charts natively as OCI artifacts in container registries.

### 19. Kustomize
- **Base & Overlays**: Maintaining environment-specific configs (dev, staging, prod).
- **Kustomize Components**: Encapsulating reusable configuration snippets.
- **ConfigMapGenerator with behavior: merge**: Avoiding config drift across overlays.

```yaml
configMapGenerator:
  - name: app-config
    behavior: merge
    literals:
      - ENV=production
```

- **Strategic Merge Patch vs JSON Patch**: When to use each.
  - *Strategic Merge*: Kubernetes-native, handles lists intelligently (default).
  - *JSON Patch*: RFC 6902 compliant, precise but verbose, ideal for complex transformations.
- **ConfigMap/Secret Patterns**: Implementing immutable ConfigMaps, projected volumes, and external stores (Vault, AWS Parameter Store).

---

## 🚀 Phase 7: GitOps & CI/CD

### 20. The GitOps Standard
- **Argo CD & Flux**: Continuous Delivery executing via constant Git state reconciliation.
- **Progressive Delivery**: Implementing Canary deployments (Flagger, Argo Rollouts) and Blue/Green transitions.

### 21. GitOps Progressive Delivery — Advanced Patterns
- **Analysis & Rollback Automation**: Argo Rollouts utilizing `AnalysisTemplates`.
  - Prometheus metrics-based promotion and SLO-based automatic rollback.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  metrics:
    - name: success-rate
      interval: 5m
      successCondition: result[0] >= 0.95
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            sum(rate(http_requests_total{service="my-app"}[5m]))
```

- **Pull Request Environments**: Generating dynamic preview environments via GitOps.
  - Utilizing Argo CD Autopilot and Weaveworks PR Env for auto-creating namespaces per PR.
- **Multi-Cluster GitOps**: Operating a single repository managing dev/staging/prod across 3 cloud providers (AWS, Azure, GCP).
- **GitOps Notification & Approval**: Integrating Argo CD Notifications for Slack approvals before prod promotion.

### 22. GitOps Security Hardening
- **Repo-to-Cluster Trust**: Branch protection rules + GPG signing commits + Kyverno policies rejecting unsigned manifests.
- **Image Promotion Strategies**: Enforcing digest-based promotion across environments (no mutating tags).
- **Cosign Integration**: Signing and verifying images in CI/CD pipelines before deployment to the cluster.
- **Policy as Code**: Deploying OPA Gatekeeper or Kyverno for organizational governance.

| Policy Engine | Best For | Language | Key Strength |
| :--- | :--- | :--- | :--- |
| **Kyverno** | Kubernetes-native, YAML teams | YAML | Mutation + generation, low learning curve |
| **OPA/Gatekeeper** | Multi-platform, complex logic | Rego | Cross-platform reuse, extreme flexibility |

---

## 📊 Phase 8: Observability

### 23. Metrics, Logs & Tracing
- **Prometheus & Grafana Stack**: The standard for scraping and visualizing metrics.
- **Logging Stacks**: EFK (Elasticsearch, Fluentd, Kibana) or Loki stack.
- **eBPF-powered Observability**: Gaining deep kernel insights using Cilium Hubble and Pixie.

### 24. Application Performance Monitoring (APM) in K8s
- **eBPF-based Auto-Instrumentation**: Pixie, Odigos, zero-code APM for Python/Java/Go.
  - No code changes required, providing automatic service graph generation.
- **Pod Logging Anti-Patterns**: Evaluating sidecar log container vs JSON logging vs stdout vs emptyDir volume for high-throughput logs.
- **Log Sampling & Throttling**: Preventing log overload and disk saturation.
  - Fluentd throttling, Loki ingestion limits, and rate-limited sidecars.

### 25. SLOs & Cost Observability
- **SLI/SLO/SLA Definition**: Formulating error budgets and actionable service-level objectives.
- **Alerting & On-Runbooks**: Utilizing Alertmanager routing, silencing, and mapped incident response paths.
- **Distributed Tracing**: Implementing OpenTelemetry, Jaeger, and Tempo for cross-service request tracking.
- **Cost Observability**: Deploying Kubecost or OpenCost for accurate resource attribution and right-sizing.

### 26. Chaos Engineering as Observability Validation
- **Chaos Mesh / Litmus**: Injecting pod failures, network latency, and IO delays to validate alerting accuracy and SLO resilience.
- **GameDays Automation**: Conducting weekly Chaos experiments in staging with automated rollback evaluation.
  - Scheduling Litmus workflows via Cron, and auto-rollback on SLO breach.

---

## 🧠 Phase 9: Advanced Kubernetes

### 27. Autoscaling
- **KEDA**: Event-driven autoscaling based on external metrics (Kafka, SQS, Prometheus).
- **Vertical Pod Autoscaler (VPA)**: Automatically adjusting pod CPU/memory requests based on historical usage.
- **Karpenter**: Fast, just-in-time node provisioning directly bypassing the auto-scaling group.

> [!TIP]
> **Karpenter Best Practices**:
> - **Consolidation**: Use `WhenEmptyOrUnderutilized` for 30-60% compute cost reduction.
> - **Spot Instances**: Achieve 60-90% savings for fault-tolerant, stateless workloads.
> - **Multi-Architecture**: Blend ARM (Graviton) and x86 support gracefully.

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      requirements:
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64", "arm64"]
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot", "on-demand"]
  disruption:
    consolidationPolicy: WhenEmptyOrUnderutilized
    consolidateAfter: 30s
```

### 28. Cluster Management
- **Cluster API**: Declarative cluster lifecycle management (Provisioning Kubernetes with Kubernetes).
- **vCluster**: Spinning up lightweight, fully functional virtual clusters within host clusters for developer isolation.

### 29. Multitenancy Models
- **Soft Multitenancy**: Employing Namespaces + NetworkPolicies + ResourceQuotas to isolate teams.
- **Hard Multitenancy**: Utilizing vCluster or Hierarchical Namespaces (HNC) for dedicated virtual control planes.
- **Tenant Isolation Economic Trade-offs**:
  - *Shared*: Lower cost, higher blast radius.
  - *Dedicated (vCluster)*: Higher cost per tenant, complete operational isolation.

### 30. Internal Developer Platform (IDP) Engineering
- **Backstage / Cortex / Humanitec**: Building developer self-service UI on top of complex Kubernetes foundations.
- **Golden Paths & Scaffolding**: Providing standardized Helm charts + `kubectl create` skeleton templates for product teams.
- **Platform as a Product**: Treating the K8s platform team as an internal product team measured by SLIs.
  - Tracking metrics like time-to-platform-ready, deployment frequency, and platform API failure rate.

### 31. Cluster Upgrades & OS Lifecycle
- **Node OS Immutability**: Embracing Flatcar, Talos Linux, Bottlerocket for API-driven, declarative node upgrades.
  - *Talos Linux*: Features no SSH, no package manager, and a dual disk image scheme with built-in rollback.
- **Cluster Upgrade Strategies**:
  - *In-place*: kubeadm upgrades (risky, requires workload draining/downtime).
  - *Blue/Green cluster*: Create a new cluster, drain workloads gradually (leveraging Cilium Cluster Mesh for seamless migration).
  - *Managed upgrades*: Executing EKS/GKE version upgrades with custom exhaustiveness checks.
- **etcd Backup & DR Drills**: Automating `etcd` snapshots to S3 and executing restore testing quarterly.

---

## 💰 Phase 10: FinOps & Cost Engineering

### 32. Resource Efficiency Metrics
- **CPU/Memory Utilization Percentiles**: Utilizing P50, P95, P99 for right-sizing instead of averages.
- **Rightsizing Automation**: Deploying Goldilocks, VPA Recommender, or KRR (Kubecost Resource Recommender) for baseline tuning.

### 33. Spot Instance Strategy
- **Spot Interruption Handling**: Configuring NodePool disruption budgets and reliable fallbacks to on-demand compute.
- **Spot Termination Tolerance**: Designing Pods to tolerate spot interruptions gracefully using `topologySpreadConstraints` and `node.kubernetes.io/unschedulable` tolerations.

### 34. Idle Workload Detection
- **Kube-downscaler**: Automating dev environment 9-to-5 cron shutdowns.
- **Cost Allocation**: Integrating Kubecost to visualize cost per namespace/team/label via FinOps dashboards.

### 35. Cluster Autoscaler vs Karpenter Cost Comparison
- **Provisioner Consolidation Cost Models**: Contrasting Karpenter's dynamic bin-packing versus Cluster Autoscaler's rigid node group expansion.
- **Real-world Savings**: Expecting 30-60% compute reduction directly from Karpenter's continuous consolidation.

---

## 🚨 Phase 11: Disaster Recovery & Business Continuity

### 36. RTO/RPO Definitions for K8s
- **RTO (Recovery Time Objective)** / **RPO (Recovery Point Objective)** per application tier:
  - **Tier 1 (Payments)**: RTO < 5min, RPO < 1min
  - **Tier 3 (Analytics)**: RTO < 4hrs, RPO < 24hrs

### 37. Cross-Region Failover
- **Cluster Mesh Active-Passive**: Utilizing Cilium Cluster Mesh with automated global traffic failover.
- **Velero Backups in Separate Region**: Implementing S3 cross-region replication to guarantee backup integrity.

### 38. Stateful Application Recovery
- **Restoring from Volume Snapshots**: Executing Postgres/Cassandra operator recovery patterns.
- **etcd Consistency**: Ensuring control plane quorum is achievable after regional failure.

### 39. Config Drift Detection
- **kubectl diff**: Comparing between backup and live state.
- **Preventing Silent State Divergence**: Enforcing GitOps as the sole source of truth and enabling drift alerts via Argo CD.

---

## 🔮 Phase 12: Emerging & Experimental

### 40. WebAssembly (WASI) on K8s
- **SpinKube**: CNCF sandbox project for executing serverless Wasm directly on Kubernetes.
  - Uses `SpinApp` CRD, `containerd-shim-spin`, and a specialized runtime class manager.
  - Achieves 50μs cold start, 2MB memory footprint, and strict capability-based security.
- **runwasi**: Containerd shim wrapper for WASM runtimes (Wasmtime, Wasmer, WasmEdge).
- **Coexistence Strategy**: Deploying Wasm for API gateways/request transformation, while retaining containers for databases and heavy logic.

### 41. Kube-native AI/ML
- **Kubeflow**: Orchestrating complex ML pipelines natively on Kubernetes.
- **Ray on K8s**: Scaling distributed computing for robust model training and inference.
- **Kueue**: Managing job queueing for batch, HPC, and AI/ML workloads.
  - Handles quota management, fair resource sharing, and priority-based scheduling.
  - Integrates seamlessly with standard kube-scheduler and cluster-autoscaler.

### 42. K8s without Kubelet
- **K3s**: Optimized, lightweight Kubernetes tailored for resource-constrained edge computing.
- **Talos Linux**: Redefining the worker node as an immutable OS with an API-driven lifecycle.

### 43. Policy Engines Evolution
- **Kyverno 2.0**: Hardening the Kubernetes-native, YAML-based mutation and generation standard.
- **OPA/Gatekeeper**: Utilizing general-purpose Rego for advanced cross-platform governance.
- **jsPolicy**: Introducing JavaScript-based policy engines to bridge the gap for web developers.

---

## 🎓 Certifications

| Certification | Level | Focus |
| :--- | :--- | :--- |
| **KCNA** | Associate | Cloud-native fundamentals. |
| **CKAD** | Specialist | Application design and deployment. |
| **CKA** | Professional | Cluster architecture and operations. |
| **CKS** | Expert | Kubernetes security hardening. |
| **KCSA** | Associate | Kubernetes and Cloud Native Security fundamentals. |

---

## 📋 Quick Reference Summary

| Category | Key Tools & Topics |
| :--- | :--- |
| **Core** | kubectl, YAML, Container Runtimes, seccomp, AppArmor |
| **Scheduling** | Affinity, PDBs, Quotas, Taints, Topology Spread, PriorityClasses, CPUManager |
| **Networking** | Gateway API, CNI (Cilium), Network Policies, Service Mesh, WireGuard, Dual Stack |
| **Security** | RBAC, Falco, Tetragon, Trivy, cert-manager, PSS, Cosign, Kyverno, Workload Identity |
| **Storage** | CSI, Storage Classes, Snapshots, Velero, Encryption, Local PV, Raw Block |
| **GitOps** | ArgoCD, Flux, Flagger, Cosign, OPA, Image Promotion, PR Environments |
| **Advanced** | Cluster API, KEDA, vCluster, Karpenter, Cluster Mesh, HNC |
| **Production** | Velero, Istio, Kubecost, OpenCost, Litmus, Chaos Mesh |
| **FinOps** | Goldilocks, KRR, Spot handling, Kube-downscaler |
| **DR/BC** | Velero cross-region, etcd snapshots, RTO/RPO, GameDays |
| **Emerging** | SpinKube, Kueue, Talos, Backstage |

---

## 🔥 Production Readiness Checklist

Before declaring any cluster production-ready, verify:

- [ ] **NetworkPolicies**: Default-deny ingress and egress enforced in all namespaces.
- [ ] **Pod Security Standards**: Enforced natively or via admission controllers.
- [ ] **RBAC**: Least-privilege access audited, no `cluster-admin` bindings for standard service accounts.
- [ ] **Secrets**: Encrypted at rest, rotated regularly, and strictly never committed in Git.
- [ ] **Backups**: Velero configured, `etcd` snapshots fully automated, DR scenarios tested.
- [ ] **Monitoring**: Prometheus + Grafana + Alertmanager actively running with defined SLOs.
- [ ] **Runtime Security**: Tetragon or Falco successfully deployed with actionable alerting.
- [ ] **Cost Control**: Karpenter consolidation enabled and Kubecost active for observability.
- [ ] **GitOps**: ArgoCD/Flux reconciling state, complete with image signature verification policies.
- [ ] **Disaster Recovery**: Quarterly DR drills conducted and documented in live runbooks.
- [ ] **Upgrade Testing**: Blue/green cluster upgrade simulated and validated in staging environment within the last 30 days.
- [ ] **etcd DR**: Automated etcd snapshot validation and restore drill performed quarterly.
- [ ] **Workload Identity**: No static IAM keys present in pods — reliant solely on OIDC IRSA / Workload Identity Federation.
- [ ] **Disruption Budgets**: Proper PDBs strictly defined for all StatefulSets and critical user-facing Deployments.
- [ ] **Spot Termination Handling**: Pods correctly tolerate spot interruption events using `topologySpreadConstraints` and node tolerations.
- [ ] **Cost Allocation**: Kubecost effectively visualizes cost per namespace/team/label, and FinOps dashboards are accessible.
- [ ] **Pull Request Environment Lifecycle**: Dynamic PR environments auto-deleted reliably after merge (< 48 hours).
- [ ] **Platform SLIs Tracked**: Real-time metrics exist for time-to-scaffold new services and platform API failure rates.

---

## 🔗 Learning Resources

- **Official Documentation**: [Kubernetes Docs](https://kubernetes.io/docs/home/), [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- **Cloud Native Ecosystem**: [CNCF Cloud Native Landscape](https://landscape.cncf.io/)
- **Networking & Security**: [Cilium Documentation](https://docs.cilium.io/), [Kyverno Policy Library](https://kyverno.io/policies/)
- **GitOps & Delivery**: [Argo CD Docs](https://argo-cd.readthedocs.io/), [Flux Documentation](https://fluxcd.io/flux/)
