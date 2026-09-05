---
title: "30 Real-World Linux Troubleshooting Scenarios for DevOps Engineers"
date: 2026-09-05
draft: false
categories: ["linux", "devops"]
description: "A practical, beginner-friendly playbook of the most common production incidents DevOps engineers face — with plain-English explanations, real examples, diagnosis commands, and fixes."
---

# 30 Real-World Linux Troubleshooting Scenarios for DevOps Engineers

When something breaks in production, you don't have time to Google slowly. You need a mental checklist: *what's the symptom, what usually causes it, how do I confirm it, and how do I fix it.*

This guide walks through 30 of the most common incidents a DevOps engineer will face, explained in plain English with real commands. Each scenario follows the same format so it's easy to memorize:

- **Symptom** — what you (or a user) actually notice
- **Common Cause** — what's usually behind it
- **How to Diagnose** — commands to confirm the cause
- **How to Fix** — the typical resolution
- **Junior Tip** — a simple way to remember it

---

# Phase 1 — Core Linux Incidents

## 1. Server Slow / High Load

**Symptom:** The server feels sluggish. Commands take longer to respond. Users complain the app is "lagging."

**Common Cause:** Too many processes competing for CPU, or the CPU is maxed out by one runaway process.

**How to Diagnose:**
```bash
uptime
top
```
`uptime` shows the "load average." If the number is much higher than your CPU core count (e.g., load of 8 on a 4-core server), the system is overloaded.

**Real Example:** A batch job that should finish in 5 minutes gets stuck in a loop and keeps consuming CPU. `top` shows it using 300% CPU non-stop.

**How to Fix:**
```bash
kill -9 <PID>
```
Stop the runaway process, then investigate why it looped (bad code, bad input data, etc.).

**Junior Tip:** Think of load average like cars waiting at a toll booth — a high number means more cars (tasks) are waiting than the booths (CPU cores) can handle.

---

## 2. Disk Full

**Symptom:** Application fails to write files. You see errors like `No space left on device`.

**Common Cause:** Log files or temp files have grown huge and filled up the disk.

**How to Diagnose:**
```bash
df -h
du -sh /var/log/* | sort -rh | head -10
```
`df -h` shows which disk/partition is full. `du` finds which folder is eating the space.

**Real Example:** An application writes verbose debug logs and nobody set up log rotation. After a few weeks, `/var/log/app.log` grows to 40GB and fills the disk.

**How to Fix:**
```bash
truncate -s 0 /var/log/app.log
```
Clear the file safely (without deleting it, so the app doesn't error out), then set up `logrotate` to prevent it from happening again.

**Junior Tip:** `df` = disk free (how much space is left). `du` = disk usage (what's using the space). Easy to mix up — remember "du digs into folders."

---

## 3. Application Down

**Symptom:** Users can't access the application. Browser shows "This site can't be reached" or similar.

**Common Cause:** The application process crashed, or the service that runs it stopped.

**How to Diagnose:**
```bash
systemctl status myapp
ps aux | grep myapp
journalctl -u myapp -n 50
```
Check if the service is active, if the process exists, and read the last log lines for an error.

**Real Example:** A Node.js app crashes after an unhandled exception, and since it wasn't managed by systemd with auto-restart, it just stays down.

**How to Fix:**
```bash
systemctl restart myapp
```
Restart the service, then check the logs to find and fix the root cause (in this case, add a `Restart=always` policy in the systemd unit file).

**Junior Tip:** Always check the logs *before* restarting blindly — restarting can hide the real problem if you don't capture the error first.

---

## 4. Nginx → Backend 502 (Bad Gateway)

**Symptom:** Users see "502 Bad Gateway" in the browser.

**Common Cause:** Nginx is running fine, but the backend application it's forwarding requests to (like a Node.js, Python, or Java app) is down or not responding.

**How to Diagnose:**
```bash
systemctl status myapp
curl -I http://127.0.0.1:8080
tail -f /var/log/nginx/error.log
```
Check if the backend is even running, and try hitting it directly to bypass Nginx.

**Real Example:** The backend app on port `8080` crashed due to a memory leak. Nginx is still up and accepting requests, but has nothing to forward them to — hence the 502.

**How to Fix:**
```bash
systemctl restart myapp
```
Restart the backend service. Then investigate the crash logs to find the root cause.

**Junior Tip:** Think of Nginx as a receptionist and the backend app as the actual doctor. A 502 means the receptionist is fine, but the doctor isn't in the office.

---

## 5. High CPU Process

**Symptom:** Server feels slow, and `top` shows one process using an unusually high percentage of CPU.

**Common Cause:** An inefficient loop, a stuck script, or a process handling more traffic than it should.

**How to Diagnose:**
```bash
top
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head -10
```
Sort processes by CPU usage to quickly spot the offender.

**Real Example:** A cron job meant to run once accidentally gets triggered multiple times in parallel, and all instances are competing for CPU.

**How to Fix:**
```bash
kill -9 <PID>
```
Kill the extra duplicate processes, then fix the cron schedule or add a lock file to prevent overlapping runs.

**Junior Tip:** `%CPU` in `top` can go above 100% on multi-core systems — 300% means the process is using 3 full cores.

---

## 6. High Memory / OOM (Out of Memory)

**Symptom:** The application randomly gets killed. Logs show "Out of memory" or the process just disappears.

**Common Cause:** A process (often due to a memory leak) consumes all available RAM, and the Linux kernel's OOM killer steps in and kills something to save the system.

**How to Diagnose:**
```bash
free -h
dmesg | grep -i "out of memory"
journalctl -k | grep -i oom
```
Check available memory and look at kernel logs — the OOM killer always logs which process it killed and why.

**Real Example:** A Java application isn't given a memory limit (`-Xmx`), so it keeps growing until it consumes all server RAM, and the kernel kills it.

**How to Fix:**
Restart the service, then set proper memory limits for the application (e.g., `-Xmx2g` for Java, or a `MemoryMax=` setting in the systemd unit).

**Junior Tip:** OOM killer is like a landlord — if one tenant (process) uses all the building's water (RAM), the landlord evicts them to keep the building running.

---

## 7. High I/O Wait

**Symptom:** CPU usage looks low, but the server still feels slow. Commands hang for a second before responding.

**Common Cause:** The disk can't keep up with read/write requests — CPU is waiting on the disk, not doing actual work.

**How to Diagnose:**
```bash
top
vmstat 2 5
iostat -x 2 5
```
In `top`, check the `%wa` (I/O wait) value. A high `%wa` means the CPU is idle, just waiting for disk operations to finish.

**Real Example:** A backup job runs during business hours and heavily reads/writes to the same disk the database uses, slowing everything down.

**How to Fix:**
Reschedule heavy disk jobs (like backups) to off-peak hours, or move them to a separate disk. If it's a genuine disk performance problem, consider upgrading to faster storage (e.g., SSD).

**Junior Tip:** High CPU = the CPU is busy working. High I/O wait = the CPU is bored, just waiting for the slow disk to respond.

---

## 8. Inode Exhaustion

**Symptom:** You get `No space left on device`, but `df -h` shows plenty of free space!

**Common Cause:** Every file needs an "inode" (a small metadata entry), and a filesystem has a limited number of them. If you have millions of tiny files, you can run out of inodes even with disk space to spare.

**How to Diagnose:**
```bash
df -i
```
This shows inode usage instead of disk space usage. If `IUse%` is at 100%, that's your problem.

**Real Example:** A caching application creates millions of tiny temp files and never cleans them up. Eventually, the filesystem runs out of inodes, even though there's 50GB of free space.

**How to Fix:**
```bash
find /path/to/cache -type f -mtime +1 -delete
```
Delete the excess small files. Long-term, fix the app to clean up its own temp files or use a proper cache expiry policy.

**Junior Tip:** Disk space = how much *room* you have. Inodes = how many *file slots* you have. You can run out of one before the other — like running out of parking spots even though the lot has room, because each spot only fits one car.

---

## 9. Deleted File Still Consuming Disk

**Symptom:** You deleted a large log file, but `df -h` still shows the disk as full.

**Common Cause:** A running process still has the file open. Linux doesn't actually free the disk space until every process holding the file closes it.

**How to Diagnose:**
```bash
lsof | grep deleted
```
This lists files that were deleted but are still held open by a process — you'll see `(deleted)` next to the file name.

**Real Example:** You `rm` a giant `app.log` file to free up space, but the application that was writing to it is still running and holding the file handle open, so the space isn't released.

**How to Fix:**
```bash
systemctl restart myapp
```
Restarting the process releases the file handle and actually frees the disk space. Going forward, use `truncate -s 0 app.log` instead of deleting log files that are actively being written to.

**Junior Tip:** Deleting a file just removes its name from the folder listing — if a program still has it "open," the disk space stays locked until that program lets go.

---

## 10. Service Repeatedly Restarting

**Symptom:** A service keeps crashing and restarting in a loop (sometimes called a "crash loop").

**Common Cause:** The application has a startup error — a bad config file, a missing dependency, or a port conflict — so it crashes immediately after systemd restarts it.

**How to Diagnose:**
```bash
systemctl status myapp
journalctl -u myapp -n 100 --no-pager
```
Look at the logs right before each crash — the error is usually the same each time.

**Real Example:** A config file has a typo in the database password. The app starts, tries to connect, fails, and exits — systemd restarts it, and the cycle repeats endlessly.

**How to Fix:**
Fix the root cause shown in the logs (in this case, correct the password in the config file), then restart the service once more to confirm it stays up.

**Junior Tip:** A crash loop is systemd doing exactly what it's told — "always restart on failure." The real bug is always in the *first* crash, so read the earliest error, not just the latest one.


---

# Phase 2 — Networking

## 11. Application Works Locally but Not Remotely

**Symptom:** `curl http://localhost:8080` works fine on the server, but nobody outside can reach it.

**Common Cause:** The app is bound to `127.0.0.1` (localhost only) instead of `0.0.0.0` (all network interfaces), or a firewall is blocking the port.

**How to Diagnose:**
```bash
ss -tulwn | grep 8080
```
If you see `127.0.0.1:8080`, that's the problem — it's only listening on the loopback interface, not the external one.

**Real Example:** A developer's app config has `host: localhost` hardcoded, so it only accepts connections from the same machine.

**How to Fix:**
Change the app's bind address to `0.0.0.0` (all interfaces) and restart it. Then double-check the firewall allows external traffic on that port.

**Junior Tip:** `127.0.0.1` means "talk to myself only." `0.0.0.0` means "accept connections from anyone who can reach me."

---

## 12. Connection Refused

**Symptom:** `curl: (7) Failed to connect... Connection refused`.

**Common Cause:** Nothing is listening on that port at all — either the service isn't running, or you're using the wrong port.

**How to Diagnose:**
```bash
ss -tulwn | grep <port>
systemctl status myapp
```
If nothing shows up for that port, the service simply isn't running there.

**Real Example:** You try to connect to port `5432` for PostgreSQL, but the database service failed to start after a server reboot.

**How to Fix:**
```bash
systemctl start postgresql
```
Start the service, and confirm it's now listening with `ss -tulwn`.

**Junior Tip:** "Connection refused" means the door isn't even there. "Timeout" (next scenario) means the door exists, but nobody's answering.

---

## 13. Connection Timeout

**Symptom:** `curl` or your browser just hangs and eventually times out — no error, no response.

**Common Cause:** A firewall (on the server, in the cloud provider's security group, or in between) is silently dropping the traffic.

**How to Diagnose:**
```bash
telnet <host> <port>
traceroute <host>
```
If `telnet` hangs instead of connecting or refusing, traffic is likely being silently blocked somewhere along the path.

**Real Example:** A new server is spun up in AWS, but the security group only allows port 22 (SSH) — port 80 (HTTP) is blocked, so web requests just hang.

**How to Fix:**
Update the firewall or security group rules to allow traffic on the required port.

**Junior Tip:** "Refused" is a fast, honest "no." "Timeout" is silence — like knocking on a door that ignores you completely, usually because a firewall is standing guard.

---

## 14. DNS Resolution Failure

**Symptom:** `curl: Could not resolve host: example.com`.

**Common Cause:** The DNS server isn't reachable, is misconfigured, or the domain simply doesn't exist / isn't propagated yet.

**How to Diagnose:**
```bash
dig example.com
cat /etc/resolv.conf
```
`dig` tells you whether the domain resolves to an IP at all. `/etc/resolv.conf` shows which DNS servers your machine is using.

**Real Example:** A server's `/etc/resolv.conf` points to an internal DNS server that was decommissioned, so every hostname lookup fails.

**How to Fix:**
Update `/etc/resolv.conf` (or your DNS settings via DHCP/cloud config) to point to a working DNS server, like `8.8.8.8`.

**Junior Tip:** DNS is like a phone book — it turns names (example.com) into numbers (IP addresses). If the phone book is broken, you can't "dial" the name even if the actual server is fine.

---

## 15. Port Listening but Inaccessible

**Symptom:** `ss -tulwn` shows the port is listening, but you still can't connect from outside.

**Common Cause:** A firewall (like `iptables`, `firewalld`, or a cloud security group) is blocking external access to that port, even though the app itself is running fine.

**How to Diagnose:**
```bash
ss -tulwn | grep <port>
sudo iptables -L -n
sudo firewall-cmd --list-all
```
Confirm the app is listening, then check firewall rules for that specific port.

**Real Example:** An app is correctly running and listening on port `443`, but the server's firewall only has port `80` open, so HTTPS traffic never gets through.

**How to Fix:**
```bash
sudo firewall-cmd --add-port=443/tcp --permanent
sudo firewall-cmd --reload
```
Open the required port in the firewall.

**Junior Tip:** A listening port is like a shop being open — but if there's a locked gate (firewall) outside, customers still can't get in.

---

## 16. TIME_WAIT / CLOSE_WAIT Explosion

**Symptom:** The server slows down or refuses new connections, and you notice thousands of connections stuck in `TIME_WAIT` or `CLOSE_WAIT` state.

**Common Cause:** `TIME_WAIT` is usually normal (connections briefly waiting to fully close) — but too many at once can exhaust available ports. `CLOSE_WAIT` piling up usually means the *application* isn't closing connections properly.

**How to Diagnose:**
```bash
ss -tan | awk '{print $1}' | sort | uniq -c | sort -rn
```
This counts how many connections are in each state — if `CLOSE_WAIT` is huge, that's an app-level bug.

**Real Example:** An application opens a connection to a database for every request but never closes it properly, so `CLOSE_WAIT` connections pile up until the server runs out of file descriptors.

**How to Fix:**
Restart the affected service short-term. Long-term, fix the application code to properly close connections (or use a connection pool).

**Junior Tip:** `TIME_WAIT` = "we said goodbye, just confirming no stray messages." `CLOSE_WAIT` = "the other side said goodbye, but *we* haven't hung up yet" — this one usually points to a code bug.

---

## 17. Network Latency / Packet Loss

**Symptom:** Requests are slow or intermittently fail, but the server itself isn't overloaded (CPU, memory, disk all look fine).

**Common Cause:** The network path between client and server has latency or is dropping packets — could be a bad network link, an overloaded switch, or a distant server.

**How to Diagnose:**
```bash
ping -c 20 <host>
traceroute <host>
mtr <host>
```
`ping` shows packet loss percentage and response times. `mtr` (a combination of ping + traceroute) shows exactly which hop along the path is causing delays or drops.

**Real Example:** A server in one cloud region talks to a database in a different region across the world, adding 150ms+ latency to every single query.

**How to Fix:**
Move services closer together (same region/availability zone), or if it's a genuine network issue, escalate to your network/cloud provider team.

**Junior Tip:** `ping` tells you *if* there's a problem. `traceroute`/`mtr` tells you *where* along the journey the problem is happening.


---

# Phase 3 — Permissions & System

## 18. Permission Denied

**Symptom:** A command or application fails with `Permission denied`.

**Common Cause:** The user or process doesn't have the right file permissions to read, write, or execute a file.

**How to Diagnose:**
```bash
ls -l /path/to/file
whoami
```
Check the file's permission bits and owner, and compare against the user trying to access it.

**Real Example:** A deployment script tries to run `deploy.sh`, but the file was uploaded without execute permission.

**How to Fix:**
```bash
chmod +x deploy.sh
```
Add execute permission to the file. For read/write issues, adjust with the correct `chmod` value instead.

**Junior Tip:** Permissions in Linux are Read (r), Write (w), Execute (x) — for Owner, Group, and Others. `chmod +x` just adds the "execute" permission without touching the rest.

---

## 19. Wrong Ownership

**Symptom:** A service fails to read its own config or log files, even though the permissions look fine.

**Common Cause:** The file is owned by the wrong user — for example, a config file owned by `root` when the app actually runs as a different, unprivileged user.

**How to Diagnose:**
```bash
ls -l /etc/myapp/config.yaml
ps -ef | grep myapp
```
Check who owns the file versus which user the application process is actually running as.

**Real Example:** An app runs as user `appuser`, but its log directory was created by `root` during setup, so `appuser` can't write logs and the app crashes on startup.

**How to Fix:**
```bash
chown -R appuser:appuser /var/log/myapp
```
Change ownership of the folder (and its contents) to match the user running the application.

**Junior Tip:** Permissions decide *what* can be done (read/write/execute). Ownership decides *who* those permissions apply to. Both need to be correct.

---

## 20. SSH Login Failure

**Symptom:** You can't SSH into a server — you get `Permission denied (publickey)` or the connection just hangs.

**Common Cause:** Wrong SSH key, incorrect permissions on the `.ssh` folder or key files, or the SSH service itself isn't running.

**How to Diagnose:**
```bash
ssh -v user@server
systemctl status sshd
ls -la ~/.ssh
```
The `-v` (verbose) flag on `ssh` shows exactly where the handshake is failing.

**Real Example:** A user's `~/.ssh` folder has overly open permissions (e.g., `777`), and SSH refuses to use it for security reasons, silently failing authentication.

**How to Fix:**
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
SSH is strict about permissions — the `.ssh` folder should be `700`, and key files inside should be `600`.

**Junior Tip:** SSH treats loose permissions on your key files as a security red flag and will refuse to use them — this is a very common "gotcha" for beginners.

---

## 21. Service Fails After Reboot

**Symptom:** Everything worked fine before a server reboot, but after restarting, the application doesn't come back up.

**Common Cause:** The service was started manually (not via systemd) and was never "enabled" to auto-start on boot.

**How to Diagnose:**
```bash
systemctl is-enabled myapp
systemctl status myapp
```
`is-enabled` tells you whether the service is configured to start automatically at boot.

**Real Example:** An engineer manually ran `myapp start` months ago and never registered it as a proper systemd service. After a routine server reboot, nobody remembers to start it again.

**How to Fix:**
```bash
systemctl enable myapp
systemctl start myapp
```
Enable the service so it starts on every future boot, then start it now.

**Junior Tip:** "Started" means running *right now*. "Enabled" means it will *automatically* start on the next boot. You usually want both.

---

## 22. systemd Service Won't Start

**Symptom:** `systemctl start myapp` fails immediately, or the service shows as `failed`.

**Common Cause:** A misconfigured systemd unit file — wrong file path, wrong user, or a missing dependency.

**How to Diagnose:**
```bash
systemctl status myapp
journalctl -xeu myapp
```
The status command usually shows the exact error, and `journalctl` gives more detailed logs.

**Real Example:** A unit file points to `/usr/bin/myapp`, but the binary was actually installed at `/usr/local/bin/myapp` — systemd can't find the executable.

**How to Fix:**
Correct the `ExecStart=` path in the unit file (`/etc/systemd/system/myapp.service`), then reload systemd and try again:
```bash
systemctl daemon-reload
systemctl start myapp
```

**Junior Tip:** Always run `daemon-reload` after editing any `.service` file — systemd caches unit files in memory and won't notice your edit otherwise.

---

## 23. Environment / Configuration Problem

**Symptom:** The application starts but behaves incorrectly — connects to the wrong database, uses the wrong API keys, or throws unexpected errors.

**Common Cause:** Missing or incorrect environment variables, or the app is loading the wrong config file for the environment (e.g., using "dev" settings in production).

**How to Diagnose:**
```bash
systemctl show myapp -p Environment
env | grep MYAPP
cat /etc/myapp/.env
```
Compare what environment variables the process actually has against what it expects.

**Real Example:** A deployment script forgets to set `DATABASE_URL` for the production environment, so the app silently falls back to a default (development) database.

**How to Fix:**
Set the correct environment variables in the systemd unit file or `.env` file, then restart the service.

**Junior Tip:** When an app "runs but behaves weird," always suspect configuration before code — check what values it's actually reading, not what you *assume* it's reading.


---

# Phase 4 — Advanced Production Incidents

## 24. Filesystem Becomes Read-Only

**Symptom:** Applications suddenly fail to write files, and you see errors like `Read-only file system`, even though nothing was intentionally changed.

**Common Cause:** The kernel automatically remounts a filesystem as read-only when it detects disk corruption or hardware errors, to prevent further data damage.

**How to Diagnose:**
```bash
mount | grep " / "
dmesg | grep -i "read-only\|error"
```
Check the current mount options, and look at kernel messages for disk errors that triggered the remount.

**Real Example:** An underlying cloud storage volume has an intermittent hardware fault. The kernel detects filesystem corruption and protects the disk by forcing it into read-only mode.

**How to Fix:**
```bash
umount /mnt/data
fsck /dev/sdb1
mount /dev/sdb1 /mnt/data
```
Run a filesystem check and repair, then remount. In cloud environments, this is often better solved by replacing the volume entirely and restoring from backup.

**Junior Tip:** A read-only filesystem is the kernel's way of saying "I don't trust this disk anymore, so I'm protecting your data by refusing to write more to it."

---

## 25. File Descriptor Exhaustion

**Symptom:** The application throws errors like `Too many open files`, even though the server has plenty of CPU, memory, and disk space.

**Common Cause:** Every open file, socket, or network connection uses a "file descriptor," and each process has a limit. A leak (like unclosed network connections) can exhaust this limit.

**How to Diagnose:**
```bash
ulimit -n
lsof -p <PID> | wc -l
```
`ulimit -n` shows the current limit; `lsof` counts how many file descriptors a specific process is actually using.

**Real Example:** A web server opens a new file descriptor for every incoming connection but never closes old ones, eventually hitting the default limit of 1024.

**How to Fix:**
Restart the affected service short-term. Increase the limit if the app genuinely needs more (`/etc/security/limits.conf`), and fix the underlying code to properly close connections.

**Junior Tip:** Think of file descriptors as numbered tickets a process hands out for every file or connection it opens. If it never gives tickets back, it eventually runs out.

---

## 26. Process Stuck in Uninterruptible I/O

**Symptom:** A process appears "stuck" — it doesn't respond to `kill`, and `top` shows its state as `D` (uninterruptible sleep).

**Common Cause:** The process is waiting on a slow or failing disk/network filesystem (like NFS) and the kernel won't let it be interrupted mid-operation, to avoid data corruption.

**How to Diagnose:**
```bash
ps -eo pid,stat,cmd | grep " D "
cat /proc/<PID>/stack
```
The `D` state in `ps`/`top` specifically means uninterruptible sleep — usually tied to I/O.

**Real Example:** An application reads from a network-mounted NFS share, and the remote NFS server becomes unresponsive. The local process freezes in `D` state, waiting forever for a response that never comes.

**How to Fix:**
You often can't kill a `D` state process directly — you may need to fix the underlying storage issue (restore the NFS server) or, as a last resort, reboot the machine.

**Junior Tip:** A process in `D` state isn't broken by choice — it's politely waiting for the disk/network to respond, and the kernel refuses to let anything interrupt it mid-write to protect your data.

---

## 27. Application Memory Leak

**Symptom:** Memory usage for a specific process climbs steadily over hours or days until the app crashes or gets OOM-killed — then the cycle repeats after a restart.

**Common Cause:** The application allocates memory (objects, caches, connections) but never releases it, so usage keeps growing over time instead of stabilizing.

**How to Diagnose:**
```bash
watch -n 5 'ps -eo pid,cmd,%mem,rss --sort=-%mem | head -5'
```
Watch the process's memory (RSS) over time — a leak shows a steady upward trend rather than memory going up and down as expected.

**Real Example:** A caching layer in an application keeps adding entries to an in-memory dictionary but never removes old ones, so memory grows a little with every request until the server runs out.

**How to Fix:**
Restart the service as an immediate fix to restore stability. Long-term, the development team needs to profile the app (e.g., with a memory profiler) and fix the code causing the leak — like adding cache expiry or properly releasing resources.

**Junior Tip:** Normal memory usage goes up and down as the app does work. A leak only ever goes up — that steady, one-directional climb is the giveaway.

---

## 28. Disk I/O Bottleneck

**Symptom:** Database queries or file operations are consistently slow, even though CPU and memory both look healthy.

**Common Cause:** The disk itself can't handle the volume of read/write requests — too many processes hitting the same disk, or the disk hardware is simply too slow for the workload.

**How to Diagnose:**
```bash
iostat -x 2 5
```
Look at `%util` (how busy the disk is) and `await` (how long requests take to complete). Consistently high values point to a genuine disk bottleneck.

**Real Example:** A database and its own backup process share the same physical disk. When the backup runs, database query times spike because both are competing for the same limited disk throughput.

**How to Fix:**
Move competing workloads to separate disks, upgrade to faster storage (like SSD/NVMe), or schedule heavy I/O tasks (backups, log shipping) during off-peak hours.

**Junior Tip:** `iostat`'s `%util` is like checking how busy a highway is — close to 100% means it's essentially a traffic jam, and `await` tells you how long each car (request) is stuck waiting.

---

## 29. Server Unexpectedly Rebooted

**Symptom:** You find that a server restarted on its own, with no one having triggered it manually.

**Common Cause:** Could be a kernel panic, hardware failure, cloud provider host maintenance, or an automatic security update that included a reboot.

**How to Diagnose:**
```bash
last reboot
journalctl --list-boots
journalctl -k -b -1 | tail -50
```
`last reboot` shows reboot history. Checking the *previous* boot's kernel logs (`-b -1`) often reveals the reason — a kernel panic message, an OOM event, or a clean shutdown signal from the cloud provider.

**Real Example:** A cloud provider performs scheduled host maintenance and live-migrates or restarts the underlying virtual machine, which shows up simply as an unexpected reboot from the guest OS's point of view.

**How to Fix:**
If it was a one-time event (like cloud maintenance), no fix is needed beyond confirming services came back up correctly (see Scenario 21). If it's a kernel panic or hardware issue, escalate to the infrastructure/hardware team and consider replacing the underlying host.

**Junior Tip:** Don't assume the worst immediately — check `journalctl --list-boots` first; sometimes it's a routine, expected event like a scheduled patch or cloud maintenance window.

---

## 30. Multiple Symptoms Simultaneously

**Symptom:** Several things seem to be going wrong at once — high CPU, slow disk, and application errors all appearing together. It's hard to tell what's cause and what's effect.

**Common Cause:** Usually one root problem is triggering a chain reaction. For example, a disk bottleneck can cause high CPU wait time, which causes requests to queue up, which causes memory to spike, which causes the app to become unresponsive.

**How to Diagnose:**
Work from the bottom up — check the most fundamental resources first, since problems often cascade upward:
```bash
df -h        # disk space
free -h      # memory
top          # CPU and load
iostat -x 2 5  # disk I/O
journalctl -xe # recent system errors
```
Look for whichever metric was abnormal *first*, using timestamps in logs — that's usually your true root cause, and everything else is a symptom of it.

**Real Example:** A disk fills up (root cause) → the database can't write → queries start failing → the application retries aggressively → CPU and memory usage spike from all the retries → the whole system looks like it's on fire, when the real issue was simply "disk full."

**How to Fix:**
Resolve the earliest, most fundamental issue first (in this example, free up disk space), then re-check the other metrics — often, fixing the root cause makes the "symptoms" disappear on their own.

**Junior Tip:** When everything looks broken at once, don't panic and chase every alert individually. Check resources in order — disk, memory, CPU, network — and look for whichever one broke *first*. That's usually your real problem; the rest is just fallout.

---

## Quick Reference Table

| # | Scenario | First Command to Run |
|---|----------|----------------------|
| 1 | Server slow / high load | `uptime`, `top` |
| 2 | Disk full | `df -h` |
| 3 | Application down | `systemctl status` |
| 4 | Nginx 502 | `curl` backend directly |
| 5 | High CPU process | `top` |
| 6 | High memory / OOM | `free -h`, `dmesg` |
| 7 | High I/O wait | `top` (`%wa`), `iostat` |
| 8 | Inode exhaustion | `df -i` |
| 9 | Deleted file still using disk | `lsof \| grep deleted` |
| 10 | Service crash loop | `journalctl -u <service>` |
| 11 | Works locally, not remotely | `ss -tulwn` |
| 12 | Connection refused | `ss -tulwn` |
| 13 | Connection timeout | `telnet host port` |
| 14 | DNS failure | `dig`, `/etc/resolv.conf` |
| 15 | Port open but blocked | firewall rules |
| 16 | TIME_WAIT/CLOSE_WAIT | `ss -tan \| sort \| uniq -c` |
| 17 | Latency / packet loss | `ping`, `mtr` |
| 18 | Permission denied | `ls -l` |
| 19 | Wrong ownership | `ls -l`, `ps -ef` |
| 20 | SSH login failure | `ssh -v` |
| 21 | Fails after reboot | `systemctl is-enabled` |
| 22 | systemd won't start | `systemctl status`, `journalctl -xeu` |
| 23 | Env/config problem | check env vars & config file |
| 24 | Filesystem read-only | `dmesg`, `mount` |
| 25 | File descriptor exhaustion | `ulimit -n`, `lsof` |
| 26 | Stuck in D state | `ps -eo stat` |
| 27 | Memory leak | watch RSS over time |
| 28 | Disk I/O bottleneck | `iostat -x` |
| 29 | Unexpected reboot | `last reboot`, `journalctl -k -b -1` |
| 30 | Multiple symptoms | check disk → memory → CPU → network in order |

---

## Final Thoughts

You don't need to memorize exact commands for every scenario on day one. What matters more is the *thinking pattern*:

1. **What's the symptom?** (What do users or logs actually show?)
2. **What's the most likely cause?** (Based on experience and common patterns.)
3. **How do I confirm it?** (Use the right diagnostic command, don't guess.)
4. **How do I fix it — now, and permanently?** (A quick fix stops the bleeding; a root-cause fix prevents it from happening again.)

Practice these scenarios in a test environment if you can. The more of these you see hands-on, the faster your instincts get — and that's really what separates a junior engineer from a senior one during a 2 AM incident.