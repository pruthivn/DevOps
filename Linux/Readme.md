# Linux

This directory contains Linux-related DevOps interview questions and resources.

## Linux Q&A from ADP

## 2. Linux File System
- `/etc` – Configuration files
- `/var` – Logs and variable data
- `/home` – User home directories
- `/tmp` – Temporary files
- `/usr` – User binaries
- `/opt` – Third-party applications

## 3. Linux Boot Process
BIOS/UEFI → GRUB → Kernel → Initramfs → Systemd → Services

## 17. Shell Scripting Automation
Examples:
- Log cleanup
- Health checks
- User management
- Backup automation

## 25. SQL & Shell Knowledge
SQL:
- SELECT
- JOIN
- GROUP BY
- ORDER BY

Shell:
- grep
- awk
- sed
- find

## Interview Questions form DigiCert

### 1. suppose there is a server called VM1. I want to ping VM1 using Salt. How will I?
A. Salt is a configuration management tool like Ansible.

pinging VM1 server using salt

```sh salt 'VM1' test.ping ```

**salt Advantages:**

Agent-Based AND Agentless Flexibility, Unmatched Execution Speed, Event-Driven Automation.

### Tell me about SAR command?
A. The sar command (System Activity Reporter) is a Linux/Unix utility used to collect, report, and save system activity information. It is part of the sysstat package.

```sh 
sar -r 
```

While tools like **top or htop** are great for seeing what is happening right now, sar is unique because it focuses on historical analysis and long-term trends

**Advantages:**

**Historical Data Review:** It runs as a background service (via cron) to log data every few minutes. If a server crashed at 2:00 AM, you can use sar to look back in time and see what caused the failure.

**All-in-One Metric Tracker:** It captures almost every hardware metric in a single tool.

It is incredibly lightweight.

easily export sar output into CSV, JSON, or XML formats.
