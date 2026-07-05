
## 🧠 Core Resource Exhaustion (CPU, Memory, Disk)

### 251. A process is stuck in D state. How do you debug?
* **Core Action**: Check the kernel execution stack to identify where the thread is blocked.
* **Commands**: `cat /proc/<PID>/stack` or `dmesg | grep -i blocked`.
* **Root Cause**: Unresponsive network mounts (NFS) or failing local drive hardware.

### 258. CPU idle is high but load average is 30.
* **Core Action**: Measure system I/O wait times. Load average tracks blocked tasks, not just active CPU execution.
* **Commands**: `vmstat 1` or `top` (monitor the `%wa` metric).
* **Root Cause**: Severe disk/storage drive saturation or multiple threads stuck in a D-state.

### 259. System randomly freezes.
* **Core Action**: Analyze hardware failure logs and kernel records captured prior to the crash.
* **Commands**: `journalctl -k -b -1` or `ipmitool sel list`.
* **Root Cause**: Faulty RAM sectors, power spikes, thermal throttling, or kernel driver deadlocks.

### 262. How do you investigate kernel logs?
* **Core Action**: Query the active ring buffer and the systemd journaling facility filtered for kernel events.
* **Commands**: `dmesg -T` or `journalctl -k --since "1h ago"`.
* **Root Cause**: Used for root-cause analysis of driver errors, hardware faults, and OOM events.

### 265. Explain your Linux troubleshooting approach during a production outage.
* **Step 1 (Triage)**: Check high-level telemetry monitoring maps (Grafana/Datadog) to isolate the fault domain.
* **Step 2 (Inspect)**: Log into the affected node and check system vitals using `top`, `df -h`, and `free -m`.
* **Step 3 (Analyze)**: Scan local files in `/var/log/` and fetch direct container/application logs for explicit exceptions.
* **Step 4 (Mitigate)**: Apply immediate relief steps (restart services, scale container pools, clear disk, or roll back code).

---

## ⚡ Resource & Performance Bottlenecks

### 252. A server has high CPU but low application traffic.
* **Core Action**: Identify high-utilization background processes, rogue worker loops, or cron scripts.
* **Commands**: `top -b -n 1 -o %CPU` or `pidstat 1`.
* **Root Cause**: Cryptominers, bad system cron configurations, or unhandled infinite loops in background code.

### 253. A server is responding slowly every 5 minutes.
* **Core Action**: Correlate the systemic performance drops with automated schedulers.
* **Commands**: `cat /etc/crontab` or `systemctl list-timers`.
* **Root Cause**: Heavy infrastructure backup scripts or aggressive cron synchronization intervals running on a 5-minute block.

### 260. One core is 100% but others are idle.
* **Core Action**: Verify thread scaling capabilities and check active CPU affinity rules.
* **Commands**: `top` (then press `1` to expand cores) or `htop`.
* **Root Cause**: Single-threaded application runtimes (e.g., Node.js or Python scripts without multiprocessing wrappers).

### 261. How do you identify a bottleneck?
* **Core Action**: Monitor and correlate utilization, queue lengths, and processing delays across hardware boundaries concurrently.
* **Commands**: `dstat 1` or `glances`.
* **Root Cause**: A single exhausted hardware subsystem stalling the rest of the operational pipeline.

### 228. Disk is full. How do you troubleshoot?
* **Core Action**: Identify the largest files or directories and locate deleted files still held open by active processes.
* **Commands**: `df -h` (check mount space), `du -ahx / | sort -rh | head -10` (find largest directories), and `lsof +L1` (find unlinked but open deleted files).
* **Root Cause**: Runaway application logs, unrotated system logs, or ballooning temporary directories.

### 229. CPU is 100%. What will you check?
* **Core Action**: Sort running processes by active resource consumption and identify multi-threaded bottlenecks.
* **Commands**: `top -b -n 1 -o %CPU | head -20` (batch sort) or `htop`.
* **Root Cause**: Infinite loops in code, unoptimized database queries, or unauthorized mining scripts.

### 230. Memory is 100%. What will you check?
* **Core Action**: Sort active threads by physical memory resident size (`RES`) and check systemic buffer allocations.
* **Commands**: `top -b -n 1 -o %MEM | head -20` or `free -h` (check swap versus actual physical buffer usage).
* **Root Cause**: Java heap configuration misallocations, memory leaks, or heavy database caching layers.

### 231. Server is slow. How do you debug?
* **Core Action**: Quickly isolate which of the 4 major pillars (CPU, Memory, Disk I/O, Network) is exhausted.
* **Commands**: `vmstat 1` or `dstat 1` (simultaneous real-time resource overview).
* **Root Cause**: Cascading system degradation due to one specific resource hitting a physical threshold.

### 236. High Load Average.
* **Core Action**: Check the system queue lengths for tasks waiting on active CPU time or waiting on Disk/Network I/O.
* **Commands**: `uptime` (check 1, 5, 15 min trends) and `top` (compare CPU usage against `%wa` state).
* **Root Cause**: Runaway computing tasks or severe system storage latency blocking process execution.

### 237. OOMKilled investigation.
* **Core Action**: Pull the kernel ring buffer logs to locate the system event where a process was reaped due to extreme memory constraints.
* **Commands**: `dmesg -T | grep -i -E 'oom[-_]killer|killed'` or `journalctl -g "OOM"`.
* **Root Cause**: The application consumed more physical memory than available on the host machine without proper cgroup constraints.

### 238. Zombie processes.
* **Core Action**: Find defunct child processes that have completed execution but their parent hasn't read their exit status code.
* **Commands**: `ps aux | awk '$8=="Z"'` or find the parent PID via `ps -o ppid= -p <zombie_pid>`.
* **Root Cause**: Poorly written application software that does not execute `wait()` or `waitpid()` system calls.

### 239. D-state process.
* **Core Action**: Inspect the specific kernel function stack trace causing the uninterruptible loop.
* **Commands**: `cat /proc/<PID>/stack` or `ps axo pid,state,cmd | grep " D "`.
* **Root Cause**: Hardware waiting loops, un-pingable network storage shares (NFS), or hardware driver blockages.

---

## 💾 Filesystem & Disk Management

### 257. Disk usage is only 40% but writes fail.
* **Core Action**: Check filesystem metadata structural limits and current partition mount states.
* **Commands**: `df -i` (check for 100% inode usage) or `mount | grep "ro"`.
* **Root Cause**: Inode exhaustion caused by millions of zero-byte or tiny files, or a kernel panic forcing the disk into protective read-only mode.

### 263. How do you debug file system corruption?
* **Core Action**: Isolate the target disk partition from active mount points and execute volume repair utilities.
* **Commands**: `umount /dev/sdX` and run `fsck -y /dev/sdX`.
* **Root Cause**: Abrupt power loss events, physical drive degradation, or unclean system shutdowns.

### 240. "No space left on device" but disk has free space.
* **Core Action**: Check filesystem infrastructure indexes (inodes) and verify unlinked open descriptors.
* **Commands**: `df -i` (check for 100% inode saturation) and `lsof | grep deleted`.
* **Root Cause**: Complete structural metadata allocation exhaustion from too many zero-byte or tiny files.

### 241. High inode usage.
* **Core Action**: Locate the exact directories hosting excessive volumes of discrete individual files.
* **Commands**: `find / -xdev -type d -exec sh -c 'echo "$(find "$1" -type f | wc -l) $1"' _ {} \; | sort -nr | head -10`.
* **Root Cause**: Runaway PHP/application session caches, infinite temporary build files, or loose microservice logs.

### 242. Disk latency.
* **Core Action**: Analyze specific disk device busy states, operations per second, and wait/service queue parameters.
* **Commands**: `iostat -xz 1` (check `%util` and `await` columns) or `iotop -o`.
* **Root Cause**: Degraded storage arrays, physical bad disk sectors, or application write volume exceeding disk IOPS limits.

### 244. File descriptor exhaustion.
* **Core Action**: Count global open descriptors versus maximum software security constraints.
* **Commands**: `sysctl fs.file-nr` (global kernel open files) and `ulimit -n` (per-process file constraints).
* **Root Cause**: Leaked unclosed application network sockets, open database connections, or unclosed file tracking handles.

### 245. Permission denied.
* **Core Action**: Map standard POSIX file permissions alongside extended access vectors.
* **Commands**: `ls -la /path/to/target`, `getfacl /path/to/target`, and `sestatus` (verify active SELinux enforcement status).
* **Root Cause**: Incorrect target user/group ownership, missing directory execution flags (`+x`), or strict SELinux/AppArmor context blocks.

---

## 🐳 Containerization & Orchestration

### 255. Docker container is healthy but application is slow.
* **Core Action**: Inspect real-time container runtime limitations and active resource throttling policies.
* **Commands**: `docker stats` or `docker inspect <container_id>`.
* **Root Cause**: Configured CPU quota throttling (`cpu-shares` limits) or internal application database connection exhaustion.

### 256. Kubernetes pod gets OOMKilled repeatedly.
* **Core Action**: Inspect the lifecycle termination codes of the container.
* **Commands**: `kubectl describe pod <pod-name>` (look for Exit Code 137).
* **Root Cause**: The application runtime memory usage exceeded its configured manifest limits (`resources.limits.memory`).

---

## 🌐 Networking & Access Control

### 254. SSH works but sudo hangs.
sudo hangs means when we are running the sudo command terminal freezes indefinitely instead of asking passwords or showing outputs. when we use sudo system must validate who you are and what machine you are on to evaluate permissions against the policy file (/etc/sudoers). If any part of that verification pipeline is blocked, sudo stalls.

* **Core Action**: Validate local host identity mapping structures and check active network authentication configurations.
* **Commands**: `cat /etc/hosts` (check local hostname matching) or `strace sudo -i`.
* **Root Cause**: 
1. The local hostname is missing from `/etc/hosts`, causing local DNS resolution loops to time out.
2. if we are using the active directory or LDAP sudo has to contact an authentication server. If the network drops or the identity daemon hangs, the authentication check freezes.

### 264. How do you investigate network packet loss?
* **Core Action**: Map individual routing paths and trace network adapter error metrics.
* **Commands**: `mtr <target_ip>`(inspects at layer3) or `ip -s link`(inspects at layer 1 & 2). these commands ae better than traceroute to identify the issue.
* **Root Cause**: Broken structural cabling, congested firewalls, faulty physical switches, or network duplex mismatches.

### 232. Application is not accessible.
* **Core Action**: Verify if the target port is actively bound locally, and validate network reachability from outside.
* **Commands**: `ss -tulpn` (check listening ports) and `curl -I localhost:<port>`.
* **Root Cause**: Application service crashed, firewalls (`iptables`/`ufw`) dropping incoming traffic, or bad proxy configurations.

### 233. SSH is not working.
* **Core Action**: Validate service daemon operations and test local authentication bindings.
* **Commands**: `systemctl status sshd` or run a connection test with verbose debug logs: `ssh -vvv user@ip`.
* **Root Cause**: Firewalls blocking port 22, explicit permission blocks in `/etc/ssh/sshd_config`, or corrupted server keys.

### 234. Website returns 502.
* **Core Action**: Check communication lines between your reverse proxy (Nginx/Apache) and the backend upstream application.
* **Commands**: `tail -n 50 /var/log/nginx/error.log` (look for "connection refused" or "timeout").
* **Root Cause**: The backend application server (Node, PHP-FPM, Gunicorn) is offline or crashed.

### 235. Website returns 503.
* **Core Action**: Determine if the proxy/gateway is temporarily overloaded or undergoing maintenance.
* **Commands**: Inspect application load-balancer health metrics or backend connection pool queues.
* **Root Cause**: Downstream server capacity limits reached, or explicit service maintenance configurations activated.

### 243. Network latency.
* **Core Action**: Trace communication latency hops continuously while monitoring network card drop rates.
* **Commands**: `mtr <target_ip>` (continuous routing trace) and `ping -c 20 <target_ip>`.
* **Root Cause**: Congested underlying ISP routers, physical cabling failures, or network routing loops.

### 246. Service won't start.
* **Core Action**: Review daemon initialization sequences and specific config validation outputs.
* **Commands**: `systemctl status <service>`, `journalctl -u <service> -n 50 --no-pager`, or run the binary manually with a config check flag (e.g., `nginx -t`).
* **Root Cause**: Syntax typos in configuration files, target port conflict overlaps, or file permission problems.

### 247. Server rebooted unexpectedly.
* **Core Action**: Trace the final events logged to the system storage before system execution abruptly halted.
* **Commands**: `last reboot`, `journalctl -b -1 -n 100`, or check remote system hardware telemetry via `ipmitool sel list`.
* **Root Cause**: Kernel panic events, unrecoverable hardware power failures, or manual hypervisor reset events.

### 248. Kernel panic.
* **Core Action**: Extract kernel crash trace text dumps to identify the crashing driver module.
* **Commands**: Review system console crash logs in `/var/log/messages` or verify configurations for `kdump`/`crash` tool setups.
* **Root Cause**: Corrupted driver kernel modules, physical memory hardware breakdown, or severe kernel software bugs.

### 249. Memory leak investigation.
* **Core Action**: Track active allocations over extended operational windows to identify consistent upward usage lines.
* **Commands**: `pidstat -r 5` (track process RSS increases over time) or instrument with profile tracing tools (`valgrind`, `gprof`).
* **Root Cause**: Application language allocation frameworks failing to clean up object heap trees.

### 250. Bottleneck identification.
* **Core Action**: Execute holistic subsystem evaluation layers to find the limiting performance component under synthetic load.
* **Commands**: Run aggregate systems screens like `glances` or systematically isolate targets via benchmarking tools (`sysbench`, `iperf`).
* **Root Cause**: System output bounds hitting physical infrastructure ceilings on a specific component line.