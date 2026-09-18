---
title: "Kubernetes Interview Questions & Answers (7+ Years DevOps Experience)"
date: 2026-09-18T15:12:44+00:00
draft: false
description: "A comprehensive, structured list of Kubernetes interview questions and answers covering architecture, networking, storage, security, troubleshooting, and real production scenarios — aimed at senior DevOps/SRE engineers with 7+ years of experience."
tags: ["kubernetes", "devops", "interview-questions", "sre", "cloud-native", "docker", "helm", "ci-cd"]
categories:
  - kubernetes
toc: true
---

## Introduction

This guide compiles Kubernetes interview questions ranging from core fundamentals to deep, scenario-based questions that a **senior DevOps/SRE engineer (7+ years experience)** would realistically be asked. Instead of just definitions, many answers include the "why" and "how it breaks in production," since senior interviews focus heavily on troubleshooting and design trade-offs.

---

## 1. Core Architecture & Concepts

### 1.1 Explain the Kubernetes architecture in detail.
Kubernetes has a **control plane** and **worker nodes**.

- **Control Plane components:**
  - `kube-apiserver` – front door for all REST operations; validates and processes requests.
  - `etcd` – distributed key-value store holding cluster state.
  - `kube-scheduler` – assigns pods to nodes based on resource requirements, affinity rules, taints/tolerations.
  - `kube-controller-manager` – runs controllers (node, replication, endpoints, service account controllers).
  - `cloud-controller-manager` – integrates with cloud provider APIs (load balancers, nodes, routes).

- **Node components:**
  - `kubelet` – agent that ensures containers described in PodSpecs are running.
  - `kube-proxy` – maintains network rules for service-to-pod communication.
  - Container runtime (containerd, CRI-O).

### 1.2 What happens end-to-end when you run `kubectl apply -f deployment.yaml`?
1. `kubectl` sends the manifest to the API server over HTTPS.
2. API server authenticates (certs/tokens), authorizes (RBAC), and runs admission controllers (mutating/validating webhooks).
3. Object is persisted in `etcd`.
4. `kube-scheduler` watches for unscheduled pods and binds them to nodes.
5. `kubelet` on the target node watches the API server, pulls the image, and starts containers via the container runtime (CRI).
6. `kube-proxy` updates iptables/IPVS rules so the Service can route traffic to the new pod.
7. Controllers (Deployment → ReplicaSet → Pod) reconcile desired vs actual state continuously.

### 1.3 What is the difference between a Deployment, ReplicaSet, and Pod?
- **Pod** – smallest deployable unit; one or more containers sharing network/storage namespace.
- **ReplicaSet** – ensures a specified number of pod replicas are running; usually not managed directly.
- **Deployment** – manages ReplicaSets, provides rolling updates, rollbacks, and declarative versioning.

### 1.4 StatefulSet vs Deployment — when do you use each?
Use a **StatefulSet** when pods need:
- Stable, unique network identities (`pod-0`, `pod-1`, ...)
- Stable persistent storage tied to pod identity
- Ordered, graceful deployment/scaling (useful for databases, Kafka, Zookeeper, Elasticsearch)

Use a **Deployment** for stateless workloads where any pod replica is interchangeable.

### 1.5 What is a DaemonSet and when would you use one?
Ensures a copy of a pod runs on every (or selected) node. Common use cases: log collectors (Fluentd/Fluent Bit), node monitoring agents (Node Exporter), CNI plugins, security agents.

### 1.6 Explain Namespaces and when NOT to use them.
Namespaces provide logical isolation for resource organization, RBAC scoping, and resource quotas. Avoid over-segmenting namespaces for tightly coupled services that need frequent cross-namespace calls — it increases network policy/DNS complexity unnecessarily.

---

## 2. Networking

### 2.1 How does Pod-to-Pod networking work across nodes?
Kubernetes requires a flat network model where every pod can reach every other pod without NAT. This is implemented by a **CNI plugin** (Calico, Cilium, Flannel, Weave). Each node gets a subnet from the cluster CIDR; the CNI plugin sets up routes/overlays (VXLAN, BGP, eBPF) so pod IPs are routable cluster-wide.

### 2.2 What's the difference between ClusterIP, NodePort, LoadBalancer, and ExternalName services?
- **ClusterIP** – internal-only virtual IP, default type.
- **NodePort** – exposes a static port on every node's IP (30000–32767 range).
- **LoadBalancer** – provisions a cloud load balancer pointing to the NodePort/ClusterIP.
- **ExternalName** – maps a Service to an external DNS name (CNAME), no proxying involved.

### 2.3 How does `kube-proxy` route traffic? iptables vs IPVS?
- **iptables mode** – uses random selection with chains of NAT rules; O(n) rule evaluation, degrades at very large service counts.
- **IPVS mode** – uses a hash table in the kernel for O(1) lookups, supports more load-balancing algorithms (round robin, least connection), scales better for large clusters.

### 2.4 Explain Ingress vs Gateway API vs Service Mesh.
- **Ingress** – L7 HTTP(S) routing into the cluster via an Ingress Controller (NGINX, Traefik, ALB).
- **Gateway API** – newer, more expressive, role-oriented successor to Ingress supporting more protocols and finer-grained routing.
- **Service Mesh** (Istio, Linkerd) – adds mTLS, retries, circuit breaking, and fine-grained traffic shaping via sidecar proxies for **east-west** (service-to-service) traffic, whereas Ingress typically handles **north-south** traffic.

### 2.5 How does DNS resolution work inside a cluster?
CoreDNS runs as a cluster add-on. Each pod's `/etc/resolv.conf` points to the CoreDNS service IP. Services get DNS records like `<service>.<namespace>.svc.cluster.local`. Pods can resolve other services within the same namespace using short names.

### 2.6 Troubleshooting scenario: A pod can't reach another service — walk through your debugging steps.
1. `kubectl get pods -o wide` – confirm both pods are Running and note their IPs/nodes.
2. `kubectl exec` into the pod and `curl`/`nc` the target service IP and pod IP directly (isolates DNS vs network issue).
3. `kubectl get svc,endpoints` – confirm the Service has valid Endpoints (labels/selectors match).
4. Check `NetworkPolicy` resources restricting traffic.
5. Check CoreDNS pod logs/health if DNS resolution fails.
6. Check CNI plugin health/logs on both nodes (`calico-node`, `cilium` pods).
7. Verify security groups/firewall rules if nodes span subnets or cloud VPCs.

---

## 3. Storage

### 3.1 Explain PV, PVC, and StorageClass.
- **PersistentVolume (PV)** – cluster-wide storage resource provisioned by an admin or dynamically.
- **PersistentVolumeClaim (PVC)** – a user's request for storage (size, access mode).
- **StorageClass** – defines a provisioner (EBS, GCE PD, Azure Disk, Ceph, etc.) and parameters for dynamic provisioning.

### 3.2 What are access modes and which one is commonly misunderstood?
- `ReadWriteOnce (RWO)` – mounted read-write by a single node.
- `ReadOnlyMany (ROX)` – read-only by many nodes.
- `ReadWriteMany (RWX)` – read-write by many nodes (needs NFS/EFS/CephFS-type backends).

Misconception: RWO means single **pod**, but it's actually single **node** — multiple pods on the same node can mount an RWO volume simultaneously.

### 3.3 What is a CSI driver and why did Kubernetes move away from in-tree volume plugins?
The **Container Storage Interface (CSI)** decouples storage vendor code from the Kubernetes core, allowing vendors to ship/update drivers independently instead of requiring core Kubernetes releases. This improves maintainability, security, and release velocity.

### 3.4 How does `volumeClaimTemplates` work in a StatefulSet?
Each StatefulSet replica gets its own PVC created from the template, named predictably (`data-mypod-0`, `data-mypod-1`), and that PVC's binding persists even if the pod is rescheduled or deleted, preserving data continuity.

---

## 4. Scheduling & Resource Management

### 4.1 How does the scheduler decide where to place a pod?
Two phases:
1. **Filtering** – eliminate nodes that don't satisfy resource requests, taints/tolerations, affinity/anti-affinity, node selectors.
2. **Scoring** – rank remaining nodes (e.g., least requested resources, spread priority) and pick the highest score.

### 4.2 Explain requests vs limits and what happens when limits are exceeded.
- **Requests** – guaranteed minimum resources; used by the scheduler for placement.
- **Limits** – hard ceiling. Exceeding a CPU limit results in throttling; exceeding a memory limit results in the container being OOMKilled.

### 4.3 What are QoS classes in Kubernetes?
- **Guaranteed** – requests == limits for all containers.
- **Burstable** – at least one container has requests < limits.
- **BestEffort** – no requests/limits set.

QoS class determines eviction priority under node pressure — BestEffort pods are evicted first.

### 4.4 Taints and tolerations vs Node affinity — what's the difference?
- **Taints/Tolerations** – repel pods from nodes unless the pod explicitly tolerates the taint (node-centric control).
- **Node Affinity** – attracts pods to nodes based on labels (pod-centric preference), can be required or preferred.
They're often combined: taint dedicated nodes (e.g., GPU nodes) and use both a toleration and node affinity to ensure only intended pods land there.

### 4.5 Scenario: Pods are stuck in `Pending`. How do you debug?
1. `kubectl describe pod <pod>` – check Events section for scheduling failures (insufficient CPU/memory, no matching nodes, unsatisfied affinity/taints).
2. `kubectl get nodes -o wide` and check node conditions (`kubectl describe node`) for `MemoryPressure`, `DiskPressure`, or `NotReady`.
3. Check if a PVC is unbound (storage provisioning issue) blocking the pod.
4. Check ResourceQuota/LimitRange in the namespace.
5. Check Cluster Autoscaler logs if using dynamic node scaling — is it hitting cloud quota limits?

---

## 5. Autoscaling

### 5.1 HPA vs VPA vs Cluster Autoscaler — how do they differ and can they be combined?
- **HPA (Horizontal Pod Autoscaler)** – scales pod replica count based on metrics (CPU, memory, custom/external metrics via Prometheus Adapter).
- **VPA (Vertical Pod Autoscaler)** – adjusts CPU/memory requests/limits of existing pods (usually requires a restart).
- **Cluster Autoscaler** – adds/removes **nodes** based on unschedulable pods or underutilized nodes.

Combining HPA + VPA on the same metric can conflict; typically HPA handles CPU-based scaling while VPA is set to "off"/recommendation-only mode alongside it, or VPA manages memory while HPA manages CPU.

### 5.2 What is KEDA and why would you use it over standard HPA?
KEDA (Kubernetes Event-Driven Autoscaling) extends HPA to scale based on external event sources — queue length (SQS, RabbitMQ, Kafka lag), cron schedules, etc., including scale-to-zero, which standard HPA can't do.

---

## 6. Security

### 6.1 Explain RBAC components: Role, ClusterRole, RoleBinding, ClusterRoleBinding.
- **Role** – namespace-scoped permission set (verbs on resources).
- **ClusterRole** – cluster-scoped (or reusable across namespaces).
- **RoleBinding** – grants a Role (or ClusterRole) to a subject within a namespace.
- **ClusterRoleBinding** – grants cluster-wide permissions.

### 6.2 What are Pod Security Standards / Pod Security Admission?
Replacing the deprecated PodSecurityPolicy, Pod Security Admission enforces three levels — `Privileged`, `Baseline`, `Restricted` — at the namespace level via labels, controlling things like host namespace usage, privilege escalation, and capabilities.

### 6.3 How do you securely manage Secrets in Kubernetes?
- Enable **encryption at rest** for etcd (`EncryptionConfiguration`).
- Avoid committing raw Secrets to Git — use **Sealed Secrets**, **External Secrets Operator**, or cloud-native secret managers (AWS Secrets Manager, Vault, GCP Secret Manager).
- Restrict RBAC access to `secrets` resource.
- Consider mounting secrets as volumes rather than env vars (env vars can leak via logs/child processes more easily).

### 6.4 How do NetworkPolicies work, and what's a common pitfall?
NetworkPolicies are **allow-list** based and require a CNI that supports them (not all do, e.g., basic Flannel doesn't enforce them). A common pitfall: once you apply *any* NetworkPolicy selecting a pod, **all non-explicitly-allowed traffic is denied** by default — teams forget to allow DNS (port 53 to kube-system) and break pod DNS resolution.

### 6.5 What is a Service Account and how is it different from a User account?
Service Accounts are for **processes/pods** to authenticate to the API server, managed within Kubernetes (bound via tokens, often projected as short-lived JWTs today). User accounts are managed externally (OIDC, certificates, cloud IAM) — Kubernetes doesn't have a built-in User object.

### 6.6 How would you harden a production cluster?
- Enable audit logging.
- Use least-privilege RBAC, avoid `cluster-admin` bindings.
- Enforce Pod Security Admission (`Restricted` where possible).
- Use NetworkPolicies for default-deny east-west traffic.
- Scan images (Trivy, Grype) in CI; use admission controllers (OPA/Gatekeeper, Kyverno) to block unscanned/untrusted images.
- Rotate certificates, enable etcd encryption at rest, restrict API server access via private endpoints/bastion.
- Run nodes with minimal OS images, enable seccomp/AppArmor profiles.

---

## 7. Helm & Package Management

### 7.1 What problem does Helm solve?
Templating and packaging of Kubernetes manifests, versioned releases, rollback support, and dependency management (subcharts) — avoiding copy-pasted YAML across environments.

### 7.2 Explain Helm's release lifecycle and how rollback works.
Helm stores release history as Secrets (or ConfigMaps) in the cluster. `helm upgrade` renders a new manifest and diffs against the live state; `helm rollback <release> <revision>` reapplies a previous rendered manifest from history.

### 7.3 What are Helm hooks?
Special annotations (`helm.sh/hook: pre-install`, `post-upgrade`, etc.) that let you run Jobs at specific points in the release lifecycle — e.g., database migrations before an upgrade completes.

### 7.4 Helm vs Kustomize — how do you decide?
- **Helm** – templating language, parameterization, packaging/versioning, good for distributing reusable charts (e.g., third-party software).
- **Kustomize** – patch-based, template-free, built into `kubectl`, good for environment-specific overlays of your own manifests without introducing a templating DSL.
Many teams use Kustomize for internal app overlays and Helm for third-party charts (ingress-nginx, cert-manager, Prometheus stack).

---

## 8. Troubleshooting & Operations (Scenario-Based)

### 8.1 A pod is in `CrashLoopBackOff`. What's your process?
1. `kubectl logs <pod> --previous` – check the last crashed container's logs.
2. `kubectl describe pod` – check exit code and reason (OOMKilled, Error, non-zero exit).
3. Check liveness probe configuration — an overly aggressive probe can kill a slow-starting app.
4. Check resource limits — OOMKilled (exit code 137) means memory limit too low.
5. Validate config/secrets are correctly mounted (missing env var causing app to exit immediately).

### 8.2 `kubectl get nodes` shows a node as `NotReady`. What do you check?
1. `kubectl describe node <node>` – check Conditions and Events.
2. SSH to node, check `kubelet` service status/logs (`journalctl -u kubelet`).
3. Check container runtime health (containerd/docker).
4. Check disk pressure (`df -h`), memory pressure.
5. Check network connectivity between node and API server (certs, security groups, clock skew for TLS).

### 8.3 How do you perform a zero-downtime rolling update, and what can still break it?
Deployments default to `RollingUpdate` strategy with `maxSurge`/`maxUnavailable`. Zero downtime also requires:
- Properly configured **readiness probes** (so traffic isn't sent to a pod that isn't ready).
- **PodDisruptionBudgets** to prevent voluntary disruptions (node drains) from taking down too many replicas at once.
- Graceful shutdown handling (`preStop` hook + `terminationGracePeriodSeconds`) so in-flight requests complete before SIGKILL.
- Connection draining at the load balancer/Service level.

### 8.4 How do you debug high memory/CPU usage across a cluster?
- `kubectl top pods/nodes` (requires metrics-server) for a quick snapshot.
- Prometheus + Grafana for historical trends and per-container breakdowns.
- `kubectl exec` + `top`/`ps` inside containers, or ephemeral debug containers (`kubectl debug`) for distroless images without a shell.
- Check for memory leaks by correlating with deployment/release timelines.

### 8.5 How do you perform a Kubernetes version upgrade safely in production?
1. Read the version's release notes/deprecated API list; run `kubectl convert`/`pluto`/`kubent` to detect deprecated API usage.
2. Upgrade control plane first (API server → controller-manager/scheduler → etcd if needed), then worker nodes.
3. Upgrade in a staging cluster first, run smoke tests.
4. Use surge node pools / blue-green node groups (common in managed services like EKS/GKE/AKS) to upgrade nodes without downtime.
5. Drain nodes (`kubectl drain --ignore-daemonsets`) respecting PodDisruptionBudgets.
6. Validate workloads, CRDs, and admission webhooks are compatible with the new API server version.

### 8.6 etcd is your source of truth — how do you back it up and restore it?
Use `etcdctl snapshot save` regularly (automated via CronJob or managed service backup), store snapshots off-cluster (S3/GCS). Restore via `etcdctl snapshot restore` into a fresh data directory, then reconfigure/restart etcd members pointing to the restored state. Always test restores — an untested backup is not a backup.

---

## 9. CI/CD, GitOps & Observability

### 9.1 How does GitOps (ArgoCD/Flux) differ from a traditional CI/CD push-based pipeline?
GitOps is **pull-based**: an in-cluster controller (ArgoCD/Flux) continuously reconciles the cluster state to match a Git repository, rather than a CI pipeline pushing `kubectl apply` from outside. Benefits: auditable history, drift detection/auto-correction, easier rollback (git revert), and no need to expose cluster credentials to external CI systems.

### 9.2 What's your strategy for managing secrets in a GitOps workflow?
Never store plaintext secrets in Git. Use **Sealed Secrets** (encrypted, only the controller in-cluster can decrypt), **External Secrets Operator** pulling from Vault/AWS Secrets Manager at sync time, or SOPS-encrypted manifests decrypted at apply time.

### 9.3 What's the three pillars of observability and how do you implement them in Kubernetes?
- **Metrics** – Prometheus + kube-state-metrics + node-exporter, visualized in Grafana; alerting via Alertmanager.
- **Logs** – Fluent Bit/Fluentd/Vector shipping to a central store (Loki, Elasticsearch, CloudWatch).
- **Traces** – OpenTelemetry instrumentation, collected via OTel Collector, visualized in Jaeger/Tempo.

### 9.4 How do you design a CI/CD pipeline for Kubernetes deployments end-to-end?
1. CI: lint, unit test, build image, scan image (Trivy), push to registry with immutable tag (git SHA, not `latest`).
2. Update manifest/Helm values (image tag) — either via CI committing to a GitOps repo or triggering an ArgoCD Image Updater.
3. CD: GitOps controller syncs changes to cluster.
4. Post-deploy: automated smoke tests / synthetic checks; progressive delivery (canary/blue-green) via Argo Rollouts or Flagger, gated by metrics (error rate, latency) before full rollout.
5. Rollback: automatic on failed health checks, or manual via `git revert`.

---

## 10. Advanced / Architect-Level Scenario Questions

### 10.1 Design a multi-tenant Kubernetes platform for multiple internal teams. What do you consider?
- **Isolation**: namespace-per-team at minimum; consider vcluster or separate clusters for teams needing hard isolation (compliance).
- **RBAC**: scoped roles per namespace, avoid shared cluster-admin.
- **ResourceQuotas/LimitRanges**: prevent noisy-neighbor resource exhaustion.
- **NetworkPolicies**: default-deny between tenant namespaces.
- **Cost visibility**: chargeback/showback via tools like Kubecost, labeling standards enforced via policy (OPA/Kyverno).
- **Shared platform services**: centralized ingress controller, cert-manager, logging/monitoring stack, self-service via GitOps.

### 10.2 How would you design for multi-region / disaster recovery with Kubernetes?
- Active-passive or active-active clusters per region behind a global load balancer (Route53/Cloud DNS failover or Anycast).
- Stateless workloads replicate easily via GitOps to both clusters.
- Stateful data needs region-aware replication strategy independent of Kubernetes (managed DB replication, or tools like Velero for backup/restore of PVs and cluster objects).
- Regularly test failover with game days; automate DNS cutover and document RTO/RPO targets.

### 10.3 How do you handle noisy-neighbor problems on a shared cluster?
Set appropriate requests/limits (Guaranteed QoS for critical workloads), use `PriorityClasses` to control eviction order, isolate heavy workloads via taints/node pools, apply `ResourceQuota` per namespace, and monitor for CPU throttling using metrics like `container_cpu_cfs_throttled_seconds_total`.

### 10.4 A production incident: API server latency spikes cluster-wide. How do you triage?
1. Check API server metrics (`apiserver_request_duration_seconds`) and etcd metrics (`etcd_disk_wal_fsync_duration_seconds`) — etcd disk latency is a very common root cause.
2. Check for a controller/operator in a reconcile loop hammering the API (`apiserver_request_total` by client/user-agent).
3. Check node count / object count growth — very large clusters or CRD/watch storms can overload the API server.
4. Check control plane resource utilization (CPU/memory) if self-managed; check managed service status page if using EKS/GKE/AKS.
5. Mitigate: rate-limit/throttle offending controllers, scale API server (if self-hosted), consider etcd defragmentation.

### 10.5 How would you migrate a stateful legacy application into Kubernetes with minimal downtime?
- Containerize and validate behavior in a non-prod cluster first.
- Use a StatefulSet with proper `volumeClaimTemplates` mapped to existing storage (or migrate data via a one-time job/replication).
- Run old and new systems in parallel; use DNS/traffic weighting to gradually shift load (canary migration).
- Ensure backward-compatible data schema during the cutover window.
- Have a tested rollback plan (keep the legacy system warm until the new system is proven stable).

### 10.6 What are Operators, and when would you build a custom one?
Operators extend Kubernetes with **Custom Resource Definitions (CRDs)** plus a controller that encodes operational knowledge (backup, scaling, failover) for a specific application — essentially automating what an SRE would do manually. Build a custom operator when you have repeatable, complex operational logic for a stateful or specialized system (e.g., managing database clusters, certificate lifecycle, or custom scaling logic) that off-the-shelf controllers don't cover.

### 10.7 How do admission webhooks (mutating/validating) fit into the request lifecycle, and what risks do they introduce?
They intercept requests after authentication/authorization but before persistence to etcd. **Mutating webhooks** run first (can modify the object), then **validating webhooks** (can only accept/reject). Risks: if a webhook's endpoint is unavailable and `failurePolicy: Fail` is set, it can block **all** matching API operations cluster-wide — including the ability to fix the webhook itself. Always scope webhooks tightly (`namespaceSelector`, `objectSelector`) and monitor their availability closely.

---

## How to Use This List

- If you're prepping for an interview, don't just memorize answers — be ready to whiteboard the architecture diagram and narrate a real incident you've handled for each troubleshooting section.
- Tailor the "Advanced/Architect" section answers with specifics from your own environment (cloud provider, cluster size, tooling) — interviewers at the 7-year level want to hear real trade-offs you've made, not textbook answers.

*Feel free to fork/extend this list on your Hugo site as you encounter new questions in interviews.*