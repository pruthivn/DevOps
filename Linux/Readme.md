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

```sh 
salt 'VM1' test.ping 
```

**salt Advantages:**

Agent-Based AND Agentless Flexibility, Unmatched Execution Speed, Event-Driven Automation.

### 2. Tell me about SAR command?
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


### 3. There is a Linux server that just rebooted unexpectedly. So how do you determine whether, like, what the problem was?
A. I first check the previous boot logs using journalctl -b -1 to identify what happened before the reboot. Then I check system logs and kernel logs using journalctl, dmesg, and /var/log/messages or /var/log/syslog. I also look for kernel panics, OOM events, hardware errors, or power failures. Finally, I verify system uptime with uptime or who -b and correlate the reboot time with monitoring alerts or recent changes to determine the root cause.

### 4. how would you troubleshoot a DNS issue?
A. I check DNS resolution using nslookup or dig, verify the configured DNS servers in /etc/resolv.conf, and test connectivity using ping or curl with both the hostname and IP address. If the IP works but the hostname doesn't, it's likely a DNS issue. In Kubernetes, I also check the CoreDNS pods and logs using kubectl get pods -n kube-system and kubectl logs to identify any DNS-related failures.

### 4. when we run free -n command, so the output is used, free, then buffer and cache, it shows all those things. So, if we have 500 MB, suppose used is 60 GB, and free is 50, 500 MB, buffer and cache is 50 GB. So is the memory exhausted?
A. No. The memory is not exhausted. In Linux, buffer and cache memory is reclaimable. If an application needs more RAM, the kernel automatically frees the buffer and cache memory and allocates it to the application. So even if the free column shows only 500 MB, having 50 GB in buffer/cache means there is still plenty of usable memory. I would check the available memory in the free -h output, as it gives the actual estimate of memory available for new applications.

### 5. Explain logrotate command?
A. logrotate is a built-in Linux system utility designed to automatically archive, compress, remove, and mail log files.

**/etc/logrotate.conf:** The main configuration file containing global system-wide defaults (e.g., rotating logs weekly)

**/etc/logrotate.d/:** A drop-in directory where specific applications (like Apache, Nginx, or MySQL) store their own custom rotation rules. if we have to rotate logs for frontend application we create file with frontendapp(no extensions needed like .conf) in that file we put below data

```sh
/path/to/frontendapplogfile.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
```
**Note:** copytruncate is the most important setting here. It ensures the application doesn't need to be restarted when the log rolls over.

### 6. we have a log file that has already consumed 200 GB of data. So, what will be your approach?
1. Never run cat, vim, or nano on a 200 GB file. It will exhaust the server's memory (RAM), freeze the CPU, and potentially crash the entire system.

2. Do not delete the file(rm filename.log). If an active application is still writing to that log, deleting the file will hide it from view, but the application will keep the disk space locked in the background, leaving your drive 100% full

3. Instead of deleting truncate it. This instantly drains the file size to 0 bytes without breaking the application's write stream.
```sh
sudo truncate -s 0 /path/to/large_file.log
```
4. Read the Last Lines of log gile to identify the Root Cause and rotate the logs using logrotate(it split daily, compress old logs into small files, and automatically delete them after a certain period).

### 7. how would you perform system monitoring in Linux systems?
A. For Linux system monitoring, I continuously monitor CPU, memory, disk, network, and running processes using standard Linux utilities. I commonly use:
```sh
top or htop → CPU, memory, and processes
sar → Historical performance statistics
vmstat → CPU, memory, and swap
iostat → Disk I/O performance
free -h → Memory and swap usage
df -h and df -i → Disk space and inode usage
iotop → Disk I/O by process
netstat or ss → Network connections
ps -ef → Running processes
journalctl and dmesg → System and kernel logs
```
In production, I also use monitoring tools like Prometheus, Grafana, and CloudWatch to monitor servers, configure alerts, and analyze performance trends