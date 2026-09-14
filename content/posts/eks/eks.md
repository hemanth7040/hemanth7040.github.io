---
title: "Amazon EKS: A Beginner-Friendly Hands-On Setup Guide"
description: "A beginner-friendly, hands-on guide to Amazon EKS with simple real-world examples for every concept — create a cluster, connect kubectl, deploy an app, expose it with a LoadBalancer, scale it, update it, troubleshoot it, and clean up."
date: 2026-09-14
draft: false
tags:
  - AWS
  - EKS
  - Kubernetes
  - DevOps
  - kubectl
  - eksctl
categories: ["eks"]
---

# Amazon EKS: A Beginner-Friendly Hands-On Setup Guide

If you are new to Amazon EKS, the best way to learn is to create a cluster yourself and deploy a small application.

This guide takes you from an AWS account to a working EKS application.

We will use:

- **Amazon EKS** for Kubernetes
- **AWS CLI** to interact with AWS
- **kubectl** to interact with Kubernetes
- **eksctl** to create the EKS cluster
- **Nginx** as our sample application
- **Amazon ECR** briefly when we deploy a custom image

The goal is:

```text
AWS Account
   |
   v
Configure AWS CLI
   |
   v
Install kubectl + eksctl
   |
   v
Create EKS Cluster
   |
   v
Connect kubectl
   |
   v
Deploy Application
   |
   v
Create Service
   |
   v
Access Application
   |
   v
Scale / Update / Rollback
   |
   v
Cleanup
```

---

# 1. What Are We Going to Build?

Our example application will be Nginx.

At the end, the architecture will look like:

```text
                    AWS Cloud
                       |
                       v
                Amazon EKS Cluster
                       |
              +--------+--------+
              |                 |
         Worker Node 1     Worker Node 2
              |                 |
           Pod Nginx         Pod Nginx
              |                 |
              +--------+--------+
                       |
                       v
               Kubernetes Service
                       |
                       v
                AWS Load Balancer
                       |
                       v
                     User
```

We will first deploy the public Nginx image so that beginners can focus on EKS.

Later, we will see how your own image from ECR fits into the same setup.

> **Easy example:** Think of EKS as a hotel. AWS builds and maintains the hotel building (the control plane). You bring your own guests (your application, Nginx in this case) and the hotel gives them rooms (worker nodes) to stay in. Your job is to check the guests in, not to build the hotel.

---

# 2. Prerequisites

You need:

- An AWS account
- An IAM identity with permissions to create the required resources
- AWS CLI
- `kubectl`
- `eksctl`
- A terminal

This tutorial uses `ap-south-1` as the example AWS Region. You can replace it with another Region that supports EKS.

For a real production environment, use your organization's approved IAM and AWS account setup.

---

# 3. Configure AWS CLI

Check whether AWS CLI is installed:

```bash
aws --version
```

Configure your AWS credentials:

```bash
aws configure
```

Example:

```text
AWS Access Key ID: ********
AWS Secret Access Key: ********
Default region name: ap-south-1
Default output format: json
```

For production, prefer temporary credentials, IAM roles, or federation instead of long-lived access keys where possible.

---

# 4. Verify AWS Access

Run:

```bash
aws sts get-caller-identity
```

Example:

```json
{
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/devops-user"
}
```

If this command fails, fix AWS authentication before continuing.

---

# 5. Install kubectl

`kubectl` is the command-line tool used to communicate with Kubernetes.

Check:

```bash
kubectl version --client
```

The relationship is:

```text
Your Laptop
     |
     | kubectl
     v
EKS Kubernetes API Server
```

We will configure `kubectl` after creating the cluster.

---

# 6. Install eksctl

`eksctl` is a command-line tool that simplifies EKS cluster creation and management.

Check:

```bash
eksctl version
```

For beginners, `eksctl` is a good way to learn EKS because it handles much of the basic cluster setup.

Later, you can learn how to create EKS infrastructure using Terraform for more controlled infrastructure-as-code workflows.

---

# 7. Choose the AWS Region

For this tutorial:

```bash
export AWS_REGION=ap-south-1
```

Verify:

```bash
echo $AWS_REGION
```

You should see:

```text
ap-south-1
```

If you use another Region, replace `ap-south-1` in the commands.

---

# 8. Create Your First EKS Cluster

Now create the cluster:

```bash
eksctl create cluster   --name my-first-eks   --region ap-south-1   --nodes 2   --node-type t3.medium
```

This means:

```text
Cluster name : my-first-eks
Region       : ap-south-1
Nodes        : 2
Instance type: t3.medium
```

Cluster creation can take several minutes because AWS needs to create and configure multiple resources.

---

# 9. What Does `eksctl` Create?

A beginner may think that an EKS cluster is only a Kubernetes control plane.

In reality, a basic EKS environment also needs networking, compute, IAM, and security configuration.

Conceptually:

```text
eksctl
  |
  +----> VPC / Subnets
  |
  +----> EKS Control Plane
  |
  +----> Worker Node Group
  |
  +----> IAM Configuration
  |
  +----> Security Groups
  |
  +----> Kubernetes Access Configuration
```

The exact resources depend on the options and `eksctl` version you use.

> **Easy example:** Ordering a "combo meal" at a restaurant is one order, but you get a burger, fries, and a drink together. Running one `eksctl create cluster` command is similar — it is one instruction, but AWS creates many resources behind the scenes (network, servers, security rules, and more) to give you a working cluster.

---

# 10. Understand the EKS Control Plane and Nodes

An EKS cluster has two major areas.

## Control Plane

AWS manages the EKS control plane.

It contains Kubernetes control-plane components such as:

- API server
- Scheduler
- Controllers
- Cluster state storage

You don't manage the underlying control-plane machines yourself.

## Worker Nodes

Worker nodes run your application Pods.

Example:

```text
EKS Cluster
 |
 +-- AWS-managed Control Plane
 |
 +-- Worker Node 1
 |      |
 |      +-- Pod
 |
 +-- Worker Node 2
        |
        +-- Pod
```

This is one of the main benefits of EKS: AWS manages the Kubernetes control plane for you.

> **Easy example:** Picture a delivery company. AWS runs the head office that plans routes and tracks packages (the control plane). Your worker nodes are the delivery trucks that actually carry the packages (your Pods). You never have to manage the head office — you just load the trucks with the right packages.

---

# 11. Verify the Cluster

Run:

```bash
eksctl get cluster
```

Example:

```text
NAME            REGION       EKSCTL CREATED
my-first-eks    ap-south-1   True
```

You can also check with AWS CLI:

```bash
aws eks list-clusters --region ap-south-1
```

---

# 12. Configure kubectl

Now connect your local `kubectl` to EKS:

```bash
aws eks update-kubeconfig   --region ap-south-1   --name my-first-eks
```

This updates your kubeconfig with the EKS cluster information.

Conceptually:

```text
kubectl
   |
   v
kubeconfig
   |
   v
EKS API Server
```

---

# 13. Verify the Worker Nodes

Run:

```bash
kubectl get nodes
```

Example:

```text
NAME                                         STATUS   ROLES
ip-10-0-1-10.ap-south-1.compute.internal   Ready    <none>
ip-10-0-2-20.ap-south-1.compute.internal   Ready    <none>
```

The important value is:

```text
STATUS = Ready
```

If your nodes are Ready, the basic EKS cluster is working.

---

# 14. Check Kubernetes Namespaces

Run:

```bash
kubectl get namespaces
```

You will normally see namespaces such as:

```text
default
kube-node-lease
kube-public
kube-system
```

For this beginner example, we will use the `default` namespace.

For real applications, use a namespace strategy that matches your project and environment structure.

---

# 15. Deploy Your First Application

Now deploy Nginx:

```bash
kubectl create deployment nginx   --image=nginx:latest
```

Check the Deployment:

```bash
kubectl get deployment
```

Example:

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   1/1     1            1
```

Check the Pod:

```bash
kubectl get pods
```

Example:

```text
NAME                     READY   STATUS    RESTARTS
nginx-7f8c6d9d4f-abc12  1/1     Running   0
```

---

# 16. Understand Deployment → ReplicaSet → Pod

When you create a Deployment, Kubernetes creates the resources needed to maintain the desired number of Pods.

The simplified relationship is:

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pod
    |
    v
Container
    |
    v
Nginx
```

If the Pod crashes, the Deployment/ReplicaSet can create another Pod to maintain the desired state.

This is one of the core ideas of Kubernetes.

> **Easy example:** Think of a call center that must always have 3 agents answering calls. The Deployment is the policy ("always keep 3 agents on duty"). The ReplicaSet is the shift supervisor who checks the count and calls in a replacement the moment someone leaves. The Pod is the actual agent taking calls. If one agent goes home sick, the supervisor immediately brings in a new one so the count stays at 3.

---

# 17. Check Pod Details

Get the Pod name:

```bash
kubectl get pods
```

Then:

```bash
kubectl describe pod <pod-name>
```

For example:

```bash
kubectl describe pod nginx-7f8c6d9d4f-abc12
```

This is an important troubleshooting command.

It shows information such as:

- Pod status
- Node
- Container
- Image
- Events
- Volumes
- Conditions

---

# 18. Check Application Logs

Run:

```bash
kubectl logs <pod-name>
```

For example:

```bash
kubectl logs nginx-7f8c6d9d4f-abc12
```

When an application fails, logs are usually one of the first things you should check.

---

# 19. Test the Application with Port Forwarding

Before creating an AWS Load Balancer, we can test Nginx locally.

Run:

```bash
kubectl port-forward deployment/nginx 8080:80
```

Now open:

```text
http://localhost:8080
```

You should see the Nginx welcome page.

Port forwarding is useful for quick testing.

---

# 20. Expose the Application Using a Service

For external access, create a Kubernetes Service:

```bash
kubectl expose deployment nginx   --type=LoadBalancer   --port=80   --target-port=80
```

Check:

```bash
kubectl get service nginx
```

Initially, the external address may show:

```text
<pending>
```

Wait for AWS to provision the load balancer and check again:

```bash
kubectl get service nginx
```

The external address will normally be an AWS load balancer hostname.

---

# 21. Understand the Service Traffic Flow

The request path becomes:

```text
User
 |
 v
AWS Load Balancer
 |
 v
Kubernetes Service
 |
 v
Nginx Pod
 |
 v
Nginx Container
```

The Service selects Pods using labels.

> **Easy example:** A Service is like a restaurant's phone number. Customers (users) always dial the same number, no matter which staff member (Pod) answers. Staff can change shifts, take breaks, or be replaced, but the phone number stays the same, so customers never notice the difference.

You can inspect the Service:

```bash
kubectl describe service nginx
```

You can also inspect the endpoints:

```bash
kubectl get endpoints nginx
```

On newer Kubernetes versions, EndpointSlices can also be inspected:

```bash
kubectl get endpointslices
```

---

# 22. Scale the Application

Suppose you want three copies of Nginx.

Run:

```bash
kubectl scale deployment nginx --replicas=3
```

Check:

```bash
kubectl get pods
```

You should see three Pods.

> **Easy example:** Imagine a coffee shop that gets busier around lunchtime. Scaling from 1 Pod to 3 Pods is like calling in two extra baristas so more customers can be served at once, without changing the recipe for the coffee itself.

The architecture becomes:

```text
                 Load Balancer
                      |
                      v
                   Service
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
        Pod 1       Pod 2       Pod 3
```

---

# 23. Use Kubernetes YAML Instead of Imperative Commands

The commands above are useful for learning.

For real projects, you should normally keep Kubernetes configuration in YAML and store it in Git.

> **Easy example:** Typing commands one by one is like giving a chef instructions verbally, step by step — nothing is written down, so it is easy to forget a step or repeat it differently next time. A YAML file is like a written recipe card: anyone on the team can follow the exact same steps, and you can keep old versions of the recipe to see what changed.

Create:

```text
deployment.yaml
```

Example:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```

Apply it:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get deployment
kubectl get pods
```

---

# 24. Create the Service Using YAML

Create:

```text
service.yaml
```

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx

spec:
  type: LoadBalancer

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f service.yaml
```

Check:

```bash
kubectl get service
```

Now your Kubernetes configuration can be version-controlled.

---

# 25. Create a Namespace for Your Project

For a real application, you may want a dedicated namespace.

Create:

```bash
kubectl create namespace my-app
```

Check:

```bash
kubectl get namespaces
```

Deploy into it:

```bash
kubectl apply -f deployment.yaml -n my-app
```

Check:

```bash
kubectl get pods -n my-app
```

A possible environment structure is:

```text
EKS Cluster
 |
 +-- dev
 |
 +-- staging
 |
 +-- production
```

The exact structure depends on your organization's requirements.

> **Easy example:** A namespace is like a labeled folder on a shared computer. Everyone uses the same computer (the cluster), but the "dev" folder, "staging" folder, and "production" folder keep each team's files separate so they don't get mixed up.

---

# 26. Update the Application

Suppose your Deployment currently uses:

```yaml
image: nginx:1.27
```

Change it to:

```yaml
image: nginx:1.28
```

Apply the updated YAML:

```bash
kubectl apply -f deployment.yaml
```

Check the rollout:

```bash
kubectl rollout status deployment/nginx
```

Kubernetes will perform a rolling update according to the Deployment strategy.

---

# 27. Check Deployment History

Run:

```bash
kubectl rollout history deployment/nginx
```

This helps you understand previous Deployment revisions.

---

# 28. Roll Back the Application

If the new version has a problem:

```bash
kubectl rollout undo deployment/nginx
```

Check:

```bash
kubectl rollout status deployment/nginx
```

This is a simple and useful production recovery technique.

> **Easy example:** Rolling back is like undoing a bad software update on your phone. If the new version has a bug, you simply go back to the last version that worked well, instead of trying to fix the new version under pressure.

---

# 29. Use Your Own Docker Image from ECR

In a real project, you usually won't deploy the public Nginx image.

You will build your own application image:

```text
Application Source Code
        |
        v
Docker Build
        |
        v
Amazon ECR
        |
        v
Amazon EKS
```

For example:

```yaml
containers:
  - name: my-app
    image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/my-app:1.0.0
```

The EKS nodes/workload identity need appropriate AWS permissions to pull the private ECR image.

Your ECR blog can contain the detailed Docker-to-ECR setup; this section only shows where ECR fits into EKS.

> **Easy example:** Using the public Nginx image is like buying a plain, ready-made shirt from a public store. Using ECR is like using your own private tailor shop: you make your own shirt (Docker image), store it in your own warehouse (ECR), and only your team can access it, instead of everyone in the world.

---

# 30. Common EKS Troubleshooting

Learning EKS also means learning how to troubleshoot it.

A useful general flow is:

```text
Application
    |
    v
Pod
    |
    v
Deployment
    |
    v
Service
    |
    v
Load Balancer
    |
    v
AWS Networking
```

Check each layer instead of guessing.

> **Easy example:** Troubleshooting EKS is like figuring out why a letter never arrived. You check step by step: Did the sender write the address correctly (Application)? Was it picked up by the mail carrier (Pod)? Did the post office sort it correctly (Service)? Did it reach the right delivery truck (Load Balancer)? Checking each step in order is faster than guessing where it went wrong.

---

# 31. Pod Is `Pending`

Check:

```bash
kubectl get pods
```

If you see:

```text
Pending
```

run:

```bash
kubectl describe pod <pod-name>
```

Look at the Events section.

Common reasons:

- Not enough CPU or memory
- Scheduling constraints
- Node problems
- Storage problems
- Resource requests that cannot be satisfied

---

# 32. Pod Is `CrashLoopBackOff`

If you see:

```text
CrashLoopBackOff
```

the container is repeatedly starting and failing.

Check:

```bash
kubectl logs <pod-name>
```

Also:

```bash
kubectl describe pod <pod-name>
```

Common causes:

- Application crash
- Wrong startup command
- Missing configuration
- Missing environment variable
- Dependency failure
- Health-check failure
- Permission issue

---

# 33. Pod Is `ImagePullBackOff`

Check:

```bash
kubectl describe pod <pod-name>
```

Common causes:

- Wrong image name
- Wrong image tag
- Image doesn't exist
- ECR permission problem
- Network problem
- Wrong AWS Region
- Private registry authentication problem

For an ECR image, verify:

```bash
aws ecr describe-images   --repository-name my-app   --region ap-south-1
```

---

# 34. Node Is `NotReady`

Check:

```bash
kubectl get nodes
```

If a node shows:

```text
NotReady
```

run:

```bash
kubectl describe node <node-name>
```

Also check the EKS node group:

```bash
eksctl get nodegroup   --cluster my-first-eks   --region ap-south-1
```

Investigate:

- EC2 instance health
- Node resources
- Kubelet
- Networking
- IAM
- Security groups
- Node group status

---

# 35. Service Is Not Accessible

Check:

```bash
kubectl get service
```

Then:

```bash
kubectl describe service nginx
```

Check whether the Service has endpoints:

```bash
kubectl get endpoints nginx
```

Also check:

```bash
kubectl get pods -o wide
```

Use this troubleshooting path:

```text
User
 |
 v
Load Balancer
 |
 v
Service
 |
 v
Endpoints
 |
 v
Pod
 |
 v
Container
 |
 v
Application
```

---

# 36. Useful EKS and Kubernetes Commands

### EKS cluster

```bash
eksctl get cluster
```

### Kubernetes cluster information

```bash
kubectl cluster-info
```

### Nodes

```bash
kubectl get nodes
```

### Pods

```bash
kubectl get pods
```

### Pods with node information

```bash
kubectl get pods -o wide
```

### Deployments

```bash
kubectl get deployments
```

### Services

```bash
kubectl get services
```

### All resources

```bash
kubectl get all
```

### Pod details

```bash
kubectl describe pod <pod-name>
```

### Logs

```bash
kubectl logs <pod-name>
```

### Shell inside a container

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

### Apply YAML

```bash
kubectl apply -f deployment.yaml
```

### Delete YAML resources

```bash
kubectl delete -f deployment.yaml
```

### Rollout status

```bash
kubectl rollout status deployment/nginx
```

### Rollback

```bash
kubectl rollout undo deployment/nginx
```

---

# 37. Basic Production Considerations

The cluster created in this tutorial is intended for learning.

A production EKS environment needs more planning.

## Networking

Consider:

- VPC design
- Multiple Availability Zones
- Public and private subnets
- Security groups
- Network policies
- NAT Gateway
- VPC endpoints

## Compute

Consider:

- Managed node groups
- Instance sizing
- On-Demand vs Spot
- Karpenter
- Cluster Autoscaler

## Security

Consider:

- Least-privilege IAM
- EKS Pod Identity
- IRSA where appropriate
- Secrets management
- Network policies
- Image scanning
- Non-root containers

## Availability

Consider:

- Multiple Availability Zones
- Multiple worker nodes
- Multiple Pod replicas
- Readiness probes
- Liveness probes
- PodDisruptionBudgets

## Deployment

Consider:

- CI/CD
- Helm
- GitOps
- Rolling deployments
- Blue/green deployments
- Canary deployments

---

# 38. Recommended Project Structure

Once you start deploying your own applications, keep Kubernetes configuration organized.

A simple project can look like:

```text
my-app/
|
+-- Dockerfile
+-- src/
|
+-- k8s/
    |
    +-- namespace.yaml
    +-- deployment.yaml
    +-- service.yaml
    +-- configmap.yaml
    +-- ingress.yaml
```

Store these files in Git.

That gives your team a history of Kubernetes configuration changes.

---

# 39. Complete Beginner Workflow

Let's summarize the entire exercise.

### Step 1: Configure AWS

```bash
aws configure
```

### Step 2: Verify AWS

```bash
aws sts get-caller-identity
```

### Step 3: Create EKS

```bash
eksctl create cluster   --name my-first-eks   --region ap-south-1   --nodes 2   --node-type t3.medium
```

### Step 4: Configure kubectl

```bash
aws eks update-kubeconfig   --region ap-south-1   --name my-first-eks
```

### Step 5: Verify nodes

```bash
kubectl get nodes
```

### Step 6: Deploy Nginx

```bash
kubectl create deployment nginx   --image=nginx:latest
```

### Step 7: Verify Pods

```bash
kubectl get pods
```

### Step 8: Expose the application

```bash
kubectl expose deployment nginx   --type=LoadBalancer   --port=80   --target-port=80
```

### Step 9: Get the Load Balancer

```bash
kubectl get service nginx
```

### Step 10: Scale

```bash
kubectl scale deployment nginx --replicas=3
```

### Step 11: Clean up

```bash
eksctl delete cluster   --name my-first-eks   --region ap-south-1
```

---

# 40. Important: AWS Costs

An EKS learning environment can create AWS resources that incur charges.

Before creating the cluster, understand that costs can come from resources such as:

- EKS cluster
- EC2 worker nodes
- Load Balancer
- NAT Gateway
- EBS volumes
- Other AWS resources

When you finish your learning exercise, delete the cluster:

```bash
eksctl delete cluster   --name my-first-eks   --region ap-south-1
```

Then check the AWS Console for resources that were created separately and are not automatically removed.

---

# 41. What You Have Learned

After completing this guide, you should be able to:

- Configure AWS CLI
- Install and use `kubectl`
- Install and use `eksctl`
- Create an EKS cluster
- Understand the basic EKS architecture
- Connect `kubectl` to EKS
- Verify worker nodes
- Deploy an application
- Understand Deployment, ReplicaSet, and Pod
- Create a Kubernetes Service
- Expose an application using a Load Balancer
- Scale an application
- Update an application
- Roll back a Deployment
- Use Kubernetes YAML
- Create namespaces
- Troubleshoot common Pod problems
- Understand where ECR fits into EKS
- Delete the EKS environment after testing

---

# 42. What to Learn Next

After successfully completing this basic EKS setup, continue with:

```text
EKS Basics
    |
    v
EKS VPC Networking
    |
    v
Managed Node Groups
    |
    v
Kubernetes Services
    |
    v
Ingress
    |
    v
AWS Load Balancer Controller
    |
    v
EKS + ECR
    |
    v
IAM / EKS Pod Identity
    |
    v
ConfigMaps and Secrets
    |
    v
EBS / EFS Storage
    |
    v
HPA
    |
    v
Karpenter
    |
    v
Monitoring and Logging
    |
    v
Helm
    |
    v
CI/CD
    |
    v
GitOps
    |
    v
Production Troubleshooting
```

The most important part is to actually run the commands yourself.

Create the cluster, deploy an application, expose it, scale it, update it, intentionally troubleshoot a failure, and finally delete the environment.

That hands-on process gives you a much stronger understanding of EKS than learning only the definitions.