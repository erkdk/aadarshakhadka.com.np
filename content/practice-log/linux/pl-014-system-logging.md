---
title: "PL - 014 — System Logging & Log Analysis"
date: 2026-08-16
draft: false
---
### The Three Pillars of Observability

In any production system, **metrics, traces, and logs** work together to detect, locate, and understand problems.

> **Metrics tell you that something is wrong.**
> **Traces tell you where it went wrong.**
> **Logs tell you why it went wrong.**

#### 1. Metrics — Detect the Problem
Metrics are numerical measurements of system health and performance.

**Examples:**

* Error rate: `8%`
* API latency: `2.5s`
* CPU usage: `90%`

**Use cases:** Monitoring system health, Setting alerts, Detecting performance issues

#### 2. Traces — Locate the Problem
Traces follow a request across services and show where time or errors occur.

**Example:**
```text
              API --> Order Service ---> Payment Service --> Database
                                     |
                                2.5s delay
```
**Use cases:** Finding bottlenecks, Debugging microservices, Tracking request flow

#### 3. Logs — Understand the Problem
Logs provide detailed context about what happened inside a service.

**Payment Service Example:**
```text
                ERROR PaymentService
                Payment API timeout — Order ID: 12345
```
**Use cases:** Finding root causes, Debugging errors, Investigating incidents

#### Real-World Example

Imagine an **e-commerce checkout** suddenly becomes slow:

1. **Metric:** API latency increases from `300ms` to `3s` --> **something is wrong.**
2. **Trace:** Shows the `Payment Service` is taking `2.7s` --> **where it went wrong.**
3. **Log:** Shows `Payment API connection timeout` --> **why it went wrong.**

**Linux Server Example:**

Imagine a Linux web server suddenly becomes slow:

**1. Metrics → Detect**
- CPU usage: `30% → 95%`
- Memory usage: `60% → 90%`
- **Something is wrong.**

**2. System/Process Tools → Locate**
- `top` shows the Java process using `90% CPU`
-  **The Java process is causing the problem.**

**3. Logs → Explain**
- Application logs show `Database connection timeout`
-  **The application is repeatedly retrying database connections.**

### System Logging & Log Analysis
 
Logs are operational telemetry in production — not optional. They enable troubleshooting, security forensics, performance monitoring, and compliance auditing (PCI-DSS, SOC2, HIPAA). Treat logs as sensitive data; they often contain tokens, PII, and stack traces.

---

### Linux Logging Architecture Overview

Modern Linux distributions run **two parallel logging systems** that coexist:

```text
                              			USER SPACE

                     +-------------------+        +----------------------+
                     |   Applications    |        |  System Services     |
                     |                   |        |    / Daemons         |
                     |   syslog()        |        |  stdout / stderr     |
                     +---------+---------+        +----------+-----------+
                               |                             |
                               +-------------+---------------+
                                             |
                                             v
                                   +----------------------+
                                   |  systemd-journald    |
                                   |                      |
                                   |  Central log service |
                                   +----------+-----------+
                                              |
                                   +----------+----------+
                                   |                     |
                                   v                     v
                          +------------------+     +----------------+
                          | Journal Files    |     |    rsyslog     |
                          |                  |     |    (optional)  |
                          | /run/log/journal |     +-------+--------+
                          |         or       |             |
                          | /var/log/journal |             v
                          |                  |      +--------------+
                          +------------------+      | /var/log/*   |
                                                    | /Remote/SIEM |
                                                    +--------------+
 ------------------------------------------------------------------------------------------		
                        		       KERNEL SPACE 

                                +----------------------+
                                |    Linux Kernel      |
                                |                      |
                                |     printk()         |
                                +----------+-----------+
                                           |
                                           v
                                +----------------------+
                                | Kernel Ring Buffer   |
                                +----------+-----------+
                                           |
                                           v
                                      /dev/kmsg
                                           |
                                           |
                                           +--------------------> systemd-journald
```
`journald` and `rsyslog` are **co-equal collectors**, not strictly primary/secondary. Applications can send logs to either path depending on configuration.

---

#### Kernel Space Logging

The kernel logs internal events (hardware detection, driver initialization, OOM kills, panics) using the `printk()` function. Messages land in a **fixed-size circular kernel ring buffer** (typically 512 KB – 4 MB, set at compile time). Because it is circular, early boot messages can be overwritten before userspace logging starts.

---

#### User Space Logging

Applications and services generate logs through three pathways:

| Pathway | Mechanism | Example |
|---------|-----------|---------|
| **Syslog API** | Standard library functions (`syslog()`, `vsyslog()`) send to `/dev/log` | Traditional C daemons |
| **stdout/stderr** | Modern systemd services write to stdout/stderr; journald intercepts via `StandardOutput=journal` | Most systemd services |
| **Direct file I/O** | Heavy applications bypass system daemons and manage their own log files | Nginx (`/var/log/nginx/`), PostgreSQL (`/var/log/postgresql/`) |

---

#### Main Logging Daemons

#### i. systemd-journald

| Attribute | Detail |
|-----------|--------|
| **Role** | First-stage collector for almost all logs on modern Linux distros (RHEL 7+, Ubuntu 15+, Debian 8+) |
| **Storage format** | Structured, indexed **binary format** with rich metadata (PID, UID, GID, unit name, SELinux context, systemd cgroup) |
| **Query tool** | `journalctl` exclusively |
| **Volatile storage** | `/run/log/journal/` — cleared on reboot |
| **Persistent storage** | `/var/log/journal/` — survives reboot |

Journald uses the `Storage =` directive in `/etc/systemd/journald.conf`:

| Setting | Behavior |
|---------|----------|
| `auto` *(default)* | Persistent **only if** `/var/log/journal/` already exists; otherwise volatile |
| `persistent` | Creates `/var/log/journal/` and stores persistently |
| `volatile` | Forces `/run/log/journal/` only |
| `none` | No local storage; forwarding only |

Never assume persistence. On fresh cloud images, `/var/log/journal/` often does not exist, and logs are lost on reboot.

#### ii. rsyslog

| Attribute | Detail |
|-----------|--------|
| **Role** | Highly customizable log processor, router, and formatter |
| **Storage format** | Traditional plain-text files in `/var/log/` |
| **Capabilities** | Complex rulesets, regex filtering, templating, TLS encryption, disk-assisted queues, remote forwarding |

#### Two Ways to Read Journald Logs

| Module | Method | Performance | Metadata |
|--------|--------|-------------|----------|
| `imuxsock` | Reads from `/run/systemd/journal/syslog` socket | Fast, low overhead | Plain text only; loses structured fields |
| `imjournal` | Reads journal files via journal API | Slower, higher CPU | Preserves all journal metadata |

Use `imuxsock` if you only need text logs in `/var/log/`. Use `imjournal` if you need to preserve journal metadata for SIEM correlation.

---

### How journald and rsyslog Interact

The relationship is **conditional**, not automatic:

1. **Journald captures everything** (kernel + userspace).
2. **If** `ForwardToSyslog=yes` is set in `/etc/systemd/journald.conf`, journald writes a copy to `/run/systemd/journal/syslog`.
3. **Rsyslog** (with `imuxsock` or `imjournal`) reads from that socket or the journal files.
4. **Rsyslog** then writes to `/var/log/` files and/or forwards remotely.

Rsyslog typically receives kernel logs via its own `imkmsg` or `imklog` module (reading `/dev/kmsg` or `/proc/kmsg`), **not** directly from journald unless explicitly bridged.

---

### Syslog Standards: Facilities & Severities

#### RFC 5424 Facilities (Who sent it)

| Code | Facility | Code | Facility |
|------|----------|------|----------|
| 0 | `kern` (kernel) | 12 | `ntp` |
| 1 | `user` (user-level) | 13 | `logaudit` |
| 2 | `mail` | 14 | `logalert` |
| 3 | `daemon` (system daemons) | 15 | `clock` |
| 4 | `auth` / `security` | 16 | `local0` |
| 5 | `syslog` (internal) | 17 | `local1` |
| 6 | `lpr` (line printer) | 18 | `local2` |
| 7 | `news` | 19 | `local3` |
| 8 | `uucp` | 20 | `local4` |
| 9 | `cron` | 21 | `local5` |
| 10 | `authpriv` (private auth) | 22 | `local6` |
| 11 | `ftp` | 23 | `local7` |

#### RFC 5424 Severity Levels (How urgent)

| Level | Severity | Description | Production Response |
|-------|----------|-------------|---------------------|
| 0 | `emerg` | System is unusable | Immediate pager; all-hands incident |
| 1 | `alert` | Action must be taken immediately | Page on-call immediately |
| 2 | `crit` | Critical conditions (hardware failure) | High-priority ticket |
| 3 | `err` | Error conditions | Standard alerting threshold |
| 4 | `warning` | Warning conditions | Monitor trend; alert if sustained |
| 5 | `notice` | Normal but significant | Info dashboards |
| 6 | `info` | Informational messages | Logged; rarely alerted |
| 7 | `debug` | Debug-level messages | Usually disabled in production |

---

### Log File System Layout (`/var/log/`)

Text log locations differ by distribution family:

| Purpose | Debian / Ubuntu | RHEL / Rocky / Fedora |
|---------|----------------|----------------------|
| Global system activity | `/var/log/syslog` | `/var/log/messages` |
| Authentication & security | `/var/log/auth.log` | `/var/log/secure` |
| Kernel events | `/var/log/kern.log` | Intercepted via `dmesg` / `messages` |
| Scheduled tasks | `/var/log/cron.log` | `/var/log/cron` |
| Boot records | `/var/log/boot.log` | `/var/log/boot.log` |
| Mail server | `/var/log/mail.log` | `/var/log/maillog` |
| Package management | `/var/log/dpkg.log` | `/var/log/dnf.rpm.log` |

`/var/log/journal/` contains **binary journald files**, not text. Do not manually edit or `logrotate` them.

---

### Log Maintenance & Rotation

#### logrotate

Prevents disk exhaustion by compressing, renaming, and purging old text logs.

On modern systemd distros, `logrotate` runs via **`logrotate.timer`** (systemd timer), not cron. On older systems or minimal containers, it may still run from `/etc/cron.daily/`.

| Directive | Meaning |
|-----------|---------|
| `daily` / `weekly` / `monthly` | Rotation frequency |
| `rotate N` | Keep N rotated archives |
| `compress` | Gzip old logs (adds `.gz`) |
| `delaycompress` | Compress the archive from the previous rotation, not the current one |
| `missingok` | Do not error if the log file is missing |
| `notifempty` | Do not rotate empty files |
| `create MODE USER GROUP` | Create new empty log with specified permissions |
| `sharedscripts` | Run postrotate script once for all matching files |

#### journald Native Rotation

Journald **does not use logrotate**. It manages its own files via `/etc/systemd/journald.conf`:

| Directive | Purpose |
|-----------|---------|
| `SystemMaxUse=` | Total max disk space for persistent logs (e.g., `500M`) |
| `SystemMaxFiles=` | Maximum number of journal files to keep |
| `MaxRetentionSec=` | Max age of entries (e.g., `1week`, `1month`) |
| `MaxFileSec=` | Max time before starting a new journal file |
| `Compress=` | Compress stored data (`yes` / `no`) |

---

### Centralized Logging & Remote Forwarding

#### Architecture Patterns

| Pattern | Mechanism | Pros | Cons |
|---------|-----------|------|------|
| **Push (syslog)** | Rsyslog forwards to central server | Simple, universal | Firewall rules needed outbound; no delivery guarantee by default |
| **Pull (agent)** | Filebeat / Fluent Bit / Vector read local files and ship | Buffers locally; backpressure handling | Requires agent deployment |
| **Hybrid** | Journald/rsyslog → local file → agent → central | Best of both worlds | More moving parts |

#### Secure Remote Forwarding with Rsyslog

**Never send sensitive logs over plaintext UDP/514 in production.** Use TLS on port **6514** (RFC 5425).

Key rsyslog forwarding parameters for reliability:

| Parameter | Why It Matters |
|-----------|----------------|
| `queue.type="linkedList"` | In-memory queue that spills to disk if full |
| `queue.saveonshutdown="on"` | Persists unsent messages across rsyslog restarts |
| `action.resumeRetryCount="-1"` | Retry forever; never drop messages |
| `StreamDriverAuthMode="x509/name"` | Mutual TLS — verifies server identity |

Always use disk-assisted queues when forwarding logs remotely. Network blips are guaranteed; message loss is optional.

---

### Terminal Session
```
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Fri Aug 14 05:07:58 2026
 
[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ hostname
labserver

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Thu Aug 13 19:20:52 +0545 2026 on tty1

 # location

 # location --> traditional log files

[root@labserver ~]# cd /var/log

[root@labserver log]# ls
anaconda       cron-20260807        hawkey.log-20260807  messages-20260730  spooler
audit          dnf.librepo.log      httpd                messages-20260807  spooler-20260730
btmp           dnf.log              lastlog              private            spooler-20260807
btmp-20260803  dnf.rpm.log          maillog              samba              sssd
chrony         firewalld            maillog-20260730     secure             tuned
cron           hawkey.log           maillog-20260807     secure-20260730    useractivity.log
cron-20260730  hawkey.log-20260730  messages             secure-20260807    wtmp
[root@labserver log]#
```

 ### Traditional Important files logs
---
```

/var/log/messages --> It contains general system logs, including DNS, DHCP, NFS, NTP, ... logs if configured to log there

/var/log/boot.log --> It contains boot-time logs

/var/log/dmesg    --> Historically contains kernel/HW-related logs; on modern systems use `dmesg` or `journalctl -k`

/var/log/cron     --> It contains cron service logs

/var/log/dnf.log  --> It contains DNF package-management logs

/var/log/secure   --> It contains authentication/security-related logs, including local login, remote login (SSH, Telnet, ...), sudo-related logs

/var/log/maillog  --> It contains mail service-related logs

/var/log/httpd/*  --> It contains Apache HTTP Server-related logs

/var/log/samba/   --> It contains Samba-related logs
```
---
```
 # Typical method to view log files

[root@labserver log]# pwd
/var/log
[root@labserver log]# cat /var/log/secure
Aug  7 05:13:01 lab su[2317]: pam_unix(su-l:session): session opened for user root(uid=0) by aadarsha(uid=1000)
...

[root@labserver log]# tail /var/log/secure
Aug 13 19:17:29 labserver sshd-session[2314]: pam_unix(sshd:session): session closed for user aadarsha
...

[root@labserver log]# tail -20 /var/log/secure
Aug 13 18:51:05 labserver sshd[942]: Server listening on :: port 22.
...


 # Format of Log files
 
   #   <Date/Time>     <Hostname>     <Process>[PID]:   <Log Message>

[root@labserver log]# tail /var/log/secure | grep failed
Aug 14 05:39:26 labserver unix_chkpwd[2641]: password check failed for user (aadarsha)

[root@labserver log]# cat /var/log/secure | grep failed
Aug  7 11:01:07 lab unix_chkpwd[1982]: password check failed for user (aadarsha)
Aug  7 11:02:09 lab unix_chkpwd[2242]: password check failed for user (root)
Aug 10 18:34:58 labserver unix_chkpwd[2165]: password check failed for user (aadarsha)
Aug 10 18:35:04 labserver unix_chkpwd[2167]: password check failed for user (aadarsha)
Aug 11 06:04:07 labserver unix_chkpwd[2822]: password check failed for user (root)
Aug 11 09:37:19 labserver unix_chkpwd[4289]: password check failed for user (aadarsha)
Aug 11 09:41:45 labserver unix_chkpwd[4360]: password check failed for user (root)
Aug 12 12:49:33 labserver unix_chkpwd[2243]: password check failed for user (aadarsha)
Aug 12 20:01:57 labserver unix_chkpwd[2235]: password check failed for user (aadarsha)
Aug 12 20:50:42 labserver unix_chkpwd[2731]: password check failed for user (root)
Aug 13 11:22:30 labserver unix_chkpwd[2354]: password check failed for user (root)
Aug 13 18:29:02 labserver unix_chkpwd[2360]: password check failed for user (aadarsha)
Aug 13 18:29:21 labserver unix_chkpwd[2402]: password check failed for user (root)
Aug 13 18:29:27 labserver unix_chkpwd[2407]: password check failed for user (root)
Aug 13 18:29:34 labserver unix_chkpwd[2413]: password check failed for user (root)
Aug 14 05:39:26 labserver unix_chkpwd[2641]: password check failed for user (aadarsha)

[root@labserver log]# cat /var/log/secure | grep -i failed
Aug  7 11:01:07 lab unix_chkpwd[1982]: password check failed for user (aadarsha)
Aug  7 11:01:09 lab login[935]: FAILED LOGIN 1 FROM tty1 FOR aadarsha, Authentication failure
Aug  7 11:01:45 lab login[935]: FAILED LOGIN 2 FROM tty1 FOR #033q, Authentication failure
Aug  7 11:02:09 lab unix_chkpwd[2242]: password check failed for user (root)
Aug 10 18:34:58 labserver unix_chkpwd[2165]: password check failed for user (aadarsha)
Aug 10 18:34:59 labserver login[940]: FAILED LOGIN 1 FROM tty1 FOR aadarsha, Authentication failure
Aug 10 18:35:04 labserver unix_chkpwd[2167]: password check failed for user (aadarsha)
Aug 10 18:35:06 labserver login[940]: FAILED LOGIN 2 FROM tty1 FOR aadarsha, Authentication failure
Aug 11 06:04:07 labserver unix_chkpwd[2822]: password check failed for user (root)
Aug 11 09:37:19 labserver unix_chkpwd[4289]: password check failed for user (aadarsha)
Aug 11 09:37:22 labserver sshd-session[4285]: Failed password for aadarsha from 192.168.254.152 port 46674 ssh2
Aug 11 09:41:45 labserver unix_chkpwd[4360]: password check failed for user (root)
Aug 12 12:49:33 labserver unix_chkpwd[2243]: password check failed for user (aadarsha)
Aug 12 12:49:36 labserver login[1066]: FAILED LOGIN 1 FROM tty1 FOR aadarsha, Authentication failure
Aug 12 20:01:57 labserver unix_chkpwd[2235]: password check failed for user (aadarsha)
Aug 12 20:01:59 labserver login[975]: FAILED LOGIN 1 FROM tty1 FOR aadarsha, Authentication failure
Aug 12 20:50:42 labserver unix_chkpwd[2731]: password check failed for user (root)
Aug 13 11:22:30 labserver unix_chkpwd[2354]: password check failed for user (root)
Aug 13 18:29:02 labserver unix_chkpwd[2360]: password check failed for user (aadarsha)
Aug 13 18:29:04 labserver sshd-session[2358]: Failed password for aadarsha from 192.168.254.152 port 39730 ssh2
Aug 13 18:29:21 labserver unix_chkpwd[2402]: password check failed for user (root)
Aug 13 18:29:27 labserver unix_chkpwd[2407]: password check failed for user (root)
Aug 13 18:29:34 labserver unix_chkpwd[2413]: password check failed for user (root)
Aug 14 05:39:26 labserver unix_chkpwd[2641]: password check failed for user (aadarsha)
Aug 14 05:39:27 labserver sshd-session[2638]: Failed password for aadarsha from 192.168.254.3 port 53476 ssh2

[root@labserver log]# cat /var/log/secure | grep -i failed | grep root
Aug  7 11:02:09 lab unix_chkpwd[2242]: password check failed for user (root)
Aug 11 06:04:07 labserver unix_chkpwd[2822]: password check failed for user (root)
Aug 11 09:41:45 labserver unix_chkpwd[4360]: password check failed for user (root)
Aug 12 20:50:42 labserver unix_chkpwd[2731]: password check failed for user (root)
Aug 13 11:22:30 labserver unix_chkpwd[2354]: password check failed for user (root)
Aug 13 18:29:21 labserver unix_chkpwd[2402]: password check failed for user (root)
Aug 13 18:29:27 labserver unix_chkpwd[2407]: password check failed for user (root)
Aug 13 18:29:34 labserver unix_chkpwd[2413]: password check failed for user (root)
 
[root@labserver log]# cat /var/log/secure | grep -i failed | grep root | wc -l
8

[root@labserver log]# cat /var/log/secure | grep -i failed | grep "192.168.254.3"
Aug 14 05:39:27 labserver sshd-session[2638]: Failed password for aadarsha from 192.168.254.3 port 53476 ssh2
Aug 14 05:41:45 labserver sshd-session[2705]: Failed password for aadarsha from 192.168.254.3 port 42494 ssh2
[root@labserver log]# 

[root@labserver log]# cat /var/log/secure | grep aadrkdk | grep -i Accepted
[root@labserver log]# cat /var/log/secure | grep aadarsha | grep -i Accepted
Aug  7 11:05:07 labserver sshd-session[2020]: Accepted password for aadarsha from 192.168.254.152 port 45930 ssh2
Aug  7 12:20:43 labserver sshd-session[2790]: Accepted password for aadarsha from 192.168.254.152 port 54754 ssh2
Aug  9 15:53:00 labserver sshd-session[2016]: Accepted password for aadarsha from 192.168.254.152 port 42354 ssh2
Aug 10 18:35:37 labserver sshd-session[2222]: Accepted password for aadarsha from 192.168.254.152 port 51030 ssh2
...

[root@labserver log]# ls 
anaconda       cron-20260807        hawkey.log-20260807  messages-20260730  spooler
audit          dnf.librepo.log      httpd                messages-20260807  spooler-20260730
btmp           dnf.log              lastlog              private            spooler-20260807
btmp-20260803  dnf.rpm.log          maillog              samba              sssd
chrony         firewalld            maillog-20260730     secure             tuned
cron           hawkey.log           maillog-20260807     secure-20260730    useractivity.log
cron-20260730  hawkey.log-20260730  messages             secure-20260807    wtmp
[root@labserver log]# 

 # By default logs are rotated after certain period. eg: secure-20260807 , secure-20260730 , secure - current

[root@labserver log]# cat /var/log/secure-20260
secure-20260730  secure-20260807  
[root@labserver log]# cat /var/log/secure-20260730 | grep -i Accepted 
Jul 13 14:49:17 localhost sshd-session[1888]: Accepted password for aadarsha from 192.168.254.32 port 53332 ssh2
Jul 18 09:43:42 localhost sshd-session[1933]: Accepted password for aadarsha from 192.168.1.98 port 32920 ssh2
Jul 18 09:53:52 localhost sshd-session[2200]: Accepted password for aadarsha from 192.168.1.98 port 36328 ssh2
Jul 21 11:38:27 labserver sshd-session[1895]: Accepted password for aadarsha from 192.168.254.32 port 57028 ssh2
Jul 21 12:28:54 labserver sshd-session[1890]: Accepted password for aadarsha from 192.168.254.32 port 43434 ssh2
Jul 21 14:13:06 labserver sshd-session[1975]: Accepted password for aadarsha from 192.168.254.32 port 55814 ssh2
Jul 22 05:13:03 labserver sshd-session[1921]: Accepted password for aadarsha from 192.168.254.32 port 57186 ssh2
...

[root@labserver log]# cat /var/log/secure-20260730 | grep -i Failed | egrep 'aadarsha|aadarkdk|root'
Jul 13 14:46:02 localhost unix_chkpwd[5775]: password check failed for user (aadarsha)
Jul 13 14:46:03 localhost login[951]: FAILED LOGIN 1 FROM tty1 FOR aadarsha, Authentication failure
Jul 21 11:29:45 labserver unix_chkpwd[1879]: password check failed for user (aadarsha)
Jul 21 12:28:47 labserver unix_chkpwd[1897]: password check failed for user (aadarsha)
Jul 21 12:28:49 labserver sshd-session[1890]: Failed password for aadarsha from 192.168.254.32 port 43434 ssh2
Jul 21 14:13:15 labserver unix_chkpwd[2012]: password check failed for user (root)
Jul 22 11:46:21 labserver unix_chkpwd[1945]: password check failed for user (root)
Jul 22 11:55:55 labserver unix_chkpwd[1979]: password check failed for user (root)
Jul 22 12:08:14 labserver unix_chkpwd[1897]: password check failed for user (aadarsha)
Jul 22 12:08:16 labserver login[974]: FAILED LOGIN 1 FROM tty1 FOR aadarsha, Authentication failure
Jul 22 13:08:04 labserver unix_chkpwd[1956]: password check failed for user (aadarsha)
Jul 22 13:08:06 labserver sshd-session[1949]: Failed password for aadarsha from 192.168.254.32 port 59394 ssh2
Jul 22 13:08:19 labserver unix_chkpwd[1957]: password check failed for user (aadarsha)
Jul 22 13:08:21 labserver sshd-session[1949]: Failed password for aadarsha from 192.168.254.32 port 59394 ssh2
Jul 22 13:08:39 labserver unix_chkpwd[1997]: password check failed for user (root)
Jul 22 16:25:54 labserver unix_chkpwd[6494]: password check failed for user (aadarsha)
Jul 22 16:25:56 labserver login[989]: FAILED LOGIN 1 FROM tty1 FOR aadarsha, Authentication failure
Jul 24 19:50:03 labserver unix_chkpwd[5168]: password check failed for user (aadarsha)
Jul 26 08:06:01 labserver unix_chkpwd[1949]: password check failed for user (aadarsha)
Jul 26 08:06:03 labserver sshd-session[1945]: Failed password for aadarsha from 192.168.254.32 port 43116 ssh2
Jul 29 05:20:08 labserver unix_chkpwd[2007]: password check failed for user (root)
Jul 29 10:12:46 labserver unix_chkpwd[1990]: password check failed for user (root)
Jul 29 20:00:31 labserver unix_chkpwd[2960]: password check failed for user (root)
[root@labserver log]#

[root@labserver ~]# cat /etc/issue.net 
\S
Kernel \r on \m
[root@labserver ~]# 

[root@labserver ~]# vi /etc/ssh/sshd_config

[root@labserver ~]# cat /etc/ssh/sshd_config
...
# no default banner path
#Banner none
Banner /etc/issue.net
...
[root@labserver ~]# 

[root@labserver ~]# systemctl restart sshd

[root@labserver ~]# vi /etc/issue.net 

[root@labserver ~]# cat /etc/issue.net 
\S
Kernel \r on \m

UN-AUTHORIZED LOGIN IS PROHIBITED!!!
[root@labserver ~]# 

[root@labserver ~]# exit
logout

[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.

aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
\S
Kernel \r on \m

UN-AUTHORIZED LOGIN IS PROHIBITED!!!
aadarsha@192.168.254.2's password: 
Last login: Sun Aug 16 18:28:12 2026 from 192.168.254.152
[aadarsha@labserver ~]$ su - root
Password: 
[root@labserver ~]# 

 # Log Rotation
 
[root@labserver ~]# cat /etc/logrotate.
logrotate.conf  logrotate.d/    
[root@labserver ~]# cat /etc/logrotate.conf 
# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may also be configured here.
[root@labserver ~]# 

[root@labserver ~]# man logrotate

[root@labserver ~]# cd /etc/logrotate.d

[root@labserver logrotate.d]# ls
btmp  chrony  dnf  firewalld  httpd  kvm_stat  rsyslog  samba  sssd  wtmp

[root@labserver logrotate.d]# cat samba 
/var/log/samba/*log* {
    compress
    dateext
    maxage 365
    rotate 99
    notifempty
    olddir /var/log/samba/old
    missingok
    copytruncate
}
[root@labserver logrotate.d]# cat httpd 
# Note that logs are not compressed unless "compress" is configured,
# which can be done either here or globally in /etc/logrotate.conf.
/var/log/httpd/*log {
    missingok
    notifempty
    sharedscripts
    delaycompress
    postrotate
        /bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
    endscript
}
[root@labserver logrotate.d]# 

[root@labserver ~]# cd /var/log/

[root@labserver log]# pwd
/var/log

[root@labserver log]# ls
anaconda       cron-20260807        hawkey.log-20260807  messages-20260730  spooler
audit          dnf.librepo.log      httpd                messages-20260807  spooler-20260730
btmp           dnf.log              lastlog              private            spooler-20260807
btmp-20260803  dnf.rpm.log          maillog              samba              sssd
chrony         firewalld            maillog-20260730     secure             tuned
cron           hawkey.log           maillog-20260807     secure-20260730    useractivity.log
cron-20260730  hawkey.log-20260730  messages             secure-20260807    wtmp
[root@labserver log]# 

[root@labserver log]# cat /etc/logrotate.conf 
# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may also be configured here.
[root@labserver log]# 

[root@labserver log]# vim /etc/logrotate.conf 

[root@labserver log]# cat /etc/logrotate.conf 
# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
#weekly
daily

# keep 4 weeks worth of backlogs
#rotate 4
rotate 90

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may also be configured here.
[root@labserver log]# 

 # logrotate 
 	--> The logrotate utility automates the rotation, compression, removal, and mailing of system log files to save disk space and maintain clean log directories.

[root@labserver log]# logrotate /etc/logrotate.conf

[root@labserver log]# ls
anaconda          dnf.rpm.log             maillog-20260816.gz   spooler
audit             firewalld               messages              spooler-20260730
btmp              hawkey.log              messages-20260730     spooler-20260807
btmp-20260803     hawkey.log-20260730     messages-20260807     spooler-20260816.gz
chrony            hawkey.log-20260807     messages-20260816.gz  sssd
cron              hawkey.log-20260816.gz  private               tuned
cron-20260730     httpd                   samba                 useractivity.log
cron-20260807     lastlog                 secure                wtmp
cron-20260816.gz  maillog                 secure-20260730
dnf.librepo.log   maillog-20260730        secure-20260807
dnf.log           maillog-20260807        secure-20260816.gz
[root@labserver log]# 

[root@labserver log]# ls -lh secure-20260730 
-rw-------. 1 root root 93K Jul 30 05:27 secure-20260730

[root@labserver log]# ls -lh secure-20260816.gz 
-rw-------. 1 root root 4.4K Aug 16 18:37 secure-20260816.gz
 
 # compress for storage management

  # Viewing logs using 'journalctl'

[root@labserver log]# journalctl
Aug 17 04:22:43 labserver kernel: Linux version 6.12.0-251.el10.x86_64 (mockbuild@15ff3073ce>
Aug 17 04:22:43 labserver kernel: Command line: BOOT_IMAGE=(hd0,gpt2)/vmlinuz-6.12.0-251.el1>
Aug 17 04:22:43 labserver kernel: x86/CPU: Model not found in latest microcode list
Aug 17 04:22:43 labserver kernel: BIOS-provided physical RAM map:
Aug 17 04:22:43 labserver kernel: BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usa>
Aug 17 04:22:43 labserver kernel: BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] res>
Aug 17 04:22:43 labserver kernel: BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] res>
Aug 17 04:22:43 labserver kernel: BIOS-e820: [mem 0x0000000000100000-0x000000007ffeffff] usa>
Aug 17 04:22:43 labserver kernel: BIOS-e820: [mem 0x000000007fff0000-0x000000007fffffff] ACP>
Aug 17 04:22:43 labserver kernel: BIOS-e820: [mem 0x00000000fec00000-0x00000000fec00fff] res>
Aug 17 04:22:43 labserver kernel: BIOS-e820: [mem 0x00000000fee00000-0x00000000fee00fff] res>
Aug 17 04:22:43 labserver kernel: BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] res>
Aug 17 04:22:43 labserver kernel: NX (Execute Disable) protection: active
Aug 17 04:22:43 labserver kernel: APIC: Static calls initialized
Aug 17 04:22:43 labserver kernel: SMBIOS 2.5 present.
Aug 17 04:22:43 labserver kernel: DMI: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 1>
Aug 17 04:22:43 labserver kernel: DMI: Memory slots populated: 0/0
Aug 17 04:22:43 labserver kernel: Hypervisor detected: KVM
Aug 17 04:22:43 labserver kernel: kvm-clock: Using msrs 4b564d01 and 4b564d00
Aug 17 04:22:43 labserver kernel: kvm-clock: using sched offset of 4441760985 cycles
Aug 17 04:22:43 labserver kernel: clocksource: kvm-clock: mask: 0xffffffffffffffff max_cycle>
Aug 17 04:22:43 labserver kernel: tsc: Detected 2419.232 MHz processor
Aug 17 04:22:43 labserver kernel: e820: update [mem 0x00000000-0x00000fff] usable ==> reserv>
Aug 17 04:22:43 labserver kernel: e820: remove [mem 0x000a0000-0x000fffff] usable
Aug 17 04:22:43 labserver kernel: last_pfn = 0x80000 max_arch_pfn = 0x400000000
Aug 17 04:22:43 labserver kernel: MTRR map: 4 entries (3 fixed + 1 variable; max 35), built >
Aug 17 04:22:43 labserver kernel: x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- >
Aug 17 04:22:43 labserver kernel: found SMP MP-table at [mem 0x0009fbf0-0x0009fbff]
Aug 17 04:22:43 labserver kernel: RAMDISK: [mem 0x33924000-0x35c89fff]
Aug 17 04:22:43 labserver kernel: ACPI: Early table checksum verification disabled
Aug 17 04:22:43 labserver kernel: ACPI: RSDP 0x00000000000E0000 000024 (v02 VBOX  )
Aug 17 04:22:43 labserver kernel: ACPI: XSDT 0x000000007FFF0030 00003C (v01 VBOX   VBOXXSDT >
Aug 17 04:22:43 labserver kernel: ACPI: FACP 0x000000007FFF00F0 0000F4 (v04 VBOX   VBOXFACP >
Aug 17 04:22:43 labserver kernel: ACPI: DSDT 0x000000007FFF02F0 002353 (v02 VBOX   VBOXBIOS >
Aug 17 04:22:43 labserver kernel: ACPI: FACS 0x000000007FFF0200 000040
Aug 17 04:22:43 labserver kernel: ACPI: FACS 0x000000007FFF0200 000040
Aug 17 04:22:43 labserver kernel: ACPI: APIC 0x000000007FFF0240 00005C (v02 VBOX   VBOXAPIC >
Aug 17 04:22:43 labserver kernel: ACPI: SSDT 0x000000007FFF02A0 000045 (v01 VBOX   VBOXCPUT >
Aug 17 04:22:43 labserver kernel: ACPI: Reserving FACP table memory at [mem 0x7fff00f0-0x7ff>
Aug 17 04:22:43 labserver kernel: ACPI: Reserving DSDT table memory at [mem 0x7fff02f0-0x7ff>
Aug 17 04:22:43 labserver kernel: ACPI: Reserving FACS table memory at [mem 0x7fff0200-0x7ff>
Aug 17 04:22:43 labserver kernel: ACPI: Reserving FACS table memory at [mem 0x7fff0200-0x7ff>
Aug 17 04:22:43 labserver kernel: ACPI: Reserving APIC table memory at [mem 0x7fff0240-0x7ff>
Aug 17 04:22:43 labserver kernel: ACPI: Reserving SSDT table memory at [mem 0x7fff02a0-0x7ff>
Aug 17 04:22:43 labserver kernel: No NUMA configuration found
Aug 17 04:22:43 labserver kernel: Faking a node at [mem 0x0000000000000000-0x000000007ffffff>
Aug 17 04:22:43 labserver kernel: NODE_DATA(0) allocated [mem 0x7ffc4b40-0x7ffeffff]
Aug 17 04:22:43 labserver kernel: crashkernel reserved: 0x000000006f000000 - 0x000000007f000>
[root@labserver log]# 

 # all logs at once

[root@labserver log]# systemctl status rsyslog
● rsyslog.service - System Logging Service
     Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-08-17 04:22:54 +0545; 57min ago
 Invocation: 28d585ee4dcc49a389ab7255992af1dc
       Docs: man:rsyslogd(8)
             https://www.rsyslog.com/doc/
    Process: 2531 ExecReload=/usr/bin/kill -HUP $MAINPID (code=exited, status=0/SUCCESS)
   Main PID: 997 (rsyslogd)
      Tasks: 3 (limit: 10630)
     Memory: 3.3M (peak: 6.6M)
        CPU: 253ms
     CGroup: /system.slice/rsyslog.service
             └─997 /usr/sbin/rsyslogd -n

Aug 17 04:22:54 labserver systemd[1]: Starting rsyslog.service - System Logging Service...
Aug 17 04:22:54 labserver rsyslogd[997]: [origin software="rsyslogd" swVersion="8.2604.0-4.e>
Aug 17 04:22:54 labserver systemd[1]: Started rsyslog.service - System Logging Service.
Aug 17 04:22:54 labserver rsyslogd[997]: imjournal: journal files changed, reloading...  [v8>
Aug 17 04:40:38 labserver systemd[1]: Reloading rsyslog.service - System Logging Service...
Aug 17 04:40:38 labserver systemd[1]: Reloaded rsyslog.service - System Logging Service.
Aug 17 04:40:38 labserver rsyslogd[997]: [origin software="rsyslogd" swVersion="8.2604.0-4.e>
[root@labserver log]# 

[root@labserver log]# rpm -q rsyslog
rsyslog-8.2604.0-4.el10.x86_64
 
[root@labserver log]# systemctl status systemd-journald
● systemd-journald.service - Journal Service
     Loaded: loaded (/usr/lib/systemd/system/systemd-journald.service; static)
     Active: active (running) since Mon 2026-08-17 04:22:52 +0545; 59min ago
 Invocation: 54c9a9e4e2a94cc58f0366b57092f274
TriggeredBy: ● systemd-journald.socket
             ○ systemd-journald-audit.socket
             ● systemd-journald-dev-log.socket
       Docs: man:systemd-journald.service(8)
             man:journald.conf(5)
   Main PID: 694 (systemd-journal)
     Status: "Processing requests..."
      Tasks: 1 (limit: 10630)
   FD Store: 12 (limit: 4224)
     Memory: 2.1M (peak: 4.6M)
        CPU: 104ms
     CGroup: /system.slice/systemd-journald.service
             └─694 /usr/lib/systemd/systemd-journald

Aug 17 04:22:52 labserver systemd-journald[694]: Collecting audit messages is disabled.
Aug 17 04:22:52 labserver systemd-journald[694]: Journal started
Aug 17 04:22:52 labserver systemd-journald[694]: Runtime Journal (/run/log/journal/6051512a1>
Aug 17 04:22:52 labserver systemd[1]: systemd-journald.service: Deactivated successfully.
Aug 17 04:22:53 labserver systemd-journald[694]: Runtime Journal (/run/log/journal/6051512a1>
Aug 17 04:22:53 labserver systemd-journald[694]: Received client request to flush runtime jo>
[root@labserver log]# 

[root@labserver log]# systemctl status rsyslog
● rsyslog.service - System Logging Service
     Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-08-17 04:22:54 +0545; 59min ago
 Invocation: 28d585ee4dcc49a389ab7255992af1dc
       Docs: man:rsyslogd(8)
             https://www.rsyslog.com/doc/
    Process: 2531 ExecReload=/usr/bin/kill -HUP $MAINPID (code=exited, status=0/SUCCESS)
   Main PID: 997 (rsyslogd)
      Tasks: 3 (limit: 10630)
     Memory: 3.3M (peak: 6.6M)
        CPU: 259ms
     CGroup: /system.slice/rsyslog.service
             └─997 /usr/sbin/rsyslogd -n

Aug 17 04:22:54 labserver systemd[1]: Starting rsyslog.service - System Logging Service...
Aug 17 04:22:54 labserver rsyslogd[997]: [origin software="rsyslogd" swVersion="8.2604.0-4.e>
Aug 17 04:22:54 labserver systemd[1]: Started rsyslog.service - System Logging Service.
Aug 17 04:22:54 labserver rsyslogd[997]: imjournal: journal files changed, reloading...  [v8>
Aug 17 04:40:38 labserver systemd[1]: Reloading rsyslog.service - System Logging Service...
Aug 17 04:40:38 labserver systemd[1]: Reloaded rsyslog.service - System Logging Service.
Aug 17 04:40:38 labserver rsyslogd[997]: [origin software="rsyslogd" swVersion="8.2604.0-4.e>
[root@labserver log]# 

 # rysyslog: 
           Traditional system logging service that processes, filters, stores, and forwards log messages, often to files under /var/log/.
 
 # systemd-journald: 
          systemd logging service that collects and stores system/service logs, viewable with journalctl and followable live with journalctl -f.
 
[root@labserver log]# ls /run/log/journal/
6051512a1e194d8fa01bc1948bcb0427
[root@labserver log]# ls /run/log/journal/6051512a1e194d8fa01bc1948bcb0427/
system.journal

 # journalctl --> reads this system.journal
 
 # Preserving System Journal

[root@labserver log]# mkdir -p /var/log/journal

[root@labserver log]# ls -ld /var/log/journal
drwxr-xr-x. 2 root root 6 Aug 17 05:29 /var/log/journal
[root@labserver log]# 

[root@labserver log]# chown root:systemd-journal /var/log/journal

[root@labserver log]# ls -ld /var/log/journal
drwxr-xr-x. 2 root systemd-journal 6 Aug 17 05:29 /var/log/journal
[root@labserver log]# 

[root@labserver log]# ls -ld /run/log/journal
drwxr-sr-x+ 3 root systemd-journal 60 Aug 17 04:22 /run/log/journal
[root@labserver log]# 

[root@labserver log]# chmod 2755 /var/log/journal

[root@labserver log]# ls -ld /var/log/journal
drwxr-sr-x. 2 root systemd-journal 6 Aug 17 05:29 /var/log/journal

[root@labserver log]# getfacl /run/log/journal
getfacl: Removing leading '/' from absolute path names
# file: run/log/journal
# owner: root
# group: systemd-journal
# flags: -s-
user::rwx
group::r-x
group:adm:r-x
group:wheel:r-x
mask::r-x
other::r-x
default:user::rwx
default:group::r-x
default:group:adm:r-x
default:group:wheel:r-x
default:mask::r-x
default:other::r-x

[root@labserver log]# 

[root@labserver log]# mkdir -p /etc/systemd/journald.conf.d

[root@labserver log]# vi /etc/systemd/journald.conf.d/10-persistent.conf
 
[root@labserver log]# cat /etc/systemd/journald.conf.d/10-persistent.conf
[journal]
Storage=persistent
[root@labserver log]# 

[root@labserver log]# systemctl restart systemd-journald

 # To save the logs of journald

[root@labserver log]# reboot
[root@labserver log]# Connection to 192.168.254.2 closed by remote host.
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
\S
Kernel \r on \m
aadarsha@192.168.254.2's password: 
Last login: Mon Aug 17 04:23:44 2026 from 192.168.254.152
[aadarsha@labserver ~]$ su -root
su: invalid option -- 'r'
Try 'su --help' for more information.
[aadarsha@labserver ~]$ su - root
Password: 
su: Authentication failure
[aadarsha@labserver ~]$ su - root
Password: 
Last login: Mon Aug 17 05:12:40 +0545 2026 on pts/0
Last failed login: Mon Aug 17 05:37:24 +0545 2026 on pts/0
There was 1 failed login attempt since the last successful login.
[root@labserver ~]# 

[root@labserver ~]# cd /var/log/journal/
[root@labserver journal]# ls
6051512a1e194d8fa01bc1948bcb0427
[root@labserver journal]# 

[root@labserver journal]# cd
[root@labserver ~]# 

[root@labserver ~]# journalctl
Aug 17 05:36:20 labserver kernel: Linux version 6.12.0-251.el10.x86_64 (mockbuild@15ff3073ce>
Aug 17 05:36:20 labserver kernel: Command line: BOOT_IMAGE=(hd0,gpt2)/vmlinuz-6.12.0-251.el1>
Aug 17 05:36:20 labserver kernel: x86/CPU: Model not found in latest microcode list
Aug 17 05:36:20 labserver kernel: BIOS-provided physical RAM map:
Aug 17 05:36:20 labserver kernel: BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usa>
Aug 17 05:36:20 labserver kernel: BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] res>
Aug 17 05:36:20 labserver kernel: BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] res>
Aug 17 05:36:20 labserver kernel: BIOS-e820: [mem 0x0000000000100000-0x000000007ffeffff] usa>
Aug 17 05:36:20 labserver kernel: BIOS-e820: [mem 0x000000007fff0000-0x000000007fffffff] ACP>
Aug 17 05:36:20 labserver kernel: BIOS-e820: [mem 0x00000000fec00000-0x00000000fec00fff] res>
Aug 17 05:36:20 labserver kernel: BIOS-e820: [mem 0x00000000fee00000-0x00000000fee00fff] res>
Aug 17 05:36:20 labserver kernel: BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] res>
Aug 17 05:36:20 labserver kernel: NX (Execute Disable) protection: active
Aug 17 05:36:20 labserver kernel: APIC: Static calls initialized
Aug 17 05:36:20 labserver kernel: SMBIOS 2.5 present.
Aug 17 05:36:20 labserver kernel: DMI: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 1>
Aug 17 05:36:20 labserver kernel: DMI: Memory slots populated: 0/0
Aug 17 05:36:20 labserver kernel: Hypervisor detected: KVM
Aug 17 05:36:20 labserver kernel: kvm-clock: Using msrs 4b564d01 and 4b564d00
Aug 17 05:36:20 labserver kernel: kvm-clock: using sched offset of 4653698686 cycles
Aug 17 05:36:20 labserver kernel: clocksource: kvm-clock: mask: 0xffffffffffffffff max_cycle>
Aug 17 05:36:20 labserver kernel: tsc: Detected 2419.232 MHz processor
Aug 17 05:36:20 labserver kernel: e820: update [mem 0x00000000-0x00000fff] usable ==> reserv>
Aug 17 05:36:20 labserver kernel: e820: remove [mem 0x000a0000-0x000fffff] usable
Aug 17 05:36:20 labserver kernel: last_pfn = 0x80000 max_arch_pfn = 0x400000000
Aug 17 05:36:20 labserver kernel: MTRR map: 4 entries (3 fixed + 1 variable; max 35), built >
Aug 17 05:36:20 labserver kernel: x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- >
Aug 17 05:36:20 labserver kernel: found SMP MP-table at [mem 0x0009fbf0-0x0009fbff]
Aug 17 05:36:20 labserver kernel: RAMDISK: [mem 0x33924000-0x35c89fff]
Aug 17 05:36:20 labserver kernel: ACPI: Early table checksum verification disabled
Aug 17 05:36:20 labserver kernel: ACPI: RSDP 0x00000000000E0000 000024 (v02 VBOX  )
Aug 17 05:36:20 labserver kernel: ACPI: XSDT 0x000000007FFF0030 00003C (v01 VBOX   VBOXXSDT >
Aug 17 05:36:20 labserver kernel: ACPI: FACP 0x000000007FFF00F0 0000F4 (v04 VBOX   VBOXFACP >
Aug 17 05:36:20 labserver kernel: ACPI: DSDT 0x000000007FFF02F0 002353 (v02 VBOX   VBOXBIOS >
Aug 17 05:36:20 labserver kernel: ACPI: FACS 0x000000007FFF0200 000040
Aug 17 05:36:20 labserver kernel: ACPI: FACS 0x000000007FFF0200 000040
Aug 17 05:36:20 labserver kernel: ACPI: APIC 0x000000007FFF0240 00005C (v02 VBOX   VBOXAPIC >
Aug 17 05:36:20 labserver kernel: ACPI: SSDT 0x000000007FFF02A0 000045 (v01 VBOX   VBOXCPUT >
Aug 17 05:36:20 labserver kernel: ACPI: Reserving FACP table memory at [mem 0x7fff00f0-0x7ff>
Aug 17 05:36:20 labserver kernel: ACPI: Reserving DSDT table memory at [mem 0x7fff02f0-0x7ff>
Aug 17 05:36:20 labserver kernel: ACPI: Reserving FACS table memory at [mem 0x7fff0200-0x7ff>
Aug 17 05:36:20 labserver kernel: ACPI: Reserving FACS table memory at [mem 0x7fff0200-0x7ff>
Aug 17 05:36:20 labserver kernel: ACPI: Reserving APIC table memory at [mem 0x7fff0240-0x7ff>
Aug 17 05:36:20 labserver kernel: ACPI: Reserving SSDT table memory at [mem 0x7fff02a0-0x7ff>
Aug 17 05:36:20 labserver kernel: No NUMA configuration found
Aug 17 05:36:20 labserver kernel: Faking a node at [mem 0x0000000000000000-0x000000007ffffff>
Aug 17 05:36:20 labserver kernel: NODE_DATA(0) allocated [mem 0x7ffc4b40-0x7ffeffff]
Aug 17 05:36:20 labserver kernel: crashkernel reserved: 0x000000006f000000 - 0x000000007f000>
[root@labserver ~]# 

[root@labserver ~]# ls /run/log/journal/

[root@labserver ~]# ls
anaconda-ks.cfg  disk_monitor.sh

[root@labserver ~]# ls /run/log/journal/
[root@labserver ~]# 

[root@labserver ~]# ls /var/log/journal/6051512a1e194d8fa01bc1948bcb0427/
system.journal  user-1000.journal
[root@labserver ~]# 

[root@labserver ~]# ls -ld /var/log/journal/
drwxr-sr-x+ 3 root systemd-journal 46 Aug 17 05:36 /var/log/journal/
[root@labserver ~]# 

[root@labserver ~]# journalctl -n
Aug 17 05:37:12 labserver systemd[1]: Starting systemd-hostnamed.service - Hostname Service.>
Aug 17 05:37:12 labserver systemd[1]: Started systemd-hostnamed.service - Hostname Service.
Aug 17 05:37:22 labserver unix_chkpwd[2332]: password check failed for user (root)
Aug 17 05:37:22 labserver su[2328]: pam_unix(su-l:auth): authentication failure; logname=aad>
Aug 17 05:37:24 labserver su[2328]: FAILED SU (to root) aadarsha on pts/0
Aug 17 05:37:30 labserver su[2333]: (to root) aadarsha on pts/0
Aug 17 05:37:30 labserver su[2333]: pam_unix(su-l:session): session opened for user root(uid>
Aug 17 05:38:00 labserver systemd[1]: systemd-hostnamed.service: Deactivated successfully.
Aug 17 05:40:08 labserver systemd[2286]: Starting grub-boot-success.service - Mark boot as s>
Aug 17 05:40:08 labserver systemd[2286]: Finished grub-boot-success.service - Mark boot as s>
[root@labserver ~]# 

[root@labserver ~]# journalctl -n 20
Aug 17 05:37:11 labserver systemd[2286]: Reached target sockets.target - Sockets.
Aug 17 05:37:11 labserver systemd[2286]: Finished systemd-tmpfiles-setup.service - Create Us>
Aug 17 05:37:11 labserver systemd[2286]: Reached target basic.target - Basic System.
Aug 17 05:37:11 labserver systemd[2286]: Reached target default.target - Main User Target.
Aug 17 05:37:11 labserver systemd[2286]: Startup finished in 146ms.
Aug 17 05:37:11 labserver systemd[1]: Started user@1000.service - User Manager for UID 1000.
Aug 17 05:37:11 labserver systemd[1]: Started session-1.scope - Session 1 of User aadarsha.
Aug 17 05:37:12 labserver systemd[1]: Starting systemd-hostnamed.service - Hostname Service.>
Aug 17 05:37:12 labserver systemd[1]: Started systemd-hostnamed.service - Hostname Service.
Aug 17 05:37:22 labserver unix_chkpwd[2332]: password check failed for user (root)
Aug 17 05:37:22 labserver su[2328]: pam_unix(su-l:auth): authentication failure; logname=aad>
Aug 17 05:37:24 labserver su[2328]: FAILED SU (to root) aadarsha on pts/0
Aug 17 05:37:30 labserver su[2333]: (to root) aadarsha on pts/0
Aug 17 05:37:30 labserver su[2333]: pam_unix(su-l:session): session opened for user root(uid>
Aug 17 05:38:00 labserver systemd[1]: systemd-hostnamed.service: Deactivated successfully.
Aug 17 05:40:08 labserver systemd[2286]: Starting grub-boot-success.service - Mark boot as s>
Aug 17 05:40:08 labserver systemd[2286]: Finished grub-boot-success.service - Mark boot as s>
Aug 17 05:43:08 labserver systemd[2286]: Created slice background.slice - User Background Ta>
Aug 17 05:43:08 labserver systemd[2286]: Starting systemd-tmpfiles-clean.service - Cleanup o>
Aug 17 05:43:08 labserver systemd[2286]: Finished systemd-tmpfiles-clean.service - Cleanup o>
[root@labserver ~]# 

 # view live log

[root@labserver ~]# tail -f /var/log/secure			
Aug 17 05:12:40 labserver su[2807]: pam_unix(su-l:session): session opened for user root(uid=0) by aadarsha(uid=1000)
Aug 17 05:36:31 labserver sshd[945]: Server listening on 0.0.0.0 port 22.
Aug 17 05:36:31 labserver sshd[945]: Server listening on :: port 22.
Aug 17 05:37:11 labserver sshd-session[2277]: Accepted password for aadarsha from 192.168.254.152 port 60090 ssh2
Aug 17 05:37:11 labserver (systemd)[2286]: pam_unix(systemd-user:session): session opened for user aadarsha(uid=1000) by aadarsha(uid=0)
Aug 17 05:37:11 labserver sshd-session[2277]: pam_unix(sshd:session): session opened for user aadarsha(uid=1000) by aadarsha(uid=0)
Aug 17 05:37:22 labserver unix_chkpwd[2332]: password check failed for user (root)
Aug 17 05:37:22 labserver su[2328]: pam_unix(su-l:auth): authentication failure; logname=aadarsha uid=1000 euid=0 tty=/dev/pts/0 ruser=aadarsha rhost=  user=root
Aug 17 05:37:30 labserver su[2333]: pam_unix(su-l:session): session opened for user root(uid=0) by aadarsha(uid=1000)

 # see: live log
 
[root@labserver ~]# journalctl -f
Aug 17 05:37:22 labserver su[2328]: pam_unix(su-l:auth): authentication failure; logname=aadarsha uid=1000 euid=0 tty=/dev/pts/0 ruser=aadarsha rhost=  user=root
Aug 17 05:37:24 labserver su[2328]: FAILED SU (to root) aadarsha on pts/0
Aug 17 05:37:30 labserver su[2333]: (to root) aadarsha on pts/0
Aug 17 05:37:30 labserver su[2333]: pam_unix(su-l:session): session opened for user root(uid=0) by aadarsha(uid=1000)
Aug 17 05:38:00 labserver systemd[1]: systemd-hostnamed.service: Deactivated successfully.
Aug 17 05:40:08 labserver systemd[2286]: Starting grub-boot-success.service - Mark boot as successful...
Aug 17 05:40:08 labserver systemd[2286]: Finished grub-boot-success.service - Mark boot as successful.
Aug 17 05:43:08 labserver systemd[2286]: Created slice background.slice - User Background Tasks Slice.
Aug 17 05:43:08 labserver systemd[2286]: Starting systemd-tmpfiles-clean.service - Cleanup of User's Temporary Files and Directories...
Aug 17 05:43:08 labserver systemd[2286]: Finished systemd-tmpfiles-clean.service - Cleanup of User's Temporary Files and Directories.
...

[root@labserver ~]# journalctl --since "05:40:00"
Aug 17 05:40:08 labserver systemd[2286]: Starting grub-boot-success.service - Mark boot as s>
Aug 17 05:40:08 labserver systemd[2286]: Finished grub-boot-success.service - Mark boot as s>
Aug 17 05:43:08 labserver systemd[2286]: Created slice background.slice - User Background Ta>
Aug 17 05:43:08 labserver systemd[2286]: Starting systemd-tmpfiles-clean.service - Cleanup o>
Aug 17 05:43:08 labserver systemd[2286]: Finished systemd-tmpfiles-clean.service - Cleanup o>
[root@labserver ~]# 

[root@labserver ~]# journalctl --since yesterday
Aug 17 05:36:20 labserver kernel: Linux version 6.12.0-251.el10.x86_64 (mockbuild@15ff3073ce>
Aug 17 05:36:20 labserver kernel: Command line: BOOT_IMAGE=(hd0,gpt2)/vmlinuz-6.12.0-251.el1>
Aug 17 05:36:20 labserver kernel: x86/CPU: Model not found in latest microcode list
Aug 17 05:36:20 labserver kernel: BIOS-provided physical RAM map:
...

[root@labserver ~]# journalctl --since "05:40:00" --until "05:45:00"
Aug 17 05:40:08 labserver systemd[2286]: Starting grub-boot-success.service - Mark boot as s>
Aug 17 05:40:08 labserver systemd[2286]: Finished grub-boot-success.service - Mark boot as s>
Aug 17 05:43:08 labserver systemd[2286]: Created slice background.slice - User Background Ta>
Aug 17 05:43:08 labserver systemd[2286]: Starting systemd-tmpfiles-clean.service - Cleanup o>
Aug 17 05:43:08 labserver systemd[2286]: Finished systemd-tmpfiles-clean.service - Cleanup o>
[root@labserver ~]# 

[root@labserver ~]# journalctl --since "2026-08-01 05:40:00" --until "2026-08-17 05:45:00"
...

[root@labserver ~]# journalctl --since "2026-08-01 05:40:00" --until "2026-08-17 05:45:00" | grep sshd
Aug 17 05:36:30 labserver systemd[1]: Created slice system-sshd\x2dkeygen.slice - Slice /system/sshd-keygen.
Aug 17 05:36:31 labserver systemd[1]: Listening on sshd-unix-local.socket - OpenSSH Server Socket (systemd-ssh-generator, AF_UNIX Local).
Aug 17 05:36:31 labserver systemd[1]: Listening on sshd-vsock.socket - OpenSSH Server Socket (systemd-ssh-generator, AF_VSOCK).
Aug 17 05:36:31 labserver systemd[1]: sshd-keygen@ecdsa.service - OpenSSH ecdsa Server Key Generation was skipped because no trigger condition checks were met.
Aug 17 05:36:31 labserver systemd[1]: sshd-keygen@ed25519.service - OpenSSH ed25519 Server Key Generation was skipped because no trigger condition checks were met.
Aug 17 05:36:31 labserver systemd[1]: sshd-keygen@rsa.service - OpenSSH rsa Server Key Generation was skipped because no trigger condition checks were met.
Aug 17 05:36:31 labserver systemd[1]: Reached target sshd-keygen.target.
Aug 17 05:36:31 labserver systemd[1]: Starting sshd.service - OpenSSH server daemon...
Aug 17 05:36:31 labserver sshd[945]: Server listening on 0.0.0.0 port 22.
Aug 17 05:36:31 labserver systemd[1]: Started sshd.service - OpenSSH server daemon.
Aug 17 05:36:31 labserver sshd[945]: Server listening on :: port 22.
Aug 17 05:37:11 labserver sshd-session[2277]: Accepted password for aadarsha from 192.168.254.152 port 60090 ssh2
Aug 17 05:37:11 labserver sshd-session[2277]: pam_unix(sshd:session): session opened for user aadarsha(uid=1000) by aadarsha(uid=0)
[root@labserver ~]# 


[root@labserver ~]# journalctl | grep -i "failed password"
[root@labserver ~]# 

[root@labserver ~]# journalctl --since "20226-08-91 05:40:00" --until "2026-08-17 05:48:00" | grep -i "failed password"
Failed to parse timestamp: 20226-08-91 05:40:00
[root@labserver ~]# 

[root@labserver ~]# journalctl -p warning
Aug 17 05:36:20 labserver kernel: APIC calibration not consistent with PM-Timer: 120ms inste>
Aug 17 05:36:20 labserver kernel: acpi PNP0A03:00: fail to add MMCONFIG information, can't a>
Aug 17 05:36:21 labserver kernel: Warning: Unmaintained driver is detected: e1000
Aug 17 05:36:21 labserver kernel: Warning: Unmaintained driver is detected: e1000_init_module
Aug 17 05:36:30 labserver systemd-journald[693]: /etc/systemd/journald.conf.d/10-persistent.>
Aug 17 05:36:30 labserver lvm[794]: PV /dev/sdd online, VG cs incomplete (need 1).
Aug 17 05:36:30 labserver lvm[795]: PV /dev/sda3 online, VG cs is complete.
Aug 17 05:36:30 labserver kernel: vmwgfx 0000:00:02.0: [drm] *ERROR* vmwgfx seems to be runn>
Aug 17 05:36:30 labserver kernel: vmwgfx 0000:00:02.0: [drm] *ERROR* This configuration is l>
Aug 17 05:36:30 labserver kernel: vmwgfx 0000:00:02.0: [drm] *ERROR* Please switch to a supp>
Aug 17 05:36:32 labserver (httpd)[1001]: httpd.service: Referenced but unset environment var>
Aug 17 05:36:32 labserver rpc.idmapd[1006]: Setting log level to 0
Aug 17 05:36:32 labserver (atd)[1023]: atd.service: Referenced but unset environment variabl>
Aug 17 05:36:32 labserver rpc.statd[1044]: Flags: TI-RPC
Aug 17 05:36:32 labserver rpc.idmapd[1006]: libnfsidmap: Unable to determine the NFSv4 domai>
Aug 17 05:36:33 labserver kernel: block dm-2: the capability attribute has been deprecated.
[root@labserver ~]# 

[root@labserver ~]# journalctl -p crit
Aug 17 05:36:21 labserver kernel: Warning: Unmaintained driver is detected: e1000
Aug 17 05:36:21 labserver kernel: Warning: Unmaintained driver is detected: e1000_init_module
[root@labserver ~]# 

[root@labserver ~]# journalctl -p emerg
-- No entries --
[root@labserver ~]# 

[root@labserver ~]# man journalctl 

[root@labserver ~]# systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-08-17 05:36:31 +0545; 25min ago
 Invocation: a84b2d3e00fc42ef92a80bbf1ab91c34
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 945 (sshd)
      Tasks: 1 (limit: 10630)
     Memory: 6.2M (peak: 23M)
        CPU: 100ms
     CGroup: /system.slice/sshd.service
             └─945 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Aug 17 05:36:31 labserver systemd[1]: Starting sshd.service - OpenSSH server daemon...
Aug 17 05:36:31 labserver sshd[945]: Server listening on 0.0.0.0 port 22.
Aug 17 05:36:31 labserver systemd[1]: Started sshd.service - OpenSSH server daemon.
Aug 17 05:36:31 labserver sshd[945]: Server listening on :: port 22.
Aug 17 05:37:11 labserver sshd-session[2277]: Accepted password for aadarsha from 192.168.25>
Aug 17 05:37:11 labserver sshd-session[2277]: pam_unix(sshd:session): session opened for use>
[root@labserver ~]# 
 
[root@labserver ~]# journalctl -u sshd
Aug 17 05:36:31 labserver systemd[1]: Starting sshd.service - OpenSSH server daemon...
Aug 17 05:36:31 labserver sshd[945]: Server listening on 0.0.0.0 port 22.
Aug 17 05:36:31 labserver systemd[1]: Started sshd.service - OpenSSH server daemon.
Aug 17 05:36:31 labserver sshd[945]: Server listening on :: port 22.
Aug 17 05:37:11 labserver sshd-session[2277]: Accepted password for aadarsha from 192.168.25>
Aug 17 05:37:11 labserver sshd-session[2277]: pam_unix(sshd:session): session opened for use>
[root@labserver ~]# 

[root@labserver ~]# journalctl -u httpd
Aug 17 05:36:32 labserver systemd[1]: Starting httpd.service - The Apache HTTP Server...
Aug 17 05:36:32 labserver (httpd)[1001]: httpd.service: Referenced but unset environment var>
Aug 17 05:36:32 labserver httpd[1001]: AH00558: httpd: Could not reliably determine the serv>
Aug 17 05:36:32 labserver httpd[1001]: Server configured, listening on: port 8080
Aug 17 05:36:32 labserver systemd[1]: Started httpd.service - The Apache HTTP Server.
[root@labserver ~]#
 
[root@labserver ~]# ls /var/log/httpd/
access_log  access_log-20260816  error_log  error_log-20260816.gz  error_log-20260817
[root@labserver ~]# 

[root@labserver ~]# journalctl -u sshd | wc -l
6
[root@labserver ~]# 

 # Task to do: Configuring central log management server of multiple machines 
```