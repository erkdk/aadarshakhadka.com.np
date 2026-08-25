---
title: "PL - 006 — Linux System Information, File Inspection & Getting Help"
date: 2026-06-10
draft: false
---

### Terminal Session

#### Lab Environment
  - OS: CentOS Stream 10
  - Architecture: x86_64
  - Virtualization: VirtualBox
  - CPU: 2 vCPU
  - Memory: ~1.7 GiB
  - Filesystem: XFS
  - Storage: LVM

#### System Identity
```
# Displays the system's network hostname.

[aadarsha@labserver ~]$ hostname
labserver

#  Shows current username

[aadarsha@labserver ~]$ whoami
aadarsha

# Show comprehensive system information including OS version, kernel, and virtualization details.

[root@labserver ~]# hostnamectl
 Static hostname: labserver
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: <redacted>
         Boot ID: <redacted>
    Product UUID: <redacted>
  Virtualization: oracle
Operating System: CentOS Stream 10 (Coughlan)                    
     CPE OS Name: cpe:/o:centos:centos:10
          Kernel: Linux 6.12.0-228.el10.x86_64
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
 Hardware Serial: <redacted>
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 19y 7month 2w 2d                                

#  Shows user ID, group ID, and group memberships

[aadarsha@labserver ~]$ id
uid=1000(aadarsha) gid=1000(aadarsha) groups=1000(aadarsha),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# Show current directory

[aadarsha@labserver ~]$ pwd
/home/aadarsha
```
#### Operating System and Kernel
```
# Displays the running kernel release.

[aadarsha@labserver ~]$ uname -r
6.12.0-228.el10.x86_64

# Displays multiple kernel/system attributes, including kernel name, hostname, kernel release, kernel version, machine architecture, and operating-system information.

[aadarsha@labserver ~]$ uname -a
Linux labserver 6.12.0-228.el10.x86_64 #1 SMP PREEMPT_DYNAMIC Thu May 14 14:51:31 UTC 2026 x86_64 GNU/Linux

# Show Linux distribution details

[aadarsha@labserver ~]$ cat /etc/os-release
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
[aadarsha@labserver ~]$ 
```
#### CPU and Memory
```
# Detailed memory statistics from the kernel

[aadarsha@labserver ~]$ head /proc/meminfo
MemTotal:        1745836 kB
MemFree:         1304920 kB
MemAvailable:    1380824 kB
Buffers:            5304 kB
Cached:           198944 kB
SwapCached:            0 kB
Active:           190272 kB
Inactive:          94280 kB
Active(anon):      85168 kB
Inactive(anon):        0 kB

# Number of processing units

[aadarsha@labserver ~]$ nproc
2 

# Memory Information

[aadarsha@labserver ~]$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       363Mi       1.1Gi       4.8Mi       308Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
```
| Field | Meaning |
|---------|---------|
| total | Total installed RAM |
| used | Memory currently used |
| free |  Memory currently unused and not allocated to anything. |
| shared | Shared memory |
| buff/cache | Memory used by kernel cache |
| available | Memory available for new processes |
| Swap | Swap partition usage |
```
```
> For practical memory availability, available is generally more useful than free.
```
```
### Storage and Filesystems

```
# Shows the block-device/storage topology: (disks and partitions) in a tree format
# LVM (Logical Volume Manager) is being used for flexible storage management

[aadarsha@labserver ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sr0          11:0    1 1024M  0 rom  

# Shows mounted filesystems

[aadarsha@labserver ~]$ mount | head
/dev/mapper/cs-root on / type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=849912k,nr_inodes=212478,mode=755,inode64)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,seclabel,inode64,usrquota)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,seclabel,nsdelegate,memory_recursiveprot)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime,seclabel)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
configfs on /sys/kernel/config type configfs (rw,nosuid,nodev,noexec,relatime)


# Disk Information
# Shows filesystem capacity and usage for mounted filesystems.

[aadarsha@labserver ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   16G  1.6G   15G  10% /
devtmpfs             830M     0  830M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            2.0G  405M  1.6G  21% /boot
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000

```
| Field | Meaning |
|--------|----------|
| Filesystem | Mounted filesystem |
| Size | Total filesystem size |
| Used | Used space |
| Avail | Available space |
| Use% | Percentage used |
| Mounted on | Mount point |

```
[aadarsha@labserver ~]$ ls
dir1  dira  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz

# stat reports filesystem metadata; it is not merely another way to display the contents or type of a file.
#  timestamp
  - Access — last access time
  - Modify — file content modification time
  - Change — inode metadata/status change time
  - Birth — creation time, when supported by the filesystem/platform

[aadarsha@labserver ~]$ stat testfile 
  File: testfile
  Size: 56        	Blocks: 8          IO Block: 4096   regular file
Device: 253,0	Inode: 526546      Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/aadarsha)   Gid: ( 1000/aadarsha)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2026-06-07 21:33:39.784384746 +0545
Modify: 2026-06-07 21:33:34.476381274 +0545
Change: 2026-06-07 21:33:34.476381274 +0545
 Birth: 2026-06-07 09:23:07.596008113 +0545

[aadarsha@labserver ~]$ stat impfiles.tar 
  File: impfiles.tar
  Size: 11704320  	Blocks: 22864      IO Block: 4096   regular file
Device: 253,0	Inode: 527801      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2026-06-08 16:35:14.826687643 +0545
Modify: 2026-06-08 16:38:29.505323470 +0545
Change: 2026-06-08 16:38:29.505323470 +0545
 Birth: 2026-06-08 16:34:03.703711086 +0545

[aadarsha@labserver ~]$ ls -lh dir1/
total 0
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  7 15:41 dir2
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 15:48 numfile

[aadarsha@labserver ~]$ ls -lh dira
total 4.8M
drwxr-xr-x. 3 aadarsha aadarsha   18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha    0 Jun  7 16:10 letfile
-rw-r--r--. 1 aadarsha aadarsha   73 Jun  8 08:47 myfile_hard
-rw-r--r--. 1 aadarsha aadarsha 4.8M Jun  8 09:52 words

[aadarsha@labserver ~]$ ls -ld dira                     # to see the details of the directory itself
drwxr-xr-x. 3 aadarsha aadarsha 65 Jun  8 12:07 dira

[aadarsha@labserver ~]$ stat dira
  File: dira
  Size: 65        	Blocks: 0          IO Block: 4096   directory
Device: 253,0	Inode: 25447633    Links: 3
Access: (0755/drwxr-xr-x)  Uid: ( 1000/aadarsha)   Gid: ( 1000/aadarsha)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2026-06-08 12:07:35.289248194 +0545
Modify: 2026-06-08 12:07:19.877766384 +0545
Change: 2026-06-08 12:07:19.877766384 +0545
 Birth: 2026-06-07 15:42:17.367604410 +0545

[aadarsha@labserver ~]$ ls
dir1  dira  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz

[aadarsha@labserver ~]$ file testfile 
testfile: ASCII text

[aadarsha@labserver ~]$ file words.gz 
words.gz: gzip compressed data, was "words", last modified: Mon Jun  8 04:07:53 2026, from Unix, original size modulo 2^32 4953598

[aadarsha@labserver ~]$ file newimpfiles.tar.gz 
newimpfiles.tar.gz: gzip compressed data, from Unix, original size modulo 2^32 6737920

# note:  
 # /etc/skel contains files and directories that are used as templates when creating a new user's home directory. The exact contents depend on the distribution and system configuration.

# cp /etc/skel/.bashrc .     ( copy .bashrc here )

-------------------------------------------------------------------------------------------------

[aadarsha@labserver ~]$ ls
dir1  dira  extracted  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz

# Getting Help 

[aadarsha@labserver ~]$ ls -l --color=auto
total 18488
drwxr-xr-x. 3 aadarsha aadarsha       33 Jun  8 06:47 dir1
drwxr-xr-x. 3 aadarsha aadarsha       65 Jun  8 12:07 dira
drwxr-xr-x. 4 root     root           28 Jun  8 20:16 extracted
-rw-r--r--. 1 root     root     11704320 Jun  8 16:38 impfiles.tar
-rw-r--r--. 1 root     root       784897 Jun  8 16:33 newimpfiles.tar.gz
drwxr-xr-x. 5 aadarsha aadarsha       54 Jun  7 07:24 testcompany
-rw-r--r--. 1 aadarsha aadarsha       56 Jun  7 21:33 testfile
-rw-r--r--. 1 aadarsha aadarsha  4953598 Jun  8 09:52 words
-rw-r--r--. 1 aadarsha aadarsha  1476067 Jun  8 12:09 words.gz

# using man command

[aadarsha@labserver ~]$ man ls

# search text in man page

 # /text
        /text   Search forward
        n       Next match
        N       Previous match
        Space   Next page
        b       Previous page
        q       Quit

  
 # use n: next forward direction  and  N: backward direction

[aadarsha@labserver ~]$ man ls

[aadarsha@labserver ~]$ ls -Sl /etc
total 1048
-rw-r--r--. 1 root root 701707 Nov 29  2023 services
...
...
-rw-r--r--. 1 root root      0 Nov 29  2023 subuid-

[aadarsha@labserver ~]$ man ls

[aadarsha@labserver ~]$ ls -Slr /etc
total 1048
-rw-r--r--. 1 root root      0 Nov 29  2023 subuid-
...
...
-rw-r--r--. 1 root root   6714 Nov 29  2023 protocols
-rw-r--r--. 1 root root   8782 May 26 15:33 kdump.conf
-rw-r--r--. 1 root root   8888 Feb 23 05:45 login.defs
-rw-r--r--. 1 root root  13567 Jun  8 16:27 ld.so.cache
-rw-r--r--. 1 root root 701707 Nov 29  2023 services

[aadarsha@labserver ~]$ ls -S -l -r /etc
total 1048
-rw-r--r--. 1 root root      0 Nov 29  2023 subuid-
-rw-r--r--. 1 root root      0 Nov 29  2023 subgid-
-rw-r--r--. 1 root root      0 Nov 29  2023 motd
...
...
-rw-r--r--. 1 root root 701707 Nov 29  2023 services

# man -k <key word related to task>
# Searches the manual-page database by keyword/description.

[aadarsha@labserver ~]$ man -k copy          # use: apropos copy

cp (1)               - copy files and directories
cpio (1)             - copy files to and from archives
dd (1)               - convert and copy a file
install (1)          - copy files and set attributes
scp (1)              - OpenSSH secure file copy
sg_copy_results (8)  - send SCSI RECEIVE COPY RESULTS command (XCOPY related)
sg_dd (8)            - copy data to and from files and devices, especially SCSI devices
sg_xcopy (8)         - copy data to and from files and devices using SCSI EXTENDED COPY (XCOPY)
sgm_dd (8)           - copy data to and from files and devices, especially SCSI devices
sgp_dd (8)           - copy data to and from files and devices, especially SCSI devices
ssh-copy-id (1)      - use locally available keys to authorise logins on a remote machine
xfs_copy (8)         - copy the contents of an XFS filesystem
xfs_metadump (8)     - copy XFS filesystem metadata to a file
xfs_rtcp (8)         - XFS realtime copy command

[aadarsha@labserver ~]$ man -k "copy file"
cp (1)               - copy files and directories
cpio (1)             - copy files to and from archives
install (1)          - copy files and set attributes

aadarkdk@pop-os:~$ ls -lS /etc/
...

 -l --> long listing format
 -S --> sort by file size, largest first (descending)

aadarkdk@pop-os:~$ ls -Slr /etc         # ls -lS --reverse /etc
...

 - -r --> reverse
```
---
### getting online help:
 - Primary references: vendor documentation, local man pages and official docs,eg: docs.redhat.com
 - Secondary references: tutorials, blogs, community discussions
---