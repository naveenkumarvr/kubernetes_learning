Phase 1: Container & Orchestration Foundations
Goal: Understand the prerequisites. You cannot master Kubernetes without deeply understanding what it is orchestrating.

Linux Container Primitives: namespaces (isolation) and cgroups (resource allocation).

Container Runtimes: Docker vs. containerd vs. CRI-O.

The OCI Standard: Open Container Initiative image and runtime specifications.

Optimizing Images: Multi-stage builds, Distroless images, and minimal attack surfaces.

The Orchestrator Problem: Why Docker Swarm failed and why Kubernetes won (declarative state vs. imperative commands).

Local Clusters: Setting up Minikube, Kind (Kubernetes in Docker), and k3d.

Phase 2: Kubernetes Core Architecture & Components
Goal: Deconstruct the cluster. Understand exactly what happens under the hood when you run a command.

The Control Plane (Master Nodes):

kube-apiserver: The gateway and brain of the cluster.

etcd: The distributed key-value store (Raft consensus, backups, and defragmentation).

kube-scheduler: How it evaluates taints, tolerations, and affinities.

kube-controller-manager: The reconciliation loops.

cloud-controller-manager: Interfacing with AWS/GCP/Azure.

The Data Plane (Worker Nodes):

kubelet: The node agent communicating with the API and container runtime via CRI.

kube-proxy: Managing local networking rules.

API Fundamentals: GVK (Group, Version, Kind), API resources, and API deprecation cycles.

Interacting with the Cluster: kubectl mastery, imperative commands vs. declarative YAML manifests.

Organizational Primitives: Namespaces, Labels, Selectors, and Annotations.

Phase 3: Workload Management & Scheduling
Goal: Deploy, manage, and scale applications reliably.

The Pod Lifecycle: Pod phases, multi-container pods (Sidecar, Ambassador, Adapter patterns), Init Containers.

ReplicaSets & Deployments: Managing stateless apps, rollouts, rollbacks, and update strategies (RollingUpdate vs. Recreate).

StatefulSets: Managing stateful apps, headless services, sticky network identities, and stable storage allocation.

DaemonSets: Running node-level agents (logging, monitoring, networking).

Batch Processing: Jobs and CronJobs.

Advanced Scheduling & Eviction:

NodeSelector and NodeName.

Node Affinity and Pod Affinity/Anti-Affinity.

Taints and Tolerations.

Pod Topology Spread Constraints (High Availability).

Pod Disruption Budgets (PDB) and safe node draining.

Resource Requests, Limits, and Quality of Service (QoS) classes (Guaranteed, Burstable, BestEffort).

Phase 4: Configuration & State Management
Goal: Decouple configuration from code and manage persistent data.

Configuration: ConfigMaps and Secrets (base64 vs. actual encryption).

Advanced Secret Management: External Secrets Operator (ESO), HashiCorp Vault integration, and Sealed Secrets.

Storage Architecture:

Volumes (emptyDir, hostPath).

Persistent Volumes (PV) and Persistent Volume Claims (PVC).

StorageClasses and Dynamic Provisioning.

Container Storage Interface (CSI) mechanics.

Volume Snapshots and Cloning.

Phase 5: Cluster Networking & Traffic Routing
Goal: Master packet flow from the internet, through the cluster, to the specific container.

The Kubernetes Network Model: IP-per-pod concept and cross-node communication.

Container Network Interface (CNI): Deep dive into Flannel, Calico, and Cilium.

Service Discovery & Internal DNS: CoreDNS architecture and troubleshooting ndots issues.

Services: ClusterIP, NodePort, LoadBalancer, and ExternalName.

Kube-Proxy Modes: iptables vs. IPVS performance differences.

Ingress Routing: Ingress Controllers (NGINX, Traefik), Ingress Classes, and TLS termination.

The Future of Routing: The Kubernetes Gateway API (replacing traditional Ingress).

Service Meshes (Intro to Advanced): Istio/Linkerd, mTLS, traffic shadowing, and circuit breaking.

Phase 6: Security, Identity, & Hard Multi-Tenancy
Goal: Lock down the cluster to enterprise and banking standards.

Authentication & Authorization:

OIDC integration, X.509 certificates, and ServiceAccounts.

Role-Based Access Control (RBAC): Roles, RoleBindings, ClusterRoles, and ClusterRoleBindings.

Network Security: Network Policies (Layer 3/4) and Cilium Clusterwide Policies (Layer 7).

Workload Security:

Pod Security Standards (Baseline, Restricted, Privileged) replacing PodSecurityPolicies (PSP).

SecurityContexts (runAsUser, readOnlyRootFilesystem, dropping capabilities).

Policy as Code (Admission Controllers):

Mutating and Validating Admission Webhooks.

Open Policy Agent (OPA) Gatekeeper vs. Kyverno.

Runtime Security & Sandboxing: Container isolation using gVisor or Kata Containers, and real-time threat detection using Falco.

Phase 7: Observability, Logging, & Troubleshooting
Goal: Keep clusters healthy and debug complex issues in seconds.

Metrics & Telemetry: Metrics Server, cAdvisor, and Kube-State-Metrics.

The Prometheus Stack: Prometheus architecture, PromQL, Alertmanager, Grafana, and Thanos/Cortex for high-availability long-term storage.

Centralized Logging: FluentBit/Fluentd daemonsets, Promtail & Loki, or the ELK/EFK stack.

Distributed Tracing: OpenTelemetry and Jaeger inside Kubernetes.

Advanced Troubleshooting:

Using ephemeral debug containers (kubectl debug).

Reading API audit logs and etcd metrics.

Network packet capture inside pods (tcpdump, Wireshark).

Phase 8: Automation, Scaling, & GitOps
Goal: Automate deployments and scale infrastructure dynamically based on load.

Workload Autoscaling:

Horizontal Pod Autoscaler (HPA) using CPU/RAM and Custom Metrics (Prometheus Adapter / KEDA).

Vertical Pod Autoscaler (VPA).

Node Autoscaling: Legacy Cluster Autoscaler vs. Karpenter (JIT node provisioning).

Package Management: Helm deep dive (templating, charts, hooks, rollbacks) vs. Kustomize (overlay architecture).

GitOps Delivery: Continuous Reconciliation using ArgoCD or Flux.

Phase 9: Extending Kubernetes (The Top 1% Level)
Goal: Stop consuming Kubernetes. Start building it using Golang.

Custom Resource Definitions (CRDs): Designing domain-specific API extensions.

The Operator Pattern: Writing custom controllers using Kubebuilder or Operator SDK (Golang).

Reconciliation Mechanics: Writing robust, idempotent controller logic.

Custom Admission Webhooks: Writing Go microservices that intercept, mutate, or validate API requests before they hit etcd.

API Aggregation: Building custom API servers extension for Kubernetes.

Phase 10: The Master Capstone Project
Goal: Prove your expertise with a production-grade portfolio piece.

The Build: Develop a custom Go-based Kubernetes Operator that monitors standard application Deployments. If an application experiences a high error rate (read via Prometheus custom metrics), your Operator automatically mutates the Gateway API rules to route traffic away, spins up an isolated sandbox pod using gVisor, runs diagnostic trace scripts, and sends an automated Slack/Teams alert with the payload trace—all natively reconciled inside the cluster.

11. Production Cluster Upgrades (Zero Downtime)
The Upgrade Order: Upgrading etcd → kube-apiserver → Control Plane Components → kubelet & kube-proxy.

Version Skew Policy: Understanding the supported version gaps between the control plane components and worker nodes.

Eviction Mechanics: How kubectl drain works under the hood (cordoning nodes, respecting PodDisruptionBudgets, and handling local storage/DaemonSets).

Blue/Green Cluster Upgrades: How to spin up a parallel cluster and flip traffic at the DNS/Load Balancer level vs. an in-place rolling upgrade.

12. Node & Control Plane Troubleshooting
Node Conditions: Diagnosing and resolving Ready=False, DiskPressure, MemoryPressure, and PIDPressure.

Kubelet Triage: Debugging systemd services, inspecting journalctl -u kubelet, and resolving certificate expiration/rotation issues.

Container Runtime Crashing: Troubleshooting containerd sockets, cgroup driver mismatches (systemd vs cgroupfs), and storage driver exhaustion.

Control Plane Failures: Triage of a crashing API server, etcd corruption recovery from a snapshot, and fixing scheduling loops.

Network Triage: Debugging CNI IPAM (IP Address Management) exhaustion, fixing CoreDNS loop errors, and tracing packet drops using iptables-save or Cilium monitor.