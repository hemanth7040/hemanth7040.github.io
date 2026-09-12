---
title: "Docker Interview & Learning Guide: From Fundamentals to Production"
date: 2026-09-12T00:00:00+05:30
draft: false
description: "A beginner-friendly, practical Docker guide covering 17 core topics — from architecture and images to multi-stage builds, security, and production workflows — with real-world examples and senior-level interview scenarios."
tags: ["Docker", "DevOps", "Containers", "Interview Prep", "Kubernetes", "CI/CD"]
categories: ["docker"]
summary: "17 essential Docker topics explained in simple language with commands, diagrams, real-world examples, and senior-level interview scenarios — built for DevOps engineers preparing for technical interviews."
showToc: true
TocOpen: true
cover:
  hidden: true
---

**Target audience:** DevOps engineers, especially those preparing for senior-level interviews.

This guide covers 17 important Docker topics, from fundamentals to production practices. Each section uses simple language, practical examples, commands, interview talking points, and real-world scenarios.

---

## 1. Docker Architecture

### What is Docker?

Docker is a containerization platform. It packages an application
together with the files, libraries, and configuration it needs to run.

Instead of installing an application directly on a server, we can
package it as a container image and run that image consistently across
environments.

For example:

``` text
Developer Laptop
       |
       | Docker Image
       v
   CI/CD Pipeline
       |
       v
 Container Registry
       |
       v
 Development / Staging / Production
```

The main benefit is consistency:

> "Build once, run consistently in different environments."

### Main Docker Components

``` text
+----------------------+
|      Docker CLI      |
|   docker build/run   |
+----------+-----------+
           |
           | Docker API
           v
+----------------------+
|    Docker Daemon     |
|       dockerd        |
+----------+-----------+
           |
     +-----+-----+----------------+
     |           |                |
     v           v                v
  Images     Containers       Networks
                              Volumes
           |
           v
      containerd
           |
           v
       OCI Runtime
         (runc)
           |
           v
       Container
```

#### Docker CLI

The CLI is what we use from the terminal:

``` bash
docker build
docker pull
docker run
docker ps
docker stop
docker exec
```

#### Docker Daemon

The Docker daemon (`dockerd`) is responsible for managing Docker objects
such as:

-   Images
-   Containers
-   Networks
-   Volumes

#### containerd

Docker uses containerd as a core container runtime component. It manages
much of the container lifecycle.

#### OCI Runtime

An OCI-compatible runtime such as `runc` creates and starts the
container process according to OCI specifications.

### Real-Time Example

Suppose you execute:

``` bash
docker run -d -p 8080:80 nginx
```

Conceptually:

``` text
docker run
    |
    v
Docker CLI
    |
    v
Docker Daemon
    |
    +--> Check whether nginx image exists locally
    |
    +--> If missing, pull image from registry
    |
    +--> Create container
    |
    +--> Create/configure network
    |
    +--> Configure filesystem
    |
    +--> Configure port mapping
    |
    +--> Start container
    |
    v
Nginx process
    |
    v
Container port 80
    |
    v
Host port 8080
```

You can then access:

``` text
http://localhost:8080
```

### Important Interview Questions

#### Docker vs Virtual Machine

A VM virtualizes hardware and normally includes a complete guest
operating system.

A container isolates processes while sharing the host kernel.

``` text
Virtual Machines:

Hardware
   |
Hypervisor
   |
+--------+--------+
| VM     | VM     |
| OS     | OS     |
| App    | App    |
+--------+--------+


Containers:

Hardware
   |
Host OS / Kernel
   |
Container Runtime
   |
+--------+--------+
| App    | App    |
| Libs   | Libs   |
+--------+--------+
```

#### Does Kubernetes require Docker Engine?

No. Modern Kubernetes commonly uses CRI-compatible runtimes such as
containerd or CRI-O. Docker images can still be used because modern
container images follow OCI-compatible formats.

------------------------------------------------------------------------

## 2. Docker Images

### What is an Image?

A Docker image is an immutable, read-only package used to create
containers.

It contains:

-   Application files
-   Required libraries
-   Runtime dependencies
-   Metadata
-   Filesystem layers

Think of an image as a **template**.

``` text
Docker Image
     |
     | docker run
     v
Container
```

### Useful Commands

List images:

``` bash
docker image ls
```

Pull an image:

``` bash
docker pull nginx:1.27
```

Inspect an image:

``` bash
docker image inspect nginx:1.27
```

Remove an image:

``` bash
docker image rm nginx:1.27
```

Show image history:

``` bash
docker history nginx:1.27
```

### Real-Time Example

A company has a Python API.

Instead of asking every developer to install:

-   Python
-   pip
-   OS packages
-   Application dependencies

the team creates:

``` text
company-api:1.5.0
```

The same image can be tested in CI and deployed to Kubernetes.

``` text
Dockerfile
    |
    v
docker build
    |
    v
company-api:1.5.0
    |
    v
Registry
    |
    +--> Dev
    +--> Staging
    +--> Production
```

### Important Point

An image does not normally contain a running process.

The container is the runtime instance created from the image.

------------------------------------------------------------------------

## 3. Containers

### What is a Container?

A container is an isolated process created from an image.

Example:

``` bash
docker run -d --name web nginx
```

Here:

``` text
nginx       -> Image
web         -> Container
```

You can run many containers from the same image:

``` bash
docker run -d --name web1 nginx
docker run -d --name web2 nginx
docker run -d --name web3 nginx
```

All three can use the same underlying image layers.

### Container Isolation

Containers use Linux kernel mechanisms such as:

-   Namespaces
-   cgroups
-   Capabilities
-   Filesystem isolation

#### Namespaces

Namespaces provide isolation for things such as:

-   Processes
-   Network interfaces
-   Mounts
-   Hostname
-   Users

#### cgroups

Control groups are used to control and measure resource usage such as:

-   CPU
-   Memory
-   PIDs
-   I/O

### Real-Time Example

Suppose a server runs:

``` text
Nginx container
API container
Redis container
```

Each process is isolated, but all containers use the host kernel.

### Useful Commands

List running containers:

``` bash
docker ps
```

List all containers:

``` bash
docker ps -a
```

See logs:

``` bash
docker logs web
```

Enter a container:

``` bash
docker exec -it web sh
```

Inspect a container:

``` bash
docker inspect web
```

------------------------------------------------------------------------

## 4. Docker Layers

### What is a Layer?

Docker images are built from filesystem layers.

Example:

``` dockerfile
FROM ubuntu:24.04

RUN apt-get update

RUN apt-get install -y nginx

COPY index.html /var/www/html/
```

Conceptually:

``` text
+------------------------------+
| COPY index.html              |
+------------------------------+
| Install nginx                |
+------------------------------+
| apt-get update               |
+------------------------------+
| ubuntu base image            |
+------------------------------+
```

Each filesystem-changing build step can produce a layer.

### Why Layers Matter

Layers provide:

-   Reuse
-   Build caching
-   Efficient storage
-   Faster image distribution

If multiple images use the same base layers, those layers can be shared.

### Real-Time Example

Suppose:

``` text
Application A -> python:3.12-slim
Application B -> python:3.12-slim
Application C -> python:3.12-slim
```

The base image layers do not need to be independently duplicated for
every image on the same host.

### Important Interview Point

Deleting a file in a later layer does not necessarily mean the data was
never included in an earlier layer.

For example, putting a secret into one image layer and deleting it later
is not a safe way to remove the secret from image history.

Never put secrets into Docker image build layers.

------------------------------------------------------------------------

## 5. Dockerfile

### What is a Dockerfile?

A Dockerfile is a text file containing instructions used to build an
image.

Example:

``` dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8080

USER 1000

CMD ["python", "app.py"]
```

### Important Instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Select base image |
| `RUN` | Execute build-time commands |
| `COPY` | Copy files into image |
| `ADD` | Copy files with additional features |
| `WORKDIR` | Set working directory |
| `ENV` | Set environment variables |
| `ARG` | Define build-time arguments |
| `EXPOSE` | Document intended port |
| `USER` | Set user for subsequent operations/runtime |
| `ENTRYPOINT` | Define main executable |
| `CMD` | Define default command/arguments |
| `HEALTHCHECK` | Define a health check |

### CMD vs ENTRYPOINT

Example:

``` dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Running:

``` bash
docker run myapp
```

results conceptually in:

``` bash
python app.py
```

The `CMD` can commonly be overridden by arguments supplied at runtime.

### Real-Time Example

For a Node.js application:

``` dockerfile
FROM node:22-slim

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 3000

USER node

CMD ["node", "server.js"]
```

Build:

``` bash
docker build -t my-node-app:1.0 .
```

Run:

``` bash
docker run -d -p 3000:3000 my-node-app:1.0
```

### Professional Practices

Use:

-   Specific base image versions
-   `.dockerignore`
-   Non-root users
-   Multi-stage builds
-   Small runtime images
-   Reproducible dependency installation
-   No secrets in Dockerfiles

------------------------------------------------------------------------

## 6. Multi-Stage Builds

### What Problem Does It Solve?

Build environments often need compilers and development tools.

Production normally does not.

For example, a Go application needs the Go compiler to build but does
not need the compiler to run.

### Example

``` dockerfile
FROM golang:1.24 AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN go build -o server .


FROM alpine:3.22

WORKDIR /app

COPY --from=builder /app/server .

USER 1000

CMD ["./server"]
```

### Architecture

``` text
Builder Stage
+----------------------+
| Go compiler          |
| Go source            |
| Build dependencies   |
+----------+-----------+
           |
           | binary
           v
Runtime Stage
+----------------------+
| Small runtime image  |
| Application binary   |
+----------------------+
```

### Benefits

-   Smaller final image
-   Reduced attack surface
-   Fewer unnecessary packages
-   Faster deployment
-   Cleaner production image

### Real-Time Example

A frontend application might require Node.js to build:

``` text
Node.js
npm
source code
build tools
    |
    v
dist/
```

The production image may only need Nginx:

``` text
Nginx
  |
  +--> dist/
```

This can significantly reduce the production image size.

------------------------------------------------------------------------

## 7. Docker Build Cache

### What is Build Cache?

Docker can reuse previously built layers when the relevant instruction
and input have not changed.

Example:

``` dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build
```

If only source files change, the dependency installation layer can
remain cached.

### Poor Dockerfile

``` dockerfile
COPY . .
RUN npm ci
```

Any change to application source may invalidate the `COPY` layer and
cause the dependency installation to run again.

### Better Dockerfile

``` dockerfile
COPY package*.json ./
RUN npm ci

COPY . .
```

### Real-Time CI Example

Suppose a CI build takes 10 minutes.

Most of the time is spent downloading dependencies.

By structuring the Dockerfile correctly and using BuildKit/registry
caching, repeated builds may reuse dependency layers and become much
faster.

### Build Command

``` bash
docker build -t myapp:1.0 .
```

For modern builds, BuildKit is commonly used by Docker.

------------------------------------------------------------------------

## 8. Docker Volumes

### Why Do We Need Volumes?

Container writable storage is tied to the container lifecycle.

If persistent application data is stored correctly in a Docker volume,
deleting the container does not automatically delete the volume.

### Types

#### Named Volume

``` bash
docker volume create app-data
```

Run:

``` bash
docker run -d \
  --name app \
  -v app-data:/data \
  myapp:1.0
```

#### Bind Mount

``` bash
docker run -d \
  -v /host/config:/app/config \
  myapp:1.0
```

#### tmpfs

``` bash
docker run -d \
  --tmpfs /tmp \
  myapp:1.0
```

### Volume vs Bind Mount

| Feature | Volume | Bind Mount |
|---|---|---|
| Managed by Docker | Yes | No |
| Host path controlled directly | No | Yes |
| Good for persistent container data | Yes | Sometimes |
| Good for local development source code | Sometimes | Yes |

### Real-Time Example

A local PostgreSQL development environment:

``` text
PostgreSQL Container
       |
       v
postgres-data volume
       |
       v
Container recreated
       |
       v
Database data remains
```

### Production Note

For production Kubernetes environments, persistent storage is normally
handled through Kubernetes storage abstractions such as PV/PVC rather
than relying on Docker volumes directly.

------------------------------------------------------------------------

## 9. Docker Networks

### Why Do Containers Need Networks?

Containers often need to communicate with:

-   Other containers
-   Databases
-   External APIs
-   Load balancers

### Common Network Types

#### Bridge

Default/common networking model for standalone Docker.

#### Host

The container uses the host's network namespace.

#### None

No normal network connectivity is provided.

#### Overlay

Used for multi-host container networking in systems that support it,
such as Docker Swarm.

### User-Defined Network

Create:

``` bash
docker network create app-net
```

Run API:

``` bash
docker run -d \
  --name api \
  --network app-net \
  my-api:1.0
```

Run database:

``` bash
docker run -d \
  --name db \
  --network app-net \
  postgres:17
```

The API can use the database service/container name:

``` text
db:5432
```

instead of depending on a hard-coded IP address.

### Real-Time Example

``` text
             app-net
+-----------+       +------------+
| API       | ----> | PostgreSQL |
| container |       | container  |
+-----------+       +------------+
```

Test from API:

``` bash
docker exec -it api sh
```

Then use tools such as `curl` or a database client to test connectivity.

### Troubleshooting

Useful commands:

``` bash
docker network ls
docker network inspect app-net
docker inspect api
```

------------------------------------------------------------------------

## 10. Container Lifecycle

A container has a lifecycle.

A simplified view:

``` text
Created
   |
   v
Running
   |
   +----> Paused
   |         |
   |         v
   |       Running
   |
   v
Stopped
   |
   v
Removed
```

### Commands

Create without starting:

``` bash
docker create --name web nginx
```

Start:

``` bash
docker start web
```

Run (create + start):

``` bash
docker run -d --name web nginx
```

Stop:

``` bash
docker stop web
```

Restart:

``` bash
docker restart web
```

Remove:

``` bash
docker rm web
```

Force remove:

``` bash
docker rm -f web
```

### Stop vs Kill

`docker stop` requests graceful termination.

``` bash
docker stop web
```

`docker kill` sends a kill signal by default and is intended for
immediate termination.

``` bash
docker kill web
```

### Real-Time Troubleshooting

A container is repeatedly stopping.

First:

``` bash
docker ps -a
```

Then:

``` bash
docker logs web
```

Inspect its configuration:

``` bash
docker inspect web
```

Check the command and exit status.

Common causes:

-   Application crashes
-   Wrong command
-   Missing environment variables
-   Missing configuration
-   Dependency failure
-   Permission problem

------------------------------------------------------------------------

## 11. Docker Compose

### What is Docker Compose?

Docker Compose lets you define and manage a multi-container application
using a YAML file.

Example:

``` yaml
services:
  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      DB_HOST: db
    depends_on:
      - db

  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

Start:

``` bash
docker compose up -d
```

Stop:

``` bash
docker compose down
```

List services:

``` bash
docker compose ps
```

View logs:

``` bash
docker compose logs -f api
```

### Real-Time Example

A developer has:

``` text
Frontend
   |
   v
Backend API
   |
   v
PostgreSQL
```

Instead of starting three containers manually, Compose defines the
complete local environment.

``` text
docker compose up
```

starts the application stack.

### Important Interview Point

`depends_on` can control startup ordering, but startup order is not the
same as application readiness.

A database process can be started while the database is still
initializing.

Health checks and application-level retry logic are important for
reliable startup.

------------------------------------------------------------------------

## 12. Container Security

Security should be considered throughout the image and container
lifecycle.

### Run as Non-Root

Bad:

``` dockerfile
FROM ubuntu
CMD ["./app"]
```

Better:

``` dockerfile
FROM ubuntu

RUN useradd -u 1000 appuser

USER 1000

CMD ["./app"]
```

### Avoid Secrets in Images

Never do:

``` dockerfile
ENV DB_PASSWORD=SuperSecret123
```

Secrets can become exposed through image configuration/history or other
build artifacts.

Use an appropriate secret-management mechanism.

### Other Security Controls

Understand:

-   Linux capabilities
-   Seccomp
-   AppArmor
-   SELinux
-   Read-only root filesystem
-   Non-root execution
-   Minimal base images
-   Image scanning
-   Image signing
-   SBOM
-   Trusted registries
-   Dependency scanning

### Read-Only Filesystem

Example:

``` bash
docker run --read-only nginx
```

If the application needs temporary storage, provide a suitable writable
mount such as tmpfs.

### Dangerous Docker Socket Mount

Be very careful with:

``` bash
-v /var/run/docker.sock:/var/run/docker.sock
```

Giving a container access to the Docker daemon can provide extremely
powerful control over the host.

### Real-Time Example

A CI runner needs to build images.

Instead of automatically exposing the host Docker socket to every
untrusted job, evaluate safer approaches such as:

-   Rootless builders
-   BuildKit-based approaches
-   Dedicated build workers
-   Isolated build environments

The right solution depends on the CI platform and threat model.

------------------------------------------------------------------------

## 13. Image Optimization

### Why Optimize Images?

Large images cause:

-   Longer build times
-   Longer image pull times
-   More storage usage
-   More network traffic
-   Larger attack surface

### Techniques

#### Use Smaller Appropriate Base Images

Examples:

``` text
python:3.12-slim
node:22-slim
alpine
distroless
```

Do not choose a base image solely because it is small. Compatibility,
security, debugging, and support also matter.

#### Multi-Stage Build

Build dependencies stay in the builder stage.

#### Use `.dockerignore`

Example:

``` text
.git
node_modules
.env
*.log
coverage
```

This reduces the Docker build context.

#### Clean Package Manager Cache

For Debian/Ubuntu-style images:

``` dockerfile
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

#### Combine Related Package Operations

This can avoid retaining unnecessary package metadata in a filesystem
layer.

### Real-Time Example

Suppose an image is:

``` text
2.1 GB
```

Investigation finds:

``` text
1.0 GB build tools
500 MB dependency cache
300 MB unnecessary OS packages
300 MB application
```

A multi-stage build plus a smaller runtime image may reduce the final
production image substantially.

The goal is not "smallest possible image"; the goal is a **secure,
maintainable, reliable image with only the required runtime content**.

------------------------------------------------------------------------

## 14. Rootless Containers

### What Are Rootless Containers?

Rootless container execution allows container workloads and the relevant
container runtime components to operate without requiring root
privileges.

This can reduce the impact of certain container escape or
daemon-compromise scenarios.

### Important Distinction

These are not the same:

``` text
USER 1000
```

and

``` text
Rootless Docker
```

`USER 1000` means the application process inside the container runs as a
non-root user.

Rootless Docker goes further by allowing the Docker daemon/runtime
operation to occur without root privileges.

### Real-Time Example

A shared development machine is used by multiple engineers.

Running Docker in a rootless configuration can reduce the privileges
associated with container management compared with a traditional
root-owned Docker daemon.

### Limitations

Rootless/container security features can have compatibility and
performance considerations.

Some workloads may require additional configuration for:

-   Networking
-   Storage
-   Privileged operations
-   Low-numbered ports
-   Hardware access

Therefore, evaluate rootless mode against workload requirements.

------------------------------------------------------------------------

## 15. Docker Registry

### What is a Registry?

A container registry stores and distributes container images.

Examples include:

-   Docker Hub
-   Amazon ECR
-   GitHub Container Registry
-   Google Artifact Registry
-   Azure Container Registry
-   Harbor

### Typical Flow

``` text
Developer
   |
   | docker build
   v
Docker Image
   |
   | docker push
   v
Container Registry
   |
   | docker pull
   v
Server / Kubernetes
```

### Commands

Login:

``` bash
docker login
```

Tag:

``` bash
docker tag myapp:1.0 registry.example.com/myapp:1.0
```

Push:

``` bash
docker push registry.example.com/myapp:1.0
```

Pull:

``` bash
docker pull registry.example.com/myapp:1.0
```

### AWS ECR Example

A common AWS workflow is:

``` text
Git
  |
  v
CI Pipeline
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
EKS
```

ECR can provide repository policies, lifecycle policies, image scanning
integrations, and controlled access through AWS IAM.

------------------------------------------------------------------------

## 16. Image Tagging

### What is a Tag?

A tag is a human-readable reference to an image.

Examples:

``` text
myapp:latest
myapp:1.0
myapp:1.0.3
myapp:2026.09.12
```

### Why `latest` Is Risky in Production

`latest` is a mutable tag.

It may point to different image content at different times.

For production, prefer meaningful immutable versioning strategies.

Example:

``` text
myapp:1.4.2
myapp:git-a81f3c2
```

You can also deploy by digest:

``` text
myapp@sha256:<digest>
```

A digest identifies image content.

### Real-Time CI/CD Example

A pipeline creates:

``` text
myapp:build-1842
myapp:git-a81f3c2
```

After testing, the exact image is promoted to production.

The important principle is:

> Build one artifact and promote the same artifact across environments.

Avoid rebuilding separately for staging and production because that can
produce different artifacts.

### Tag vs Digest

``` text
Tag
 |
 +--> Human-friendly reference
 +--> Can be mutable

Digest
 |
 +--> Content-addressed reference
 +--> Identifies exact image content
```

------------------------------------------------------------------------

## 17. Image Scanning

### What is Image Scanning?

Image scanning checks an image for known security issues.

A scanner may inspect:

-   OS packages
-   Application dependencies
-   Known CVEs
-   Vulnerable libraries
-   Sometimes configuration/security issues

Examples of tools include:

-   Trivy
-   Grype
-   Snyk
-   Registry-integrated scanners

### Example

``` bash
trivy image myapp:1.0
```

The output can identify vulnerabilities and their severity.

### CI/CD Example

``` text
Git Commit
    |
    v
Build Image
    |
    v
Unit Tests
    |
    v
Image Scan
    |
    +---- Critical vulnerability ----> Fail pipeline
    |
    v
Push Image
    |
    v
Registry
    |
    v
Deploy
```

### Real-Time Example

Suppose a base image contains a critical vulnerability.

Your application code has not changed.

A security scan detects the vulnerable package.

The correct response is usually to:

1.  Identify the affected package.
2.  Determine whether the vulnerability is exploitable in your context.
3.  Update the base image/package where appropriate.
4.  Rebuild the image.
5.  Scan again.
6.  Test.
7.  Redeploy the fixed image.

### Important Senior-Level Point

Scanning is not a one-time activity.

New vulnerabilities can be discovered after an image has already been
deployed. Images should therefore be monitored and rescanned according
to the organization's security process.

------------------------------------------------------------------------

## Production Docker Workflow

A mature Docker workflow can look like this:

``` text
                    Developer
                        |
                        v
                       Git
                        |
                        v
                    CI Pipeline
                        |
              +---------+---------+
              |                   |
              v                   v
          Unit Tests          Docker Build
                                  |
                                  v
                           Image Optimization
                                  |
                                  v
                            Image Scanning
                                  |
                         +--------+--------+
                         |                 |
                      Failed            Passed
                         |                 |
                         v                 v
                       Stop              Tag
                                           |
                                           v
                                     Push to Registry
                                           |
                                           v
                                     Deploy to Dev
                                           |
                                           v
                                      Integration
                                         Tests
                                           |
                                           v
                                    Promote Same
                                       Image
                                           |
                                           v
                                      Production
                                           |
                                           v
                                     Monitoring
```

------------------------------------------------------------------------

## Senior-Level Docker Interview Scenarios

### Scenario 1: Container Keeps Restarting

Question:

> A production container keeps restarting. How do you troubleshoot it?

Approach:

``` bash
docker ps -a
docker logs <container>
docker inspect <container>
docker stats <container>
```

Check:

-   Exit code
-   Application logs
-   Entrypoint/CMD
-   Environment variables
-   Configuration
-   Permissions
-   Dependencies
-   Memory
-   Health checks
-   Signals

Do not randomly restart the container repeatedly without finding the
cause.

------------------------------------------------------------------------

### Scenario 2: Container Works Locally but Fails in CI

Investigate:

``` text
Environment differences
Credentials
Build context
Architecture
Dependency versions
Network access
Proxy configuration
Docker version/build behavior
Environment variables
File permissions
```

A good DevOps engineer compares the environments instead of assuming the
application is broken.

------------------------------------------------------------------------

### Scenario 3: Image Is 2 GB

Investigate:

``` text
Base image
Build dependencies
Application dependencies
Caches
Dockerfile layers
Build context
Unnecessary files
Multi-stage build opportunity
```

Use:

``` bash
docker history <image>
```

and appropriate image inspection tools.

------------------------------------------------------------------------

### Scenario 4: Production Deployment Uses `latest`

Problem:

``` text
production -> myapp:latest
```

Potential issue:

The tag can change.

Better:

``` text
production -> myapp:1.8.3
```

or preferably pin the exact immutable image digest where the deployment
platform supports it.

------------------------------------------------------------------------

### Scenario 5: Developer Accidentally Commits a Secret

Do not simply delete the Git line.

The immediate priority is:

``` text
Revoke / rotate secret
        |
        v
Investigate exposure
        |
        v
Remove secret from source/history where appropriate
        |
        v
Move secret to proper secret management
        |
        v
Add prevention controls
```

The exposed credential should be considered compromised.

------------------------------------------------------------------------

## Docker Commands You Should Know

### Images

``` bash
docker image ls
docker pull nginx
docker build -t myapp:1.0 .
docker image inspect myapp:1.0
docker history myapp:1.0
docker image rm myapp:1.0
```

### Containers

``` bash
docker ps
docker ps -a
docker run
docker start
docker stop
docker restart
docker rm
docker logs
docker exec
docker inspect
docker stats
```

### Networks

``` bash
docker network ls
docker network create app-net
docker network inspect app-net
docker network connect app-net container
docker network disconnect app-net container
```

### Volumes

``` bash
docker volume ls
docker volume create app-data
docker volume inspect app-data
docker volume rm app-data
```

### Registry

``` bash
docker login
docker tag
docker push
docker pull
```

### Compose

``` bash
docker compose up -d
docker compose down
docker compose ps
docker compose logs
docker compose exec
```

------------------------------------------------------------------------

## Key Concepts to Remember

### Image vs Container

``` text
Image     = immutable template
Container = running/created instance
```

### Dockerfile vs Image

``` text
Dockerfile
    |
    | docker build
    v
Image
```

### Image vs Registry

``` text
Image      = artifact
Registry   = storage/distribution system
```

### Container vs VM

``` text
Container
    |
    +--> Shares host kernel
    +--> Process isolation
    +--> Lightweight

VM
    |
    +--> Guest operating system
    +--> Virtualized hardware
    +--> Stronger OS-level isolation
```

### Tag vs Digest

``` text
Tag
    |
    +--> Human-friendly
    +--> Can change

Digest
    |
    +--> Identifies exact image content
    +--> Suitable for immutable references
```

### `USER` vs Rootless

``` text
USER 1000
    |
    +--> Application runs as non-root inside container

Rootless
    |
    +--> Container management/runtime can operate without root privileges
```

------------------------------------------------------------------------

## Recommended Learning Order

For interviews, study in this order:

``` text
1. Docker Architecture
        |
2. Images
        |
3. Containers
        |
4. Layers
        |
5. Dockerfile
        |
6. Build Cache
        |
7. Multi-stage Builds
        |
8. Networks
        |
9. Volumes
        |
10. Container Lifecycle
        |
11. Docker Compose
        |
12. Image Optimization
        |
13. Container Security
        |
14. Rootless Containers
        |
15. Registry
        |
16. Image Tagging
        |
17. Image Scanning
```

After learning these topics, practice connecting Docker to:

``` text
Docker
  |
  +--> Jenkins / GitHub Actions
  |
  +--> AWS ECR
  |
  +--> Kubernetes / EKS
  |
  +--> Terraform
  |
  +--> Prometheus / Grafana
  |
  +--> DevSecOps
```

For a 7-year DevOps interview, the goal is not just to define Docker
concepts. You should be able to explain **why a design was chosen, what
happens internally, how to troubleshoot failures, and how to operate
Docker securely in production**.