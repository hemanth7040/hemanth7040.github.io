---
title: "Linux File System Hierarchy Explained (For Beginners)"
date: 2026-09-05
draft: false
categories: ["linux"]
description: "A beginner-friendly guide to the Linux file system hierarchy — what each top-level folder like /home, /etc, /var, and /usr is for, with simple examples."
---

# Linux File System Hierarchy Explained (For Beginners)

When you start using Linux, one of the first things that feels strange is the folder structure. There is no `C:\` drive like Windows. Instead, everything starts from a single folder called `/` (root).

In this post, we will look at the most important folders in Linux, what they are used for, and simple examples so you can remember them easily.

---

## The Root of Everything: `/`

In Linux, **everything begins at `/`**. This is called the "root" folder. Every other folder — whether it's for your files, system settings, or programs — lives inside `/`.

Think of `/` like the trunk of a tree, and all other folders are branches growing out of it.

---

## Important Folders and What They Do

### `/home` — Your Personal Space
This is where normal users keep their personal files, like Desktop, Downloads, Documents, and Pictures.

**Example:** If your username is `ravi`, your personal folder is:
```
/home/ravi
```
This is similar to `C:\Users\ravi` in Windows.

---

### `/root` — The Admin's Home
This is the home folder, but only for the **root user** (the superadmin of the system). Normal users cannot access this folder.

**Example:** If you log in as root, your files will be saved in:
```
/root
```

---

### `/etc` — Settings and Configuration Files
Short for "et cetera," this folder holds configuration files for the system and installed applications.

**Example:** The file that stores network settings or the file listing all system users:
```
/etc/passwd
```

---

### `/var` — Data That Keeps Changing
"Var" stands for "variable." This folder stores data that changes often, like logs, caches, and mail files.

**Example:** System log files are usually found at:
```
/var/log/syslog
```

---

### `/tmp` — Temporary Files
Programs use this folder to store temporary files while they are running. Files here are usually deleted automatically when you restart your computer.

**Example:** A program downloading a file temporarily might save it as:
```
/tmp/update_123.tmp
```

---

### `/usr` — User Programs and Libraries
Despite the name, this is **not** for personal user files. It contains most of the installed programs, libraries, and documentation on the system.

**Example:** Many applications you install live inside:
```
/usr/bin
/usr/lib
```

---

### `/bin` — Basic Commands Everyone Needs
This folder has essential command-line tools that both normal users and admins need, like `ls`, `cp`, and `mv`.

**Example:**
```
/bin/ls
```
This is the actual program that runs when you type `ls` in the terminal.

---

### `/sbin` — Admin-Only Commands
Similar to `/bin`, but these commands are for **system administration** tasks, like shutting down or managing disks. Normal users usually don't need these.

**Example:**
```
/sbin/shutdown
```

---

### `/opt` — Optional Software
Some extra or third-party software (not part of the core system) gets installed here.

**Example:** A company's custom software might be installed at:
```
/opt/myapp
```

---

### `/dev` — Device Files
Linux treats hardware devices like files! This folder holds special files representing your hardware, like hard drives and USB devices.

**Example:** Your first hard disk might appear as:
```
/dev/sda
```

---

### `/proc` — Live Info About Running Processes
This is a special, virtual folder that doesn't store real files on disk. Instead, it gives live information about the system and currently running processes.

**Example:** To see info about your CPU, you can check:
```
/proc/cpuinfo
```

---

### `/sys` — Kernel and Device Information
Similar to `/proc`, this folder gives detailed information about the kernel, devices, and drivers.

**Example:** You might find power settings for your laptop battery in:
```
/sys/class/power_supply
```

---

## Quick Summary Table

| Folder  | Simple Meaning                          |
|---------|------------------------------------------|
| `/`     | The starting point of everything         |
| `/home` | Personal files for normal users          |
| `/root` | Personal files for the root user         |
| `/etc`  | Configuration and settings files         |
| `/var`  | Logs, caches, and changing data          |
| `/tmp`  | Temporary files, cleared often           |
| `/usr`  | Installed programs and libraries         |
| `/bin`  | Basic commands for everyone              |
| `/sbin` | Commands only for system admin tasks     |
| `/opt`  | Extra/optional software                  |
| `/dev`  | Files representing hardware devices      |
| `/proc` | Live info about running processes        |
| `/sys`  | Info about kernel and devices            |

---

## Final Thoughts

You don't need to memorize all of this in one day. As you use Linux more — installing programs, checking logs, or running commands — you will naturally start remembering where things live.

A simple way to think about it:
- **`/home`** → your stuff
- **`/etc`** → settings
- **`/var`** and **`/tmp`** → changing/temporary data
- **`/bin`, `/sbin`, `/usr`** → programs and commands
- **`/dev`, `/proc`, `/sys`** → hardware and system info

Once this clicks, navigating Linux becomes much easier!