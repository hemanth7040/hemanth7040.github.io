---
title: "15 Kubernetes Scenario-Based Interview Questions and Answers for 7+ Years DevOps Experience"
description: "Production-oriented Kubernetes troubleshooting scenarios, commands, root causes, and interview-ready answers for senior DevOps engineers."
tags:
  - kubernetes
  - devops
  - interview
  - troubleshooting
  - eks
categories:
  - kubernetes
---

# 15 Kubernetes Scenario-Based Interview Questions and Answers

For a DevOps engineer with 7+ years of experience, Kubernetes interviews are usually less about definitions and more about **production troubleshooting**.

The interviewer may give you a symptom such as:

> "The Pod is running, but users cannot access the application. What would you check?"

A strong senior-level answer should show a systematic approach:

```text
Observe → Isolate → Verify → Fix → Verify again
```

---

## 1. Pod is stuck in `Pending`

### Scenario

> A newly deployed Pod is stuck in `Pending`. How would you troubleshoot it?

### Step 1 — Check the Pod

```bash
kubectl get pod <pod> -n <namespace>
kubectl describe pod <pod> -n <namespace>
```

Pay special attention to the **Events** section.

Typical messages include:

```text
Insufficient cpu
Insufficient memory
node(s) had untolerated taint
pod has unbound immediate PersistentVolumeClaims
didn't match Pod's node affinity
```

### Step 2 — Check nodes

```bash
kubectl get nodes
kubectl describe node <node>
```

Check:

- CPU and memory availability
- Taints
- Node conditions
- Disk pressure
- Memory pressure

### Step 3 — Check scheduling constraints

```bash
kubectl get pod <pod> -o yaml
```

Look for:

- `nodeSelector`
- Node affinity
- Pod affinity/anti-affinity
- Tolerations
- Topology spread constraints

### Step 4 — Check PVC

```bash
kubectl get pvc
kubectl describe pvc <pvc-name>
```

### Interview-ready answer

> "First, I would describe the Pod and inspect Events to identify why it hasn't been scheduled. Then I'd check node availability and resource capacity, taints and tolerations, node selectors or affinity rules, and PVC status. I wouldn't assume the scheduler is down just because the Pod is Pending."

### Key takeaway

```text
Pending
  ↓
Scheduling / Resources / Storage
```

---

## 2. Pod is in `CrashLoopBackOff`

### Scenario

> A Pod is continuously restarting and shows `CrashLoopBackOff`. What do you do?

### Step 1 — Check Pod state

```bash
kubectl get pod <pod> -n <namespace>
kubectl describe pod <pod> -n <namespace>
```

Look at:

- Last State
- Termination reason
- Exit code
- Events

### Step 2 — Check logs

```bash
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

`--previous` is particularly important because the previous container instance may contain the actual crash reason.

### Investigate

Common causes include:

- Application crash
- Wrong `command` or `args`
- Missing ConfigMap or Secret
- Bad environment variables
- Dependency failure
- Permission issue
- Liveness probe failure
- `OOMKilled`
- Application exits immediately

### Interview-ready answer

> "I'd check the Pod Events and termination reason first, then inspect the current and previous container logs. I'd look at the exit code and determine whether the application crashed, the container was OOMKilled, a command or configuration is wrong, or a liveness probe is repeatedly killing the container."

### Key takeaway

```text
CrashLoopBackOff
  ↓
Container starts
  ↓
Container/application fails
  ↓
Container restarts repeatedly
```

---

## 3. Pod is in `ImagePullBackOff`

### Scenario

> A Pod is stuck in `ImagePullBackOff`. What do you check?

### Step 1 — Check Events

```bash
kubectl describe pod <pod>
```

Look for errors such as:

```text
manifest unknown
repository does not exist
unauthorized
authentication required
i/o timeout
```

### Step 2 — Verify image

```bash
kubectl get pod <pod> -o yaml
```

Check:

- Repository name
- Image name
- Tag

Example:

```yaml
image: nginx:1.27
```

### Step 3 — Private registry

Check whether authentication is configured:

```bash
kubectl get pod <pod> -o yaml
```

Look for:

```yaml
imagePullSecrets:
```

For EKS/ECR, investigate the node/workload's AWS permissions and registry access.

### Step 4 — Check network connectivity

If image name and authentication are correct, investigate:

- DNS
- NAT/internet connectivity
- Security controls
- Registry availability
- Registry rate limits

### Interview-ready answer

> "I'd start with `kubectl describe pod` and inspect the Events for the exact image-pull error. Then I'd verify the image repository and tag. If it's a private registry, I'd check authentication and permissions. If those are correct, I'd investigate node-to-registry connectivity, DNS, and registry availability or rate limits."

### Key takeaway

```text
ImagePullBackOff
  ↓
Image / Registry / Authentication / Network
```

---

## 4. Pod is `Running` but `NotReady`

### Scenario

```text
NAME       READY   STATUS
api-pod    0/1     Running
```

### Meaning

The Pod has one container, the container is running, but the Pod is not considered ready to receive traffic.

### Step 1 — Check Events

```bash
kubectl describe pod <pod>
```

You may see:

```text
Readiness probe failed:
HTTP probe failed with statuscode: 503
```

### Step 2 — Check application logs

```bash
kubectl logs <pod>
```

### Step 3 — Check readiness probe configuration

```bash
kubectl get pod <pod> -o yaml
```

Check:

- Probe path
- Port
- Initial delay
- Timeout
- Failure threshold

### Other causes

- Application is still initializing
- Database is unavailable
- Dependency is unavailable
- Wrong readiness endpoint
- Wrong port

### Interview-ready answer

> "`Running` means the container is running, but `0/1 Ready` means it isn't currently eligible to receive traffic. I'd check Events for readiness probe failures, inspect application logs, and verify the readiness probe path, port, and configuration. I'd also check whether the application is still initializing or has an unavailable dependency."

### Key takeaway

```text
Running ≠ Ready
```

---

## 5. Service has no endpoints

### Scenario

```bash
kubectl get endpoints api-service
```

Output:

```text
NAME          ENDPOINTS
api-service   <none>
```

### Step 1 — Check Service configuration

```bash
kubectl get svc api-service -o yaml
```

Look at the selector:

```yaml
selector:
  app: api
```

### Step 2 — Check Pod labels

```bash
kubectl get pods --show-labels
```

The Service selector must match the Pod labels.

Example:

```text
Service selector: app=api
Pod label:        app=backend
```

No match means no endpoints.

### Step 3 — Check Pod readiness

```bash
kubectl get pods
```

A Pod can be:

```text
Running
0/1 Ready
```

and therefore not be a usable backend.

### Step 4 — Check namespace

The Service and selected Pods need to be in the appropriate same namespace for normal Service selection.

### Interview-ready answer

> "I'd first confirm the Service has no endpoints. Then I'd compare the Service selector with the Pod labels and verify that the Pods are Ready and in the correct namespace. If the selector doesn't match the labels, the Service won't select those Pods."

### Key takeaway

```text
Service
  ↓
Selector
  ↓
Matching + Ready Pods
  ↓
EndpointSlices
```

---

## 6. Service exists but traffic doesn't reach Pods

### Scenario

> The Service exists, Pods are running, but traffic isn't reaching the application.

### Troubleshooting path

```text
Client
  ↓
Service
  ↓
EndpointSlice
  ↓
Pod
  ↓
Application
```

### Step 1 — Check Service

```bash
kubectl get svc <service>
kubectl describe svc <service>
```

Check:

- Selector
- `port`
- `targetPort`

For example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

means:

```text
Service :80
    ↓
Pod :8080
```

### Step 2 — Check endpoints

```bash
kubectl get endpoints <service>
kubectl get endpointslice
```

Confirm that expected Pod IPs are present.

### Step 3 — Test application directly

```bash
kubectl exec <pod> -- curl localhost:8080
```

### Step 4 — Test Service

Create a temporary debugging Pod:

```bash
kubectl run debug --rm -it \
  --image=curlimages/curl -- sh
```

Then:

```bash
curl http://<service-name>:80
```

### If the Pod works but Service doesn't

Investigate:

- Service selector
- `targetPort`
- EndpointSlices
- NetworkPolicy
- CNI
- Service routing/kube-proxy

### Interview-ready answer

> "I'd verify the Service selector and port mapping, then check EndpointSlices to confirm the expected Pod IPs are registered. I'd test the application directly inside the Pod and then test the Service from another Pod. This isolates whether the problem is the application, Service configuration, or cluster networking."

---

## 7. Ingress returns `502` or `503`

### Scenario

> Users are receiving HTTP 502 or 503 errors through the Ingress/ALB.

### Architecture

```text
Internet
   ↓
ALB / Ingress
   ↓
Service
   ↓
EndpointSlice
   ↓
Pod
   ↓
Application
```

### Step 1 — Check Ingress

```bash
kubectl describe ingress <ingress>
```

Check:

- Host rules
- Paths
- Backend Service
- Events

### Step 2 — Check Service

```bash
kubectl describe svc <service>
```

### Step 3 — Check endpoints

```bash
kubectl get endpoints <service>
kubectl get endpointslice
```

### Step 4 — Check Pods

```bash
kubectl get pods -o wide
kubectl describe pod <pod>
```

### Step 5 — Test application

```bash
kubectl exec <pod> -- curl localhost:8080
```

Then test the Service from another Pod.

### For AWS ALB

Also investigate:

- Target health
- Target group
- Listener rules
- Security groups
- Subnets
- Health-check path
- AWS Load Balancer Controller events/logs

### 502 vs 503

The exact meaning depends on the proxy/load-balancer implementation, but generally:

- `502` often indicates a problem communicating with the backend.
- `503` often indicates no healthy/available backend.

### Interview-ready answer

> "I'd trace the request from the Ingress down to the Pod. I'd check Ingress configuration, the backend Service, EndpointSlices, Pod readiness, and application connectivity. For an AWS ALB, I'd also check target health, listener rules, health-check configuration, security groups, and the AWS Load Balancer Controller."

---

## 8. DNS failure

### Scenario

> Pods cannot resolve `my-service.default.svc.cluster.local`.

### Step 1 — Test DNS from a Pod

```bash
kubectl exec -it <pod> -- nslookup my-service
```

or:

```bash
kubectl exec -it <pod> -- nslookup \
  my-service.default.svc.cluster.local
```

### Step 2 — Check CoreDNS

```bash
kubectl get pods -n kube-system
```

Check CoreDNS Pods and logs:

```bash
kubectl logs -n kube-system <coredns-pod>
```

### Step 3 — Check CoreDNS Service

```bash
kubectl get svc -n kube-system
```

### Step 4 — Check Pod DNS configuration

```bash
kubectl exec <pod> -- cat /etc/resolv.conf
```

### Investigate

- CoreDNS Pods
- CoreDNS Service
- `/etc/resolv.conf`
- NetworkPolicy
- CNI/networking
- kube-proxy/service routing
- Upstream DNS if external resolution is failing

### Interview-ready answer

> "I'd first reproduce the DNS failure from inside a Pod using `nslookup` or `dig`. Then I'd check CoreDNS Pods and logs, the CoreDNS Service, and the Pod's `/etc/resolv.conf`. I'd also verify that NetworkPolicies or CNI issues aren't blocking DNS traffic."

---

## 9. Node is `NotReady`

### Scenario

```bash
kubectl get nodes
```

```text
worker-1   NotReady
```

### Step 1 — Describe the node

```bash
kubectl describe node worker-1
```

Check node Conditions:

```text
MemoryPressure
DiskPressure
PIDPressure
Ready
```

### Step 2 — Check node components

Investigate:

- kubelet
- container runtime
- CNI

On the node:

```bash
systemctl status kubelet
journalctl -u kubelet
```

### Step 3 — Check resources

```bash
df -h
free -m
```

### Common causes

- Kubelet down
- Container runtime failure
- Disk full
- Memory pressure
- CNI failure
- Network connectivity issue
- Certificate problem

### Interview-ready answer

> "I'd start with `kubectl describe node` and inspect the node Conditions and Events. Then I'd check kubelet and container runtime health, disk and memory pressure, and CNI/network connectivity. I'd also determine what happened to workloads running on that node and whether they need to be rescheduled."

---

## 10. Pod is `OOMKilled`

### Scenario

```text
Last State:
  Terminated
  Reason: OOMKilled
```

### Meaning

The container exceeded its available memory limit, or the node experienced memory pressure and the container was killed.

### Step 1 — Confirm the termination reason

```bash
kubectl describe pod <pod>
```

Look for:

```text
Reason: OOMKilled
```

### Step 2 — Check resources

```bash
kubectl get pod <pod> -o yaml
```

Example:

```yaml
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"
```

### Investigate

- Application memory usage
- Memory leak
- JVM heap or runtime memory
- Container memory limit
- Node memory pressure
- Recent application release

### Important

Don't automatically increase the memory limit.

First determine:

> Is the application consuming unexpectedly high memory, or is the limit incorrectly configured?

### Interview-ready answer

> "I'd confirm the termination reason is OOMKilled and inspect the container's memory requests and limits. Then I'd correlate memory usage with application behavior and recent deployments to determine whether there's a memory leak, excessive workload, or an incorrectly configured limit. I'd fix the root cause rather than simply increasing the limit."

---

## 11. HPA scales Pods but Pods remain `Pending`

### Scenario

```text
HPA:
5 replicas → 20 replicas

Pods:
5 Running
15 Pending
```

### Why?

HPA increased the desired number of Pods, but the cluster doesn't have enough capacity.

### Step 1 — Inspect Pending Pods

```bash
kubectl get pods
kubectl describe pod <pending-pod>
```

You may see:

```text
Insufficient cpu
Insufficient memory
```

### Step 2 — Check nodes

```bash
kubectl get nodes
kubectl describe nodes
```

### Step 3 — Check Cluster Autoscaler

If using Cluster Autoscaler:

```bash
kubectl get pods -n kube-system
```

Then inspect its logs.

### Architecture

```text
Traffic increases
      ↓
HPA
      ↓
More Pods requested
      ↓
Cluster lacks resources
      ↓
Pods Pending
      ↓
Cluster Autoscaler
      ↓
New node
      ↓
Pods scheduled
```

### Interview-ready answer

> "I'd first inspect the Pending Pods and their Events. If they're Pending due to insufficient CPU or memory, I'd check node capacity and Cluster Autoscaler behavior. HPA increases the number of Pods, but it doesn't create node capacity itself; the Cluster Autoscaler may need to add nodes."

---

## 12. PVC is stuck in `Pending`

### Scenario

```bash
kubectl get pvc
```

```text
NAME       STATUS
data-pvc   Pending
```

### Step 1 — Describe PVC

```bash
kubectl describe pvc data-pvc
```

Check Events.

### Step 2 — Check StorageClasses

```bash
kubectl get storageclass
```

Check:

- StorageClass exists
- Correct provisioner
- Default StorageClass
- Parameters

### Step 3 — Check PVs

```bash
kubectl get pv
```

### Common causes

- No StorageClass
- Wrong StorageClass
- CSI driver unavailable
- Provisioning failure
- Storage quota/capacity issue
- Unsupported access mode
- Cloud-provider permission problem

### EKS

Check the relevant CSI driver, such as the EBS CSI driver for EBS-backed volumes.

### Interview-ready answer

> "I'd describe the PVC and inspect its Events to identify the provisioning failure. Then I'd check the StorageClass, PVs, and CSI driver. In EKS I'd also verify that the CSI driver is healthy and has the required AWS permissions and that the requested storage configuration is supported."

---

## 13. NetworkPolicy is blocking traffic

### Scenario

> Frontend cannot reach backend after a NetworkPolicy was introduced.

### Step 1 — Check policies

```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <policy>
```

### Step 2 — Check Pod labels

```bash
kubectl get pods --show-labels
```

Compare the labels with the selectors used by the NetworkPolicy.

### Step 3 — Consider both directions

For:

```text
frontend → backend
```

consider:

```text
frontend egress
+
backend ingress
```

depending on the policies and networking implementation.

### Step 4 — Test connectivity

```bash
kubectl exec <frontend-pod> -- \
  curl http://backend-service:8080
```

### Also check DNS

A NetworkPolicy can unintentionally block DNS traffic, so verify:

```bash
kubectl exec <frontend-pod> -- nslookup backend-service
```

### Interview-ready answer

> "I'd first verify whether a NetworkPolicy was recently introduced or changed. Then I'd inspect ingress and egress rules, compare the policy selectors with actual Pod labels and namespaces, and test connectivity from the frontend Pod to the backend Service. I'd also verify that DNS traffic hasn't been unintentionally blocked."

---

## 14. EKS node is not joining the cluster

### Scenario

> An EKS worker node was created, but it doesn't appear in `kubectl get nodes`.

### Step 1 — Check AWS resources

Verify:

- EC2 instance is running
- Correct subnet
- Security groups
- IAM role
- Route/NAT connectivity
- EKS cluster endpoint accessibility

### Step 2 — Check node services

On the node:

```bash
systemctl status kubelet
journalctl -u kubelet
```

Also investigate the container runtime.

### Step 3 — Check bootstrap/configuration

Investigate:

- Cluster endpoint
- Cluster CA configuration
- Node IAM permissions
- Bootstrap process
- Network connectivity to the EKS API server

### Common causes

- Incorrect IAM permissions
- Security group/network issue
- Node cannot reach EKS API endpoint
- Bootstrap failure
- Wrong cluster configuration
- Instance profile problem

### Interview-ready answer

> "I'd first verify the EC2 instance, IAM role, networking, and security groups. Then I'd inspect kubelet and bootstrap logs on the node to see why it can't register with the EKS API server. I'd verify connectivity to the cluster endpoint and ensure the node IAM permissions and bootstrap configuration are correct."

---

## 15. Production zero-downtime deployment and rollback

### Scenario

> You need to deploy version 2 of a production application without downtime. How would you approach it?

### Use multiple replicas

```yaml
replicas: 3
```

Use a rolling update:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Use a properly configured readiness probe.

### Deployment flow

```text
v1   v1   v1
 ↓
Create v2
 ↓
v1   v1   v1   v2
 ↓
v2 becomes Ready
 ↓
Traffic moves toward v2
 ↓
Terminate one v1
 ↓
v1   v1   v2
 ↓
Create another v2
 ↓
...
```

### Monitor rollout

```bash
kubectl rollout status deployment/<deployment>
kubectl get pods
kubectl get rs
```

### Rollback

If the new version fails:

```bash
kubectl rollout undo deployment/<deployment>
```

Check rollout history:

```bash
kubectl rollout history deployment/<deployment>
```

### Production considerations

Also consider:

- Readiness probes
- Graceful shutdown
- `preStop`
- `terminationGracePeriodSeconds`
- PodDisruptionBudget
- Multiple replicas
- Appropriate `maxUnavailable`
- Application backward compatibility
- Database migration strategy
- Monitoring and alerting
- Canary or blue-green deployment for higher-risk releases

### Interview-ready answer

> "I'd use a rolling deployment with multiple replicas, a properly configured readiness probe, and `maxUnavailable: 0` so existing healthy Pods continue serving traffic while new Pods are deployed. I'd monitor the rollout and application metrics. If the new version shows errors, I'd use `kubectl rollout undo` to return to the previous ReplicaSet. For higher-risk releases, I'd consider canary or blue-green deployment."

---

# Quick Revision Cheat Sheet

| Scenario | First thing to check | Main area |
|---|---|---|
| `Pending` | `describe pod` → Events | Scheduling/resources/storage |
| `CrashLoopBackOff` | `logs --previous` + Events | Application/container |
| `ImagePullBackOff` | `describe pod` → Events | Image/registry/auth |
| `Running` + `0/1 Ready` | Readiness probe + logs | Application/readiness |
| Service no endpoints | EndpointSlices + selectors | Service/Pod selection |
| Service can't reach Pods | Service → endpoints → Pod | Networking |
| Ingress `502/503` | Ingress → Service → endpoints | External routing |
| DNS failure | `nslookup` + CoreDNS | DNS/networking |
| Node `NotReady` | `describe node` | Node/kubelet/CNI |
| `OOMKilled` | Termination reason + resources | Memory |
| HPA + Pending Pods | Pod Events + node capacity | Scaling/capacity |
| PVC `Pending` | PVC Events + StorageClass | Storage/CSI |
| NetworkPolicy issue | Policies + labels + connectivity | Network security |
| EKS node not joining | kubelet/bootstrap/IAM/network | EKS/node |
| Zero-downtime deployment | RollingUpdate + readiness | Deployment/release |

---

# Senior Kubernetes Troubleshooting Pattern

When you get any Kubernetes production scenario, use this thought process:

```text
1. What is the exact symptom?
          ↓
2. Which Kubernetes layer is failing?
          ↓
3. Gather evidence
          ↓
4. Check Events / logs / status
          ↓
5. Narrow the problem
          ↓
6. Apply the smallest appropriate fix
          ↓
7. Verify the application again
```

A useful request path to visualize is:

```text
Internet
   ↓
ALB / NLB
   ↓
Ingress / Ingress Controller
   ↓
Service
   ↓
EndpointSlice
   ↓
Pod
   ↓
Container
   ↓
Application
```

Not every application uses every component, but this model helps you identify **which layer to investigate**.

## Most important distinctions

```text
Pending
→ Pod exists but containers haven't successfully started;
  commonly waiting for scheduling/resources/storage.

CrashLoopBackOff
→ Container starts and repeatedly fails/restarts.

ImagePullBackOff
→ Kubernetes cannot successfully pull the container image.

Running but NotReady
→ Container is running, but Pod is not currently eligible
  to receive Service traffic.

Service has no endpoints
→ Service currently has no matching/usable backend Pods.

OOMKilled
→ Container was killed because of memory exhaustion.

Node NotReady
→ Node is not currently healthy/available to Kubernetes.
```

> **Senior-level interview tip:** Don't just list commands. Explain what you expect to see, how you interpret the result, and what you would check next.
