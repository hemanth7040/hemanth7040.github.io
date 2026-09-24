---
title: "Terraform Interview Questions & Answers"
date: 2026-09-24
draft: false
tags: ["terraform", "devops", "interview", "aws", "iac"]
categories: ["terraform"]
description: "A complete, tiered Terraform interview question bank with practical answers and code examples, for engineers with 7+ years of experience."
---

> A structured Terraform interview prep guide, organized into three tiers: **must-master** fundamentals and senior scenarios, **strong-understanding** AWS/security/workflow topics, and **basic-awareness** ecosystem topics. Each answer is written to be spoken naturally in an interview, with code examples where they add real value.

---

## 🔴 Tier 1 — Master These

### 1. State & Backend

**1. What is Terraform state?**

A JSON file (`terraform.tfstate`) that maps the resources in your `.tf` config to the real-world objects Terraform created, storing their IDs and current attribute values.

**2. Why does Terraform need a state file?**

Without it, Terraform would have no way to know what it already created, so every `apply` would try to create everything from scratch. State lets it compute a diff between desired and actual infrastructure.

**3. What information is stored in terraform.tfstate?**

Resource metadata, all attributes (including sensitive ones, in plain text), dependency graph data, and output values.

**4. Why shouldn't you commit terraform.tfstate to Git?**

It often contains secrets in plaintext (passwords, keys), and concurrent commits offer no locking, so two people could silently overwrite each other's state.

**5. What happens if you delete the state file?**

Terraform loses all knowledge of existing resources. The next `plan` will propose creating everything again, potentially duplicating live infrastructure.

**6. Local state vs remote state?**

Local state lives on one person's disk (`terraform.tfstate` in the working directory) — no locking, no sharing. Remote state lives in a shared backend (S3, Terraform Cloud, etc.), supports locking, and is accessible to the whole team/CI.

**7. What is a remote backend?**

A configured location (e.g., S3, Azure Blob, GCS, Terraform Cloud) where Terraform stores and retrieves the state file instead of on local disk.

**8. What is an S3 backend?**

Using an AWS S3 bucket to store `terraform.tfstate` remotely.

```hcl
terraform {
  backend "s3" {
    bucket         = "my-tf-state-bucket"
    key            = "prod/network/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-locks"
    encrypt        = true
  }
}
```

**9. Why use S3 for Terraform state?**

Durable, versionable (via S3 versioning, so you can roll back a bad state), supports encryption at rest, and integrates cleanly with DynamoDB for locking.

**10. How do you configure an S3 backend?**

As shown above — a `backend "s3"` block specifying bucket, key (path within the bucket), region, and optionally a `dynamodb_table` for locking and `encrypt = true`.

**11. What is state locking?**

A mechanism that prevents two `apply`/`plan` operations from running against the same state simultaneously, avoiding race conditions and corruption.

**12. Why is state locking required?**

Without it, two concurrent applies could both read the same state, make conflicting writes, and the second write could silently overwrite or corrupt the first person's changes.

**13. How do you implement state locking with AWS?**

Pair the S3 backend with a DynamoDB table (a simple table with a `LockID` primary key). Terraform writes a lock item to that table for the duration of the operation.

**14. What happens if two engineers run terraform apply simultaneously?**

With locking configured, the second `apply` fails immediately with a "state is locked" error rather than proceeding. Without locking, both could write to state and corrupt it.

**15. What happens when Terraform state is locked?**

Any other `plan`/`apply` attempting to use that state is blocked and returns an error identifying who/what holds the lock.

**16. How do you troubleshoot a locked state?**

Check who's actually running an operation (e.g., a colleague, or a stuck CI job) before doing anything. If the lock is confirmed stale (crashed process), only then force-unlock.

**17. What is terraform force-unlock?**

A command (`terraform force-unlock <LOCK_ID>`) that manually removes a lock from state, used when a process crashed and left a stale lock behind.

**18. When would you use terraform force-unlock?**

Only after confirming no other apply/plan is genuinely running — e.g., a CI job was killed mid-run and left the DynamoDB lock item behind.

**19. What happens if Terraform state becomes corrupted?**

Terraform can't reliably reconcile config vs. reality — plans may error out or show nonsensical diffs. You typically restore from a backup/versioned copy.

**20. How do you recover Terraform state?**

Restore the previous version from S3 versioning (if enabled) or a `terraform.tfstate.backup` file, or rebuild it resource-by-resource using `terraform import`.

**21. How do you back up Terraform state?**

Enable S3 bucket versioning on the backend bucket, so every state write is automatically versioned and recoverable.

**22. How do you secure Terraform state?**

Encrypt at rest (S3 SSE), encrypt in transit (TLS), restrict IAM access to the bucket/table to only the CI role and admins, and never commit state to Git.

**23. Why is Terraform state considered sensitive?**

Because it stores full resource attributes in plaintext, including things like database passwords or generated secrets, regardless of `sensitive = true` on the variable.

**24. How do you encrypt Terraform state?**

Enable server-side encryption on the S3 bucket (`encrypt = true` in the backend block, plus bucket-level SSE-KMS for stronger control).

---

### 2. Drift & Import

**25. What is Terraform drift?**

When real-world infrastructure no longer matches what's defined in your Terraform config/state — usually caused by manual changes outside Terraform.

**26. How does Terraform detect drift?**

`terraform plan` refreshes state against the actual cloud resources, then diffs that against your `.tf` config, surfacing any mismatch.

**27. What happens if someone manually modifies an AWS resource?**

The next `plan` shows a diff trying to revert that manual change back to what's declared in code (unless the attribute is under `ignore_changes`).

**28. How do you identify infrastructure drift?**

Run `terraform plan` regularly (or on a schedule in CI) and review any unexpected diffs — that's your drift signal.

**29. How do you resolve drift?**

Either accept the manual change by updating your `.tf` code to match it, or run `apply` to revert infrastructure back to the declared config — decide based on which is "correct."

**30. What happens during terraform plan when drift exists?**

Plan shows a diff (e.g., "~ update in-place" or "will be replaced") reflecting the difference between actual and desired state.

**31. Can Terraform automatically fix drift?**

Only if you run `apply` after detecting it — Terraform doesn't auto-remediate on its own; `plan` only reports drift, it doesn't fix it silently.

**32. How would you handle a production resource that was manually modified?**

Investigate why (was it an emergency fix?), then either codify the change into Terraform or coordinate reverting it via `apply`, communicating with the team first to avoid surprises.

**33. How do you prevent manual infrastructure changes?**

Restrict IAM console/CLI write permissions for engineers, funnel all changes through a CI/CD Terraform pipeline, and use SCPs (Service Control Policies) to block direct changes to critical resources.

**34. What is terraform import?**

A command that brings an already-existing real-world resource under Terraform's management by adding it to the state file, without recreating it.

**35. Why do we use Terraform import?**

To adopt manually-created resources into Terraform management without destroying and recreating them.

**36. How do you import an existing AWS resource?**

```bash
terraform import aws_instance.web i-0123456789abcdef0
```

You must first write a matching `resource` block in your `.tf` file (even a placeholder), then import maps that real resource ID to it.

**37. What happens after importing a resource?**

The resource exists in state, but Terraform doesn't generate the matching `.tf` config for you (in versions before `terraform plan -generate-config-out`) — you must write the config to match, or `plan` will show a diff trying to "correct" attributes you haven't declared.

**38. Does terraform import automatically generate Terraform configuration?**

Classic `import` does not — you write the resource block yourself. Newer Terraform (1.5+) supports `import` blocks with `-generate-config-out` to scaffold config automatically.

**39. How do you bring manually created infrastructure under Terraform management?**

Write the resource block, run `terraform import`, then run `plan` and adjust your config until it shows no diff (meaning your code accurately reflects the resource).

**40. What problems can occur during Terraform import?**

Config drift right after import if your `.tf` block doesn't exactly match every attribute; import only handles one resource at a time, so it's slow for large environments; some resources have quirky/composite import IDs.

**41. How would you import an existing VPC into Terraform?**

```bash
terraform import aws_vpc.main vpc-0abc12345
```
Then flesh out the `aws_vpc.main` resource block's arguments (CIDR, tags, DNS settings) until `plan` shows zero changes.

---

### 3. Modules

**42. What is a Terraform module?**

A reusable, self-contained package of `.tf` resource files that you can call with different inputs to create the same set of infrastructure repeatedly.

**43. Why do we use modules?**

To avoid duplicating the same resource blocks across environments/projects, enforce consistency, and encapsulate complexity behind a clean interface (inputs/outputs).

**44. Root module vs child module?**

The root module is the top-level directory where you run `terraform apply`. A child module is any module called from within it via a `module` block.

**45. How do you create a reusable module?**

Put resource blocks in their own directory with `variables.tf` (inputs), `main.tf` (resources), and `outputs.tf` (exposed values), then call it via `source`.

**46. How do you pass variables to a module?**

```hcl
module "network" {
  source   = "./modules/network"
  vpc_cidr = "10.0.0.0/16"
  az_count = 2
}
```

**47. How do you expose outputs from a module?**

Define `output` blocks inside the module, then reference them from the caller as `module.network.vpc_id`.

**48. How do modules communicate with each other?**

Through outputs — one module's output is passed as another module's input variable in the root module.

**49. How do you version Terraform modules?**

Using Git tags (`source = "git::https://example.com/module.git?ref=v1.2.0"`) or a version constraint when pulling from the Terraform Registry (`version = "~> 3.0"`).

**50. How do you handle module dependencies?**

Terraform infers them automatically when one module's output feeds another module's input; for anything implicit, use `depends_on` at the module block level.

**51. How do you structure Terraform modules for an organization?**

A common pattern: a `modules/` directory with generic building blocks (network, compute, database), and per-environment root directories (`envs/dev`, `envs/staging`, `envs/prod`) that call those modules with different variables.

**52. How do you design reusable VPC modules?**

Parameterize CIDR ranges, AZ count, and whether NAT gateways are needed; output the VPC ID and subnet IDs so other modules (compute, database) can consume them.

**53. How do you design reusable EKS modules?**

Parameterize cluster version, node group instance types/counts, and networking inputs (VPC/subnet IDs from the network module); output the cluster endpoint and OIDC provider ARN for IAM roles for service accounts.

**54. How do you prevent breaking changes in modules?**

Semantic versioning (major version bump for breaking changes), maintain a changelog, and avoid renaming/removing variables without a deprecation period.

**55. How do you maintain multiple versions of a module?**

Consumers pin exact or constrained versions (`version = "2.3.0"`); the module repo tags releases so old consumers keep working while new ones adopt newer tags.

**56. Terraform Registry vs private module registry?**

The public Registry hosts community/vendor modules anyone can use; a private registry (Terraform Cloud/Enterprise, or a Git-based private source) hosts internal, company-specific modules with access control.

**57. How would you create a private Terraform module repository?**

Host it in an internal Git repo and reference it with a Git source URL and `ref`, or publish it to a private registry (e.g., Terraform Cloud's private registry) for versioned, discoverable consumption.

---

### 4. count & for_each

**58. What is count?**

A meta-argument that creates N copies of a resource, indexed numerically (`count.index`).

**59. What is for_each?**

A meta-argument that creates one resource per item in a map or set of strings, keyed by that value rather than a numeric index.

**60. count vs for_each?**

`count` uses numeric indexes, so removing a middle item shifts indexes and can cause unrelated resources to be destroyed/recreated. `for_each` uses stable string keys, so removing one item only affects that one resource.

**61. When would you use count?**

For simple, identical resources where order/identity doesn't matter much (e.g., a fixed number of nearly-identical EC2 instances at the start of a small project).

**62. When would you use for_each?**

Whenever resources have distinct identities (subnets per AZ, IAM users by name) or the list might change over time — the stability of the key matters.

**63. What happens when you remove an item from a count list?**

All items after the removed index shift down by one, so Terraform destroys and recreates every resource after that index — not just the removed one.

**64. What happens when you remove an item from a for_each map?**

Only the resource matching that specific key is destroyed; every other resource is untouched.

**65. Can you use both count and for_each on the same resource?**

No — a resource block can use one or the other, not both simultaneously.

**66. How do you reference a resource created using count?**

`aws_instance.web[0].id`, `aws_instance.web[count.index].id`, or `aws_instance.web[*].id` for all of them.

**67. How do you reference a resource created using for_each?**

`aws_instance.web["app1"].id`, or `aws_instance.web` as a whole is a map keyed by your `for_each` keys.

**68. How would you create multiple AWS subnets using for_each?**

```hcl
variable "subnets" {
  type = map(string)
  default = {
    "public-a"  = "10.0.1.0/24"
    "public-b"  = "10.0.2.0/24"
  }
}

resource "aws_subnet" "this" {
  for_each          = var.subnets
  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value
  availability_zone = each.key == "public-a" ? "us-east-1a" : "us-east-1b"

  tags = { Name = each.key }
}
```

---

### 5. Lifecycle

**69. What is the Terraform lifecycle block?**

A nested block inside a resource that customizes how Terraform manages that resource's create/update/destroy behavior.

```hcl
resource "aws_instance" "web" {
  # ...
  lifecycle {
    create_before_destroy = true
    prevent_destroy        = true
    ignore_changes          = [tags]
  }
}
```

**70. What is create_before_destroy?**

Tells Terraform to provision the replacement resource first, then destroy the old one — avoiding downtime during a forced replacement.

**71. What is prevent_destroy?**

A safety flag that makes Terraform error out and refuse to destroy that resource, even if `apply` or `destroy` would otherwise remove it — commonly used on databases.

**72. What is ignore_changes?**

Tells Terraform to ignore diffs on specific attributes going forward, even if they drift from the declared config (e.g., ignore tags added by an auto-tagging Lambda).

**73. When would you use ignore_changes?**

When another process legitimately manages certain attributes outside Terraform — e.g., autoscaling changes `desired_capacity`, or a tagging automation adds tags you don't want reverted every apply.

**74. What are the risks of ignore_changes?**

It can mask real drift you actually care about, and if overused, your `.tf` code stops being a reliable source of truth for those attributes.

**75. How would you prevent Terraform from accidentally destroying a production database?**

Add `lifecycle { prevent_destroy = true }` to the RDS resource, and additionally require manual approval gates in the CI/CD pipeline before any apply touching production.

**76. How would you achieve zero-downtime infrastructure changes using Terraform?**

Use `create_before_destroy` on the resource lifecycle, combine with load balancer health checks so new instances are only added to rotation once healthy, and use `for_each`-based deployments so unrelated resources aren't disturbed.

---

### 6. Providers

**77. What is a Terraform provider?**

A plugin that translates Terraform's HCL configuration into API calls for a specific platform (AWS, Azure, GCP, Kubernetes, etc.).

**78. How does Terraform communicate with AWS?**

Through the `aws` provider plugin, which calls the AWS API using credentials you supply (env vars, shared credentials file, or an assumed IAM role).

**79. What is provider versioning?**

Pinning which version(s) of a provider plugin Terraform should use, to avoid unexpected breaking changes from provider updates.

**80. What is required_providers?**

A block inside `terraform { }` declaring which providers and version constraints the configuration needs.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

**81. What is a provider alias?**

A named additional instance of a provider, letting you use the same provider type with different configurations (e.g., different regions or accounts) within one config.

**82. Why would you use provider aliases?**

To manage resources across multiple regions or accounts in a single Terraform run, e.g., replicating an S3 bucket to a second region.

**83. How do you deploy resources to multiple AWS regions?**

```hcl
provider "aws" {
  alias  = "us_west"
  region = "us-west-2"
}

resource "aws_instance" "west_app" {
  provider = aws.us_west
  # ...
}
```

**84. How do you manage multiple AWS accounts?**

Use separate provider blocks with different `assume_role` configurations (one per account), or separate state/backends entirely per account for stronger isolation.

**85. How would you use different AWS providers for dev and production?**

Typically via separate root modules/state per environment, each with its own provider block pointing at the respective account/role, rather than mixing both in one config.

**86. How do you assume an IAM role from Terraform?**

```hcl
provider "aws" {
  assume_role {
    role_arn = "arn:aws:iam::123456789012:role/terraform-exec"
  }
}
```

**87. How do you authenticate Terraform with AWS?**

Preferably via IAM roles (assumed role, or an EC2/CI instance profile) rather than static keys; alternatively environment variables or the shared credentials file for local dev.

**88. What is the difference between AWS access keys and IAM roles?**

Access keys are long-lived static credentials that must be stored/rotated manually; IAM roles provide short-lived, automatically-rotated temporary credentials — much safer, especially in CI.

**89. How do you avoid hardcoding AWS credentials?**

Never put keys in `.tf` files; use IAM roles/instance profiles, environment variables, or a secrets manager, and rely on the provider's default credential chain.

---

### 7. Terraform + CI/CD

**90. How do you integrate Terraform with Jenkins?**

A pipeline with stages that run `terraform init`, `fmt`, `validate`, `plan` (posting the plan output for review), and a gated `apply` stage requiring manual approval for production.

**91. How do you integrate Terraform with GitHub Actions?**

Use the `hashicorp/setup-terraform` action, run `init`/`plan` on pull requests (commenting the plan output), and run `apply` on merge to main with environment protection rules for production.

**92. What stages would you include in a Terraform pipeline?**

`fmt` (style check) → `validate` (syntax) → `plan` (preview) → manual approval (for prod) → `apply`.

**93. Where should terraform fmt run?**

Early — as a pre-commit hook or the first CI stage, to catch style issues before anything else runs.

**94. Where should terraform validate run?**

Right after `fmt`, before `plan`, to catch syntax/type errors cheaply and quickly.

**95. Where should terraform plan run?**

On every pull request, so reviewers can see the exact infrastructure diff before merging.

**96. Where should terraform apply run?**

Only after merge to the main branch, ideally gated behind a manual approval step for production environments.

**97. How do you require approval before production apply?**

Use environment protection rules (GitHub Actions environments, or a manual approval stage in Jenkins/GitLab) that pause the pipeline until an authorized approver clicks "approve."

**98. How do you prevent unauthorized Terraform changes?**

Restrict who can merge to main / trigger the apply job, enforce PR reviews, and give only the CI service role actual apply permissions in the cloud account (not individual engineers).

**99. How do you handle concurrent Terraform pipelines?**

Use state locking (so a second run blocks/fails safely) and pipeline-level concurrency controls (e.g., GitHub Actions `concurrency` groups) to queue rather than run in parallel.

**100. How do you handle a failed Terraform pipeline?**

Inspect the error, don't blindly retry; run `plan` again once the underlying issue is fixed to confirm the intended changes before re-running `apply`.

**101. How do you store Terraform secrets in CI/CD?**

In the CI platform's encrypted secrets store (GitHub Secrets, Jenkins Credentials), injected as environment variables at runtime — never committed to the repo.

**102. How do you securely authenticate AWS from GitHub Actions?**

Use OIDC federation (GitHub's OIDC provider assuming an AWS IAM role) instead of long-lived access keys stored as secrets.

**103. How do you implement Terraform plan on pull requests?**

A workflow triggered on `pull_request` that runs `terraform plan` and posts the output as a PR comment for reviewers.

**104. How do you prevent developers from directly running terraform apply?**

Don't give individual engineers apply-level cloud credentials at all — only the CI pipeline's service role has them, so applying is only possible through the pipeline.

**105. How do you implement separate pipelines for dev, stage, and production?**

Separate workflow files or pipeline stages per environment, each pointing at its own backend/state and cloud credentials, often with progressively stricter approval gates.

---

### 8. Terraform Troubleshooting

**106. terraform plan is showing unexpected changes. How do you troubleshoot?**

Check if a provider version bump changed defaults, if someone made a manual change (drift), or if the config itself changed. Run `plan` with `-out` and inspect the diff attribute-by-attribute.

**107. terraform apply fails. What do you check?**

The actual error message first — IAM permissions, quota limits, invalid arguments, or a dependency ordering issue — then whether partial resources were created.

**108. Terraform says the state is locked. What do you do?**

Confirm no one else (or no CI job) is genuinely running an operation, then `force-unlock` only if the lock is confirmed stale.

**109. Terraform cannot authenticate with AWS. How do you troubleshoot?**

Verify credentials are set (env vars, profile, or assumed role), check IAM permissions for the action being performed, and confirm the region is correct.

**110. Terraform cannot find a resource. What do you check?**

Whether it exists in state (`terraform state list`), whether it's in the right region/account, and whether the resource was deleted outside Terraform (drift).

**111. Terraform wants to destroy an existing resource. What do you investigate?**

Whether a required argument change forces replacement, whether the resource was removed from `.tf` code by mistake, or whether it's genuinely no longer needed.

**112. Terraform keeps recreating the same resource. Why?**

Usually a diff caused by a computed value that changes every apply (e.g., a timestamp), or an attribute forcing replacement that's being set inconsistently (e.g., an unstable data source).

**113. Terraform is showing perpetual diffs. How do you troubleshoot?**

Look for attributes with default values set differently on the provider side than in your config, or a value the cloud API normalizes differently than what you wrote (common with JSON policy documents — use `jsonencode` consistently).

**114. Terraform resource exists in AWS but not in state. What do you do?**

`terraform import` it into state so Terraform stops trying to recreate it and instead manages the existing one.

**115. Resource exists in state but not in AWS. What do you do?**

Run `terraform apply` — Terraform will try to recreate it since it thinks it should exist; or, if it was intentionally deleted, remove it from state instead using `terraform state rm`.

**116. Terraform provider initialization fails. How do you troubleshoot?**

Check network access to the provider registry, version constraints in `required_providers`, and whether `.terraform.lock.hcl` is stale (delete and re-`init` if needed).

**117. Terraform module isn't downloading. What do you check?**

Source URL correctness, Git/network access, authentication for private repos, and whether the version/ref tag actually exists.

**118. Terraform backend initialization fails. How do you troubleshoot?**

Verify the backend bucket/table exists and you have permission to access it, and that the backend config block has correct region/credentials.

**119. Terraform apply fails halfway through. What do you do?**

Run `plan` again — state reflects what was actually created — investigate and fix the root cause of the failure, then re-`apply`; Terraform will only touch what's still missing.

**120. How do you safely troubleshoot Terraform in production?**

Always run `plan` first and review the diff carefully, never blindly re-run `apply`, and make changes in a maintenance window with rollback steps ready for anything stateful.

---

### 9. Senior Real-World Scenarios

**121. Terraform wants to destroy a production database. What will you do?**

Stop and investigate why — likely a forced-replacement attribute change or someone removed it from code. Add `prevent_destroy` proactively, and if the destroy is truly intended, ensure a backup/snapshot exists first.

**122. Two engineers run Terraform at the same time. How do you prevent conflicts?**

Remote backend with state locking (S3 + DynamoDB) ensures the second run is blocked until the first completes.

**123. Someone manually changes a production security group. What happens during the next Terraform plan?**

Plan detects the drift and proposes reverting the security group back to what's declared in code (unless that attribute is in `ignore_changes`).

**124. Terraform state is accidentally deleted. How do you recover?**

Restore from S3 bucket versioning if enabled; otherwise, rebuild the state by re-importing every resource one by one — tedious but doable.

**125. A Terraform apply fails halfway through. How do you recover?**

Run `plan` to see what state now reflects, diagnose and fix the actual cause of failure, then `apply` again — idempotency means it only creates what's missing.

**126. You have 50 AWS accounts. How would you design your Terraform architecture?**

A shared "landing zone" or "account factory" module to provision each account consistently, separate state per account, centralized remote backend (e.g., one state bucket per account or a dedicated management account), and provider aliases/assumed roles per account.

**127. You have dev, stage, and production. How would you structure your repository?**

Either separate directories per environment (`envs/dev`, `envs/stage`, `envs/prod`) each with their own backend/state calling shared modules, or Terraform workspaces if the team is comfortable with the extra discipline required.

**128. How would you design Terraform for multiple AWS regions?**

Provider aliases per region within one config for cross-region resources (e.g., replication), or fully separate state per region for independent regional stacks.

**129. How would you design a reusable VPC module?**

— see Q52.

**130. How would you design a reusable EKS module?**

— see Q53.

**131. How would you implement Terraform through GitHub Actions?**

— see Q91.

**132. How would you ensure production changes require approval?**

— see Q97.

**133. How would you prevent developers from modifying production infrastructure directly?**

— see Q104.

**134. How would you detect and remediate Terraform drift?**

— see Q25–33.

**135. How would you migrate existing manually created AWS infrastructure to Terraform?**

Inventory the existing resources, write matching `.tf` blocks, `terraform import` each one, then iterate on the config until `plan` shows zero diff.

**136. How would you migrate Terraform state from one backend to another?**

Change the `backend` block to the new target, then run `terraform init -migrate-state`, which copies state over and prompts for confirmation.

**137. How would you rename a Terraform resource without destroying the AWS resource?**

Use a `moved` block (Terraform 1.1+) or `terraform state mv`:

```hcl
moved {
  from = aws_instance.web
  to   = aws_instance.app_server
}
```
or
```bash
terraform state mv aws_instance.web aws_instance.app_server
```
Either tells Terraform "this is the same resource, just renamed," avoiding destroy/recreate.

**138. How would you move a resource from one module to another?**

```bash
terraform state mv module.old_module.aws_instance.web module.new_module.aws_instance.web
```
Or the `moved` block equivalent, pointing from the old module path to the new one.

**139. How would you upgrade a Terraform module without breaking production?**

Test the new version in dev/staging first, review the module's changelog for breaking changes, bump the version constraint, run `plan` to review the diff carefully, and roll out gradually (e.g., dev → stage → prod).

**140. How would you perform a zero-downtime infrastructure change?**

— see Q76.

---

## 🟠 Tier 2 — Strong Understanding

### AWS Infrastructure

**141. How would you create a VPC using Terraform?**

```hcl
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = { Name = "main-vpc" }
}
```

**142. How would you design public and private subnets?**

Public subnets route `0.0.0.0/0` to an Internet Gateway (for internet-facing resources like ALBs); private subnets route `0.0.0.0/0` to a NAT Gateway (for outbound-only resources like app servers and databases).

**143. How would you create an Internet Gateway?**

```hcl
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}
```

**144. How would you create a NAT Gateway?**

```hcl
resource "aws_eip" "nat" { domain = "vpc" }

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public.id
}
```

**145. How would you configure route tables?**

One route table for public subnets pointing `0.0.0.0/0` to the Internet Gateway, another for private subnets pointing `0.0.0.0/0` to the NAT Gateway, each associated with their respective subnets.

**146. How would you create security groups?**

```hcl
resource "aws_security_group" "web" {
  vpc_id = aws_vpc.main.id
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

**147. How would you create an ALB?**

Define `aws_lb`, `aws_lb_target_group`, and `aws_lb_listener` resources, attaching your EC2/ECS targets to the target group and the listener to the load balancer.

**148. How would you create an EC2 instance?**

```hcl
resource "aws_instance" "app" {
  ami           = "ami-0abcd1234"
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.private.id
}
```

**149. How would you create an IAM role?**

```hcl
resource "aws_iam_role" "app_role" {
  name = "app-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Effect    = "Allow"
      Principal = { Service = "ec2.amazonaws.com" }
    }]
  })
}
```

**150. How would you create an EKS cluster?**

Use `aws_eks_cluster` with a VPC/subnet configuration and an IAM role granting EKS the necessary permissions, typically via the community `terraform-aws-modules/eks/aws` module rather than raw resources.

**151. How would you create an EKS node group?**

`aws_eks_node_group`, referencing the cluster name, a node IAM role, subnet IDs, and scaling config (min/max/desired size).

**152. How would you configure EBS CSI using Terraform?**

Install the EBS CSI driver as an EKS add-on (`aws_eks_addon`) and attach an IAM role (via IRSA) granting it permission to create/attach EBS volumes.

**153. How would you create an S3 bucket securely?**

Enable default encryption, block public access, enable versioning, and attach a restrictive bucket policy.

```hcl
resource "aws_s3_bucket" "data" {
  bucket = "my-secure-bucket"
}

resource "aws_s3_bucket_public_access_block" "data" {
  bucket                  = aws_s3_bucket.data.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

**154. How would you create an RDS database?**

```hcl
resource "aws_db_instance" "main" {
  engine            = "postgres"
  instance_class    = "db.t3.medium"
  allocated_storage = 20
  db_name           = "appdb"
  username          = var.db_username
  password          = var.db_password
  skip_final_snapshot = false
}
```

**155. How do you manage AWS resources across multiple accounts?**

Provider aliases with `assume_role` per account, or fully separate state/pipelines per account for stronger isolation — the latter is more common at scale.

### Security

**156. How do you manage secrets in Terraform?**

Fetch them at runtime from a secrets manager (AWS Secrets Manager, Vault) via a data source rather than hardcoding, mark variables `sensitive = true`, and encrypt the state backend.

**157. Should passwords be stored in .tfvars?**

No — `.tfvars` files are often committed to Git and aren't inherently secret; passwords should come from a secrets manager or environment variables injected at runtime.

**158. What does sensitive = true do?**

Prevents Terraform from printing that value in CLI output (`plan`/`apply` logs); it does not encrypt or hide it from the state file.

**159. Does sensitive = true encrypt the value?**

No — it's purely a display/output redaction; the value is still stored in plaintext in the state file.

**160. How do you prevent secrets from appearing in Terraform state?**

You largely can't avoid it entirely if Terraform manages the resource — the real mitigation is encrypting the state backend and tightly restricting who can read it.

**161. How do you use AWS Secrets Manager with Terraform?**

```hcl
data "aws_secretsmanager_secret_version" "db" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db.secret_string
}
```

**162. How do you use SSM Parameter Store?**

Similarly, via `data "aws_ssm_parameter"`, often used for less sensitive config values or where Secrets Manager's cost isn't justified.

**163. How do you scan Terraform code for security issues?**

Run static analysis tools like `tfsec` or `Checkov` in CI, failing the pipeline on high-severity findings (e.g., an S3 bucket without encryption).

**164. What is Checkov?**

An open-source static analysis tool that scans Terraform (and other IaC) for security and compliance misconfigurations.

**165. What is tfsec?**

A static analysis security scanner specifically for Terraform, checking for common misconfigurations (open security groups, unencrypted storage, etc.).

**166. How do you enforce security policies in Terraform?**

Policy-as-code tools like Sentinel (Terraform Cloud/Enterprise) or Open Policy Agent, which can block an `apply` if it violates a defined rule (e.g., "no public S3 buckets").

**167. How do you implement least privilege for Terraform?**

Give the CI/CD execution role only the specific IAM permissions needed for the resources it manages, scoped by resource ARN where possible, rather than broad admin access.

### Workspaces

**168. What are Terraform workspaces?**

A built-in feature letting you maintain multiple named state files from the same configuration directory (e.g., `dev`, `staging`, `prod` workspaces).

**169. Why use Terraform workspaces?**

Quick way to spin up parallel environments from identical code without duplicating directories.

**170. Workspace vs separate directory?**

Workspaces share the same `.tf` code and just swap state; separate directories can have genuinely different code/variables per environment and make the active environment explicit in the file path, reducing accidental cross-environment applies.

**171. Workspace vs separate state file?**

A workspace *is* a way of having separate state files, but tied to one shared codebase; a fully separate directory approach naturally gives separate state too, plus more isolation.

**172. When should you avoid Terraform workspaces?**

For environments with meaningfully different configurations (not just different variable values), or when the risk of an engineer forgetting which workspace is selected and applying to the wrong one is too high — common in production-sensitive setups.

**173. How do you manage dev/stage/prod?**

Either via workspaces with a shared config and per-workspace `.tfvars`, or (more common at senior/production level) separate directories per environment with separate backends.

**174. Would you use workspaces for production environments? Why?**

Many senior engineers avoid them for production specifically because the workspace context isn't visually obvious in the terminal/file path, raising the risk of an accidental production apply; separate directories make the target environment explicit.

### Variables & Functions

**175. What are Terraform variables?**

Named inputs (`variable` blocks) that parameterize your configuration so the same code can be reused with different values.

**176. What are local values?**

`locals` — named expressions computed once and reused within a module, useful for reducing repetition (not exposed as inputs).

**177. Variables vs locals?**

Variables are inputs supplied from outside the module; locals are internal computed values derived from variables/resources, not set externally.

**178. What are Terraform outputs?**

`output` blocks that expose values from a module (or root config) for use by other modules, or for display after `apply`.

**179. What are Terraform variable types?**

`string`, `number`, `bool`, `list`, `map`, `set`, `object`, `tuple`, and `any`.

**180. What are complex variable types?**

The structured types: `list`, `map`, `set`, `object`, and `tuple`, as opposed to simple scalars like `string`/`number`/`bool`.

**181. What is a map?**

A collection of key-value pairs, all values of the same type, e.g., `map(string)`.

**182. What is a list?**

An ordered collection of values of the same type, accessed by numeric index.

**183. What is a set?**

An unordered collection of unique values, useful when order doesn't matter and duplicates should be disallowed.

**184. What is an object?**

A structured type with named attributes of potentially different types, e.g., `object({ name = string, size = number })`.

**185. What is a tuple?**

An ordered sequence of elements where each position can have a different type, defined explicitly, e.g., `tuple([string, number, bool])`.

**186. What are Terraform functions?**

Built-in functions for transforming values inside expressions — e.g., `lookup()`, `merge()`, `join()`, `length()`, `jsonencode()`.

**187. Which Terraform functions have you used?**

Commonly: `lookup()` and `try()` for safe map access, `merge()` for combining tag maps, `jsonencode()` for IAM policies, `element()`/`length()` with `count`, `cidrsubnet()` for subnet math.

**188. What are conditional expressions?**

Ternary-style expressions: `condition ? true_val : false_val`, e.g., `var.env == "prod" ? "t3.large" : "t3.micro"`.

**189. What are Terraform expressions?**

Any value-producing construct in HCL — references, function calls, conditionals, and operators used to compute argument values.

**190. What is a dynamic block?**

A way to generate repeated nested blocks (like multiple `ingress` rules in a security group) from a list/map, instead of writing each one manually.

```hcl
dynamic "ingress" {
  for_each = var.allowed_ports
  content {
    from_port   = ingress.value
    to_port     = ingress.value
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Repository Architecture

**191. How would you structure a large Terraform repository?**

`modules/` for reusable building blocks, `envs/{dev,stage,prod}/` for environment-specific root configs that call those modules, each with its own backend and `.tfvars`.

**192. Monorepo vs separate repositories for Terraform?**

A monorepo keeps everything discoverable and easy to cross-reference but can get large and needs careful path-based CI triggers; separate repos give stronger team-level ownership/access control but complicate cross-referencing shared modules.

**193. How do you separate environments?**

Separate directories (preferred at senior level) or workspaces, each with isolated state/backend so a mistake in one environment can't touch another.

**194. How do you separate infrastructure by application/team?**

Separate state files per application/team ("micro state" approach), often with a shared network/foundation layer that individual app teams consume via remote state data sources.

**195. How do you manage Terraform module versions?**

Git tags/semantic versioning, with consumers pinning specific version constraints and upgrading deliberately after testing.

**196. How do you manage Terraform provider versions?**

`required_providers` version constraints plus the `.terraform.lock.hcl` file, which pins exact provider versions/checksums for reproducible runs.

**197. How do you manage Terraform upgrades?**

Read the changelog/upgrade guide, test in a non-production environment first, and roll out gradually; use `tfenv` or similar to manage multiple installed CLI versions during migration.

**198. How do you test Terraform code?**

`terraform validate` for syntax, `plan` review for logical correctness, static analysis (`tfsec`/`Checkov`) for security, and tools like Terratest for actual integration testing against real infrastructure.

**199. How do you perform Terraform security scanning?**

Integrate `tfsec` or `Checkov` into CI, failing builds on critical findings, run against every PR before merge.

**200. How do you review Terraform changes?**

Require `plan` output on every PR, have a reviewer read the diff carefully (especially anything showing "destroy" or "force replacement"), and require approval before merge/apply for production.

---

## 🟢 Tier 3 — Basic Awareness

### Terraform Cloud / HCP Terraform

**201. What is Terraform Cloud/HCP Terraform?**

HashiCorp's managed service for running Terraform, providing remote state storage, remote plan/apply execution, a private module registry, and team/policy controls.

**202. Terraform Cloud vs local Terraform?**

Terraform Cloud runs plans/applies on HashiCorp's infrastructure (not your laptop/CI runner), manages state and locking automatically, and adds collaboration features like PR integration and Sentinel policies.

**203. What are remote runs?**

Plan/apply operations executed on Terraform Cloud's own runners rather than locally, giving consistent execution environment and centralized logs.

**204. What is HCP Terraform workspace?**

A Terraform Cloud concept distinct from CLI workspaces — a top-level container binding a specific config, variable set, and state together, roughly like a project.

**205. How does HCP Terraform manage state?**

It stores and versions state automatically, with built-in locking, so teams don't need to configure their own S3/DynamoDB backend.

**206. How does HCP Terraform integrate with VCS?**

It connects directly to GitHub/GitLab/Bitbucket, automatically triggering `plan` on PRs and `apply` on merge, without needing a separate CI pipeline.

**207. What are Terraform Cloud run tasks?**

Integration points that let external tools (security scanners, cost estimators) inject checks into the plan/apply workflow before it's allowed to proceed.

### Policy-as-Code

**208. What is policy-as-code?**

Defining infrastructure governance rules (e.g., "no public S3 buckets," "must have cost-center tag") as code that's automatically enforced during the Terraform workflow.

**209. Why is policy-as-code useful with Terraform?**

It catches violations automatically before `apply`, rather than relying on manual review, and scales consistently across every team/project.

**210. What is HashiCorp Sentinel?**

HashiCorp's policy-as-code framework, integrated into Terraform Cloud/Enterprise, that can block runs violating defined policies.

**211. What is Open Policy Agent (OPA)?**

An open-source, general-purpose policy engine (using the Rego language) that can be used to enforce policies against Terraform plans, independent of HashiCorp's ecosystem.

**212. How can you prevent insecure Terraform configurations?**

Combine static analysis (`tfsec`/`Checkov`) with policy-as-code (Sentinel/OPA) gates in the pipeline, so insecure plans are blocked automatically.

### Terraform Testing

**213. How do you test Terraform code?**

— see Q198.

**214. What is terraform validate?**

A command that checks configuration syntax and internal consistency (e.g., referencing an undefined variable) without contacting any cloud provider.

**215. What is Terraform testing?**

The practice of verifying Terraform configurations behave as expected — ranging from static validation to full integration tests that actually provision and check real resources.

**216. What is Terratest?**

A Go testing library for writing integration tests that apply real Terraform configs, assert on the resulting infrastructure, then tear it down.

**217. What is Checkov?**

— see Q164.

**218. What is TFLint?**

A linter for Terraform that catches provider-specific errors and best-practice violations beyond what `terraform validate` checks.

**219. What is tfsec?**

— see Q165.

**220. How would you integrate Terraform testing into CI/CD?**

Run `fmt`/`validate`/`TFLint`/`tfsec` on every PR as fast checks, and run Terratest suites (which actually provision resources) on a schedule or before major releases due to their cost/time.

### Terraform Ecosystem

**221. What is the Terraform Registry?**

HashiCorp's public catalog of community and vendor-published providers and modules.

**222. What are private Terraform registries?**

Internally-hosted or Terraform Cloud-hosted registries for publishing and versioning your organization's own modules/providers with access control.

**223. What are Terraform providers?**

— see Q77.

**224. How do you find and evaluate a Terraform provider?**

Check the Registry for official/verified badges, review download counts and maintenance activity (recent commits/releases), and check open issues for known problems.

**225. How do you manage provider upgrades?**

Bump the version constraint deliberately, review the provider's changelog for breaking changes, test in a non-production environment, and regenerate the lock file.

**226. What are Terraform modules from the Registry?**

Pre-built, versioned modules published publicly (e.g., `terraform-aws-modules/vpc/aws`) that you can call directly instead of writing common infrastructure patterns from scratch.

**227. What is Infrastructure as Code security?**

The practice of applying security scanning, least-privilege access, and policy enforcement to infrastructure code itself, catching misconfigurations before they're ever deployed.

**228. What is policy enforcement in Terraform?**

Automatically blocking non-compliant infrastructure changes using tools like Sentinel or OPA, integrated into the plan/apply pipeline, rather than relying on after-the-fact audits.

---

*Good luck with your interview! Practice explaining these out loud, not just reading them — interviewers are grading how clearly you reason through trade-offs, not whether you recite definitions.*