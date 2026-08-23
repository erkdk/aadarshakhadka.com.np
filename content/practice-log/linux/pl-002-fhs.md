---
title: "PL - 002 — Linux Filesystem"
date: 2026-06-06
draft: false
---

### Linux Filesystem Hierarchy

Linux systems organize directories and resources under a unified root namespace (`/`). This structure is guided by the **Filesystem Hierarchy Standard (FHS)**, ensuring consistent file locations across Unix-like operating systems.

> Unix-like systems present a unified file-descriptor interface for interacting with system resources—including regular files, directories, device nodes, pipes, sockets, and virtual kernel interfaces. While summarized by the phrase *"everything is a file,"* processes and abstract kernel objects remain distinct internal entities managed by the kernel.

### Filesystem Hierarchy 

| Directory |  Purpose |
| :--- | :--- |
| `/` | Root of the entire filesystem hierarchy |
| `/boot` | Kernel and boot-related files |
| `/dev` | Device nodes and device interfaces |
| `/etc` | System-wide configuration |
| `/home` | Regular users' home directories |
| `/media` | Removable-media mount points |
| `/mnt` | Manual/temporary mount point |
| `/opt` | Optional or third-party software |
| `/proc` | Process and kernel information |
| `/root` | Root user's home directory |
| `/run` | Volatile runtime state |
| `/srv` | Data served by system services |
| `/sys` | Kernel objects, devices, and drivers |
| `/tmp` | Temporary files |
| `/usr` | User-space programs, libraries, and shared data |
| `/var` | Variable and persistent system/application data |


A system may also contain additional directories such as `/lost+found`, `/snap`, or distribution-specific directories.

### Linux File Types

Linux filesystems support several fundamental file types. The first character in the output of `ls -l` identifies the type.

| Character | Type              | Description                                                       |
| --------- | ----------------- | ----------------------------------------------------------------- |
| `-`       | Regular file      | Contains ordinary data such as text, programs, images, or scripts |
| `d`       | Directory         | Contains directory entries that organize files hierarchically     |
| `l`       | Symbolic link     | Refers to another filesystem path                                 |
| `c`       | Character device  | Provides a character-oriented interface to a device               |
| `b`       | Block device      | Provides block-oriented access to a device such as a disk         |
| `p`       | Named pipe (FIFO) | Provides FIFO-style inter-process communication                   |
| `s`       | Socket            | Provides bidirectional communication between processes            |

---
### Terminal Session
```
[aadarsha@mainserver ~]$ cat /etc/os-release 
NAME="CentOS Stream"
VERSION="10 (Coughlan)"
RELEASE_TYPE=stable
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="10"
PLATFORM_ID="platform:el10"
PRETTY_NAME="CentOS Stream 10 (Coughlan)"
ANSI_COLOR="0;31"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:centos:centos:10"
HOME_URL="https://centos.org/"
VENDOR_NAME="CentOS"
VENDOR_URL="https://centos.org/"
BUG_REPORT_URL="https://issues.redhat.com/"
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux 10"
REDHAT_SUPPORT_PRODUCT_VERSION="CentOS Stream"

[aadarsha@mainserver ~]$ whoami
aadarsha

[aadarsha@mainserver ~]$ pwd
/home/aadarsha

[aadarsha@mainserver ~]$ cd /

[aadarsha@mainserver /]$ pwd
/

[aadarsha@mainserver /]$ ls
afs  boot  etc   lib    media  opt   root  sbin  sys  usr
bin  dev   home  lib64  mnt    proc  run   srv   tmp  var

[aadarsha@mainserver /]$ ls -l
total 20
dr-xr-xr-x.   2 root root    6 Apr  2  2025 afs
lrwxrwxrwx.   1 root root    7 Apr  2  2025 bin -> usr/bin
dr-xr-xr-x.   5 root root 4096 Jul 13 14:45 boot
drwxr-xr-x.  20 root root 3300 Aug 23 19:30 dev
drwxr-xr-x.  82 root root 8192 Aug 23 19:30 etc
drwxr-xr-x.   5 root root  100 Aug  5 21:47 home
lrwxrwxrwx.   1 root root    7 Apr  2  2025 lib -> usr/lib
lrwxrwxrwx.   1 root root    9 Apr  2  2025 lib64 -> usr/lib64
drwxr-xr-x.   2 root root    6 Apr  2  2025 media
drwxr-xr-x.   2 root root    6 Apr  2  2025 mnt
drwxr-xr-x.   2 root root    6 Apr  2  2025 opt
dr-xr-xr-x. 200 root root    0 Aug 23 19:29 proc
dr-xr-x---.   4 root root  186 Aug  5 21:46 root
drwxr-xr-x.  32 root root  880 Aug 23 19:30 run
lrwxrwxrwx.   1 root root    8 Apr  2  2025 sbin -> usr/sbin
drwxr-xr-x.   2 root root    6 Apr  2  2025 srv
dr-xr-xr-x.  13 root root    0 Aug 23 19:30 sys
drwxrwxrwt.  10 root root 4096 Aug 23 19:32 tmp
drwxr-xr-x.  12 root root  144 Jul 13 14:29 usr
drwxr-xr-x.  19 root root  264 Jul 13 14:45 var
[aadarsha@mainserver /]$ 

# here : 
  bin -> usr/bin,  lib -> usr/lib,  lib64 -> usr/lib64, sbin -> usr/sbin  are soft links 
```
---
### Primary Directories

#### Root & System Boot
* **`/` (Root):** Top-level directory; every absolute path originates here.
* **`/boot`:** Contains files required to initialize the system during boot:
  * `vmlinuz-*`: Compressed Linux kernel image.
  * `initramfs-*`: Initial RAM filesystem used for early userspace initialization.
  * `grub2/` or `loader/`: Bootloader configurations and binary modules.

#### System Configuration & Hardware Interfaces
* **`/etc`:** Contains global administrative system and service configuration files (e.g., `fstab`, `hosts`, `passwd`, `sysctl.conf`).
Configuration in `/etc` may relate to:
    * networking
    * authentication
    * users and groups
    * SSH
    * DNS
    * package management
    * boot configuration
    * security
    * system services
    * installed applications

* **`/dev`:** Exposes kernel device nodes managed dynamically (typically via `udev`).
  * Block devices: `/dev/sda`
  * Character interfaces: `/dev/tty`
  * Pseudo-devices: `/dev/null`, `/dev/zero`, `/dev/urandom`

#### User Spaces & Mounting
* **`/home`:** Houses persistent personal storage and configurations for regular users (`/home/<username>`).
* **`/root`:** Dedicated home directory for the `root` administrative user, intentionally kept separate from `/home` on the primary root partition.
* **`/media`:** Automated mount location for removable storage media (e.g., USB drives, optical disks).
* **`/mnt`:** Standard temporary mount point for manual filesystem attachment by administrators.
* **`/opt`:** Standard destination for standalone, third-party software packages installed outside system package management (e.g., Google, Oracle, VMware).
* **`/srv`:** Holds site-specific data served by network daemons (e.g., `/srv/www` for web applications).

#### User-Space Programs (`/usr`)
The `/usr` directory serves as the primary secondary hierarchy for read-only user space data and binaries:

* **`/usr/bin`:** Primary executable commands for user applications (`bash`, `git`, `gcc`, `python`).
* **`/usr/sbin`:** Executable binaries for system administration and management tasks.
* **`/usr/lib` / `/usr/lib64`:** Architecture-dependent shared libraries and kernel modules.
* **`/usr/local`:** Primary destination for locally compiled or non-packaged custom software.
* **`/usr/share`:** Platform-independent architecture-neutral assets (man pages, documentation, locales).
* **`/usr/include`:** C/C++ header files required during source code compilation.
* **`/usr/src`:** Kernel and application source code files.

> **UsrMerge Note:** Modern distributions simplify this layout by converting legacy directories (`/bin`, `/sbin`, `/lib`, `/lib64`) into symbolic links pointing directly to their `/usr` counterparts (`/usr/bin`, `/usr/sbin`, `/usr/lib`, `/usr/lib64`).

#### Variable Application Data (`/var`)
Contains stateful data that continually changes during normal system runtime:

* **`/var/cache`:** Cached application-level data intended to speed up operations.
* **`/var/lib`:** Stateful persistent database engine files, container storage, and service states (do not delete).
* **`/var/log`:** System and service logs (`messages`, `journal/`, `dnf.log`).
* **`/var/spool`:** Queued tasks awaiting execution (print queues, cron schedules, mail queues).
* **`/var/tmp`:** Temporary data designed to survive system reboots longer than `/tmp`.

---
### Virtual & Runtime Filesystems

Linux implements pseudo-filesystems to expose dynamic kernel and hardware states as accessible file interfaces:

| Path | Virtual Type | Primary Purpose | Persistent Disk Storage? |
| :--- | :--- | :--- | :--- |
| **`/proc`** | `procfs` | Exposes active process data (PID folders) and runtime kernel metrics (`/proc/meminfo`, `/proc/cpuinfo`) | No (In-Memory) |
| **`/sys`** | `sysfs` | Exposes structural kernel objects, hardware topology, device drivers, and subsystem tunables | No (In-Memory) |
| **`/run`** | `tmpfs` | Stores volatile runtime state, PID files, and IPC sockets generated during the current boot cycle | No (In-Memory) |
| **`/tmp`** | `tmpfs` / Disk | Short-lived temporary workspace cleared on reboot or periodic cleanup policies | Varies by distro |

---
#### Structural Distinctions
* **`/` vs `/root`:** `/` is the global filesystem root; `/root` is the home directory for the administrative user.
* **`/tmp` vs `/var/tmp`:** `/tmp` is for ephemeral scratch space; `/var/tmp` persists across reboots for longer-term temporary processing.
* **`/proc` vs `/sys`:** `/proc` focuses on process status and kernel parameters; `/sys` focuses on hardware bus topology, drivers, and device representations.
* **`/run` vs `/var`:** `/run` stores non-persistent volatile boot session data; `/var` stores persistent changing system logs and application databases.

---

#### Further Documentation & References
* `man hier` — Local system manual page for directory structures
* `man proc` — System documentation for `/proc` interfaces
* `man sysfs` — Kernel object export subsystem documentation
* `man findmnt` — Command tool for inspecting system mount trees
* `man file` — Identify filesystem object types
* `man ls` — List directory contents
* [Filesystem Hierarchy Standard (FHS) 3.0 Specification](https://refspecs.linuxfoundation.org/fhs.shtml)
* [Linux Kernel Documentation on procfs](https://www.kernel.org/doc/html/latest/filesystems/proc.html)
* [Linux Kernel Documentation on sysfs](https://www.kernel.org/doc/html/latest/admin-guide/sysfs-rules.html)
---