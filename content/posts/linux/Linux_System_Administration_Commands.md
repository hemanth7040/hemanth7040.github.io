---
title: "Linux System Administration Commands: A Practical Guide"
date: 2026-09-05
draft: false
categories: ["linux"]
description: "A simple, real-world guide to Linux commands for process management, disk usage, networking, logs, and automation — with practical examples for everyday system administration."
---

# Linux System Administration Commands: A Practical Guide

As a system administrator or engineer, you deal with servers every day — checking what's running, freeing up disk space, reading logs, or troubleshooting a network issue. This post covers the essential Linux commands you'll actually use, grouped by task, with simple real-world examples.

---

## 1. Process Management

These commands help you see what's running on your server and control it.

### `ps` — Snapshot of running processes
```bash
ps -ef
```
Lists all running processes with their process IDs (PID). Useful when you need to find a specific process before killing it.

### `top` — Live process monitor
```bash
top
```
Shows CPU and memory usage in real time, updating every few seconds. Good first command to run when a server feels slow.

### `htop` — A friendlier version of top
```bash
htop
```
Same purpose as `top`, but with colors, scrolling, and mouse support. Easier to read (needs to be installed separately).

### `pgrep` — Find a process ID by name
```bash
pgrep nginx
```
Returns the PID(s) of any process named `nginx`, without needing to scroll through the full process list.

### `pidof` — Find PID of a running program
```bash
pidof nginx
```
Similar to `pgrep`, gives you the PID of the running `nginx` process — handy in scripts.

### `kill` — Stop a process by PID
```bash
kill -9 4521
```
Force-stops the process with PID `4521`. Use this when a process is stuck or unresponsive.

### `pkill` — Stop a process by name
```bash
pkill -9 nginx
```
Kills all processes matching the name `nginx`, without needing to look up the PID first.

---

## 2. CPU and Memory Monitoring

Use these when a server is slow and you need to find out why.

### `uptime` — How long the server has been running
```bash
uptime
```
Shows system uptime along with the "load average" — a quick way to check if the CPU is under heavy load.

### `free` — Check memory usage
```bash
free -h
```
Shows total, used, and available RAM in a human-readable format (MB/GB).

### `vmstat` — Overview of system performance
```bash
vmstat 2 5
```
Reports CPU, memory, and I/O stats every 2 seconds, 5 times. Useful for spotting performance bottlenecks over a short period.

### `top` — (see above)
Also doubles as a quick CPU/memory monitor, showing which processes are consuming the most resources.

---

## 3. Disk Management

Essential when a server runs low on storage.

### `df` — Check disk space
```bash
df -h
```
Shows how much space is used and free on each mounted disk, in human-readable format.

### `du` — Check size of files and folders
```bash
du -sh /var/log/*
```
Shows the size of each item inside `/var/log` — great for finding what's eating up disk space.

### `lsblk` — List block devices
```bash
lsblk
```
Shows all disks and partitions attached to the system, in a tree view.

### `blkid` — Show disk UUIDs and filesystem types
```bash
blkid /dev/sda1
```
Displays the UUID and filesystem type (like ext4 or xfs) of a specific partition — often needed when editing `/etc/fstab`.

### `mount` — Attach a filesystem
```bash
mount /dev/sdb1 /mnt/data
```
Mounts the partition `/dev/sdb1` to the `/mnt/data` folder, making it accessible.

### `umount` — Detach a filesystem
```bash
umount /mnt/data
```
Safely unmounts `/mnt/data` before removing a disk or drive.

### `findmnt` — Show mounted filesystems
```bash
findmnt /mnt/data
```
Displays details about a specific mount point, including its source device and options.

---

## 4. File Operations

Basic but essential commands for managing files.

### `find` — Search for files by name, size, or date
```bash
find /var/log -name "*.log" -mtime +7
```
Finds all `.log` files inside `/var/log` that are older than 7 days — useful for cleanup scripts.

### `locate` — Quickly find files by name
```bash
locate nginx.conf
```
Searches a prebuilt index for files matching `nginx.conf`. Much faster than `find`, but the index needs to be updated periodically (`updatedb`).

### `stat` — Detailed file information
```bash
stat app.log
```
Shows file size, permissions, and last modified/accessed time — more detail than a simple `ls`.

### `ls` — List directory contents
```bash
ls -la
```
Lists all files, including hidden ones, with permissions and sizes.

### `cp` — Copy files
```bash
cp config.yaml config.yaml.bak
```
Creates a backup copy before editing an important file.

### `mv` — Move or rename files
```bash
mv app.log.bak /backup/
```
Moves a file into the `/backup` folder.

### `rm` — Delete files
```bash
rm -f old_report.txt
```
Deletes `old_report.txt` without asking for confirmation. **Use carefully.**

---

## 5. Text Processing

These commands help you search, filter, and transform text — especially useful for logs and config files.

### `grep` — Search for text in files
```bash
grep "ERROR" app.log
```
Finds every line in `app.log` containing the word `ERROR`.

### `awk` — Process and extract columns of text
```bash
awk '{print $1, $NF}' access.log
```
Prints the first and last column of each line — for example, pulling out IP addresses and status codes from a web server log.

### `sed` — Find and replace text
```bash
sed -i 's/staging/production/g' app.conf
```
Replaces every occurrence of `staging` with `production` directly inside `app.conf`.

### `cut` — Extract specific fields
```bash
cut -d ':' -f1 /etc/passwd
```
Splits each line by `:` and prints only the first field — here, the usernames on the system.

### `sort` — Sort lines of text
```bash
sort -n numbers.txt
```
Sorts the contents of `numbers.txt` numerically.

### `uniq` — Remove duplicate lines
```bash
sort access.log | uniq -c | sort -nr
```
Counts how many times each line appears — a common way to find the most frequent IP addresses hitting a server.

### `tr` — Translate or delete characters
```bash
echo "Hello World" | tr 'a-z' 'A-Z'
```
Converts lowercase letters to uppercase — useful in shell scripts for formatting text.

---

## 6. Log Viewing

Reading logs quickly is one of the most common admin tasks.

### `journalctl` — View systemd logs
```bash
journalctl -u nginx -f
```
Streams live logs for the `nginx` service managed by systemd.

### `tail` — View the end of a file
```bash
tail -f app.log
```
Shows the last lines of `app.log` and keeps updating as new lines are added — great for watching logs during a deployment.

### `head` — View the beginning of a file
```bash
head -n 50 app.log
```
Shows the first 50 lines of `app.log` — useful for checking when logging started.

### `less` — View large files page by page
```bash
less app.log
```
Opens `app.log` in a scrollable view without loading the whole file into memory — much better than `cat` for large files.

---

## 7. Networking

Commands to check connectivity and diagnose network issues.

### `ip` — Show and manage network interfaces
```bash
ip addr show
```
Displays IP addresses assigned to each network interface on the machine.

### `ss` — Show open ports and connections
```bash
ss -tulwn
```
Lists all listening ports and active connections — the modern replacement for `netstat`.

### `ping` — Check if a host is reachable
```bash
ping -c 4 google.com
```
Sends 4 test packets to `google.com` to check connectivity.

### `curl` — Test APIs and web servers
```bash
curl -I https://example.com
```
Fetches just the HTTP response headers — a quick way to check if a service is responding.

### `dig` — Look up DNS records
```bash
dig example.com
```
Shows the DNS resolution details for `example.com`, including its IP address.

### `traceroute` — Trace the network path to a host
```bash
traceroute example.com
```
Shows every network hop between your machine and `example.com` — helpful for diagnosing where a connection is slow or failing.

---

## 8. Service Management

Controlling background services on modern Linux systems.

### `systemctl` — Manage system services
```bash
systemctl restart nginx
systemctl status nginx
```
Restarts the `nginx` service and checks whether it's running properly.

### `journalctl` — (see above)
Also used to check logs specifically related to a service's startup or failure.

---

## 9. User and Permission Management

Commands for checking identity and controlling access.

### `id` — Show user and group IDs
```bash
id ravi
```
Displays the user ID (UID) and group memberships for the user `ravi`.

### `who` — Show who is logged in
```bash
who
```
Lists all users currently logged into the system.

### `w` — Show logged-in users and their activity
```bash
w
```
Similar to `who`, but also shows what each user is currently doing.

### `chmod` — Change file permissions
```bash
chmod 644 config.yaml
```
Sets read/write for the owner and read-only for everyone else.

### `chown` — Change file ownership
```bash
chown ravi:devops app.conf
```
Changes the owner of `app.conf` to user `ravi` and group `devops`.

### `sudo` — Run a command as another user (usually root)
```bash
sudo systemctl restart nginx
```
Runs the restart command with root privileges, which are often needed to manage services.

---

## 10. File and Process Investigation (Advanced)

Deeper tools for debugging tricky issues.

### `lsof` — List open files
```bash
lsof -i :8080
```
Shows which process is using port `8080` — very useful when you get a "port already in use" error.

### `strace` — Trace system calls made by a program
```bash
strace -p 4521
```
Shows the low-level system calls a running process (PID `4521`) is making — helpful for debugging why a program is hanging or failing.

---

## 11. Automation

Commands for scripting and scheduling tasks.

### `bash` — Run shell scripts
```bash
bash backup.sh
```
Executes the script `backup.sh` using the Bash shell.

### `cron` — Schedule recurring tasks
```bash
crontab -e
```
Opens the crontab editor, where you can schedule a command to run automatically. For example, adding this line runs a backup script every day at 2 AM:
```
0 2 * * * /home/ravi/backup.sh
```

---

## Quick Reference Table

| Category      | Commands                                              |
|---------------|--------------------------------------------------------|
| Process       | `ps`, `top`, `htop`, `pgrep`, `pidof`, `kill`, `pkill`  |
| CPU/Memory    | `uptime`, `free`, `vmstat`, `top`                       |
| Disk          | `df`, `du`, `lsblk`, `blkid`, `mount`, `umount`, `findmnt` |
| Files         | `find`, `locate`, `stat`, `ls`, `cp`, `mv`, `rm`        |
| Text          | `grep`, `awk`, `sed`, `cut`, `sort`, `uniq`, `tr`       |
| Logs          | `journalctl`, `tail`, `head`, `less`                    |
| Network       | `ip`, `ss`, `ping`, `curl`, `dig`, `traceroute`         |
| Services      | `systemctl`, `journalctl`                                |
| Users         | `id`, `who`, `w`, `chmod`, `chown`, `sudo`               |
| Investigation | `lsof`, `strace`                                         |
| Automation    | `bash`, `cron`                                            |

---

## Final Thoughts

You don't need to memorize every flag for every command. Start by learning what each command is *for*, and practice the ones you use most — `top`, `df`, `grep`, and `systemctl` are a great place to begin.

Over time, as you troubleshoot real issues on real servers, these commands will become second nature.