---
title: "Amazon ECR Interview Guide for Senior DevOps Engineers"
description: "A practical, beginner-friendly guide to Amazon Elastic Container Registry covering authentication, IAM, EKS integration, image tagging, scanning, lifecycle policies, cross-account access, replication, security, CI/CD, and troubleshooting."
date: 2026-09-12
draft: false
tags:
  - AWS
  - ECR
  - Docker
  - Kubernetes
  - EKS
  - DevOps
  - CI/CD
categories: ["ecr"]
summary: "A practical Amazon ECR guide for senior DevOps engineers — authentication, IAM, EKS integration, tagging, scanning, lifecycle policies, cross-account access, replication, and troubleshooting."
showToc: true
TocOpen: true
cover:
  hidden: true
---

Amazon Elastic Container Registry (ECR) is an AWS-managed container registry used to **store, manage, scan, and distribute container images**.

For a senior DevOps engineer, knowing only `docker push` and `docker pull` is not enough. You should understand ECR from a **production, security, CI/CD, Kubernetes, and troubleshooting perspective**.

This guide explains the important ECR concepts in simple English with practical examples.

---

## 1. What is Amazon ECR?

Amazon ECR is a managed container registry provided by AWS.

It allows organizations to store Docker/OCI container images privately and make them available to services such as:

- Amazon EKS
- Amazon ECS
- AWS Batch
- EC2-based container workloads
- CI/CD pipelines

Typical workflow:

```text
Developer
    |
    v
Git Repository
    |
    v
CI/CD Pipeline
    |
    v
Docker Build
    |
    v
Security Scan
    |
    v
Amazon ECR
    |
    +--------> Amazon EKS
    |
    +--------> Amazon ECS
```

### Real-world example

Suppose a company has a payment application.

The application is packaged as:

```text
payment-service:1.5.0
```

The CI/CD pipeline builds the image and pushes it to:

```text
Amazon ECR
    |
    +-- payment-service
         |
         +-- 1.4.0
         +-- 1.5.0
         +-- 1.5.1
```

The production EKS cluster can then pull the required image from ECR.

---

## 2. ECR Architecture

A simple architecture is:

```text
                       AWS Account
                           |
                           v
                    Amazon ECR
                           |
                    +------+------+
                    |             |
                    v             v
               Repository A   Repository B
                    |
                    v
               Container Images
                    |
              +-----+-----+
              |           |
              v           v
             EKS         ECS
```

A more complete DevOps workflow:

```text
Developer
    |
    v
Git
    |
    v
CI/CD
    |
    +--> Unit Tests
    |
    +--> Docker Build
    |
    +--> Image Scan
    |
    +--> Image Tag
    |
    v
Amazon ECR
    |
    +--> Development
    |
    +--> Staging
    |
    +--> Production
```

---

## 3. ECR Repository

An ECR repository is where container images are stored.

For example:

```text
ECR
 |
 +-- frontend
 +-- backend
 +-- payment-service
 +-- user-service
```

Create a repository:

```bash
aws ecr create-repository \
  --repository-name payment-service \
  --region ap-south-1
```

List repositories:

```bash
aws ecr describe-repositories --region ap-south-1
```

Describe a specific repository:

```bash
aws ecr describe-repositories \
  --repository-names payment-service \
  --region ap-south-1
```

### Real-world example

An organization with multiple microservices may have:

```text
ECR
 |
 +-- user-service
 +-- payment-service
 +-- order-service
 +-- notification-service
 +-- frontend
```

Each repository can contain multiple image versions.

---

## 4. Docker Image to ECR Workflow

This is one of the most important ECR workflows for interviews.

### Step 1: Build the image

```bash
docker build -t payment-service:1.0 .
```

### Step 2: Tag the image for ECR

```bash
docker tag payment-service:1.0 123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment-service:1.0
```

### Step 3: Authenticate Docker with ECR

```bash
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

### Step 4: Push the image

```bash
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment-service:1.0
```

Complete flow:

```text
Dockerfile
    |
    v
docker build
    |
    v
Local Docker Image
    |
    | docker tag
    v
ECR Image Reference
    |
    | docker push
    v
Amazon ECR
```

---

## 5. ECR Authentication

A common interview question is:

> How does Docker authenticate with Amazon ECR?

Typical command:

```bash
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

Conceptually:

```text
AWS CLI
   |
   | Request ECR authentication token
   v
Amazon ECR
   |
   v
Authentication token
   |
   v
Docker login
   |
   v
Docker can push/pull
```

The AWS identity executing the command must have the required IAM permissions.

### Important point

Do not treat the ECR login password as a permanent AWS access key. The AWS identity is used to obtain an ECR authentication token, and Docker uses that authentication for registry operations.

---

## 6. IAM Permissions for ECR

ECR access depends heavily on IAM.

A CI/CD pipeline that pushes images may need permissions such as:

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:CompleteLayerUpload
ecr:InitiateLayerUpload
ecr:UploadLayerPart
ecr:PutImage
```

A client pulling images may need permissions such as:

```text
ecr:GetAuthorizationToken
ecr:BatchGetImage
ecr:GetDownloadUrlForLayer
```

Use least privilege and tailor permissions to the actual workflow.

### Troubleshooting example

Suppose Jenkins can authenticate to AWS but `docker push` returns `AccessDeniedException`.

Investigate:

1. Which AWS identity is Jenkins using?
2. Run:

```bash
aws sts get-caller-identity
```

3. Check IAM permissions.
4. Check the ECR repository policy.
5. Verify AWS account.
6. Verify region.
7. Check permission boundaries or SCPs if applicable.

---

## 7. ECR and Amazon EKS

This is especially important for AWS DevOps engineers.

Typical architecture:

```text
Developer
    |
    v
Git
    |
    v
CI/CD
    |
    v
Docker Build
    |
    v
Security Scan
    |
    v
Amazon ECR
    |
    v
Amazon EKS
    |
    v
Kubernetes Pod
    |
    v
Container
```

Example Kubernetes deployment:

```yaml
containers:
  - name: payment-service
    image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment-service:1.5.0
```

When the pod needs the image, the node/runtime must have appropriate authorization to pull the private ECR image.

---

## 8. How Does EKS Pull an Image from Private ECR?

Interview question:

> How does EKS pull an image from private ECR?

At a high level:

```text
EKS
 |
 | Image pull
 v
Container Runtime
 |
 | AWS authorization
 v
Amazon ECR
 |
 | Image layers
 v
Container Runtime
 |
 v
Container
```

You should understand:

- Node IAM role
- EKS Pod Identity
- IAM Roles for Service Accounts (IRSA)
- ECR repository policies
- ECR authentication
- Kubernetes image pulling
- Container runtime

### Troubleshooting

If you see:

```text
ImagePullBackOff
```

run:

```bash
kubectl describe pod <pod-name>
```

Look at the Events section.

Check:

```text
Image name correct?
        |
        v
Correct account?
        |
        v
Correct region?
        |
        v
Repository exists?
        |
        v
Image/tag exists?
        |
        v
IAM permissions correct?
        |
        v
Network access available?
```

---

## 9. ECR Image Tagging

Tags are human-readable references to images.

Examples:

```text
payment-service:1.0
payment-service:1.5.2
payment-service:2026.09.12
payment-service:git-a81f3c2
```

Avoid using:

```text
payment-service:latest
```

as the only production identifier.

### Why?

Tags can be mutable.

For example:

```text
payment-service:latest
          |
          v
       Image A
```

Later:

```text
payment-service:latest
          |
          v
       Image B
```

The same tag now points to different content.

---

## 10. Recommended CI/CD Tagging Strategy

A CI/CD pipeline can generate tags from:

- Application version
- Git commit SHA
- Build number
- Release number

Example:

```text
payment-service:1.5.2
payment-service:a81f3c2
```

Example pipeline:

```bash
IMAGE_TAG=$(git rev-parse --short HEAD)

docker build -t payment-service:$IMAGE_TAG .

docker tag payment-service:$IMAGE_TAG \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment-service:$IMAGE_TAG

docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment-service:$IMAGE_TAG
```

### Important DevOps principle

> Build once and promote the same artifact across environments.

For example:

```text
Build
  |
  v
Image A
  |
  +--> Dev
  |
  +--> Staging
  |
  +--> Production
```

This improves artifact consistency and makes rollback easier.

---

## 11. ECR Image Digest

An image can also be referenced by its digest:

```text
payment-service@sha256:abc123...
```

A digest identifies specific image content.

Compare:

```text
payment-service:latest
```

with:

```text
payment-service@sha256:abc123...
```

A tag is a named reference that can be changed.

A digest is content-addressed and identifies a particular image manifest/content.

### Senior interview question

> Why deploy using an image digest?

Answer:

> To ensure the deployment references the exact image artifact that was tested and approved, avoiding unexpected changes caused by mutable tags.

---

## 12. ECR Tag Immutability

ECR supports tag immutability settings.

The purpose is to prevent an existing tag from being overwritten.

For example:

```text
payment-service:1.5.0
       |
       +--> First push: allowed
       |
       +--> Different image with same tag: rejected
```

### Real-world example

Without immutability:

```text
1.5.0 -> Image A
```

A developer accidentally pushes:

```text
1.5.0 -> Image B
```

Now the meaning of `1.5.0` has changed.

With immutable tags, this accidental overwrite can be prevented.

---

## 13. ECR Image Scanning

Image scanning checks container images for known vulnerabilities.

Potential findings include vulnerabilities in:

- Operating system packages
- Application dependencies
- Libraries
- Runtime components

Common tools include:

- ECR scanning capabilities
- Trivy
- Grype
- Snyk

Example:

```bash
trivy image payment-service:1.5.0
```

### CI/CD workflow

```text
Build
  |
  v
Docker Image
  |
  v
Security Scan
  |
  +---- Critical vulnerability ----> Fail pipeline
  |
  v
Push
  |
  v
ECR
  |
  v
Deploy
```

### Important senior-level point

Scanning is not a one-time activity. New vulnerabilities can be discovered after an image has already been built and deployed, so organizations need a process for reassessment and remediation.

---

## 14. Handling Critical Vulnerabilities

Suppose a scanner reports a critical vulnerability.

A professional response is:

```text
Identify vulnerability
        |
        v
Identify affected package
        |
        v
Check exploitability/exposure
        |
        v
Update base image/package
        |
        v
Rebuild image
        |
        v
Run tests
        |
        v
Scan again
        |
        v
Deploy fixed image
```

Do not simply delete the image and assume the security issue is solved.

---

## 15. ECR Lifecycle Policies

CI/CD systems can create many images:

```text
build-1001
build-1002
build-1003
...
build-5000
```

Keeping every image forever increases storage usage and management overhead.

ECR lifecycle policies can automatically expire images according to retention rules.

Typical policies may consider:

- Number of images to retain
- Image age
- Tagged vs untagged images
- Development vs production retention

### Real-world example

A development repository might retain a limited number of recent CI images, while production repositories retain release images longer for rollback and compliance requirements.

Do not delete images blindly. Align retention with rollback and compliance needs.

---

## 16. Untagged Images

ECR can accumulate untagged image manifests/layers.

Example:

```text
payment-service
 |
 +-- 1.5.0
 +-- 1.5.1
 +-- 1.5.2
 +-- untagged
 +-- untagged
 +-- untagged
```

Lifecycle policies can help clean up images that are no longer needed.

This is an important storage-management practice.

---

## 17. ECR Repository Policies

Two concepts should be distinguished:

```text
IAM Policy
    |
    v
Controls AWS principal permissions

ECR Repository Policy
    |
    v
Controls access to a particular ECR repository
```

Repository policies are particularly useful for cross-account access scenarios.

---

## 18. Cross-Account ECR Access

A common enterprise architecture is:

```text
AWS Account A
    |
    | Build
    v
Central ECR
    |
    +------------------+
    |                  |
    v                  v
Account B           Account C
   EKS                 EKS
```

Another possible model:

```text
Shared Services Account
        |
        v
Central ECR
        |
        +----> Dev Account
        |
        +----> Staging Account
        |
        +----> Production Account
```

Cross-account access generally requires the correct combination of:

- IAM permissions
- ECR repository policy
- Authentication
- Correct AWS account
- Correct region

### Interview question

> How would you allow Account B to pull images from an ECR repository owned by Account A?

Mention:

1. Repository policy in Account A.
2. Appropriate IAM permissions in Account B.
3. ECR authentication.
4. Correct repository URI.
5. Correct account and region.
6. Testing the actual pull operation.

---

## 19. ECR Encryption

ECR supports encryption for stored images.

Organizations can use AWS-managed encryption or configure customer-managed KMS keys where appropriate.

### Why use a customer-managed KMS key?

Potential reasons include:

- Organizational control
- Key policies
- Audit requirements
- Compliance requirements
- Separation of duties

Simplified view:

```text
Docker Image
     |
     v
ECR
     |
     v
Encrypted Storage
     |
     v
AWS KMS
```

Encryption at rest does not replace:

- IAM
- Image scanning
- Secure CI/CD
- Secret management
- Network security

---

## 20. ECR Replication

ECR can be used in multi-region or multi-account architectures.

Example:

```text
ECR ap-south-1
       |
       | Replication
       v
ECR ap-southeast-1
```

This can help with:

- Disaster recovery
- Multi-region deployments
- Business continuity
- Lower image transfer latency

Another model:

```text
Build Account
      |
      v
Central ECR
      |
      +----> Production Account
      |
      +----> DR Account
```

Understand:

- Cross-region replication
- Cross-account replication
- Replication configuration
- Destination image availability

---

## 21. Private vs Public ECR

### Private ECR

Used for internal/private application images.

```text
Company
   |
   v
Private ECR
   |
   +--> EKS
   +--> ECS
   +--> CI/CD
```

### Public ECR

Used when an organization intentionally publishes container images publicly.

For most enterprise DevOps interviews, private ECR is the more important area.

---

## 22. ECR + Jenkins

A common CI/CD architecture:

```text
Developer
    |
    v
GitHub
    |
    v
Jenkins
    |
    +--> Checkout
    |
    +--> Unit Tests
    |
    +--> Docker Build
    |
    +--> Security Scan
    |
    +--> ECR Authentication
    |
    +--> Docker Push
    |
    v
Amazon ECR
    |
    v
EKS
```

Example shell logic:

```bash
IMAGE_TAG=$(git rev-parse --short HEAD)

docker build -t payment-service:$IMAGE_TAG .

aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com

docker tag payment-service:$IMAGE_TAG \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment-service:$IMAGE_TAG

docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment-service:$IMAGE_TAG
```

In production, prefer short-lived AWS credentials/roles rather than long-lived static credentials where your CI platform supports that model.

---

## 23. ECR + GitHub Actions

Typical architecture:

```text
GitHub
   |
   v
GitHub Actions
   |
   v
AWS IAM Authentication
   |
   v
ECR
   |
   v
EKS
```

Modern AWS integrations commonly use GitHub Actions OIDC federation to obtain temporary AWS credentials instead of storing long-lived AWS access keys.

---

## 24. ECR Troubleshooting

### Problem 1: `no basic auth credentials`

Possible causes:

- Docker is not authenticated to ECR.
- Authentication token is no longer valid.
- Wrong AWS region.
- Wrong registry URI.
- Wrong AWS identity.

Check:

```bash
aws sts get-caller-identity
```

Then authenticate again:

```bash
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

---

### Problem 2: `AccessDeniedException`

Check:

```text
AWS identity
     |
     v
IAM permissions
     |
     v
Repository policy
     |
     v
SCP / permission boundary
     |
     v
Account / region
```

---

### Problem 3: Repository Does Not Exist

Check:

```bash
aws ecr describe-repositories \
  --repository-names payment-service \
  --region ap-south-1
```

Possible causes:

- Wrong repository name
- Wrong region
- Wrong AWS account
- Typo in ECR URI

---

### Problem 4: EKS `ImagePullBackOff`

Run:

```bash
kubectl describe pod <pod-name>
```

Check Events.

Investigate:

```text
Image name
Image tag
Repository
AWS account
Region
IAM permissions
Repository policy
Network connectivity
Node/pod identity
```

---

## 25. ECR Cost Optimization

Important areas include:

- Lifecycle policies
- Removing unnecessary images
- Cleaning untagged images
- Avoiding duplicate image builds
- Choosing sensible retention periods
- Monitoring repository storage
- Using appropriate replication architecture

### Real-world example

Suppose a company runs:

```text
100 CI builds/day
```

and every build pushes an image.

After one year:

```text
36,500+ images
```

If most are never used again, a sensible lifecycle policy can significantly reduce unnecessary storage.

Do not delete images blindly. Retain enough versions for:

- Rollbacks
- Incident investigation
- Compliance
- Release support

---

## 26. Enterprise ECR Architecture

A mature enterprise setup could look like:

```text
                         Git
                          |
                          v
                    CI/CD Pipeline
                          |
                 +--------+--------+
                 |                 |
                 v                 v
             Unit Tests      Docker Build
                                   |
                                   v
                              Image Scan
                                   |
                                   v
                         Central ECR / Build
                                   |
                         +---------+---------+
                         |                   |
                         v                   v
                    Dev Registry        Prod Registry
                         |                   |
                         v                   v
                        EKS                 EKS
                         |                   |
                         v                   v
                       Pods                 Pods
```

For larger organizations, you may also introduce:

```text
Multi-account
Multi-region
KMS
Lifecycle policies
Tag immutability
Image scanning
Replication
IAM
Repository policies
Audit logging
```

---

## 27. ECR Interview Questions

### Fundamentals

1. What is Amazon ECR?
2. Why use ECR instead of Docker Hub?
3. What is an ECR repository?
4. What is the difference between private and public ECR?
5. How do you push an image to ECR?
6. How do you pull an image from ECR?

### Authentication and IAM

7. How does Docker authenticate with ECR?
8. What IAM permissions are required to push an image?
9. What permissions are required to pull an image?
10. How do you troubleshoot an ECR `AccessDeniedException`?
11. IAM policy vs ECR repository policy?

### Docker and ECR

12. How do you tag an image for ECR?
13. What happens during `docker push`?
14. How are image layers handled?
15. Tag vs digest?
16. Why is `latest` risky in production?
17. What is tag immutability?

### EKS

18. How does EKS pull images from private ECR?
19. How do you troubleshoot `ImagePullBackOff`?
20. Node IAM role vs IRSA vs EKS Pod Identity?
21. What happens if an EKS workload cannot authenticate to ECR?

### Security

22. How do you scan ECR images?
23. How do you handle critical CVEs?
24. How do you encrypt ECR images?
25. Why use KMS?
26. How do you prevent image tampering?
27. How do you secure a CI/CD pipeline that pushes to ECR?

### Enterprise

28. How do you implement cross-account ECR access?
29. How do you implement cross-region replication?
30. How do you design ECR for multi-account environments?
31. How do you implement lifecycle policies?
32. How do you reduce ECR storage costs?

### CI/CD

33. How would you integrate ECR with Jenkins?
34. How would you integrate ECR with GitHub Actions?
35. How would you implement build → scan → push → deploy?
36. How do you guarantee that the same image tested in staging is deployed to production?

---

## 28. The Most Important ECR Flow to Remember

For interviews, remember this complete flow:

```text
                         Developer
                             |
                             v
                            Git
                             |
                             v
                          CI/CD
                             |
                             v
                       Docker Build
                             |
                             v
                       Image Created
                             |
                             v
                       Security Scan
                             |
                             v
                     Immutable Tag/Digest
                             |
                             v
                    ECR Authentication
                             |
                             v
                         IAM Check
                             |
                             v
                       Amazon ECR
                             |
                  +----------+----------+
                  |                     |
                  v                     v
                 EKS                   ECS
                  |                     |
                  v                     v
                 Pod                   Task
                  |                     |
                  +----------+----------+
                             |
                             v
                       Application
```

---

## 29. What You Should Be Able to Explain in an Interview

For a senior DevOps role, you should be able to explain this scenario:

> We have a microservices application running on EKS. Developers commit code to GitHub. Jenkins builds Docker images, scans them for vulnerabilities, pushes them to ECR, and then deploys them to EKS. Explain how you would design this securely and reliably.

Your answer should cover:

- Git and CI/CD
- Docker build
- Testing
- Image scanning
- Immutable tags/digests
- IAM least privilege
- ECR authentication
- Repository policies
- ECR lifecycle policies
- EKS image pulling
- Secrets management
- Rollback
- Monitoring
- Cross-account access when required
- Multi-region strategy when required
- `ImagePullBackOff` troubleshooting

---

## 30. ECR Quick Cheat Sheet

| Topic | Remember |
|---|---|
| ECR | AWS managed container registry |
| Repository | Stores container images |
| Authentication | `aws ecr get-login-password` + `docker login` |
| Push | `docker push` |
| Pull | `docker pull` |
| IAM | Controls AWS principal permissions |
| Repository policy | Controls repository access |
| Tag | Human-readable image reference |
| Digest | Exact content-addressed image reference |
| Tag immutability | Helps prevent tag overwrite |
| Scanning | Finds known vulnerabilities |
| Lifecycle policy | Cleans old images |
| KMS | Encryption key management |
| Replication | Copies images across configured regions/accounts |
| EKS | Common consumer of private ECR images |
| ImagePullBackOff | Investigate image, registry, IAM, network, and identity |
| CI/CD | Build → scan → tag → push → deploy |

---

## Final Takeaway

For a senior DevOps interview, think of ECR as more than a place where Docker images are stored.

Think about the entire lifecycle:

```text
                    BUILD
                      |
                      v
                   SCAN
                      |
                      v
              TAG / DIGEST
                      |
                      v
                  AUTHENTICATE
                      |
                      v
                     IAM
                      |
                      v
                    ECR
                      |
             +--------+--------+
             |                 |
             v                 v
          LIFECYCLE        REPLICATION
             |
             v
          SECURITY
             |
             v
            EKS
             |
             v
        IMAGE PULL
             |
             v
          RUNNING
          CONTAINER
```

If you understand **Docker + ECR + EKS + IAM + CI/CD + security + troubleshooting** as one complete workflow, you will be well prepared for the ECR portion of a senior AWS DevOps interview.