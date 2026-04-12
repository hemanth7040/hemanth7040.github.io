---
title: "Linux Commands Every DevOps Engineer Must Know"
date: 2024-01-02
draft: false
categories: ["linux"]
description: "A practical reference of must-know Linux commands for DevOps and MLOps engineers — covering files, processes, networking, Docker, Kubernetes, and log analysis."
---

> Every command below is explained in plain English. If you are new to Linux, read the "What this does" section under each command — don't just memorize the syntax, understand **why** you are running it.

---

## How to Read Command Syntax

Before we start, here is how to read the examples in this guide:

```
command  -flag  argument
   |       |       |
   |       |    What you operate on (a file, a name, a path)
   |    Modifies how the command behaves
The program you are running
```

- A `-` followed by a single letter is a **short flag**: `-l`, `-a`, `-h`
- `--` followed by a word is a **long flag**: `--all`, `--follow`, `--namespace`
- You can combine short flags: `-lah` is the same as `-l -a -h`
- Words in `<angle brackets>` are placeholders — replace them with your actual value

---

## 1. File & Directory Management

These are the commands you will use every single day. Master these first.

---

### `ls` — List files in a directory

```bash
ls
ls -l
ls -lah /var/log
```

**Breaking down `ls -lah`:**

| Flag | Meaning |
|------|---------|
| `-l` | Long format — shows permissions, owner, size, date |
| `-a` | All files — includes hidden files (files starting with `.`) |
| `-h` | Human readable — shows sizes as KB, MB, GB instead of raw bytes |

**What the output looks like:**
```
drwxr-xr-x  3 root root 4.0K Jan  1 10:00 nginx/
-rw-r--r--  1 root root  12K Jan  1 09:55 syslog
```

Reading the first column (`drwxr-xr-x`):
- First character: `d` = directory, `-` = file, `l` = symbolic link
- Next 3 characters (`rwx`): owner's permissions (read, write, execute)
- Next 3 (`r-x`): group's permissions
- Last 3 (`r-x`): everyone else's permissions

**When to use it:** Before doing anything in a new directory — always `ls -lah` first to see what is there.

---

### `find` — Search for files recursively

```bash
find /etc -name "*.conf" -type f
find /var -name "*.log" -mtime -1
find . -name "*.py" -size +1M
```

**Breaking it down:**

| Part | Meaning |
|------|---------|
| `/etc` | Start searching from this directory (`.` means current directory) |
| `-name "*.conf"` | Match files whose name ends in `.conf` (`*` is a wildcard) |
| `-type f` | Only files (`f`). Use `-type d` for directories |
| `-mtime -1` | Modified in the last 1 day (`-` means "less than") |
| `-size +1M` | Larger than 1 Megabyte |

**Real-world use:** You deployed an app and it created config files but you don't know where. Run:
```bash
find / -name "app.conf" -type f 2>/dev/null
```
The `2>/dev/null` part silences "Permission denied" errors so your output stays clean.

---

### `du` — Disk Usage — how much space is a folder using?

```bash
du -sh /var/lib/docker
du -sh /* 2>/dev/null | sort -rh | head -10
```

**Breaking it down:**

| Flag | Meaning |
|------|---------|
| `-s` | Summary — show only the total, not every subfolder |
| `-h` | Human readable (MB, GB) |
| `/*` | Check every top-level directory |
| `sort -rh` | Sort by size, largest first (`r` = reverse, `h` = human-readable sizes) |
| `head -10` | Show only the top 10 results |

**Real-world use:** Your server disk is at 95%. Run the second command to find the biggest directories and figure out what is eating your space. Docker image cache (`/var/lib/docker`) is a very common culprit.

---

### `mkdir`, `cp`, `mv`, `rm` — Create, Copy, Move, Delete

```bash
# Create a directory (and all parent directories if they don't exist)
mkdir -p /app/config/envs

# Copy a file
cp config.yml config.yml.backup

# Copy an entire folder
cp -r ./myapp ./myapp-backup

# Move or rename
mv old-name.txt new-name.txt
mv ./file.txt /opt/app/file.txt

# Delete a file
rm file.txt

# Delete a folder and everything inside it (DANGEROUS — no undo!)
rm -rf /tmp/build-artifacts
```

**`-p` in mkdir:** Creates all intermediate directories. Without `-p`, if `/app/config` doesn't exist, the command fails. With `-p`, it creates everything needed.

**`-r` in cp and rm:** Recursive — means "apply to the folder AND everything inside it".

**`-f` in rm:** Force — no confirmation prompt. Combined with `-r`, `rm -rf` is one of the most dangerous commands in Linux. There is no Recycle Bin. Files are gone immediately.

> **Beginner tip:** Before running `rm -rf`, run `ls` on the path first to confirm you are in the right place. A typo here can wipe critical files.

---

### `tar` — Archive and compress files

```bash
# Compress a folder into a .tar.gz file
tar -czf backup.tar.gz ./myapp

# Extract a .tar.gz file into a specific folder
tar -xzf backup.tar.gz -C /opt/myapp
```

**Breaking down the flags:**

| Flag | Meaning |
|------|---------|
| `-c` | Create a new archive |
| `-x` | Extract from an archive |
| `-z` | Use gzip compression (makes the file smaller) |
| `-f` | The next argument is the filename of the archive |
| `-C /path` | Extract into this directory (must already exist) |

**Memory trick:** Think **c**reate, e**x**tract, g**z**ip, **f**ile.

---

### `chmod` and `chown` — Change permissions and ownership

```bash
chmod 755 deploy.sh
chmod 600 ~/.ssh/id_rsa
chown -R appuser:appgroup /var/app
```

**Understanding the numbers in chmod:**

Each digit represents permissions for: Owner | Group | Everyone else

| Number | Permission | What it means |
|--------|-----------|---------------|
| 7 | rwx | Read + Write + Execute |
| 6 | rw- | Read + Write only |
| 5 | r-x | Read + Execute only |
| 4 | r-- | Read only |
| 0 | --- | No permissions at all |

So `chmod 755` means:
- Owner → 7 → rwx (full access)
- Group → 5 → r-x (can read and run, not write)
- Others → 5 → r-x (can read and run, not write)

**`chmod 600 ~/.ssh/id_rsa`** — Your SSH private key must be readable ONLY by you. If group or others can read it, SSH will refuse to use the key and throw a "Permissions are too open" error.

**`chown -R appuser:appgroup /var/app`** — Changes the owner to `appuser` and the group to `appgroup` for the folder and everything inside it (`-R` = recursive).

---

## 2. Process & System Monitoring

When something is wrong — high CPU, a crashed service, a frozen app — these commands tell you what is happening inside your system.

---

### `top` and `htop` — Real-time process monitor

```bash
top
htop
```

**`top` is built-in to every Linux system.** `htop` is a prettier version you install separately (`apt install htop`).

**What to look at in `top`:**
```
top - 14:23:01 up 5 days, load average: 0.52, 0.48, 0.45
Tasks: 142 total,   1 running, 141 sleeping
%Cpu(s):  5.2 us,  1.3 sy, 92.8 id
MiB Mem :   3934.0 total,    412.3 free,   2100.4 used
```

| Field | Meaning |
|-------|---------|
| `load average: 0.52` | Average number of processes waiting for CPU in last 1/5/15 min. If this exceeds your CPU count, your system is overloaded |
| `%Cpu id: 92.8` | 92.8% of CPU is idle (good). If this drops below 10%, CPU is very busy |
| `MiB Mem used: 2100` | 2.1 GB of RAM is in use |

**Keyboard shortcuts inside top:**
- `P` — Sort by CPU usage (highest first)
- `M` — Sort by Memory usage
- `k` — Kill a process (type the PID when prompted)
- `q` — Quit

---

### `ps` — Snapshot of all running processes

```bash
ps aux
ps aux | grep nginx
ps -ef --forest
```

**Breaking down `ps aux`:**

| Flag | Meaning |
|------|---------|
| `a` | Show processes from ALL users (not just you) |
| `u` | User-oriented format (shows username, CPU%, MEM%) |
| `x` | Include processes not attached to a terminal |

**Understanding the output columns:**
```
USER   PID  %CPU  %MEM   VSZ    RSS  TTY   STAT  START   TIME  COMMAND
root   123   0.0   0.1  5120   1024  ?     Ss    10:00   0:00  nginx: master
```

| Column | Meaning |
|--------|---------|
| `PID` | Process ID — the unique number for this process |
| `%CPU` | How much CPU it is using right now |
| `%MEM` | How much RAM it is using |
| `STAT` | S=sleeping (idle), R=running, Z=zombie (stuck/dead) |

**`ps aux | grep nginx`** — The `|` (pipe) sends the output of `ps aux` into `grep`, which filters to only show lines containing "nginx". This is how you quickly check if a specific service is running.

**`ps -ef --forest`** — Shows processes as a tree so you can see parent-child relationships. Useful for understanding which process launched which.

---

### `kill`, `pkill`, `killall` — Stop a process

```bash
kill -9 1234
pkill -f "python app.py"
killall nginx
```

**Why `-9`?** Linux sends signals to processes. Signal `-9` is `SIGKILL` — the strongest, meaning "stop immediately, no cleanup". Use this when a process is frozen and won't respond normally.

| Signal | Number | Meaning |
|--------|--------|---------|
| SIGTERM | 15 | "Please shut down gracefully" — this is the default |
| SIGKILL | 9 | "Stop immediately, no choice" — cannot be ignored |
| SIGHUP | 1 | "Reload your config" — used for nginx, etc. |

**Best practice:** First try `kill 1234` (graceful SIGTERM). If the process does not stop within a few seconds, escalate to `kill -9 1234`.

**`pkill -f "python app.py"`** — Kill by matching the full command line string. Useful when you don't know the PID but know the command name.

---

### `systemctl` — Manage system services (the modern way)

```bash
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx
systemctl enable nginx
systemctl disable nginx
```

**The difference between `restart` and `reload`:**
- `restart` — Stops the process completely, then starts it fresh. There will be a brief downtime.
- `reload` — Asks the process to re-read its config without stopping. Zero downtime. Not all services support this.

**The difference between `enable` and `start`:**
- `start` — Starts the service **right now**, but if the server reboots, it won't auto-start
- `enable` — Makes it **auto-start on every boot**, but does not start it right now
- **You usually want both:** `systemctl enable --now nginx` does both in one command

**Reading `systemctl status` output:**
```
● nginx.service - A high performance web server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled)
   Active: active (running) since Mon 2024-01-01 10:00:00; 5 days ago
 Main PID: 1234 (nginx)
```

- `active (running)` — Service is healthy ✓
- `failed` — Service crashed ✗
- `inactive (dead)` — Service is stopped
- `enabled` — Will auto-start on boot

---

### `journalctl` — Read system and service logs

```bash
journalctl -u nginx
journalctl -u nginx -f
journalctl -u nginx --since "2 hours ago"
journalctl -u nginx --since "2024-01-01 10:00" --until "2024-01-01 11:00"
journalctl -p err -b
journalctl --disk-usage
```

**Breaking down the flags:**

| Flag | Meaning |
|------|---------|
| `-u nginx` | Show logs only for the nginx unit/service |
| `-f` | Follow — keep printing new lines as they appear (live stream) |
| `--since "2 hours ago"` | Only show logs from the last 2 hours |
| `-p err` | Only show error-level messages and above |
| `-b` | Only show logs from the current boot (since last restart) |
| `-n 50` | Show only the last 50 lines |

**Real-world use:** Your nginx service failed to start. Run:
```bash
journalctl -u nginx -n 50 --no-pager
```
The last 50 lines will almost always show exactly what went wrong — usually a config syntax error or a port that is already in use.

---

### `free` and `df` — Memory and disk space at a glance

```bash
free -h
df -h
df -h /var
```

**`free -h` output explained:**
```
              total        used        free      shared  buff/cache   available
Mem:          3.8Gi       2.1Gi       400Mi       150Mi       1.3Gi       1.4Gi
Swap:         2.0Gi       100Mi       1.9Gi
```

- `available` is more important than `free` — it includes memory that can be freed from cache quickly
- If `available` is near zero and Swap is heavily used, your system is under memory pressure

**`df -h` output explained:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       50G   38G   12G  77% /
/dev/xvdb1      100G   80G   20G  80% /var/lib/docker
```

- `Use%` above 90% is a warning — above 95%, your services will likely start failing

---

## 3. User & Permission Management

---

### Adding and managing users

```bash
# Create a new user (also creates their home directory at /home/username)
adduser devops

# Add an existing user to the sudo group (grants admin privileges)
usermod -aG sudo devops

# Change a user's password
passwd devops

# Delete a user AND their home directory
userdel -r olduser
```

**`usermod -aG sudo devops` explained carefully:**
- `-a` = Append (add to the group WITHOUT removing from their existing groups)
- `-G sudo` = The group to add them to
- Without `-a`, you would **replace ALL their existing groups** with just `sudo` — a dangerous mistake that could break things

**`sudo` group:** On Ubuntu/Debian, members of the `sudo` group can run any command as root by prefixing it with `sudo`. On CentOS/RHEL, the equivalent group is called `wheel`.

---

### `su` and `sudo` — Run as another user

```bash
su - devops          # Switch to devops user (need THEIR password)
sudo -i              # Get a root shell (need YOUR own password)
sudo systemctl restart nginx   # Run one command as root
sudo -u postgres psql          # Run as a specific non-root user
```

**`su -` vs `su`:**
- `su devops` — Switch user but keep current environment variables and directory
- `su - devops` — Switch user AND load their full environment (home directory, PATH, shell settings). The `-` is important — always use it to avoid environment issues.

---

## 4. Networking

Networking commands are used constantly in DevOps — to debug why a service is unreachable, check if a port is listening, or trace a DNS resolution problem.

---

### `ping`, `traceroute`, `mtr` — Test connectivity

```bash
ping -c 4 8.8.8.8
traceroute google.com
mtr google.com
```

**`ping -c 4 8.8.8.8`:**
- Sends 4 ICMP "echo request" packets to Google's DNS (8.8.8.8)
- `-c 4` = count — stop after 4 packets. Without this, ping runs forever.
- If you get replies: your basic internet connectivity is working
- If you get `Request timeout`: either the host is down or ICMP is blocked by a firewall

**`traceroute`** shows every router ("hop") your packet passes through on the way to the destination. Useful for finding where along the path packets are getting dropped or delayed.

**`mtr`** combines ping and traceroute in real-time. It is the best single tool for diagnosing network path issues. Install with `apt install mtr`.

---

### `dig` and `nslookup` — DNS lookup and debugging

```bash
nslookup example.com
dig example.com
dig example.com A              # only get IPv4 address (A) records
dig example.com MX             # get mail server records
dig @8.8.8.8 example.com       # use Google's DNS instead of your system DNS
dig +short example.com         # just the IP, no extra output
```

**When to use this:** Your service is deployed but the domain name isn't working. Run `dig` to check if DNS is resolving correctly and pointing to the right IP address.

**`dig @8.8.8.8 example.com`** — Queries Google's public DNS directly, bypassing your local resolver. Useful for checking if a DNS change has propagated publicly while your local cache still shows the old value.

**What to look at in `dig` output:**
```
;; ANSWER SECTION:
example.com.    3600    IN    A    93.184.216.34
                  |                    |
           TTL in seconds         The IP address
```

If the ANSWER SECTION is empty, DNS is not resolving. If it shows the wrong IP, your DNS record is misconfigured.

---

### `curl` — Make HTTP requests from the terminal

```bash
# Basic GET request — see the response body
curl https://api.example.com

# Get only the HTTP response headers
curl -I https://api.example.com/health

# Get just the HTTP status code (e.g., 200, 404, 500)
curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health

# Send a POST request with a JSON body
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer mytoken" \
  -d '{"name": "hemanth", "role": "devops"}' \
  https://api.example.com/users

# Download a file, saving with the same filename
curl -O https://example.com/file.tar.gz

# Download and save with a custom name
curl -o custom-name.tar.gz https://example.com/file.tar.gz
```

**Breaking down the flags:**

| Flag | Meaning |
|------|---------|
| `-I` | HEAD request — fetch only response headers, not the body |
| `-s` | Silent — suppress progress bar and error messages |
| `-o /dev/null` | Throw away the response body (`/dev/null` is a black hole in Linux) |
| `-w "%{http_code}"` | After the request, print this info (here: the HTTP status code) |
| `-X POST` | Use HTTP POST method (default is GET) |
| `-H "..."` | Add a header to the request (can use multiple `-H` flags) |
| `-d '...'` | The request body data to send |

**Real-world DevOps use:** In a CI/CD pipeline, check if your newly deployed service is healthy before marking the deployment as successful:
```bash
STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://myapp.com/health)
if [ "$STATUS" != "200" ]; then
  echo "Health check failed! Got HTTP $STATUS"
  exit 1
fi
echo "Service is healthy!"
```

---

### `ss` — Check which ports are open and what is listening

```bash
ss -tulnp
ss -tulnp | grep 8080
```

**Breaking down `ss -tulnp`:**

| Flag | Meaning |
|------|---------|
| `-t` | Show TCP connections |
| `-u` | Show UDP connections |
| `-l` | Only show LISTENING sockets (services waiting for connections) |
| `-n` | Show port numbers instead of service names (e.g., show 80 not "http") |
| `-p` | Show which process owns the socket |

**Sample output:**
```
Netid  State   Local Address:Port   Process
tcp    LISTEN  0.0.0.0:80          users:(("nginx",pid=1234))
tcp    LISTEN  0.0.0.0:443         users:(("nginx",pid=1234))
tcp    LISTEN  127.0.0.1:5432      users:(("postgres",pid=5678))
```

**Real-world use:** You are trying to start an app on port 8080 but it says "address already in use". Run `ss -tulnp | grep 8080` to see which process is already holding that port.

---

### `ssh` and `scp` — Connect to remote servers and copy files

```bash
# Connect to a remote server using a password
ssh username@192.168.1.10

# Connect using an SSH private key (common for AWS EC2 instances)
ssh -i ~/.ssh/mykey.pem ubuntu@ec2-52-10-x-x.compute.amazonaws.com

# Create a tunnel: access remote PostgreSQL on your local port 5432
ssh -L 5432:localhost:5432 user@remote-server
# Now open another terminal and: psql -h localhost -U myuser mydb

# Copy a file TO a remote server
scp ./deploy.sh ubuntu@10.0.0.5:/home/ubuntu/

# Copy a file FROM a remote server to your current folder
scp ubuntu@10.0.0.5:/var/log/app.log ./

# Copy an entire folder recursively
scp -r ./myapp ubuntu@10.0.0.5:/opt/myapp

# Sync a folder efficiently (only transfers changed files)
rsync -avz ./local/ user@remote:/remote/
# Also delete files on remote that were deleted locally:
rsync -avz --delete ./local/ user@remote:/remote/
```

**SSH port forwarding (`-L`) explained:**
- `-L 5432:localhost:5432` means: "On MY local machine, listen on port 5432. Forward all traffic through this SSH tunnel to port 5432 on the REMOTE machine."
- This lets you securely access a remote database using your local database client, without exposing the database port to the internet.

**`rsync -avz` flags:**
- `-a` = Archive mode — preserves permissions, timestamps, and symbolic links
- `-v` = Verbose — shows each file being transferred
- `-z` = Compress during transfer — faster over slow network connections

---

## 5. Text Processing & Log Analysis

This is where Linux really shines. You can process millions of lines of logs in seconds by chaining these tools together.

---

### `grep` — Search for text inside files

```bash
grep "ERROR" app.log
grep -rn "ERROR" /var/log/ --include="*.log"
grep -v "DEBUG" app.log
grep -E "ERROR|WARN|CRITICAL" app.log
grep -c "500" access.log
grep -A 5 -B 2 "NullPointerException" app.log
```

**Breaking down the flags:**

| Flag | Meaning |
|------|---------|
| `-r` | Recursive — search inside all files in the directory and subdirectories |
| `-n` | Show line numbers alongside matches |
| `--include="*.log"` | Only search files matching this pattern |
| `-v` | Invert match — show lines that do NOT contain the pattern |
| `-E` | Extended regex — enables the `\|` pipe for OR logic |
| `-c` | Count — print the number of matching lines instead of the lines themselves |
| `-i` | Case-insensitive — matches ERROR, error, Error all the same |
| `-A 5` | Print 5 lines After each match (for context) |
| `-B 2` | Print 2 lines Before each match |

**Real-world use:** Your Java app crashed. Get the full stack trace (exception + the 10 lines before it):
```bash
grep -B 10 "Exception" app.log | tail -60
```

---

### `tail` and `head` — Read the end or beginning of a file

```bash
# Show the last 50 lines
tail -n 50 app.log

# Follow a log file live — stays open, prints new lines as they arrive
tail -f /var/log/syslog

# Follow AND filter at the same time — only show 5xx HTTP errors
tail -f /var/log/nginx/access.log | grep " 5[0-9][0-9] "

# Show only the first 20 lines (good for reading config file headers)
head -n 20 config.yml
```

**`tail -f`** — The `-f` flag stands for "follow". The terminal stays open and prints each new line as it is written to the file. This is how you monitor live application logs. Press `Ctrl+C` to stop.

The pipe combination `tail -f app.log | grep "ERROR"` is one of the most useful patterns in DevOps — live log monitoring with filtering.

---

### `awk` — Extract and process specific columns from text

`awk` processes text line by line, splitting each line into fields (columns) separated by whitespace by default.

```bash
# Print only the first field (column) of each line
awk '{print $1}' access.log

# Print the 1st and 9th columns (IP and HTTP status in nginx logs)
awk '{print $1, $9}' access.log

# Count unique IP addresses in an nginx access log — full pipeline
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# Change the field separator (for CSV files use comma)
awk -F',' '{print $1, $3}' data.csv

# Sum a numeric column (e.g., bytes sent — column 10 in nginx logs)
awk '{sum += $10} END {print "Total bytes:", sum}' access.log

# Skip the header row (NR = line Number of Record)
awk -F',' 'NR>1 {sum += $3} END {print sum}' data.csv
```

**Understanding `awk` field variables:**
- `$0` = the entire line
- `$1` = first field
- `$2` = second field
- `$NF` = the very last field
- `NR` = current line number

**The "count unique IPs" pipeline — step by step:**
```bash
awk '{print $1}' access.log     # Step 1: Extract IP column
  | sort                        # Step 2: Sort them (groups same IPs together)
  | uniq -c                     # Step 3: Count consecutive identical lines
  | sort -rn                    # Step 4: Sort by count, highest first
  | head -20                    # Step 5: Show only the top 20
```
Result: You instantly see which IP addresses hit your server the most — useful for detecting abuse.

---

### `sed` — Find and replace text in files

```bash
# Replace all occurrences and save back to the file (-i means in-place)
sed -i 's/localhost/prod-db.internal/g' config.yml

# Preview the replacement WITHOUT modifying the file (no -i flag)
sed 's/localhost/prod-db.internal/g' config.yml

# Delete all comment lines (lines starting with #)
sed -i '/^#/d' config.conf

# Delete all blank/empty lines
sed -i '/^$/d' file.txt

# Print only specific line numbers (useful for huge log files)
sed -n '10,20p' bigfile.log
```

**Understanding `'s/localhost/prod-db.internal/g'`:**
- `s` = substitute command
- `localhost` = the text pattern to find
- `prod-db.internal` = what to replace it with
- `g` = global flag — replace ALL occurrences on each line. Without `g`, only the first match per line is replaced.

**Always test without `-i` first** — run `sed 's/.../.../' file` to preview the output before committing the change with `-i`.

**Real-world DevOps use:** You have a config file that was written for staging and need to promote it to production:
```bash
# Preview first
sed 's/staging-db.internal/prod-db.internal/g' app.yml | head -30

# Apply if it looks right
sed -i 's/staging-db.internal/prod-db.internal/g' app.yml
```

---

### `jq` — Parse and query JSON output

`jq` is essential for working with Kubernetes, cloud APIs, and any modern tool that outputs JSON.

```bash
# Pretty-print a JSON response (adds indentation and colors)
kubectl get pod my-pod -o json | jq .

# Extract a specific field from JSON
kubectl get pods -o json | jq '.items[].metadata.name'

# Reshape the output — create a new JSON object with selected fields
kubectl get nodes -o json | jq '.items[] | {name: .metadata.name, cpu: .status.capacity.cpu}'

# Filter results based on a condition
kubectl get pods -o json | jq '.items[] | select(.status.phase != "Running")'

# Navigate nested paths
cat response.json | jq '.data.user.email'

# Get the count of items in an array
cat pods.json | jq '.items | length'
```

**`jq` path syntax:**
- `.` = the entire JSON object
- `.key` = access a field named "key"
- `.items[]` = iterate over each element of the "items" array
- `.items[0]` = first element of the "items" array
- `select(condition)` = filter — only return elements where the condition is true

**Real-world use:** AWS CLI returns JSON. Extract just the instance IDs from a describe-instances response:
```bash
aws ec2 describe-instances | jq '.Reservations[].Instances[].InstanceId'
```

---

## 6. Docker Commands

Docker packages your application and its dependencies into a **container** — a portable, isolated unit that runs the same on any machine. Think of it as a lightweight virtual machine that shares the host OS kernel.

---

### Container lifecycle

```bash
# See running containers
docker ps

# See ALL containers including stopped ones
docker ps -a

# Run a new container from an image
docker run -d -p 8080:80 --name web nginx
```

**Breaking down `docker run -d -p 8080:80 --name web nginx`:**

| Part | Meaning |
|------|---------|
| `run` | Create and start a new container |
| `-d` | Detached mode — run in the background (don't take over your terminal) |
| `-p 8080:80` | Port mapping: host port 8080 → container port 80 |
| `--name web` | Give the container a memorable name instead of a random one |
| `nginx` | The image to use (downloaded from Docker Hub automatically if not local) |

**Port mapping `-p 8080:80`** — Format is `HOST_PORT:CONTAINER_PORT`. Nginx inside the container listens on port 80. You access it from your browser on port 8080 (`http://localhost:8080`). The container's port 80 is not directly accessible from outside without this mapping.

```bash
# Start a stopped container
docker start web

# Stop gracefully (sends SIGTERM, waits 10s, then SIGKILL)
docker stop web

# Restart
docker restart web

# Remove a stopped container
docker rm web

# Force remove even if it's still running
docker rm -f web

# Remove ALL containers (stopped and running)
docker rm -f $(docker ps -aq)
# $(command) runs the inner command first and substitutes its output
# docker ps -aq = list all container IDs quietly (just IDs, no headers)
```

---

### Images

```bash
# List all local images
docker images

# Download an image from Docker Hub
docker pull ubuntu:22.04
# Format: image_name:tag. If you omit the tag, "latest" is used.

# Build an image from your Dockerfile in the current directory
docker build -t myapp:v1.0 .

# Tag an image for a private registry
docker tag myapp:v1.0 myregistry.io/myteam/myapp:v1.0

# Push the tagged image to the registry
docker push myregistry.io/myteam/myapp:v1.0

# Remove a local image
docker rmi myapp:v1.0
```

**`docker build -t myapp:v1.0 .`**:
- `-t myapp:v1.0` = tag (name and version) for the image
- `.` = the build context — the directory containing your Dockerfile and source files

---

### Inspecting and debugging containers

```bash
# View container logs
docker logs web

# Stream live logs with the last 100 lines shown first
docker logs -f --tail=100 web

# Get an interactive shell inside a running container
docker exec -it web bash
docker exec -it web sh          # use sh if bash is not installed in the image

# Run a single command inside the container without opening a shell
docker exec web nginx -t        # test nginx config

# Inspect all details about a container (network, mounts, env vars, etc.)
docker inspect web

# Live resource usage for all running containers
docker stats
docker stats web                # for one specific container
```

**`docker exec -it` explained:**
- `exec` = run a command inside an already-running container
- `-i` = interactive — keep STDIN open so you can type
- `-t` = allocate a pseudo-terminal (TTY) so it feels like a real terminal
- Together, `-it` gives you a proper interactive shell session inside the container

**Extracting specific info from `docker inspect` using `jq`:**
```bash
# Get the container's internal IP address
docker inspect web | jq '.[0].NetworkSettings.IPAddress'

# Get all environment variables
docker inspect web | jq '.[0].Config.Env'

# Get mounted volumes
docker inspect web | jq '.[0].Mounts'
```

---

### Cleanup

```bash
# Nuclear option — removes ALL unused containers, networks, images, and build cache
docker system prune -af

# Only remove dangling (unnamed/untagged) images
docker image prune -f

# Remove volumes not attached to any container (WARNING: deletes data!)
docker volume prune -f

# Check how much disk space Docker is using overall
docker system df
```

**`docker system prune -af`** — Use this when your disk fills up from accumulated Docker layers. `-a` removes ALL unused images (not just untagged ones). `-f` skips the "Are you sure?" prompt.

---

### Docker Compose — manage multi-container apps

Docker Compose lets you define a complete multi-container application in one `docker-compose.yml` file and start everything with a single command.

```bash
# Start all services defined in docker-compose.yml (in background)
docker-compose up -d

# Rebuild images before starting (do this after changing Dockerfile or source code)
docker-compose up -d --build

# Stop and remove containers (keeps named volumes — your data is safe)
docker-compose down

# Stop, remove containers AND delete volumes (DESTROYS DATA!)
docker-compose down -v

# Check the status of your services
docker-compose ps

# Stream logs from one specific service
docker-compose logs -f app

# Get a shell inside a running service container
docker-compose exec app bash

# Run a one-off command in a fresh container (removed automatically after)
docker-compose run --rm app python manage.py migrate
```

**Why `--build` matters:** Without `--build`, `docker-compose up` uses cached images even if you changed your Dockerfile or application source code. Always use `--build` after making code changes.

---

## 7. Kubernetes (kubectl) Commands

Kubernetes (K8s) is a system for running and managing containers across multiple servers automatically. It handles automatic restarts, scaling, load balancing, and rolling deployments.

**Key concepts you must know before the commands:**

| Term | What it is |
|------|-----------|
| **Pod** | Smallest deployable unit. One or more containers running together on the same node |
| **Deployment** | Manages a set of identical Pods. Handles rolling updates, scaling, and self-healing |
| **Service** | A stable network endpoint in front of Pods (Pods come and go, Services stay stable) |
| **Namespace** | A virtual cluster inside the cluster — used to separate environments (dev, staging, prod) |
| **Node** | A physical or virtual machine that runs Pods |
| **ConfigMap** | Store non-sensitive configuration data (key-value pairs) |
| **Secret** | Store sensitive data like passwords and tokens (base64 encoded) |

---

### Context and namespace management

```bash
# List all clusters you are configured to talk to
kubectl config get-contexts

# Switch to a different cluster
kubectl config use-context prod-cluster

# Set a default namespace so you don't have to type -n on every command
kubectl config set-context --current --namespace=ml-team

# Check which cluster/context you are currently using
kubectl config current-context
```

**Always check your context before running destructive commands.** Running `kubectl delete` on the wrong cluster is a very common and painful mistake.

---

### Pod operations

```bash
# List pods in the current namespace
kubectl get pods

# List pods in a specific namespace
kubectl get pods -n kube-system

# List pods across ALL namespaces
kubectl get pods -A

# Show more detail: which Node the pod runs on and its cluster IP
kubectl get pods -o wide

# Filter pods by label
kubectl get pods -l app=myapp

# Get FULL details about a pod — the go-to debugging command
kubectl describe pod my-pod-xyz

# Delete a pod (if managed by a Deployment, Kubernetes will recreate it automatically)
kubectl delete pod my-pod-xyz

# Delete all pods in a namespace (the Deployment will recreate them)
kubectl delete pods --all -n staging
```

**`kubectl describe pod` is your #1 debugging tool.** Look at the **Events** section at the bottom of the output — it tells you exactly why a pod failed:
```
Events:
  Warning  Failed    2m   kubelet  Failed to pull image "myapp:v2": not found
  Warning  BackOff   1m   kubelet  Back-off restarting failed container
  Normal   Pulling   30s  kubelet  Pulling image "myapp:v2"
```

This tells you the image tag doesn't exist in the registry — something you'd never guess from just `kubectl get pods` showing `ImagePullBackOff`.

---

### Viewing logs

```bash
# Basic log view
kubectl logs my-pod

# Stream live logs (stays open, prints new lines)
kubectl logs -f my-pod

# Only show the last 100 lines
kubectl logs my-pod --tail=100

# Logs from a specific container in a multi-container Pod
kubectl logs -f my-pod -c sidecar-container

# Logs from the PREVIOUS (crashed) container — critical for crash loops!
kubectl logs my-pod --previous

# Stream logs from ALL pods of a Deployment at once
kubectl logs -f deploy/my-app --tail=50
```

**`kubectl logs my-pod --previous`** is invaluable for **CrashLoopBackOff** situations. When a container crashes and restarts, its logs are lost. The `--previous` flag retrieves the logs from the crashed instance. Without this, you cannot see why it crashed.

---

### Exec into pods

```bash
# Open an interactive bash shell inside a running pod
kubectl exec -it my-pod -- bash

# If bash is not installed in the image, use sh
kubectl exec -it my-pod -- sh

# Open a shell in a specific container within a multi-container pod
kubectl exec -it my-pod -c sidecar -- sh

# Run a single command without opening a shell
kubectl exec my-pod -- cat /etc/config/app.yml
kubectl exec my-pod -- env | grep DB_HOST
```

**The `--` separator:** Everything after `--` is the command to run inside the pod. The `--` prevents `kubectl` from trying to interpret your command as its own flags.

**Real-world debugging — check if your app can reach the database from inside the pod:**
```bash
kubectl exec -it my-pod -- bash
# You are now inside the pod
curl http://postgres-service:5432        # can we reach the service?
nslookup postgres-service               # is DNS resolving inside the cluster?
env | grep DB_                          # are the right env variables injected?
```

---

### Deployments — managing your application

```bash
# List all deployments
kubectl get deployments -A

# See detailed info about a deployment
kubectl describe deployment my-app

# Watch the progress of a rolling update in real time
kubectl rollout status deploy/my-app

# See the history of all rollouts
kubectl rollout history deploy/my-app

# Roll back to the previous version (immediately!)
kubectl rollout undo deploy/my-app

# Roll back to a specific older version
kubectl rollout undo deploy/my-app --to-revision=3

# Trigger a rolling restart — replaces all pods one by one with zero downtime
kubectl rollout restart deploy/my-app

# Scale the number of pods up or down
kubectl scale deploy/my-app --replicas=5

# Update the container image (triggers a rolling update)
kubectl set image deploy/my-app app=myregistry.io/myapp:v2.0
```

**Rolling restart explained:** `kubectl rollout restart` replaces pods one by one — it waits until the new pod is healthy before terminating the old one. This means zero downtime. Use this when you need to reload env variables, secrets, or config maps without changing the image.

**`kubectl rollout undo`** is your emergency rollback. If a bad deployment is causing errors in production, this command reverts to the last known-good version in seconds.

---

### Applying manifests

```bash
# Apply a YAML manifest (creates resources if they don't exist, updates if they do)
kubectl apply -f manifest.yaml

# ALWAYS dry-run first in production — see what would change without doing it
kubectl apply -f manifest.yaml --dry-run=client

# Apply all YAML files in a directory
kubectl apply -f ./k8s/

# Delete all resources defined in a manifest
kubectl delete -f manifest.yaml

# See a human-readable diff of what will change
kubectl diff -f manifest.yaml
```

**Best practice for production deployments:**
```bash
# Step 1: Check the diff
kubectl diff -f deployment.yaml

# Step 2: Dry run
kubectl apply -f deployment.yaml --dry-run=client

# Step 3: Apply for real
kubectl apply -f deployment.yaml

# Step 4: Watch the rollout
kubectl rollout status deploy/my-app
```

---

### Secrets and ConfigMaps

```bash
# List secrets in a namespace
kubectl get secrets -n production

# See metadata about a secret (the values are hidden)
kubectl describe secret db-credentials

# Read the actual value of a secret (decodes base64 for you)
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 --decode

# Create a secret from the command line
kubectl create secret generic db-creds \
  --from-literal=username=admin \
  --from-literal=password=super-secret-password

# Create a secret from a file (e.g., a TLS certificate)
kubectl create secret tls my-tls \
  --cert=tls.crt \
  --key=tls.key

# View a ConfigMap
kubectl get configmap app-config -o yaml
```

**Why are Kubernetes secrets base64 encoded?** Base64 encoding allows binary data (like TLS certificates) to be safely stored as text. It is NOT encryption — anyone with access to the secret can decode it. For production, use tools like HashiCorp Vault, AWS Secrets Manager, or Sealed Secrets for proper encryption.

---

### Node management

```bash
# List all nodes with their status and OS/version info
kubectl get nodes -o wide

# Full details and events for a specific node
kubectl describe node worker-node-1

# Cordon — prevent new pods from being scheduled on this node
kubectl cordon worker-node-1

# Uncordon — allow pods to be scheduled on this node again
kubectl uncordon worker-node-1

# Drain — safely evict all pods off the node (for maintenance/shutdown)
kubectl drain worker-node-1 \
  --ignore-daemonsets \        # skip DaemonSet pods (they run on every node by design)
  --delete-emptydir-data       # allow eviction of pods that use emptyDir volumes

# Check resource usage across all nodes
kubectl top nodes
kubectl top pods -A --sort-by=memory
```

**Node maintenance workflow:**
1. `kubectl cordon worker-node-1` — Stop new pods going there
2. `kubectl drain worker-node-1 --ignore-daemonsets` — Safely move existing pods away
3. Perform your maintenance (OS patching, hardware fix, etc.)
4. `kubectl uncordon worker-node-1` — Allow pods to schedule there again

---

### Troubleshooting workflow — step by step

When something is broken in Kubernetes, follow this order:

```bash
# Step 1: Find what is not in a healthy state
kubectl get pods -A | grep -v Running | grep -v Completed

# Step 2: Look at recent events to see what just happened
kubectl get events -n mynamespace --sort-by='.lastTimestamp' | tail -20

# Step 3: Describe the failing pod — read the Events section carefully
kubectl describe pod failing-pod -n mynamespace

# Step 4: Read the logs — current and previous
kubectl logs failing-pod -n mynamespace --tail=100
kubectl logs failing-pod -n mynamespace --previous

# Step 5: If the pod is running but behaving wrongly, shell in and investigate
kubectl exec -it failing-pod -n mynamespace -- bash

# Step 6: If OOMKilled (out of memory), check resource usage
kubectl top pods -n mynamespace
```

---

## 8. Environment Variables & Shell Tricks

---

### Environment variables

```bash
# Set a variable (available for this terminal session and any commands you run)
export DB_HOST=prod-db.internal
export DB_PORT=5432

# Use a variable
echo $DB_HOST
echo "Connecting to ${DB_HOST}:${DB_PORT}"

# List all environment variables
printenv
printenv | grep DB_        # only show DB-related ones

# Remove a variable from the current session
unset MY_VAR

# Load variables from a .env file (skip comment lines)
export $(cat .env | grep -v '#' | xargs)
```

**Defensive scripting — always set these at the top of your shell scripts:**
```bash
#!/bin/bash
set -euo pipefail
# -e: Exit immediately if any command returns a non-zero exit code
# -u: Treat unset variables as errors (avoids silent bugs)
# -o pipefail: A pipeline fails if ANY command in it fails (not just the last)

# Use default values for optional variables
DB_HOST=${DB_HOST:-"localhost"}     # use "localhost" if DB_HOST is not set
DB_PORT=${DB_PORT:-"5432"}
```

---

### Useful aliases to add to `~/.bashrc` or `~/.zshrc`

```bash
# Kubernetes shortcuts
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgpa='kubectl get pods -A'
alias kgn='kubectl get nodes -o wide'
alias kns='kubectl config set-context --current --namespace'
alias kdp='kubectl describe pod'
alias klf='kubectl logs -f'

# Docker shortcuts
alias dps='docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'
alias dex='docker exec -it'

# General shortcuts
alias ll='ls -lah'
alias grep='grep --color=auto'
alias ..='cd ..'
alias ...='cd ../..'
```

After adding aliases, run `source ~/.bashrc` to apply them immediately.

---

### Shell history tricks

```bash
# Interactive reverse history search — start typing to find past commands
Ctrl + R

# Run the previous command again
!!

# Run the previous command with sudo (very useful!)
sudo !!

# Run the last command that started with "kubectl"
!kubectl

# Show your recent command history
history | tail -20
history | grep "docker build"
```

---

## 9. Cron Jobs — Scheduling Recurring Tasks

Cron runs commands automatically on a schedule. Think of it as Task Scheduler on Linux.

```bash
# Open your personal crontab file in an editor
crontab -e

# View your current cron jobs
crontab -l

# View system-wide cron jobs
cat /etc/cron.d/*
ls /etc/cron.daily/
```

**Cron syntax:**
```
┌───── minute (0-59)
│ ┌─────── hour (0-23)
│ │ ┌───────── day of month (1-31)
│ │ │ ┌─────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0=Sunday, 6=Saturday)
│ │ │ │ │
* * * * *   /path/to/command
```

**Common examples:**
```bash
# Run at 2:30 AM every day
30 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# Run every 5 minutes
*/5 * * * * /opt/scripts/healthcheck.sh >> /var/log/health.log 2>&1

# Run every Monday at 9 AM
0 9 * * 1 /opt/scripts/weekly-report.sh

# Run at midnight on the 1st of every month
0 0 1 * * /opt/scripts/monthly-cleanup.sh

# Run every hour at the top of the hour
0 * * * * /opt/scripts/sync.sh
```

**`>> /var/log/backup.log 2>&1` explained:**
- `>>` = append stdout (normal output) to this log file
- `2>&1` = also redirect stderr (error messages) to the same file (file descriptor 2 → 1)
- Without this, all output is silently discarded or emailed (if mail is configured)
- Always log your cron output so you can debug if something goes wrong

> **Tip:** Use [crontab.guru](https://crontab.guru) to build and visualize cron expressions interactively.

---

## 10. Performance Debugging Cheatsheet

When something is slow or broken, work through this checklist:

| Symptom | First command | Follow-up |
|---------|--------------|-----------|
| High CPU usage | `top` (press `P` to sort by CPU) | `ps aux --sort=-%cpu \| head -10` |
| High memory | `free -h` | `ps aux --sort=-%mem \| head -10` |
| Disk full | `df -h` | `du -sh /* 2>/dev/null \| sort -rh \| head -10` |
| Port already in use | `ss -tulnp \| grep PORT` | `lsof -i :PORT` |
| Service won't start | `systemctl status svc` | `journalctl -u svc -n 100 --no-pager` |
| DNS not resolving | `dig example.com` | `cat /etc/resolv.conf` |
| Container crash loop | `kubectl describe pod name` | `kubectl logs name --previous` |
| Pod stuck Pending | `kubectl describe pod name` | Look at Events — usually a resource quota or node selector issue |
| Node not ready | `kubectl describe node name` | SSH to node: `journalctl -u kubelet -n 50` |
| Slow response times | `curl -w "\nTime: %{time_total}s\n" URL` | Check `top` for CPU/memory pressure |

---

## Quick Reference One-Liners

```bash
# Watch a command refresh every 2 seconds (live dashboard)
watch -n 2 kubectl get pods -A

# Find and kill whatever process is using port 8080
kill -9 $(lsof -t -i:8080)

# Count how many times each HTTP status code appears in nginx logs
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Monitor logs live with errors highlighted (the trailing | keeps stream open)
tail -f app.log | grep --color=always -E "ERROR|WARN|$"

# Decode a Kubernetes secret value (the full pipeline)
kubectl get secret mysecret -o jsonpath='{.data.password}' | base64 --decode && echo

# List all unique container images running across the entire cluster
kubectl get pods -A -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}' | sort -u

# Check SSL certificate expiry for a domain
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Run the same command across multiple servers
for host in web1 web2 web3; do
  echo "=== $host ==="
  ssh $host "systemctl status nginx --no-pager | head -5"
done

# Find the 10 largest files anywhere on the system
find / -type f -printf '%s %p\n' 2>/dev/null | sort -rn | head -10

# Test if a remote port is reachable (without curl or telnet)
cat < /dev/null > /dev/tcp/hostname/8080 && echo "Port is open" || echo "Port is closed"
```

---

## What to Practice Next

- **[KillerCoda](https://killercoda.com)** — Free browser-based Linux and Kubernetes labs. No setup needed. Highly recommended.
- **[Play with Docker](https://labs.play-with-docker.com)** — Free Docker playground in the browser.
- **[Linux Journey](https://linuxjourney.com)** — Structured beginner course with interactive exercises.
- **[crontab.guru](https://crontab.guru)** — Visual cron expression builder.
- **CKA (Certified Kubernetes Administrator)** — The gold-standard Kubernetes certification.
- **AWS CLI** — Pair these Linux skills with cloud operations using the AWS command-line tool.

> The fastest way to learn is to have a real goal — deploy an app, set up monitoring, build a CI/CD pipeline. Commands stick when they solve an actual problem. Build, break, fix, repeat.

← Back to **[Linux Theory](/blog/linux-theory)**