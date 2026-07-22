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

### 1. suppose there is a server called VM1. I want to ping VM1 using **Salt**. How will I?
A. Salt is a configuration management tool like Ansible.

pinging VM1 server using salt

```sh 
salt 'VM1' test.ping 
```

**salt Advantages:**

Agent-Based AND Agentless Flexibility, Unmatched Execution Speed, Event-Driven Automation.

### 2. Tell me about SAR command?
A. The sar command (System Activity Reporter) is a Linux/Unix utility used to collect, report, and save system activity information. It is part of the sysstat package.

**syntax:** sar [options] [interval] [count]

Below command displays the Memory and Swap utilization percentages for every second 3 times
```sh 
sar -r 1 3
```
**Imp:** if we want display usage for every 2 seconds for 5 times?

```sh 
sar 2 5
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

### 8. The root file system is unresponsive. How did you investigate that?
A. To investigate an unresponsive root file system, safely drop into a system rescue mode or emergency shell.
I first verify whether it's a disk I/O issue or if the filesystem mounted has become read-only.

I start with:

df -h to check disk space.
df -i to verify inode usage.
mount or cat /proc/mounts to check if the root filesystem is mounted as read-only.
dmesg and journalctl -xe to look for filesystem, disk, or I/O errors.
iostat -x 1 and vmstat 1 to check for high disk latency or I/O wait.
lsblk and smartctl (if available) to verify disk health.

If the filesystem is full, I identify large files using du -sh /* and clean up logs or unnecessary files. If there are filesystem corruption or hardware errors, I schedule an fsck during maintenance or replace the faulty disk if required."

### 9. Could you please explain the difference between fork(), exec(), and wait()?
A. "fork(), exec(), and wait() are Linux system calls used for process management.

fork() creates a new child process by copying the parent process.
exec() replaces the current process(means child process) with a new program. It does not create a new process; it reuses the existing process ID.
wait() makes the parent process wait until the child process finishes execution, preventing zombie processes.

A typical flow is: the parent calls fork(), the child calls exec() to run a new program, and the parent calls wait() until the child exits."

Ex: we have parent process 100 fork() creates child process 101 child running exact same code/program as parent because child is clone of parent exec() is called inside the child remove the original code and runs the exec() code(ls -l) wait() makes the parent process wait until the child process finishes execution, preventing zombie processes.

**Note:** it is never guranteed to pid 101 while creating child process any other  if another background service, cron job, or system thread forks a split second before or at the exact same time as your process, that other process will grab PID 101. Your child process would then get PID 102 or higher.
```text
[Parent Process: Shell]
 (PID: 5001)
     │
     ▼
  fork()  ───────────────────────► [Child Process: Clone of Shell]
     │                              (PID: 5002)
     │                                  │
     │ (waits for PID 5002)             ▼
  wait()                            exec("ls")
     │                                  │  (Wipes parent process memory/code,
     │                                  ▼   but retains PID 5002)
     │                             [Process: ls]
     │                             (PID: 5002)
     │                                  │
     ▼                                  ▼
Wakes up when PID 5002 exits ◄────── terminates
     │
     ▼
parent starts
 (PID: 5001)
```



### 10. what is zombie and orphan process?
A. **Zombie process** is a child process that has finished execution, but its entry still exists in the process table because the parent process has not yet collected its exit status using wait(). Zombie processes don't consume CPU or memory, but they occupy a process table entry.

**Orphan process** is a process whose parent has terminated before it. The orphan process is automatically adopted by the init or systemd process (PID 1), which eventually cleans it up when it exits/completes."

### 11. What is lsof command?
A. The lsof command in Linux stands for "List Open Files". it is used to display list of all files currently opened by running processes on the system.

The below command dispalys files opened/using by process id 1234
```sh
sudo lsof -p 1234
```

### 12. What is swap memory & Buffer memory?
A. **SWAP memory:** Swap memory is a disk space that Linux uses as an extension of RAM. When RAM memory becomes full, inactive memory pages are moved from RAM to swap space to free up memory for active processes. Since swap resides on disk, it is much slower than RAM, so excessive swap usage can impact system performance. I usually monitor swap usage using commands like **free -h, swapon -s, and vmstat.**

**Buffer Memory:** Buffer memory is RAM used by the Linux kernel to temporarily store data during input/output operations, such as reading from or writing to disks. It helps reduce the number of direct disk accesses, improving I/O performance. Unlike application memory, buffer memory is automatically reclaimed by the operating system whenever applications need more RAM.

### 12. Buffer v/s cache memory?
A. Buffer: Temporary storage for data in transit (moving between devices). It holds data that hasn't been read or written yet.

Cache: Temporary storage for frequently accessed data to make future reads faster. It holds copies of data you already used, guessing you might need it again soon.