---
title: "Git for DevOps Engineers – The Complete Guide"
date: 2026-04-14
draft: false
tags: ["git", "devops", "version-control", "ci-cd", "devsecops"]
categories: ["git"]
description: "A comprehensive guide covering what Git is, why it matters, when to use it, real-world DevOps use cases, and every mandatory Git command every DevOps Engineer must know — with clear explanations and practical examples."

---

## What is Git?

**Git** is a free and open-source **distributed version control system** (DVCS). It tracks changes in files over time, allowing you and your team to collaborate on code, roll back mistakes, and maintain a complete history of every change ever made.

> Think of Git as a time machine for your code. Every save point (commit) is a snapshot you can always return to.

Git was created by **Linus Torvalds** in 2005 (yes, the same person who created Linux). Today it is the industry standard used by virtually every software team on the planet.

---

## Why Git?

As a DevOps engineer, you sit at the intersection of development and operations. Git is your single source of truth for everything — application code, infrastructure-as-code (IaC), CI/CD pipelines, configuration files, Helm charts, Kubernetes manifests, and more.

Here is why Git is non-negotiable:

| Reason | Explanation |
|---|---|
| **Collaboration** | Multiple engineers can work on the same codebase simultaneously without overwriting each other's work. |
| **History & Audit Trail** | Every change is recorded — who made it, when, and why. Critical for compliance and debugging. |
| **Rollback** | Made a bad deployment? Git lets you revert to any previous state within seconds. |
| **Branching** | Isolate features, hotfixes, and experiments in separate branches without touching production. |
| **GitOps** | The entire infrastructure (Terraform, Ansible, K8s manifests) can be managed and deployed through Git workflows. |
| **CI/CD Integration** | Every major pipeline tool (Jenkins, GitHub Actions, GitLab CI, ArgoCD) is triggered by Git events. |

---

## When to Use Git?

Git should be used **always and for everything** in a DevOps context. Specifically:

- Writing or modifying **application code**
- Creating or updating **Terraform / CloudFormation** infrastructure templates
- Managing **Kubernetes manifests** and Helm charts
- Writing **Ansible playbooks** or Chef/Puppet recipes
- Configuring **CI/CD pipeline** files (`.github/workflows`, `Jenkinsfile`, `.gitlab-ci.yml`)
- Tracking **Dockerfile** and **docker-compose** changes
- Managing **configuration files** for servers and services
- Documenting infrastructure with **README** and runbooks

> **Rule of thumb:** If it's a text file that can change over time, it belongs in Git.

---

## Core Concepts Before Commands

Before jumping into commands, understand these key building blocks:

| Term | Meaning |
|---|---|
| **Repository (Repo)** | The project folder tracked by Git. Contains all files and their entire history. |
| **Commit** | A saved snapshot of your changes at a point in time. |
| **Branch** | An independent line of development. `main` is the default production branch. |
| **Remote** | A version of your repo hosted on a server (GitHub, GitLab, Bitbucket, Azure Repos). |
| **Staging Area (Index)** | A preparation area where you decide what goes into your next commit. |
| **HEAD** | A pointer to the current commit you are working from. |
| **Merge** | Combining changes from one branch into another. |
| **Rebase** | Replaying commits from one branch on top of another — creates a cleaner history. |

---

## Mandatory Git Commands for DevOps Engineers

---

### 1. `git init` — Initialize a Repository

**What it does:** Creates a brand new empty Git repository in the current directory.

```bash
git init
```

**Example — Start tracking an Ansible project:**

```bash
mkdir ansible-playbooks
cd ansible-playbooks
git init
# Output: Initialized empty Git repository in /ansible-playbooks/.git/
```

> Use this when starting a new project from scratch locally.

---

### 2. `git clone` — Copy a Remote Repository

**What it does:** Downloads a complete copy of a remote repository (including all history and branches) to your local machine.

```bash
git clone <repository-url>
git clone <repository-url> <target-folder-name>
```

**Example — Clone a Terraform project from GitHub:**

```bash
git clone https://github.com/org/terraform-infra.git
git clone https://github.com/org/terraform-infra.git my-infra
```

> This is almost always the first command you run when joining a project.

---

### 3. `git status` — Check What's Changed

**What it does:** Shows the current state of your working directory — which files are modified, staged, or untracked.

```bash
git status
```

**Example output:**

```
On branch feature/add-nginx-config
Changes not staged for commit:
  modified:   nginx/nginx.conf

Untracked files:
  nginx/ssl.conf
```

> Run this constantly. It's your GPS — it tells you exactly where you are.

---

### 4. `git add` — Stage Changes

**What it does:** Moves changes from your working directory to the staging area, preparing them for a commit.

```bash
git add <filename>         # Stage a specific file
git add .                  # Stage all changed files
git add -p                 # Stage changes interactively (hunk by hunk)
```

**Example — Stage only the Dockerfile:**

```bash
git add Dockerfile
```

**Example — Stage everything:**

```bash
git add .
```

> Always review what you're staging before committing, especially in production pipelines.

---

### 5. `git commit` — Save a Snapshot

**What it does:** Records the staged changes into the repository history with a descriptive message.

```bash
git commit -m "your message here"
git commit -am "message"      # Stage tracked files and commit in one step
```

**Example — Commit a Kubernetes deployment update:**

```bash
git commit -m "feat: update nginx deployment replicas to 3 for HA"
```

**Best practice for commit messages — follow Conventional Commits:**

```
feat: add Redis cache configuration
fix: correct wrong port in service manifest
chore: update .gitignore for terraform state files
docs: add runbook for database failover
```

> Good commit messages are like documentation. Future-you (and your team) will thank present-you.

---

### 6. `git log` — View Commit History

**What it does:** Displays a list of all commits in the current branch — who made them, when, and the message.

```bash
git log                        # Full history
git log --oneline              # Compact one-line view
git log --oneline --graph      # Visual branch tree
git log --oneline -10          # Last 10 commits
git log --author="Alice"       # Filter by author
git log -- path/to/file        # History of a specific file
```

**Example:**

```bash
git log --oneline --graph --all
# Output:
# * a3f1c2d (HEAD -> main) feat: add Prometheus scrape config
# * b7e90ca fix: correct Grafana data source URL
# * 1d4e8f0 chore: update terraform provider versions
```

> Use `--graph` to visualize branch merges — very helpful in complex projects.

---

### 7. `git branch` — Manage Branches

**What it does:** Lists, creates, or deletes branches.

```bash
git branch                     # List all local branches
git branch -a                  # List all branches (local + remote)
git branch feature/my-feature  # Create a new branch
git branch -d feature/done     # Delete a branch (safe — only if merged)
git branch -D feature/force    # Force delete a branch
```

**Example — List branches in a CI/CD repo:**

```bash
git branch -a
# Output:
# * main
#   feature/add-github-actions
#   remotes/origin/main
#   remotes/origin/release/v1.2
```

---

### 8. `git checkout` / `git switch` — Switch Branches

**What it does:** Moves you to a different branch. `git switch` is the modern, cleaner alternative to `git checkout` for branch switching.

```bash
git checkout branch-name           # Old way
git switch branch-name             # New way (recommended)
git switch -c feature/new-branch   # Create and switch in one step
git checkout -b feature/new-branch # Old equivalent
```

**Example — Create and move to a new feature branch:**

```bash
git switch -c feature/add-helm-chart
# Output: Switched to a new branch 'feature/add-helm-chart'
```

---

### 9. `git pull` — Fetch and Merge Remote Changes

**What it does:** Downloads changes from the remote repository and immediately merges them into your current branch.

```bash
git pull
git pull origin main           # Pull from a specific remote and branch
git pull --rebase origin main  # Pull using rebase instead of merge
```

**Example — Sync your local branch before starting work:**

```bash
git switch main
git pull origin main
# Always pull before you start coding to avoid conflicts later.
```

> `git pull --rebase` is preferred in many teams as it avoids unnecessary merge commits.

---

### 10. `git fetch` — Download Without Merging

**What it does:** Downloads all changes from the remote but does NOT merge them into your local branch. It just updates your knowledge of the remote.

```bash
git fetch
git fetch origin
git fetch --all        # Fetch from all remotes
```

**Example — Check what changed on remote before merging:**

```bash
git fetch origin
git log HEAD..origin/main --oneline
# Shows what commits are on remote that you don't have yet
```

> Use `git fetch` when you want to inspect remote changes before applying them.

---

### 11. `git push` — Upload to Remote

**What it does:** Sends your local commits to the remote repository.

```bash
git push
git push origin main                      # Push to a specific remote branch
git push -u origin feature/my-feature     # Push and set upstream tracking
git push --force-with-lease               # Force push safely (preferred over --force)
```

**Example — Push a new feature branch:**

```bash
git push -u origin feature/add-argo-workflow
```

> **Never use `git push --force` on shared branches.** Use `--force-with-lease` instead — it checks that nobody else has pushed before you.

---

### 12. `git merge` — Combine Branches

**What it does:** Merges changes from one branch into the current branch.

```bash
git merge feature/branch-name
git merge --no-ff feature/branch-name    # Always create a merge commit
git merge --squash feature/branch-name   # Squash all commits into one
```

**Example — Merge a feature into main:**

```bash
git switch main
git pull origin main
git merge feature/add-monitoring-stack
git push origin main
```

> `--no-ff` (no fast-forward) creates a merge commit even if the history is linear. This preserves context about the merge in the history.

---

### 13. `git rebase` — Rewrite History Cleanly

**What it does:** Moves or replays your commits on top of another branch. Produces a linear, cleaner history compared to merge.

```bash
git rebase main                  # Rebase current branch on top of main
git rebase -i HEAD~5             # Interactive rebase — edit last 5 commits
```

**Example — Keep your feature branch up to date with main:**

```bash
git switch feature/add-fluentd
git rebase main
# Your commits are now replayed on top of the latest main
```

**Interactive rebase — clean up messy commits before PR:**

```bash
git rebase -i HEAD~3
# Opens an editor where you can squash, rename, or reorder commits
```

> ⚠️ Never rebase commits that have already been pushed to a shared remote branch.

---

### 14. `git stash` — Temporarily Save Uncommitted Work

**What it does:** Saves your uncommitted changes to a hidden stack so you can switch context without losing work.

```bash
git stash                          # Stash current changes
git stash push -m "work in progress on nginx config"   # Stash with a label
git stash list                     # See all stashes
git stash pop                      # Apply latest stash and remove it
git stash apply stash@{2}          # Apply a specific stash without removing it
git stash drop stash@{0}           # Delete a stash
```

**Example — Urgent hotfix interrupts your work:**

```bash
# You're midway through editing a Terraform file
git stash push -m "wip: vpc peering changes"

# Switch to fix the urgent bug
git switch hotfix/fix-lb-timeout
# ... make fix, commit, push ...

# Come back and restore your work
git switch feature/vpc-peering
git stash pop
```

---

### 15. `git diff` — Compare Changes

**What it does:** Shows the exact line-by-line differences between files, commits, or branches.

```bash
git diff                          # Unstaged changes vs last commit
git diff --staged                 # Staged changes vs last commit
git diff main..feature/my-branch  # Difference between two branches
git diff commit1 commit2          # Difference between two commits
```

**Example — Review what you're about to commit:**

```bash
git diff --staged
```

> Always run `git diff --staged` before committing to catch accidental changes.

---

### 16. `git reset` — Undo Changes

**What it does:** Moves HEAD (and optionally the working directory) back to a previous state.

```bash
git reset --soft HEAD~1    # Undo last commit, keep changes staged
git reset --mixed HEAD~1   # Undo last commit, keep changes unstaged (default)
git reset --hard HEAD~1    # Undo last commit and DISCARD all changes (destructive)
git reset HEAD filename    # Unstage a specific file
```

**Example — Undo a bad commit (but keep the changes):**

```bash
git reset --soft HEAD~1
# Commit is removed, but your file changes are still staged
```

> ⚠️ `--hard` permanently destroys uncommitted changes. Use with extreme caution.

---

### 17. `git revert` — Safely Undo a Commit

**What it does:** Creates a new commit that reverses the changes of a previous commit. This is the **safe way** to undo in shared/production branches because it doesn't rewrite history.

```bash
git revert <commit-hash>
git revert HEAD            # Revert the latest commit
```

**Example — Revert a bad deployment config:**

```bash
git log --oneline
# a3f1c2d fix: wrong memory limit in deployment

git revert a3f1c2d
# Creates a new commit that undoes the change
git push origin main
```

> Use `git revert` on production/shared branches. Use `git reset` only on your private local branches.

---

### 18. `git tag` — Mark Releases

**What it does:** Creates a permanent, named pointer to a specific commit. Used to mark releases and versions.

```bash
git tag                           # List all tags
git tag v1.0.0                    # Create a lightweight tag
git tag -a v1.0.0 -m "Release 1.0.0 — initial stable release"  # Annotated tag (preferred)
git push origin v1.0.0            # Push a specific tag
git push origin --tags            # Push all tags
```

**Example — Tag a production release:**

```bash
git tag -a v2.3.1 -m "Release v2.3.1 — security patch for CVE-2026-1234"
git push origin v2.3.1
```

> Tags are critical in CI/CD — pipelines often trigger production deployments only on tagged commits.

---

### 19. `git remote` — Manage Remote Connections

**What it does:** Lets you view, add, rename, or remove connections to remote repositories.

```bash
git remote -v                            # List remotes with URLs
git remote add origin <url>              # Add a remote called "origin"
git remote set-url origin <new-url>      # Change remote URL
git remote remove origin                 # Remove a remote
```

**Example — Point your repo to a new GitLab instance:**

```bash
git remote set-url origin https://gitlab.company.com/infra/terraform-aws.git
git remote -v
# origin  https://gitlab.company.com/infra/terraform-aws.git (fetch)
# origin  https://gitlab.company.com/infra/terraform-aws.git (push)
```

---

### 20. `git cherry-pick` — Apply a Specific Commit

**What it does:** Takes a single commit from any branch and applies it to your current branch. Useful for applying a hotfix across multiple branches.

```bash
git cherry-pick <commit-hash>
git cherry-pick abc123 def456     # Cherry-pick multiple commits
```

**Example — Apply a critical security fix to a release branch:**

```bash
# Fix was committed on main with hash a1b2c3d
git switch release/v1.5
git cherry-pick a1b2c3d
git push origin release/v1.5
```

---

### 21. `git bisect` — Find the Bad Commit

**What it does:** Uses binary search to find which commit introduced a bug. Incredibly powerful for debugging regressions.

```bash
git bisect start
git bisect bad                    # Current commit is broken
git bisect good v1.0.0            # This older tag was working
# Git checks out a middle commit — you test and mark it:
git bisect good                   # or
git bisect bad
# Repeat until Git identifies the first bad commit
git bisect reset                  # End the session
```

**Example — Find when a memory leak was introduced:**

```bash
git bisect start
git bisect bad                    # HEAD is broken
git bisect good v2.1.0            # v2.1.0 was stable
# Git will help you narrow it down in ~7 steps for 100 commits
```

---

### 22. `git blame` — Track Who Changed What

**What it does:** Shows who last modified each line of a file and in which commit.

```bash
git blame filename
git blame -L 10,20 filename       # Show only lines 10–20
```

**Example — Find who changed the Jenkinsfile:**

```bash
git blame Jenkinsfile
# Output shows commit hash, author, date, and line content for each line
```

> Not about assigning blame (despite the name) — it's about finding context for changes.

---

## Git Workflow Cheat Sheet for DevOps

```bash
# Daily Workflow
git switch main && git pull          # Start from the latest
git switch -c feature/my-task        # Create a new branch
# ... make your changes ...
git status                           # Check what changed
git diff --staged                    # Review staged changes
git add .                            # Stage changes
git commit -m "feat: describe change" # Save a snapshot
git push -u origin feature/my-task   # Push to remote
# Open a Pull Request / Merge Request on GitHub/GitLab

# Keeping your branch up to date
git fetch origin
git rebase origin/main

# Quick fix on production
git stash
git switch hotfix/urgent-fix
# Fix, commit, push
git switch feature/my-task
git stash pop
```

---

## Git Configuration — Do This First

```bash
# Set your identity (required before first commit)
git config --global user.name "Your Name"
git config --global user.email "you@company.com"

# Set default branch name to main
git config --global init.defaultBranch main

# Set VS Code as default editor
git config --global core.editor "code --wait"

# Enable colored output
git config --global color.ui auto

# View all your config
git config --global --list

# Set up aliases for common commands
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.lg "log --oneline --graph --all"
```

---

## Useful `.gitignore` Patterns for DevOps

```gitignore
# Terraform
*.tfstate
*.tfstate.backup
.terraform/
.terraform.lock.hcl
*.tfvars           # Contains secrets — never commit!

# Ansible
*.retry
vault_password_file

# Kubernetes
kubeconfig
*.kubeconfig

# General secrets
.env
*.pem
*.key
secrets/
credentials.json

# OS files
.DS_Store
Thumbs.db
```

---

## Quick Reference Table

| Command | Purpose |
|---|---|
| `git init` | Start a new repo |
| `git clone <url>` | Copy a remote repo |
| `git status` | Check current state |
| `git add .` | Stage all changes |
| `git commit -m "msg"` | Save a snapshot |
| `git push` | Upload to remote |
| `git pull` | Download and merge |
| `git fetch` | Download without merging |
| `git branch` | List/create branches |
| `git switch -c <name>` | Create and switch branch |
| `git merge <branch>` | Merge a branch |
| `git rebase <branch>` | Rebase onto a branch |
| `git stash` | Save work temporarily |
| `git log --oneline` | View commit history |
| `git diff --staged` | Review staged changes |
| `git reset --soft HEAD~1` | Undo last commit (keep changes) |
| `git revert <hash>` | Safely undo a commit |
| `git tag -a v1.0.0` | Mark a release |
| `git cherry-pick <hash>` | Apply a specific commit |
| `git bisect` | Find which commit broke things |
| `git blame <file>` | See who changed each line |

---

## Summary

Git is not just a developer tool — it is the **backbone of modern DevOps**. Every pipeline, every deployment, every infrastructure change starts with a Git event. Mastering Git means you can collaborate faster, recover from failures with confidence, and maintain a clean, auditable trail of every change in your environment.

Start with the basics (`clone`, `add`, `commit`, `push`, `pull`), build habits around branching and tagging, and gradually add tools like `rebase`, `stash`, `cherry-pick`, and `bisect` to your daily toolkit. These commands separate a good DevOps engineer from a great one.

---

*Last updated: April 2026 | Tags: git, devops, version-control, gitops, ci-cd*