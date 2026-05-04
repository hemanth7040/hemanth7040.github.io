---
title: "Phase 1: The Local Automator — OS Mastery for DevOps Engineers"
date: 2026-04-19T08:00:00+05:30
draft: false
categories: ["python"]
description: "Learn os, pathlib, subprocess, json, and logging — the five built-in Python modules every DevOps engineer needs to control the file system, run terminal commands, manage config files, and track script health in production."
series: ["Python Zero to MLOps"]
series_order: 4
tags: ["os", "pathlib", "subprocess", "json", "logging", "DevOps", "LocalAutomator", "Beginners", "Python"]
---

# Phase 1: The Local Automator

> **Goal:** Master the local Operating System layer before touching the cloud.
> Written for beginners — including people from non-IT backgrounds.

---

## What You Will Learn

| Module | What it does |
|---|---|
| `os` and `pathlib` | Move files, check directories, read secret keys from the OS |
| `subprocess` | Run any terminal command (`git`, `kubectl`, `docker`) from inside Python |
| `json` | Convert Python data to and from JSON files — the language every API speaks |
| `logging` | Write timestamped records of every action (the professional alternative to `print()`) |

---

## Before You Start — Two Concepts Every Beginner Must Understand

### What is a Terminal?

Think of Python as a very smart assistant sitting inside your computer. The **terminal** (also called command prompt, shell, or CLI) is the black screen where you type commands like `git status` or `ls` and the computer responds.

Normally a human types those commands by hand. `subprocess` lets your Python script type those commands *automatically* — no human needed.

### What is stdout vs stderr?

Every program that runs in a terminal has two separate output channels:

```
stdout  (Standard Output)
        The normal result — what the program prints when everything is OK.
        Example: git status printing "On branch main"

stderr  (Standard Error)
        The error channel — what the program prints when something goes wrong.
        Example: git printing "fatal: not a git repository"
```

**Real-world analogy:** Imagine calling a bank helpline.

- **stdout** = the agent giving you your account balance (normal answer)
- **stderr** = the automated voice saying "Invalid account number" (error message)

They are two separate phone lines. Python can listen to both at the same time.

---

## Module 1 — `os` and `pathlib`: File and Directory Control

### Why you need this

Think of your computer's file system as a massive office building.

- `os` is your **master key** — it lets Python open any door, read any notice board (environment variables), and check which floor you are on.
- `pathlib.Path` is your **GPS navigator** — it builds safe directions to any room (file path) that work whether you are on Windows, Mac, or Linux.

### Real DevOps scenario

> Your deployment script must: (1) read the secret API key from the OS, (2) check if the releases folder exists, (3) create it if missing, (4) build the path to the config file safely.

---

### Step 1 — Import the tools

`import` means: go fetch this toolbox and make it available in this script. Both `os` and `pathlib` are built into Python — no installation needed.

```python
import os
from pathlib import Path
```

`from pathlib import Path` means: from the pathlib toolbox, give me only the `Path` tool. `Path` is smarter than a plain string like `"/var/app/releases"` because it understands how file systems work and adjusts automatically for each operating system.

---

### Step 2 — Read secrets from the environment

Never hardcode secrets in your code. If you write `api_key = "sk-abc123"` and push it to GitHub, the whole world can see your API key.

The safe way is to read it from the operating system instead:

```python
api_key = os.getenv("DEPLOY_API_KEY", "not-set")
```

`os.getenv()` takes two arguments — the variable name to look up, and a default value if it is missing. Someone sets this variable beforehand by running `export DEPLOY_API_KEY=sk-abc123` in their terminal. Using `"not-set"` as the default means your script fails gracefully instead of crashing with a confusing error.

```python
print(f"API key loaded: {'YES' if api_key != 'not-set' else 'MISSING — check your environment'}")
```

The `f` before the quote means "format this string." Anything inside `{}` is evaluated as Python code and inserted into the string.

---

### Step 3 — Build directory paths safely

```python
deploy_dir = Path("/var/app/releases")
```

This creates a `Path` object — a smart reference to a folder location. **Nothing happens on disk at this line.** You are just telling Python "I want to work with this folder." Think of it like writing an address on a piece of paper — you have not gone there yet.

You can inspect the path object immediately:

```python
print(deploy_dir)           # /var/app/releases
print(deploy_dir.name)      # releases  (just the folder name)
print(deploy_dir.parent)    # /var/app  (the folder that contains it)
```

---

### Step 4 — Create the directory if it does not exist

```python
if not deploy_dir.exists():
    deploy_dir.mkdir(parents=True, exist_ok=True)
    print(f"Created: {deploy_dir}")
```

`.exists()` asks the OS whether the folder actually exists on disk. `not` flips the result — so this block runs only when the folder is missing.

`.mkdir()` creates the folder. Two important arguments:

- `parents=True` — if `/var/app` does not exist either, create all missing parent folders along the way. Without this, Python crashes saying `/var/app` does not exist.
- `exist_ok=True` — if the folder already exists, do nothing instead of crashing. This makes the script safe to run multiple times.

---

### Step 5 — Join paths with the `/` operator

```python
config_file = deploy_dir / "config.json"
print(f"Config path: {config_file}")
# Config path: /var/app/releases/config.json
```

The `/` operator between two `Path` objects joins them into a new path. This is always correct — on Windows it uses `\`, on Mac and Linux it uses `/`, automatically. Plain string concatenation like `"/var/app" + "/" + "config.json"` breaks on Windows.

---

### Step 6 — Read and write files

Always create the parent directory before writing a file, then use `.write_text()` and `.read_text()`:

```python
log_file = Path("/var/log/deploy/app.log")
log_file.parent.mkdir(parents=True, exist_ok=True)

log_file.write_text("Deploy started\n")

content = log_file.read_text()
print(content)
# Deploy started
```

`.write_text()` creates the file and writes the string in one step. If the file already exists, it overwrites it. `.read_text()` opens the file and returns its entire content as a string — no need to open and close manually.

---

**Full terminal output for this module:**

```
API key loaded: MISSING — check your environment
/var/app/releases
releases
/var/app
Created: /var/app/releases
Config path: /var/app/releases/config.json
Deploy started
```

---

## Module 2 — `subprocess`: Run Terminal Commands from Python

### Why you need this

As a DevOps engineer you will spend a lot of time running terminal commands — `git pull`, `docker build`, `kubectl apply`, `systemctl restart nginx`.

Normally you type these by hand. Automation means your **Python script types them for you** — at 3 AM, inside a CI pipeline, on 50 servers simultaneously. `subprocess` is the bridge between Python and the terminal.

### The four key parameters

Before writing code, understand what each parameter controls.

**`capture_output`** — should Python capture the output or let it print freely?

- `False` (default) — output goes directly to your terminal screen. Python cannot read or store it.
- `True` — Python intercepts the output silently. It lands in `result.stdout` and `result.stderr` where your script can read, log, and act on it.

**Real-world analogy:** `capture_output=False` is like a colleague answering out loud — everyone hears it. `capture_output=True` is like getting a written note — only you see it, and you can keep it.

---

**`text`** — what format should the output be in?

- `False` (default) — output comes back as raw bytes: `b"On branch main\n"`. You cannot search or split bytes like a normal string.
- `True` — Python decodes the bytes into a readable string: `"On branch main\n"`. Always use `text=True` unless you are handling binary files like images.

---

**`check`** — what should Python do if the command fails?

- `False` (default) — Python continues running no matter what. You check `result.returncode` yourself and decide what to do.
- `True` — if the command fails, Python immediately raises a `CalledProcessError` and stops your script. Use this when a command **must** succeed for the script to be safe.

---

**`returncode`** — every terminal command finishes and sends a number back to Python.

- `0` means success — the command worked perfectly.
- Any other number means failure — the number tells you the type of failure.

This is a convention followed by every terminal program: `git`, `docker`, `kubectl`, `npm`, `pip` — all of them. Think of it like a delivery confirmation SMS: `0` means "delivered successfully," `1` means "address not found."

---

### Real DevOps scenario

> A CI/CD pipeline script: verify git is clean, pull latest code, check kubectl cluster health. Log everything. Stop if any critical step fails.

First, set up the logger (covered fully in Module 4):

```python
import subprocess
import logging
from pathlib import Path

logger = logging.getLogger("ci_pipeline")
logger.setLevel(logging.DEBUG)
formatter = logging.Formatter("%(asctime)s [%(levelname)s] %(name)s — %(message)s")
console_handler = logging.StreamHandler()
console_handler.setFormatter(formatter)
file_handler = logging.FileHandler("ci_pipeline.log")
file_handler.setFormatter(formatter)
logger.addHandler(console_handler)
logger.addHandler(file_handler)
```

Next, build a reusable helper function that runs any terminal command safely. The `critical` parameter controls what happens on failure — `True` stops the whole pipeline, `False` logs the error and continues:

```python
def run_command(command: list, step_name: str, critical: bool = True):
    """Run a terminal command and log the result."""
    logger.info(f"Running step: {step_name}")
    logger.debug(f"Command: {' '.join(command)}")

    try:
        result = subprocess.run(
            command,
            capture_output=True,
            text=True,
            check=False,
            timeout=60
        )

        if result.returncode == 0:
            logger.info(f"✓ {step_name} passed")
            if result.stdout.strip():
                logger.debug(f"Output:\n{result.stdout.strip()}")
            return result
        else:
            logger.error(
                f"✗ {step_name} failed "
                f"(exit code: {result.returncode})\n"
                f"stderr: {result.stderr.strip()}"
            )
            if critical:
                raise RuntimeError(f"Critical step '{step_name}' failed. Pipeline stopped.")
            return None

    except subprocess.TimeoutExpired:
        logger.error(f"✗ {step_name} timed out after 60 seconds")
        if critical:
            raise
        return None

    except FileNotFoundError:
        logger.error(f"✗ Command not found: '{command[0]}' — is it installed and in PATH?")
        if critical:
            raise
        return None
```

Notice that `command` is a **list**, not a string. `subprocess.run(["git", "status"])` is safe. `subprocess.run("git status")` passes the command through the OS shell, which allows shell injection attacks — a malicious input could run extra commands. Always use a list.

We use `check=False` and inspect `result.returncode` ourselves so we can log the exact error and decide whether to stop or continue based on the `critical` flag.

On failure we always log `result.stderr`, not `result.stdout` — programs print their error messages to the error channel, not the normal output channel.

Now use the helper to build the pipeline:

```python
def run_ci_pipeline():
    logger.info("=" * 60)
    logger.info("CI Pipeline started")
    logger.info("=" * 60)

    result = run_command(
        command=["git", "status"],
        step_name="Git Status Check",
        critical=True
    )

    if result and "nothing to commit" not in result.stdout:
        logger.warning("Working directory has uncommitted changes — aborting deploy")
        raise RuntimeError("Git working tree is not clean.")

    run_command(["git", "pull", "origin", "main"], "Git Pull", critical=True)

    result = run_command(["kubectl", "get", "nodes"], "Kubernetes Node Health", critical=False)
    if result:
        node_count = result.stdout.count("Ready")
        logger.info(f"Cluster has {node_count} Ready node(s)")

    run_command(
        command=["python", "-m", "pytest", "tests/", "--tb=short"],
        step_name="Unit Tests",
        critical=True
    )

    logger.info("=" * 60)
    logger.info("CI Pipeline completed successfully")
    logger.info("=" * 60)


if __name__ == "__main__":
    try:
        run_ci_pipeline()
    except RuntimeError as e:
        logger.critical(f"Pipeline aborted: {e}")
        raise SystemExit(1)
```

`SystemExit(1)` exits Python with return code `1`, which signals to GitHub Actions, Jenkins, or GitLab CI that the pipeline **failed** and marks the build as red.

**ci_pipeline.log:**

```
2026-04-19 10:00:00 [INFO]  ci_pipeline — CI Pipeline started
2026-04-19 10:00:00 [INFO]  ci_pipeline — Running step: Git Status Check
2026-04-19 10:00:00 [INFO]  ci_pipeline — ✓ Git Status Check passed
2026-04-19 10:00:01 [INFO]  ci_pipeline — Running step: Git Pull
2026-04-19 10:00:01 [INFO]  ci_pipeline — ✓ Git Pull passed
2026-04-19 10:00:02 [INFO]  ci_pipeline — Running step: Kubernetes Node Health
2026-04-19 10:00:02 [INFO]  ci_pipeline — ✓ Kubernetes Node Health passed
2026-04-19 10:00:02 [INFO]  ci_pipeline — Cluster has 3 Ready node(s)
2026-04-19 10:00:05 [INFO]  ci_pipeline — Running step: Unit Tests
2026-04-19 10:00:05 [INFO]  ci_pipeline — ✓ Unit Tests passed
2026-04-19 10:00:05 [INFO]  ci_pipeline — CI Pipeline completed successfully
```

---

## Module 3 — `json`: Read and Write Config Files

### Why you need this

JSON is the **universal language of the internet**. Every API response, every config file, every Kubernetes manifest speaks JSON.

Think of JSON as a standardized government form with fixed fields. A Python dictionary is that form filled out on your desk. `json.dump()` photocopies it into the standard format and files it away. `json.load()` takes the filed form and brings it back to your desk as a Python dictionary.

### Real DevOps scenario

> Build a deployment config manager that writes environment-specific configs, validates required fields, and reads them back safely.

---

### Writing — Python dictionary to JSON file

`open()` with `"w"` mode creates the file if it does not exist, or overwrites it if it does. Always wrap it in a `with` block — this guarantees the file is closed when the block ends, even if an error occurs mid-write.

```python
import json
import os
import logging
from pathlib import Path

logger = logging.getLogger("config_manager")
logger.setLevel(logging.INFO)
formatter = logging.Formatter("%(asctime)s [%(levelname)s] %(name)s — %(message)s")
console_handler = logging.StreamHandler()
console_handler.setFormatter(formatter)
logger.addHandler(console_handler)


def write_config(config: dict, path: Path) -> None:
    """Write a Python dictionary to a JSON file safely."""
    path.parent.mkdir(parents=True, exist_ok=True)

    with open(path, "w", encoding="utf-8") as file:
        json.dump(config, file, indent=2, ensure_ascii=False)

    logger.info(f"Config written to {path}")
```

`json.dump()` converts the Python dictionary to JSON text and writes it to the file. Two important arguments:

- `indent=2` — adds 2-space indentation so the file is human-readable. Without it, everything is on one unreadable line.
- `ensure_ascii=False` — allows non-English characters to be written as-is. Without it, the Rupee symbol becomes `\u20b9`.

Always specify `encoding="utf-8"` — without it, special characters may get corrupted on Windows machines.

---

### Reading — JSON file back to Python dictionary

`open()` with `"r"` mode opens the file for reading only. Always check that the file exists first — otherwise `open()` crashes with a `FileNotFoundError`.

```python
def read_config(path: Path) -> dict:
    """Read a JSON file and return it as a Python dictionary."""
    if not path.exists():
        logger.error(f"Config file not found: {path}")
        raise FileNotFoundError(f"Config file missing: {path}")

    with open(path, "r", encoding="utf-8") as file:
        config = json.load(file)

    logger.info(f"Config loaded from {path}")
    return config
```

`json.load()` reads all the JSON text from the file and converts it into a normal Python dictionary. After this line you can access values with `config["key"]` or `config.get("key")`.

---

### Safe value access — `.get()` vs `[]`

There are two ways to read a value from a dictionary, and the difference matters:

```python
app_name = config["app_name"]
```

If `"app_name"` is missing from the dictionary, this crashes with a `KeyError`. Only use `[]` when you are 100% certain the key will always be present.

```python
replicas = config.get("replicas", 1)
timeout  = config.get("timeout_seconds", 30)
debug    = config.get("debug", False)
```

`.get(key, default)` returns the default value instead of crashing when the key is missing. Always choose a sensible default — `1` replica, `30` seconds, `False` for debug mode.

For keys that are truly required, validate them explicitly at startup rather than letting the script fail later with a confusing error:

```python
def validate_and_use_config(config: dict) -> None:
    """Validate required fields and log the loaded config."""
    required_fields = ["app_name", "environment", "replicas"]
    missing = [field for field in required_fields if field not in config]

    if missing:
        raise ValueError(f"Config is missing required fields: {missing}")

    logger.info(f"App:         {config['app_name']}")
    logger.info(f"Environment: {config['environment']}")
    logger.info(f"Replicas:    {config.get('replicas', 1)}")
    logger.info(f"Timeout:     {config.get('timeout_seconds', 30)}s")
    logger.info(f"Debug mode:  {config.get('debug', False)}")
```

This is called "fail fast" — crash loudly at startup with a clear message rather than silently deploying with broken config.

---

### Putting it all together

Read environment-specific values from the OS, build the config dictionary, write it, and read it back:

```python
deploy_config = {
    "app_name":        "payment-service",
    "environment":     os.getenv("APP_ENV", "staging"),
    "replicas":        3,
    "version":         "2.1.0",
    "timeout_seconds": 45,
    "debug":           False,
    "database": {
        "host": os.getenv("DB_HOST", "localhost"),
        "port": 5432,
        "name": "payments_db"
    }
}

config_path = Path("configs") / "deploy_config.json"

write_config(deploy_config, config_path)
loaded_config = read_config(config_path)
validate_and_use_config(loaded_config)
```

`os.getenv("APP_ENV", "staging")` reads the environment from the OS — not hardcoded. The default is `"staging"` rather than `"production"` so a missing variable never accidentally targets production.

**configs/deploy_config.json (written to disk):**

```json
{
  "app_name": "payment-service",
  "environment": "staging",
  "replicas": 3,
  "version": "2.1.0",
  "timeout_seconds": 45,
  "debug": false,
  "database": {
    "host": "localhost",
    "port": 5432,
    "name": "payments_db"
  }
}
```

---

## Module 4 — `logging`: Track Script Health (Production-Grade)

### Why `print()` is not enough in production

`print("Deploy started")` works fine on your laptop. In production it falls apart:

- No timestamp — when exactly did this happen?
- No severity — is this normal info or a critical error?
- Goes to terminal only — lost forever when the terminal closes
- Cannot be filtered — you see everything or nothing

`logging` solves all of this. Every line gets a timestamp, a severity level, and a logger name automatically. You can write to a file and filter by severity — show only `ERROR` and above in production, `DEBUG` and above in development.

### The five severity levels

Use the right level for the right situation:

| Level | When to use | Example |
|---|---|---|
| `DEBUG` | Detailed developer notes, never in production | `"Opening file at /etc/app/config.json"` |
| `INFO` | Normal expected operations | `"Deployment started for version 2.1.0"` |
| `WARNING` | Something unexpected but recovered | `"Timeout not set — using default 30s"` |
| `ERROR` | A real problem that needs fixing | `"Database connection failed after 3 retries"` |
| `CRITICAL` | Script cannot continue — wake someone up | `"Disk full — deployment aborted"` |

### Real DevOps scenario

> Build a production-grade logger — one named logger per module, different levels for console vs file, environment-aware configuration.

---

### Step 1 — Create a named logger

```python
import logging
import os
from pathlib import Path


def create_logger(name: str, log_file: str = None) -> logging.Logger:
    """Factory function that creates a production-grade named logger."""
    logger = logging.getLogger(name)
```

`logging.getLogger(name)` creates a logger with a specific name. If you call it twice with the same name, Python returns the same object — so it is safe to call from multiple files.

Why not use `logging.basicConfig()`? That configures one global logger shared by all code — including third-party libraries like `boto3` or `requests`. Their debug messages flood your log. Named loggers are isolated — only your code writes to your logger.

---

### Step 2 — Set the log level based on environment

```python
    env = os.getenv("APP_ENV", "development")
    log_level = logging.DEBUG if env == "development" else logging.INFO
    logger.setLevel(log_level)
```

`setLevel()` sets the minimum severity the logger will process. Messages below this level are discarded before they even reach the handlers. In development you want every detail. In production you want only meaningful events.

---

### Step 3 — Create the formatter

```python
    formatter = logging.Formatter(
        fmt="%(asctime)s [%(levelname)-8s] %(name)s — %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S"
    )
```

The formatter controls how each log line looks. The `-8s` after `levelname` left-aligns the level name in an 8-character column so all lines stay vertically aligned:

```
2026-04-19 10:00:00 [INFO    ] deploy_pipeline — Step started
2026-04-19 10:00:01 [WARNING ] deploy_pipeline — Using default value
2026-04-19 10:00:02 [CRITICAL] deploy_pipeline — Disk full
```

---

### Step 4 — Add handlers

A handler decides where the log messages go. Add one for the terminal and one for a file:

```python
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.DEBUG)
    console_handler.setFormatter(formatter)
    logger.addHandler(console_handler)

    if log_file:
        log_path = Path(log_file)
        log_path.parent.mkdir(parents=True, exist_ok=True)

        file_handler = logging.FileHandler(
            filename=log_path,
            mode="a",
            encoding="utf-8"
        )
        file_handler.setLevel(logging.WARNING)
        file_handler.setFormatter(formatter)
        logger.addHandler(file_handler)

    logger.propagate = False
    return logger
```

The file handler uses `mode="a"` (append) — new log lines are added to the end of the file, never overwriting history. Always use append mode in production.

The file handler level is set to `WARNING` — `INFO` and `DEBUG` would create enormous log files very quickly. The terminal still shows `INFO` and above for the human watching live.

`logger.propagate = False` stops messages from being passed up to the root logger, which would print them a second time if `basicConfig()` was called anywhere.

---

### Using the logger in a deployment script

```python
logger = create_logger(name="deploy_pipeline", log_file="logs/deploy.log")


def deploy_application(version: str, environment: str) -> bool:
    """Deploy the application with every step logged."""
    logger.info(f"Starting deployment: version={version} environment={environment}")

    if environment not in ["staging", "production"]:
        logger.error(f"Invalid environment '{environment}' — must be staging or production")
        return False

    if environment == "production":
        logger.warning("Deploying to PRODUCTION — double-check version and approvals")

    try:
        logger.info("Step 1/4 — pulling Docker image")
        logger.info("Step 2/4 — running database migrations")
        logger.info("Step 3/4 — updating Kubernetes deployment")
        logger.info("Step 4/4 — running smoke tests")
        logger.info(f"Deployment successful: {version} is live on {environment}")
        return True

    except Exception as e:
        logger.critical(f"Deployment FAILED: {e}", exc_info=True)
        return False


deploy_application(version="2.1.0", environment="production")
```

`exc_info=True` attaches the full Python traceback to the log line. Without it you see the error message but not where it happened. With it you see the file name, line number, and full call chain. Always use `exc_info=True` inside an `except` block on `ERROR` and `CRITICAL`.

**Terminal output (APP_ENV=production):**

```
2026-04-19 10:00:00 [INFO    ] deploy_pipeline — Starting deployment: version=2.1.0 environment=production
2026-04-19 10:00:00 [WARNING ] deploy_pipeline — Deploying to PRODUCTION — double-check version and approvals
2026-04-19 10:00:01 [INFO    ] deploy_pipeline — Step 1/4 — pulling Docker image
2026-04-19 10:00:02 [INFO    ] deploy_pipeline — Step 2/4 — running database migrations
2026-04-19 10:00:03 [INFO    ] deploy_pipeline — Step 3/4 — updating Kubernetes deployment
2026-04-19 10:00:04 [INFO    ] deploy_pipeline — Step 4/4 — running smoke tests
2026-04-19 10:00:05 [INFO    ] deploy_pipeline — Deployment successful: 2.1.0 is live on production
```

**logs/deploy.log (WARNING and above only):**

```
2026-04-19 10:00:00 [WARNING ] deploy_pipeline — Deploying to PRODUCTION — double-check version and approvals
```

---

## Complete Production Script — All Modules Together

All four modules combined into one unified script. Read through it now — every pattern should look familiar.

```python
import os
import json
import logging
import subprocess
from pathlib import Path


def create_logger(name: str, log_file: str) -> logging.Logger:
    """Reusable production logger factory."""
    logger = logging.getLogger(name)
    env = os.getenv("APP_ENV", "development")
    logger.setLevel(logging.DEBUG if env == "development" else logging.INFO)
    fmt = logging.Formatter(
        "%(asctime)s [%(levelname)-8s] %(name)s — %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S"
    )
    ch = logging.StreamHandler()
    ch.setLevel(logging.DEBUG)
    ch.setFormatter(fmt)
    Path(log_file).parent.mkdir(parents=True, exist_ok=True)
    fh = logging.FileHandler(log_file, mode="a", encoding="utf-8")
    fh.setLevel(logging.WARNING)
    fh.setFormatter(fmt)
    logger.addHandler(ch)
    logger.addHandler(fh)
    logger.propagate = False
    return logger


logger = create_logger("devops_automator", "logs/pipeline.log")


def run_command(command: list, step_name: str, critical: bool = True):
    logger.info(f"Running: {step_name}")
    logger.debug(f"Command: {' '.join(command)}")
    try:
        result = subprocess.run(
            command,
            capture_output=True,
            text=True,
            check=False,
            timeout=60
        )
        if result.returncode == 0:
            logger.info(f"✓ {step_name} passed")
            return result
        else:
            logger.error(f"✗ {step_name} failed (code {result.returncode}): {result.stderr.strip()}")
            if critical:
                raise RuntimeError(f"Critical step failed: {step_name}")
            return None
    except subprocess.TimeoutExpired:
        logger.error(f"✗ {step_name} timed out")
        if critical:
            raise
        return None
    except FileNotFoundError:
        logger.error(f"✗ Command not found: {command[0]}")
        if critical:
            raise
        return None


def setup_directories(base_path: Path) -> Path:
    releases = base_path / "releases"
    releases.mkdir(parents=True, exist_ok=True)
    logger.info(f"Directories ready: {releases}")
    return releases


def write_deploy_config(releases: Path) -> Path:
    config = {
        "app_name":    "payment-service",
        "version":     os.getenv("APP_VERSION", "0.0.0"),
        "environment": os.getenv("APP_ENV", "staging"),
        "replicas":    int(os.getenv("REPLICAS", "2")),
        "debug":       os.getenv("APP_ENV") == "development"
    }
    cfg_path = releases / "config.json"
    with open(cfg_path, "w", encoding="utf-8") as f:
        json.dump(config, f, indent=2, ensure_ascii=False)
    logger.info(f"Config written: {cfg_path}")
    return cfg_path


def run_pipeline():
    logger.info("=" * 60)
    logger.info("DevOps Automation Pipeline — starting")
    logger.info("=" * 60)

    releases = setup_directories(Path("/var/app"))
    write_deploy_config(releases)

    run_command(["git", "status"],                          "Git Status",      critical=True)
    run_command(["git", "pull", "origin", "main"],          "Git Pull",        critical=True)
    run_command(["python", "-m", "pytest", "tests/", "-q"], "Unit Tests",      critical=True)
    run_command(["kubectl", "get", "nodes"],                 "K8s Node Health", critical=False)

    logger.info("=" * 60)
    logger.info("Pipeline completed — ready to deploy")
    logger.info("=" * 60)


if __name__ == "__main__":
    try:
        run_pipeline()
    except (RuntimeError, Exception) as e:
        logger.critical(f"Pipeline aborted: {e}", exc_info=True)
        raise SystemExit(1)
```

---

## Key Takeaways

| Rule | Why |
|---|---|
| Never hardcode secrets — use `os.getenv()` | Secrets in code leak when pushed to GitHub |
| Always use `pathlib.Path` with `/` operator | Works on Windows, Mac, Linux — string concat breaks |
| Pass commands as **lists** to `subprocess.run()` | Prevents shell injection attacks |
| Always set `capture_output=True, text=True` | So you can read, log, and act on command output |
| Use `check=False` and manual `returncode` check | Full control over failure handling |
| Always check `result.stderr` on failure | Error messages go to stderr, not stdout |
| Use `json.dump(indent=2)` when writing files | Human-readable config files are easier to debug |
| Always use `.get(key, default)` for optional keys | Prevents KeyError crashes on missing config fields |
| Replace `print()` with `logging` before production | Timestamps, severity, file output, filtering |
| Use `getLogger(__name__)` not `basicConfig()` | Isolated logger — no third-party library pollution |
| Set `logger.propagate = False` | Prevents duplicate messages in the terminal |
| Use `exc_info=True` on ERROR/CRITICAL | Attaches full traceback to the log line |

---

*Phase 1 complete. Next: Phase 2 — Cloud SDK and API Calls.*