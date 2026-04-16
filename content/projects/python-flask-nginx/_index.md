---
title: "Automated CI/CD Pipeline for a Python Flask Web App"
date: 2026-04-16
draft: false
#project: ["Production-Ready Flask"]
projects: ["Basic"]
description: "A beginner-friendly walkthrough of deploying a Python Flask app to AWS EC2 using Docker, Nginx, and GitHub Actions"
tags: ["DevOps", "Flask", "Docker", "Nginx", "GitHub Actions", "AWS EC2", "CI/CD"]
---

---
title: "Projects"
description: "A showcase of my DevOps and MLOps engineering work."
layout: "simple"
showAuthor: true
showDate: false
showPagination: false
---


## Project Overview

This project demonstrates how to take a Python Flask web application from your local machine and deploy it to a real cloud server on AWS — automatically, every time you push code to GitHub.

**What you will build:** A Flask app running inside a Docker container on an AWS EC2 server, sitting behind Nginx, with a GitHub Actions pipeline that deploys new changes with zero manual effort.

**Tools used:** Python Flask, Docker, Nginx, AWS EC2, GitHub Actions, Gunicorn, Bash

---

## Project Folder Structure

When you open the project, you see these files and folders:

```
python-flask/
├── app.py                          ← Your Flask web application
├── requirements.txt                ← Python package list
├── Dockerfile                      ← Instructions to containerize the app
├── .gitignore                      ← Files Git should ignore
├── .github/
│   └── workflows/
│       └── deploy.yml              ← GitHub Actions CI/CD pipeline
└── venv/                           ← Local Python virtual environment
```

### What Each Folder and File Does

**`app.py`** — This is the heart of the project. It contains your Flask web application: the routes, the logic, and the responses. When someone visits your website, this file decides what they see.

**`requirements.txt`** — Think of this as a shopping list for Python. It tells Python's package manager (pip) exactly which libraries your app needs, and which versions. This ensures the app runs identically on your laptop, in Docker, and on the production server.

**`Dockerfile`** — This is a recipe for building a Docker image. It tells Docker how to create a self-contained box (a container) that has Python, your dependencies, and your app all bundled together. Anyone can take this recipe and run your app without installing anything manually.

**`.gitignore`** — This file tells Git which files and folders to skip when saving your code. For example, the `venv/` folder (your local virtual environment) is huge and machine-specific — there is no point uploading it to GitHub.

**`.github/workflows/deploy.yml`** — This is the automation file. GitHub reads this every time you push code and automatically runs the steps inside it: build the Docker image, SSH into your EC2 server, and deploy the new version.

**`venv/`** — This is your local Python virtual environment. It is a sandboxed Python installation where your project's packages live, completely isolated from other Python projects on your machine. This folder never goes to GitHub (it is in `.gitignore`) because it is generated locally and is specific to your machine.

---

## File-by-File Code Explanation

### `app.py` — The Flask Application

```python
from flask import Flask
```

This line imports the Flask class from the Flask library. Think of Flask as a toolkit for building web servers. By importing `Flask`, you are saying "give me the tools to create a web server."

```python
app = Flask(__name__)
```

This creates your web application. `Flask(__name__)` tells Flask the name of the current file so it knows where to find templates, static files, and configuration. The variable `app` is your web server — everything flows through it.

```python
@app.route("/")
def home():
    return "<h1>Hello from my DevOps Pipeline! 🚀</h1>"
```

This is a **route** — it maps a URL path to a Python function. Breaking it down:

- `@app.route("/")` — The `@` symbol makes this a **decorator**. It tells Flask: "When someone visits the path `/` (the homepage, e.g. `http://yoursite.com/`), call the function below."
- `def home():` — This defines the function that handles that request.
- `return "<h1>Hello from my DevOps Pipeline! 🚀</h1>"` — This sends back an HTML response to the user's browser. Whatever you return here is what the user sees on screen.

```python
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

This block only runs when you execute `python app.py` directly — it does not run when Gunicorn or another server imports the file. `host="0.0.0.0"` means "accept connections from any network interface, not just localhost." `port=5000` means "listen on port 5000."

> **Note:** In production (inside Docker), Gunicorn starts the app instead of this block. Gunicorn is a proper production-grade server that can handle many simultaneous users, while Flask's built-in server is only meant for development.

---

### `requirements.txt` — Python Dependencies

```
flask==3.0.0
gunicorn==21.2.0
```

**`flask==3.0.0`** — Installs Flask version 3.0.0 exactly. Flask is the web framework your app is built on. Pinning to a specific version (`==3.0.0` rather than just `flask`) prevents surprise breakages if a newer version of Flask changes something.

**`gunicorn==21.2.0`** — Installs Gunicorn, which stands for "Green Unicorn." It is a production-grade Python web server. While Flask has its own built-in server, it is single-threaded and not suitable for real traffic. Gunicorn can handle many requests simultaneously. In Docker, Gunicorn is the program that actually starts and serves your Flask app.

---

### `Dockerfile` — Containerization Recipe

```dockerfile
FROM python:3.14-slim
```

Every Dockerfile starts with `FROM`, which specifies the base image. You are starting from `python:3.14-slim` — an official Docker image that already has Python 3.14 installed. The `-slim` variant is a stripped-down version that removes unnecessary files, making the final image smaller and faster to download and deploy.

Think of it like choosing a pre-furnished apartment: instead of starting from an empty room, you start from a room that already has the essentials.

```dockerfile
WORKDIR /app
```

This sets the working directory inside the container. From this point forward, every command in the Dockerfile runs from `/app`. It also means your app files will live at `/app` inside the container. If `/app` does not exist, Docker creates it automatically.

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
```

These two lines work together. First, `COPY requirements.txt .` copies only the requirements file into the container (the `.` means "current directory" which is `/app` because of `WORKDIR`).

Then `RUN pip install -r requirements.txt` installs all the packages listed in that file.

**Why copy requirements first and the rest of the code second?** This is a Docker performance trick. Docker builds images in layers and caches each layer. If your app code changes but your requirements do not, Docker can skip reinstalling packages and use the cached layer. This makes rebuilds significantly faster.

```dockerfile
COPY . .
```

Now copy everything else — your `app.py`, and any other project files — into the container's `/app` directory. The first `.` means "everything in the current folder on your machine," and the second `.` means "put it in the current directory inside the container."

```dockerfile
EXPOSE 5000
```

This documents that the container listens on port 5000. It is like labeling a door — it tells anyone reading the Dockerfile "this container's service is accessible through port 5000." Note that `EXPOSE` does not actually open the port to the outside world; that happens when you run the container with `-p 5000:5000`.

```dockerfile
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

This is the command Docker runs when the container starts. Breaking it down:

- `gunicorn` — Use Gunicorn as the server (not Flask's development server).
- `--bind 0.0.0.0:5000` — Listen on all network interfaces on port 5000.
- `app:app` — The first `app` refers to the file `app.py`. The second `app` refers to the Flask instance (the variable called `app` inside that file). Gunicorn needs this to know where your Flask application lives.

---

### `.github/workflows/deploy.yml` — GitHub Actions CI/CD Pipeline

```yaml
name: Deploy to EC2
```

This gives your workflow a name. It shows up in the GitHub Actions tab so you can easily identify it.

```yaml
on:
  push:
    branches: [main]
```

This defines the **trigger** — what causes this pipeline to run. Here, it runs automatically whenever you push code to the `main` branch. Every `git push origin main` will kick off this pipeline.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
```

A workflow contains one or more **jobs**. This job is named `deploy`. `runs-on: ubuntu-latest` tells GitHub to run this job on a fresh Ubuntu Linux virtual machine (GitHub provides these for free). Every run starts from a clean slate.

```yaml
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
```

**Steps** are the individual tasks inside a job. This first step uses a pre-built action (`actions/checkout@v4`) published by GitHub itself. It clones your repository onto the runner machine so the next steps have access to your code. Without this, the runner would have no files to work with.

```yaml
      - name: Build Docker image
        run: docker build -t my-devops-app .
```

This step runs a shell command directly on the runner. `docker build -t my-devops-app .` builds a Docker image using the Dockerfile in your repository. The `-t my-devops-app` gives the image a name (a "tag") so you can reference it in the next step.

```yaml
      - name: Deploy to EC2 via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
```

This step uses a community-built action (`appleboy/ssh-action`) that establishes an SSH connection to your EC2 server. The `${{ secrets.EC2_HOST }}` and `${{ secrets.EC2_SSH_KEY }}` values are pulled from **GitHub Secrets** — a secure vault where you store sensitive information like server IPs and private keys, so they never appear in your code.

`username: ubuntu` is the default SSH user for Ubuntu-based EC2 instances.

```yaml
          script: |
            cd /home/ubuntu

            docker stop my-app || true
            docker rm my-app || true

            docker build -t my-devops-app https://github.com/${{ github.repository }}.git#main
            docker run -d --name my-app -p 5000:5000 my-devops-app
```

This is the script that runs on your EC2 server over SSH. Line by line:

- `cd /home/ubuntu` — Navigate to the ubuntu home directory.
- `docker stop my-app || true` — Stop the currently running container. The `|| true` prevents the pipeline from failing if no container is running yet (the first time you deploy, there is nothing to stop).
- `docker rm my-app || true` — Remove the stopped container to free up the name for the next run.
- `docker build -t my-devops-app https://github.com/...` — Build a fresh Docker image directly from your GitHub repository on the EC2 server itself.
- `docker run -d --name my-app -p 5000:5000 my-devops-app` — Start the new container. `-d` means run in the background (detached mode). `--name my-app` gives it a name. `-p 5000:5000` maps port 5000 on the EC2 to port 5000 inside the container, making it accessible.

---

### Nginx Configuration — The Traffic Director

The Nginx config is placed on your EC2 server at `/etc/nginx/sites-available/default`.

```nginx
server {
    listen 80;
```

`server { }` defines a virtual server — a single block of rules for handling incoming traffic. `listen 80` tells Nginx to watch port 80, which is the standard HTTP port. When a browser navigates to `http://your-ec2-ip`, it automatically goes to port 80, and Nginx is waiting there.

```nginx
    location / {
```

`location /` matches any URL path that starts with `/`. Since `/` is the root, it matches everything: `/`, `/about`, `/api/users`, all of it. This is where the rules for forwarding traffic are defined.

```nginx
        proxy_pass http://127.0.0.1:5000;
```

This is the most important line. It tells Nginx: "Forward this request to port 5000 on this same machine." Your Flask app (running in Docker) is listening on port 5000 internally. Nginx receives traffic on port 80 (the public internet-facing port) and passes it to Flask on port 5000 (the internal application port). The user never needs to know Flask exists.

`127.0.0.1` means "this very machine" — it is the loopback address, also known as `localhost`.

```nginx
        proxy_set_header Host $host;
```

When Nginx forwards a request, it creates a new internal request to Flask. By default, Flask would see Nginx's IP as the requester and lose the original domain name. This line attaches the original domain name (`$host`) to the forwarded request so Flask knows which website the user was actually trying to visit. This matters for multi-domain setups and for accurate logging.

```nginx
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Similarly, without this line, Flask would see every request as coming from `127.0.0.1` (Nginx itself). `X-Real-IP` passes the actual visitor's IP address through to your app. This is essential for analytics, rate limiting, security, and understanding real traffic patterns.

---

## How Everything Connects

Here is the complete picture of what happens when a visitor loads your website:

```
User's Browser
      │
      │  HTTP request (port 80)
      ▼
AWS EC2 Server
      │
      ▼
  Nginx (port 80)         ← reads your nginx config
      │
      │  proxy_pass to localhost:5000
      ▼
Docker Container (port 5000)
      │
      ▼
  Gunicorn                ← production Python server
      │
      ▼
  Flask app.py            ← your Python code runs
      │
      │  HTML response
      ▼
Back to the user's browser ✅
```

And when you push code to GitHub:

```
git push origin main
      │
      ▼
GitHub Actions triggers
      │
      ▼
Runner (Ubuntu VM)
  → Checkout your code
  → Build Docker image
  → SSH into EC2
      │
      ▼
EC2 Server
  → Stop old container
  → Remove old container
  → Build new Docker image
  → Start new container
      │
      ▼
Updated app is live ✅
```

---

## What You Learned

By completing this project you have hands-on experience with the core building blocks that real DevOps engineers use every day. You understand how Flask serves web traffic, how Docker packages that app into a portable container, how Nginx sits in front to handle routing cleanly, and how GitHub Actions automates the entire delivery process so no manual deployment is ever needed.

This foundation carries you directly into Project 2 (Infrastructure as Code with Terraform) and beyond.