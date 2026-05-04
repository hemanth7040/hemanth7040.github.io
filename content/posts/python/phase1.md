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
> Every parameter, every keyword, every line is explained in plain English.
> Written for beginners — including people from non-IT backgrounds.

---

## What You Will Learn

| Module | What it does |
|---|---|
| `os` & `pathlib` | Move files, check directories, read secret keys from the OS |
| `subprocess` | Run any terminal command (`git`, `kubectl`, `docker`) from inside Python |
| `json` | Convert Python data ↔ JSON files — the language every API speaks |
| `logging` | Write timestamped records of every action (the professional alternative to `print()`) |

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

Think of Python as a very smart assistant sitting inside your computer.
The **terminal** (also called command prompt, shell, or CLI) is the black screen
where you type commands like `git status` or `ls` and the computer responds.

Normally a human types those commands by hand.
`subprocess` lets your Python script type those commands *automatically* — no human needed.

### What is stdout vs stderr?

Every program that runs in a terminal has two separate output channels:

```
stdout  (Standard Output)
        "The normal result" — what the program prints when everything is OK
        Example: git status printing "On branch main"

stderr  (Standard Error)
        "The error channel" — what the program prints when something goes wrong
        Example: git printing "fatal: not a git repository"
```

**Real-world analogy:**
Imagine calling a bank helpline.

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

```python
import os
# `import` means: "go fetch this toolbox and make it available in this script."
# `os` is a toolbox built into Python — no installation needed.
# It gives you tools to interact with the Operating System (Linux / Mac / Windows).

from pathlib import Path
# `from pathlib import Path` means: "from the pathlib toolbox, give me only the Path tool."
# Path is a special object that represents a file or folder location.
# It is smarter than a plain string like "/var/app/releases" because it understands
# how file systems work and adjusts automatically for each operating system.


# -----------------------------------------------------------------
# ENVIRONMENT VARIABLES — Where you store secrets safely
# -----------------------------------------------------------------

# Bad practice — NEVER do this:
# api_key = "sk-abc123secret"   <- hardcoded secret in your code
# If you push this to GitHub, the whole world can see your API key.

# Good practice — read it from the OS instead:
api_key = os.getenv("DEPLOY_API_KEY", "not-set")

# Breaking this down word by word:
# os          -> the os toolbox
# .getenv()   -> a function inside os that reads environment variables
# "DEPLOY_API_KEY" -> the NAME of the variable to look up in the OS
#                    (someone set this earlier by running: export DEPLOY_API_KEY=sk-abc123)
# "not-set"   -> the DEFAULT VALUE — if DEPLOY_API_KEY does not exist in the OS,
#               return "not-set" instead of crashing with an error.
#               Always provide a default so your script fails gracefully.

print(f"API key loaded: {'YES' if api_key != 'not-set' else 'MISSING — check your environment'}")
# f-string: the f before the quote means "format this string".
# Anything inside {} is evaluated as Python code and inserted into the string.
# Here we check: if api_key is not "not-set", print YES, otherwise print the warning.


# -----------------------------------------------------------------
# PATH OPERATIONS — Navigate and create directories safely
# -----------------------------------------------------------------

deploy_dir = Path("/var/app/releases")
# Path() creates a Path object — a smart reference to a folder location.
# IMPORTANT: Nothing happens on disk at this line.
# You are just telling Python "I want to work with this folder."
# Think of it like writing an address on a piece of paper — you have not gone there yet.

print(deploy_dir)           # -> /var/app/releases
print(deploy_dir.name)      # -> releases          (just the folder name)
print(deploy_dir.parent)    # -> /var/app           (the folder that contains it)


if not deploy_dir.exists():
    # .exists() asks the OS: "Does this folder actually exist on disk right now?"
    # Returns True if yes, False if no.
    # `not` flips the answer:
    #   not True  -> False  (folder exists     -> skip this block)
    #   not False -> True   (folder is missing -> run this block)

    deploy_dir.mkdir(parents=True, exist_ok=True)
    # .mkdir() tells the OS: "Create this folder now."
    #
    # parents=True  -> What this means:
    #   Imagine you want to create /var/app/releases
    #   but /var/app does not exist either.
    #   parents=True says: "Create ALL missing parent folders along the way."
    #   Without parents=True -> Python crashes saying /var/app does not exist.
    #   With    parents=True -> Python creates /var, then /var/app, then /var/app/releases.
    #
    # exist_ok=True -> What this means:
    #   If the folder ALREADY exists, do nothing — do not raise an error.
    #   Without exist_ok=True -> running mkdir() twice crashes with "folder already exists."
    #   With    exist_ok=True -> running mkdir() twice is safe — it silently skips.
    #   Rule of thumb: always set both parents=True and exist_ok=True in DevOps scripts.

    print(f"Created: {deploy_dir}")


config_file = deploy_dir / "config.json"
# The / operator between two Path objects (or a Path and a string) JOINS them.
# Result: /var/app/releases/config.json
#
# Why not just write: "/var/app/releases" + "/" + "config.json"  ?
# Because string concatenation breaks on Windows (which uses \ not /).
# Path's / operator always uses the correct separator for the current OS.
# This single habit makes your scripts portable across all operating systems.

print(f"Config path: {config_file}")
# -> Config path: /var/app/releases/config.json


# -----------------------------------------------------------------
# READING AND WRITING FILES with pathlib
# -----------------------------------------------------------------

log_file = Path("/var/log/deploy/app.log")
log_file.parent.mkdir(parents=True, exist_ok=True)
# Always create the parent directory before writing a file.
# log_file.parent -> /var/log/deploy
# .mkdir()        -> create that folder if it does not exist

log_file.write_text("Deploy started\n")
# .write_text() creates the file and writes the string into it in one step.
# If the file already exists, it overwrites it completely.
# \n is a newline character — moves the cursor to the next line.

content = log_file.read_text()
# .read_text() opens the file and returns its entire content as a single string.
# No need to open/close manually — Path handles that for you.

print(content)  # -> Deploy started
```

**Terminal output:**

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

As a DevOps engineer you will spend a lot of time running terminal commands:
`git pull`, `docker build`, `kubectl apply`, `systemctl restart nginx`.

Normally you type these by hand. But automation means your **Python script types them for you** — at 3 AM, inside a CI pipeline, on 50 servers simultaneously.

`subprocess` is the bridge between Python and the terminal.

### The anatomy of subprocess — every parameter explained

Before the code, understand the three most important switches:

```
subprocess.run(command, capture_output, text, check)

capture_output  ->  Should Python CAPTURE the output or let it print freely?
text            ->  What FORMAT should the output be in?
check           ->  What happens on FAILURE?
```

#### `capture_output` — Should Python capture the output?

```
capture_output=False  (default)
    The command prints directly to YOUR terminal screen.
    Python does NOT save the output anywhere.
    You can see it with your eyes but cannot use it in your script.

    Use when: you just want to run a command and see its output live.
    Example: running a long database migration where you watch the progress.

capture_output=True
    Python intercepts the output BEFORE it reaches your screen.
    The output is saved silently inside result.stdout and result.stderr.
    Nothing appears on your terminal screen.
    You CAN read, process, search, and log the output inside your script.

    Use when: you need to check the output, parse it, or store it in a log file.
    Example: checking if "On branch main" is in git status output.
```

**Real-world analogy:**

- `capture_output=False` — You ask a colleague a question and they answer out loud. Everyone in the office hears it.
- `capture_output=True`  — You ask the same question but they hand you a written note. Only you see the answer — and you can keep the note.

#### `text` — What format should the output be in?

```
text=False  (default)
    Output is returned as RAW BYTES — the computer's native binary format.
    result.stdout -> b"On branch main\n"
                     The b prefix means BYTES — not a normal string.
    You cannot do normal string operations like .split() or .upper()

text=True
    Python automatically DECODES the bytes into a normal readable string.
    result.stdout -> "On branch main\n"
    Now you can search it, split it, print it, compare it — just like any other string.

    Always set text=True unless you are dealing with binary data (images, executables).
```

**Real-world analogy:**

- `text=False` — You receive a fax but it comes out as raw transmission signals. Unreadable.
- `text=True`  — The fax machine decodes those signals into readable printed text automatically.

#### `check` — What should Python do if the command fails?

```
check=False  (default)
    Python runs the command and returns the result NO MATTER WHAT.
    Even if the command failed (returncode != 0), Python continues running.
    YOU are responsible for checking result.returncode and handling the failure.

    Use when: you expect the command might fail and you want to handle it yourself.
    Example: checking git status in a folder that might not be a git repo.

check=True
    If the command fails (returncode != 0), Python automatically raises an exception
    called CalledProcessError and STOPS your script immediately.
    You do not need to manually check result.returncode.

    Use when: the command MUST succeed for the script to continue safely.
    Example: running git pull before a deployment — if it fails, do NOT deploy.
```

**Real-world analogy:**

- `check=False` — You ask a contractor to build a wall. If they fail, you personally inspect the work and decide what to do next.
- `check=True`  — You ask the same contractor but tell them upfront: "If anything goes wrong, stop work immediately and call me." They halt automatically.

#### `returncode` — The command's exit signal

```
Every terminal command finishes and sends a number back to Python.
This number is called the return code or exit code.

returncode == 0    -> SUCCESS  (the command worked perfectly)
returncode != 0    -> FAILURE  (something went wrong)

This is a Unix/Linux convention used by EVERY terminal program:
git, docker, kubectl, npm, pip — all of them follow this same rule.
```

**Real-world analogy:**
Think of returncode like a delivery confirmation SMS.

- `0`   — "Your package was delivered successfully."
- `1`   — "Delivery failed — address not found."
- `128` — "Delivery failed — recipient refused the package."

Different numbers mean different types of failure.

---

### Real DevOps scenario

> A CI/CD pipeline script: verify git is clean, pull latest code, check kubectl cluster health. Log everything. Stop if any critical step fails.

```python
import subprocess
import logging
from pathlib import Path

# Set up logger first (production-grade — covered fully in Module 4)
logger = logging.getLogger("ci_pipeline")
logger.setLevel(logging.DEBUG)
formatter = logging.Formatter("%(asctime)s [%(levelname)s] %(name)s — %(message)s")
console_handler = logging.StreamHandler()
console_handler.setFormatter(formatter)
file_handler = logging.FileHandler("ci_pipeline.log")
file_handler.setFormatter(formatter)
logger.addHandler(console_handler)
logger.addHandler(file_handler)


def run_command(command: list, step_name: str, critical: bool = True):
    """
    A reusable helper function that runs any terminal command safely.

    Parameters:
        command    -> the command as a list of strings e.g. ["git", "status"]
        step_name  -> a human-readable label for logging e.g. "Git Status Check"
        critical   -> if True and command fails, raise an exception and stop the script
                      if False and command fails, log the error and continue

    Returns:
        result object containing .stdout, .stderr, .returncode
        OR None if the command failed and critical=False
    """

    logger.info(f"Running step: {step_name}")
    logger.debug(f"Command: {' '.join(command)}")
    # ' '.join(command) converts ["git", "status"] back to "git status" for readable logging.

    try:
        result = subprocess.run(
            command,
            # Why a LIST and not a string?
            # subprocess.run("git status")          <- WRONG — dangerous
            # subprocess.run(["git", "status"])     <- CORRECT — safe
            #
            # With a plain string, Python passes it through the OS shell (/bin/sh).
            # A malicious input could inject extra commands — called shell injection.
            # With a list, each item is one argument. No shell. No injection risk.

            capture_output=True,
            # We want to CAPTURE the output, not print it live to the terminal.
            # Captured output lands in:
            #   result.stdout -> everything the command printed normally
            #   result.stderr -> everything the command printed as an error

            text=True,
            # Decode the output from raw bytes into a normal Python string.
            # Without text=True:
            #   result.stdout -> b"On branch main\nnothing to commit\n"  (bytes object)
            # With text=True:
            #   result.stdout -> "On branch main\nnothing to commit\n"   (string)

            check=False,
            # Do NOT auto-raise an exception on failure.
            # We handle success/failure ourselves below using result.returncode.

            timeout=60
            # Wait a maximum of 60 seconds for the command to finish.
            # Without timeout, a hanging command would freeze your script FOREVER.
            # After 60 seconds, Python raises TimeoutExpired and you can handle it.
        )

        # Check if the command succeeded
        if result.returncode == 0:
            # returncode 0 = the command completed successfully (Unix convention).
            logger.info(f"✓ {step_name} passed")

            if result.stdout.strip():
                # .strip() removes leading/trailing whitespace and newlines.
                logger.debug(f"Output:\n{result.stdout.strip()}")

            return result

        else:
            # returncode != 0 = the command failed.
            # result.stderr contains the error message the command printed.
            logger.error(
                f"✗ {step_name} failed "
                f"(exit code: {result.returncode})\n"
                f"stderr: {result.stderr.strip()}"
            )
            # Why log stderr and not stdout on failure?
            # Programs print their ERROR messages to stderr — not stdout.

            if critical:
                raise RuntimeError(f"Critical step '{step_name}' failed. Pipeline stopped.")
            return None

    except subprocess.TimeoutExpired:
        # Raised when the command takes longer than `timeout` seconds.
        logger.error(f"✗ {step_name} timed out after 60 seconds")
        if critical:
            raise
        return None

    except FileNotFoundError:
        # Raised when the command itself does not exist on the system.
        # Example: running ["kubectl", "get", "pods"] on a machine where kubectl is not installed.
        logger.error(f"✗ Command not found: '{command[0]}' — is it installed and in PATH?")
        if critical:
            raise
        return None


# -----------------------------------------------------------------
# THE CI PIPELINE — using our helper function
# -----------------------------------------------------------------

def run_ci_pipeline():
    logger.info("=" * 60)
    logger.info("CI Pipeline started")
    logger.info("=" * 60)

    # STEP 1 — Check git status (critical: must be clean before deploying)
    result = run_command(
        command=["git", "status"],
        step_name="Git Status Check",
        critical=True
        # critical=True -> if git fails, raise an exception and stop everything.
    )

    if result and "nothing to commit" not in result.stdout:
        # result.stdout contains the full text output of `git status`.
        # We check if "nothing to commit" is IN that text.
        logger.warning("Working directory has uncommitted changes — aborting deploy")
        raise RuntimeError("Git working tree is not clean.")

    # STEP 2 — Pull latest code (critical)
    run_command(
        command=["git", "pull", "origin", "main"],
        step_name="Git Pull",
        critical=True
        # If git pull fails (network issue, merge conflict), stop immediately.
    )

    # STEP 3 — Check kubectl cluster health (non-critical — informational)
    result = run_command(
        command=["kubectl", "get", "nodes"],
        step_name="Kubernetes Node Health",
        critical=False
        # critical=False -> if kubectl is not installed, log the error but CONTINUE.
    )

    if result:
        node_count = result.stdout.count("Ready")
        # result.stdout.count("Ready") counts how many times "Ready" appears.
        # Each healthy node shows "Ready" in the kubectl output.
        logger.info(f"Cluster has {node_count} Ready node(s)")

    # STEP 4 — Run tests (critical)
    run_command(
        command=["python", "-m", "pytest", "tests/", "--tb=short"],
        step_name="Unit Tests",
        critical=True
        # Never deploy without passing tests.
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
        # SystemExit(1) exits the Python process with return code 1.
        # This signals to the CI system (GitHub Actions, Jenkins, GitLab CI)
        # that the pipeline FAILED — it will mark the build as red/failed.
        # SystemExit(0) would signal success.
```

**ci_pipeline.log:**

```
2026-04-19 10:00:00 [INFO]  ci_pipeline — ============================================================
2026-04-19 10:00:00 [INFO]  ci_pipeline — CI Pipeline started
2026-04-19 10:00:00 [INFO]  ci_pipeline — ============================================================
2026-04-19 10:00:00 [INFO]  ci_pipeline — Running step: Git Status Check
2026-04-19 10:00:00 [DEBUG] ci_pipeline — Command: git status
2026-04-19 10:00:00 [INFO]  ci_pipeline — ✓ Git Status Check passed
2026-04-19 10:00:01 [INFO]  ci_pipeline — Running step: Git Pull
2026-04-19 10:00:01 [INFO]  ci_pipeline — ✓ Git Pull passed
2026-04-19 10:00:02 [INFO]  ci_pipeline — Running step: Kubernetes Node Health
2026-04-19 10:00:02 [INFO]  ci_pipeline — ✓ Kubernetes Node Health passed
2026-04-19 10:00:02 [INFO]  ci_pipeline — Cluster has 3 Ready node(s)
2026-04-19 10:00:05 [INFO]  ci_pipeline — Running step: Unit Tests
2026-04-19 10:00:05 [INFO]  ci_pipeline — ✓ Unit Tests passed
2026-04-19 10:00:05 [INFO]  ci_pipeline — ============================================================
2026-04-19 10:00:05 [INFO]  ci_pipeline — CI Pipeline completed successfully
2026-04-19 10:00:05 [INFO]  ci_pipeline — ============================================================
```

---

## Module 3 — `json`: Read and Write Config Files

### Why you need this

JSON is the **universal language of the internet**.
Every API response, every config file, every Kubernetes manifest speaks JSON.

Think of JSON as a **standardized form** — like a government form with fixed fields.
Python dictionaries are like that form filled out on your desk (in Python's format).
`json.dump()` photocopies your filled form into the standard format and files it away.
`json.load()` takes a filed form and brings it back to your desk in Python's format.

### Real DevOps scenario

> Build a deployment config manager that writes environment-specific configs, validates required fields, and reads them back safely.

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


# -----------------------------------------------------------------
# WRITING — Python dictionary -> JSON file
# -----------------------------------------------------------------

def write_config(config: dict, path: Path) -> None:
    """Write a Python dictionary to a JSON file safely."""

    path.parent.mkdir(parents=True, exist_ok=True)
    # Always ensure the directory exists before writing a file.

    with open(path, "w", encoding="utf-8") as file:
        # `with open(...) as file:` — the context manager pattern.
        #
        # "w"              -> write mode. Creates the file if it does not exist.
        #                     Overwrites it completely if it does exist.
        #                     Use "a" (append) if you want to add to the end instead.
        #
        # encoding="utf-8" -> always specify UTF-8 encoding.
        #                     Without this, special characters (e, n, Rs, Chinese)
        #                     may get corrupted on Windows machines.
        #
        # The `with` block guarantees the file is CLOSED when the block ends —
        # even if an error occurs mid-write.

        json.dump(config, file, indent=2, ensure_ascii=False)
        #
        # json.dump(data, file) -> converts Python dict to JSON text and writes to file.
        #
        # indent=2         -> add 2-space indentation to make the file human-readable.
        #                     Without indent, the entire JSON is one single unreadable line.
        #                     With indent=2, it becomes a clean multi-line format.
        #
        # ensure_ascii=False -> allow non-English characters to be written as-is.
        #                     Without this, the Rupee symbol becomes \u20b9

    logger.info(f"Config written to {path}")


# -----------------------------------------------------------------
# READING — JSON file -> Python dictionary
# -----------------------------------------------------------------

def read_config(path: Path) -> dict:
    """Read a JSON file and return it as a Python dictionary."""

    if not path.exists():
        # Always check if the file exists before trying to open it.
        # Without this check, open() raises FileNotFoundError and crashes.
        logger.error(f"Config file not found: {path}")
        raise FileNotFoundError(f"Config file missing: {path}")

    with open(path, "r", encoding="utf-8") as file:
        # "r" -> read mode. Opens the file for reading only.
        #        Python will raise an error if you try to write in read mode.

        config = json.load(file)
        # json.load(file) -> reads ALL the JSON text from the file
        #                    and converts it into a Python dictionary.
        # After this line, `config` is a normal Python dict.
        # You can access values with config["key"] or config.get("key").

    logger.info(f"Config loaded from {path}")
    return config


# -----------------------------------------------------------------
# SAFE VALUE ACCESS — .get() vs [] — the difference that prevents crashes
# -----------------------------------------------------------------

def validate_and_use_config(config: dict) -> None:
    """Show the safe way to read values from a config dictionary."""

    # Method 1 — Square brackets [] — UNSAFE for optional keys
    app_name = config["app_name"]
    # If "app_name" EXISTS in the dict -> returns its value. Fine.
    # If "app_name" is MISSING from the dict -> raises KeyError and CRASHES.
    # Only use [] when you are 100% certain the key will always be present.

    # Method 2 — .get(key, default) — SAFE for optional or uncertain keys
    replicas = config.get("replicas", 1)
    # If "replicas" EXISTS -> returns its value.
    # If "replicas" is MISSING -> returns 1 (the default) instead of crashing.

    timeout = config.get("timeout_seconds", 30)
    # Not every config file will have "timeout_seconds".
    # .get() handles this gracefully. 30 seconds is a safe default.

    debug_mode = config.get("debug", False)
    # Boolean config value. Default False is safe — do not accidentally run
    # production with debug mode on.

    # Validate required fields — raise early if critical config is missing
    required_fields = ["app_name", "environment", "replicas"]
    missing = [field for field in required_fields if field not in config]
    # List comprehension: for each field in our required list,
    # check if it is NOT in the config dict. Collect the missing ones.

    if missing:
        raise ValueError(f"Config is missing required fields: {missing}")
        # Raise an error immediately with a clear message listing what is missing.
        # This is called "fail fast" — better to crash loudly at startup
        # than silently deploy with broken config.

    logger.info(f"App:         {app_name}")
    logger.info(f"Environment: {config['environment']}")
    logger.info(f"Replicas:    {replicas}")
    logger.info(f"Timeout:     {timeout}s")
    logger.info(f"Debug mode:  {debug_mode}")


# -----------------------------------------------------------------
# REAL USAGE — putting it all together
# -----------------------------------------------------------------

deploy_config = {
    "app_name": "payment-service",
    "environment": os.getenv("APP_ENV", "staging"),
    # Read environment from OS variable — not hardcoded.
    # os.getenv() default is "staging" — safer than "production" as a fallback.
    "replicas": 3,
    "version": "2.1.0",
    "timeout_seconds": 45,
    "debug": False,
    "database": {
        "host": os.getenv("DB_HOST", "localhost"),
        "port": 5432,
        "name": "payments_db"
    }
    # Nested dict -> nested JSON object. json.dump handles this automatically.
}

config_path = Path("configs") / "deploy_config.json"
# Builds: configs/deploy_config.json
# On Windows this becomes: configs\deploy_config.json automatically.

write_config(deploy_config, config_path)
loaded_config = read_config(config_path)
validate_and_use_config(loaded_config)
```

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

Using `print()` has serious problems in production:

- No timestamp — when exactly did this happen?
- No severity — is this normal info or a critical error?
- No file name — which script printed this?
- Goes to terminal only — lost forever when the terminal closes
- Cannot be filtered — you see everything or nothing

Using `logging` gives you all of that for free:

- Automatic timestamp on every line
- Severity level (DEBUG / INFO / WARNING / ERROR / CRITICAL)
- Logger name tells you which module produced the message
- Writes to file — survives terminal close, server reboot
- Filter by level — show only ERROR and above in production, DEBUG and above in development

### The five severity levels — with real examples

```
DEBUG    -> "I am about to open the file at /etc/app/config.json"
            Extremely detailed developer notes. Never needed in production.

INFO     -> "Deployment pipeline started for version 2.1.0"
            Normal, expected operations. The script is working as designed.

WARNING  -> "Config key 'timeout' not found — using default 30s"
            Something unexpected happened but the script recovered and continued.

ERROR    -> "Failed to connect to database after 3 retries"
            A real problem that needs fixing.

CRITICAL -> "Disk 100% full — cannot write logs — deployment aborted"
            The script cannot continue at all. Requires IMMEDIATE human action.
```

### Real DevOps scenario

> Build a production-grade logger used by a real deployment pipeline — one named logger per module, different levels for console vs file, environment-aware configuration.

```python
import logging
import os
from pathlib import Path


def create_logger(name: str, log_file: str = None) -> logging.Logger:
    """
    Factory function that creates a production-grade named logger.

    Parameters:
        name     -> the logger's name — appears in every log line.
                    Use the module name so you know where each message came from.
                    Convention: pass __name__ when calling from another module.
        log_file -> optional path to a log file.
                    If None, logs only to the terminal.

    Returns:
        A configured Logger object ready to use.
    """

    # STEP 1: Create the logger
    logger = logging.getLogger(name)
    # logging.getLogger(name) creates a NEW logger with the given name.
    # If you call getLogger("deploy") twice, you get the SAME logger object
    # both times — Python reuses it. So it is safe to call in multiple files.
    #
    # Why NOT use the root logger (logging.basicConfig)?
    # basicConfig() configures ONE global logger shared by ALL code —
    # including third-party libraries you import (boto3, requests, paramiko).
    # Their debug messages flood YOUR log file.
    # Named loggers are isolated — only YOUR code writes to YOUR logger.

    # STEP 2: Set the logger level
    env = os.getenv("APP_ENV", "development")
    log_level = logging.DEBUG if env == "development" else logging.INFO
    # In development: show EVERYTHING including debug details.
    # In production:  show INFO and above only — skip noisy debug lines.
    logger.setLevel(log_level)
    # setLevel() sets the MINIMUM severity the logger will process.
    # Messages below this level are discarded before they reach the handlers.

    # STEP 3: Create the formatter
    formatter = logging.Formatter(
        fmt="%(asctime)s [%(levelname)-8s] %(name)s — %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S"
    )
    # Formatter defines how EACH LOG LINE looks.
    #
    # fmt tokens — each is replaced automatically:
    # %(asctime)s    -> timestamp: "2026-04-19 10:00:01"
    # %(levelname)s  -> severity word: "INFO", "ERROR", etc.
    # -8s            -> left-align the level name in an 8-character wide column:
    #                   [INFO    ]
    #                   [WARNING ]
    #                   [CRITICAL]
    # %(name)s       -> the logger name you set in getLogger("deploy_pipeline")
    # %(message)s    -> the actual message you passed to logger.info(...)

    # STEP 4a: Console handler — logs to terminal
    console_handler = logging.StreamHandler()
    # StreamHandler() sends log messages to the terminal (stdout by default).

    console_handler.setLevel(logging.DEBUG)
    # Handler-level filter — separate from the logger-level filter above.
    # Logger level:  messages below this are DISCARDED completely.
    # Handler level: messages below this are not sent to THIS handler,
    #                but may still go to other handlers.

    console_handler.setFormatter(formatter)
    # Attach the formatter — tell this handler how to format each line.

    logger.addHandler(console_handler)
    # addHandler() connects the handler to the logger.

    # STEP 4b: File handler — logs to disk (optional)
    if log_file:
        log_path = Path(log_file)
        log_path.parent.mkdir(parents=True, exist_ok=True)
        # Ensure the log directory exists before creating the file handler.

        file_handler = logging.FileHandler(
            filename=log_path,
            mode="a",
            encoding="utf-8"
        )
        # FileHandler(filename) writes every log line to a file on disk.
        #
        # mode="a" -> APPEND mode. New log lines are added to the END of the file.
        #             The file is never overwritten — you keep the full history.
        #             Use mode="w" only if you want a fresh log file every time.
        #             In production: always use "a".

        file_handler.setLevel(logging.WARNING)
        # File only receives WARNING and above.
        # Reason: INFO and DEBUG create huge log files very quickly.

        file_handler.setFormatter(formatter)
        logger.addHandler(file_handler)

    # Prevent duplicate log messages
    logger.propagate = False
    # By default, a named logger passes its messages UP to the root logger,
    # which may print them again if basicConfig() was called anywhere.
    # propagate=False stops that — your logger handles its own messages only.

    return logger


# -----------------------------------------------------------------
# USING THE LOGGER — in a real deployment script
# -----------------------------------------------------------------

logger = create_logger(
    name="deploy_pipeline",
    log_file="logs/deploy.log"
)

def deploy_application(version: str, environment: str) -> bool:
    """Deploy the application — every step logged with appropriate severity."""

    logger.info(f"Starting deployment: version={version} environment={environment}")
    # INFO — normal start message. Confirms the function was called with correct args.

    logger.debug(f"Reading config from configs/{environment}.json")
    # DEBUG — low-level detail. Only visible in development (APP_ENV=development).

    if environment not in ["staging", "production"]:
        logger.error(f"Invalid environment '{environment}' — must be staging or production")
        # ERROR — invalid input that prevents the deploy from proceeding.
        return False

    if environment == "production":
        logger.warning("Deploying to PRODUCTION — double-check version and approvals")
        # WARNING — not an error, but demands the human's attention.

    try:
        logger.info("Step 1/4 — pulling Docker image")
        logger.info("Step 2/4 — running database migrations")
        logger.info("Step 3/4 — updating Kubernetes deployment")
        logger.info("Step 4/4 — running smoke tests")

        logger.info(f"Deployment successful: {version} is live on {environment}")
        return True

    except Exception as e:
        logger.critical(
            f"Deployment FAILED catastrophically: {e}",
            exc_info=True
        )
        # exc_info=True -> attaches the full Python traceback (stack trace) to the log line.
        # Without exc_info: you see the error message but not WHERE it happened.
        # With    exc_info: you see the file name, line number, and full call chain.
        # Always use exc_info=True on CRITICAL and ERROR when inside an except block.
        return False


deploy_application(version="2.1.0", environment="production")
```

**Terminal output (APP_ENV=production — INFO level):**

```
2026-04-19 10:00:00 [INFO    ] deploy_pipeline — Starting deployment: version=2.1.0 environment=production
2026-04-19 10:00:00 [WARNING ] deploy_pipeline — Deploying to PRODUCTION — double-check version and approvals
2026-04-19 10:00:01 [INFO    ] deploy_pipeline — Step 1/4 — pulling Docker image
2026-04-19 10:00:02 [INFO    ] deploy_pipeline — Step 2/4 — running database migrations
2026-04-19 10:00:03 [INFO    ] deploy_pipeline — Step 3/4 — updating Kubernetes deployment
2026-04-19 10:00:04 [INFO    ] deploy_pipeline — Step 4/4 — running smoke tests
2026-04-19 10:00:05 [INFO    ] deploy_pipeline — Deployment successful: 2.1.0 is live on production
```

**logs/deploy.log (only WARNING and above):**

```
2026-04-19 10:00:00 [WARNING ] deploy_pipeline — Deploying to PRODUCTION — double-check version and approvals
```

---

## Complete Production Script — All Modules Together

> File: `devops_automator.py`
> One unified script: environment config, directory setup, JSON config write, CI checks via subprocess, everything logged with named logger.

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
        "environment": os.getenv("APP_ENV",     "staging"),
        "replicas":    int(os.getenv("REPLICAS", "2")),
        "debug":       os.getenv("APP_ENV") == "development"
    }
    cfg_path = releases / "config.json"
    with open(cfg_path, "w", encoding="utf-8") as f:
        json.dump(config, f, indent=2, ensure_ascii=False)
    logger.info(f"Config written: {cfg_path}")
    logger.debug(f"Config contents: {json.dumps(config)}")
    return cfg_path


def run_pipeline():
    logger.info("=" * 60)
    logger.info("DevOps Automation Pipeline — starting")
    logger.info("=" * 60)

    releases = setup_directories(Path("/var/app"))
    cfg_path  = write_deploy_config(releases)

    run_command(["git", "status"],                           "Git Status",      critical=True)
    run_command(["git", "pull", "origin", "main"],           "Git Pull",        critical=True)
    run_command(["python", "-m", "pytest", "tests/", "-q"],  "Unit Tests",      critical=True)
    run_command(["kubectl", "get", "nodes"],                  "K8s Node Health", critical=False)

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