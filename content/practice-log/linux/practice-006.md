---
title: "Practice 006 — Linux Basic Information & Getting Help "
date: 2026-06-10
draft: false
---

### Terminal Session

#### Getting some information

```
# Displays the system's network hostname.

[aadarsha@labserver ~]$ hostname
labserver
[aadarsha@labserver ~]$

[aadarsha@labserver ~]$ cat /etc/redhat-release 
CentOS Stream release 10 (Coughlan)
[aadarsha@labserver ~]$ 

# Show comprehensive system information including OS version, kernel, and virtualization details.

[root@labserver ~]# hostnamectl
 Static hostname: labserver
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 6051312a1e194d8ga01bc1948bcb0427
         Boot ID: 9299b2f276724g30b3efeb72835f0131
    Product UUID: 28702045-a8d5-de9b-a9ca-a7526d5a6ece
  Virtualization: oracle
Operating System: CentOS Stream 10 (Coughlan)                    
     CPE OS Name: cpe:/o:centos:centos:10
          Kernel: Linux 6.12.0-228.el10.x86_64
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
 Hardware Serial: VirtualBox-45206028-d5a9-7bde-a5ca-a7526d7a6ece
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 19y 7month 2w 2d                               
[root@labserver ~]# 

# Shows only kernel version

[aadarsha@labserver ~]$ uname -r
6.12.0-228.el10.x86_64
[aadarsha@labserver ~]$

# Show complete system information

[aadarsha@labserver ~]$ uname -a
Linux labserver 6.12.0-228.el10.x86_64 #1 SMP PREEMPT_DYNAMIC Thu May 14 14:51:31 UTC 2026 x86_64 GNU/Linux
[aadarsha@labserver ~]$ 

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

#  Shows current username

[aadarsha@labserver ~]$ whoami
aadarsha
[aadarsha@labserver ~]$ 

#  Shows user ID, group ID, and group memberships

[aadarsha@labserver ~]$ id
uid=1000(aadarsha) gid=1000(aadarsha) groups=1000(aadarsha),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[aadarsha@labserver ~]$ 

# Show current directory

[aadarsha@labserver ~]$ pwd
/home/aadarsha
[aadarsha@labserver ~]$ 

# Detailed memory statistics from the kernel

[aadarsha@labserver ~]$ cat /proc/meminfo | head
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
[aadarsha@labserver ~]$ 

# Number of processing units

[aadarsha@labserver ~]$ nproc
2
[aadarsha@labserver ~]$ 

# Shows block devices (disks and partitions) in a tree format
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
[aadarsha@labserver ~]$

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
[aadarsha@labserver ~]$ 



# Memory Information

[aadarsha@labserver ~]$ free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       363Mi       1.1Gi       4.8Mi       308Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[aadarsha@labserver ~]$ 
```

| Field | Meaning |
|---------|---------|
| total | Total installed RAM |
| used | Memory currently used |
| free | Completely unused memory |
| shared | Shared memory |
| buff/cache | Memory used by kernel cache |
| available | Memory available for new processes |
| Swap | Swap partition usage |


```
# Disk Information

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
[aadarsha@labserver ~]$ 

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
# Detailed CPU architecture information.

[aadarsha@labserver ~]$ lscpu
Architecture:                x86_64
  CPU op-mode(s):            32-bit, 64-bit
  Address sizes:             39 bits physical, 48 bits virtual
  Byte Order:                Little Endian
CPU(s):                      2
  On-line CPU(s) list:       0,1
Vendor ID:                   GenuineIntel
  Model name:                11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
    CPU family:              6
    Model:                   140
    Thread(s) per core:      1
    Core(s) per socket:      2
    Socket(s):               1
    Stepping:                1
    BogoMIPS:                4838.46
    Flags:                   fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology n
                             onstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefet
                             ch fsgsbase bmi1 avx2 bmi2 invpcid rdseed adx clflushopt sha_ni arat md_clear flush_l1d arch_capabilities
Virtualization features:     
  Hypervisor vendor:         KVM
  Virtualization type:       full
Caches (sum of all):         
  L1d:                       96 KiB (2 instances)
  L1i:                       64 KiB (2 instances)
  L2:                        2.5 MiB (2 instances)
  L3:                        16 MiB (2 instances)
NUMA:                        
  NUMA node(s):              1
  NUMA node0 CPU(s):         0,1
Vulnerabilities:             
  Gather data sampling:      Unknown: Dependent on hypervisor status
  Indirect target selection: Mitigation; Aligned branch/return thunks
  Itlb multihit:             Not affected
  L1tf:                      Not affected
  Mds:                       Not affected
  Meltdown:                  Not affected
  Mmio stale data:           Not affected
  Old microcode:             Not affected
  Reg file data sampling:    Not affected
  Retbleed:                  Not affected
  Spec rstack overflow:      Not affected
  Spec store bypass:         Vulnerable
  Spectre v1:                Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:                Mitigation; Retpolines; STIBP disabled; RSB filling; PBRSB-eIBRS Not affected; BHI Retpoline
  Srbds:                     Not affected
  Tsa:                       Not affected
  Tsx async abort:           Not affected
  Vmscape:                   Not affected
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[aadarsha@labserver ~]$ 

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
[aadarsha@labserver ~]$ 

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
[aadarsha@labserver ~]$

[aadarsha@labserver ~]$ ls -lh dir1/
total 0
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  7 15:41 dir2
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 15:48 numfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -lh dira
total 4.8M
drwxr-xr-x. 3 aadarsha aadarsha   18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha    0 Jun  7 16:10 letfile
-rw-r--r--. 1 aadarsha aadarsha   73 Jun  8 08:47 myfile_hard
-rw-r--r--. 1 aadarsha aadarsha 4.8M Jun  8 09:52 words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -ld dira
drwxr-xr-x. 3 aadarsha aadarsha 65 Jun  8 12:07 dira
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -ld dira                     # to see the details of the directory itself
drwxr-xr-x. 3 aadarsha aadarsha 65 Jun  8 12:07 dira
[aadarsha@labserver ~]$ 

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
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ file testfile 
testfile: ASCII text
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ file words.gz 
words.gz: gzip compressed data, was "words", last modified: Mon Jun  8 04:07:53 2026, from Unix, original size modulo 2^32 4953598
[aadarsha@labserver ~]$

[aadarsha@labserver ~]$ file newimpfiles.tar.gz 
newimpfiles.tar.gz: gzip compressed data, from Unix, original size modulo 2^32 6737920
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ 
 
# note:  /etc/skel/ --> contains .bashrc ( main location which contain .bashrc, which comes in the user's home directory when the user is created )

# cp /etc/skel/.bashrc .     ( copy .bashrc here )


-------------------------------------------------------------------------------------------------

[aadarsha@labserver ~]$ ls
dir1  dira  extracted  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -l /etc/
total 1048
-rw-r--r--. 1 root root     16 May 26 15:36 adjtime
-rw-r--r--. 1 root root   1529 Nov 29  2023 aliases
drwxr-xr-x. 2 root root   4096 May 26 15:34 alternatives
-rw-r--r--. 1 root root    541 Jan 14 05:45 anacrontab
drwxr-x---. 4 root root    100 May 26 15:33 audit
drwxr-xr-x. 3 root root   4096 May 26 15:34 authselect
drwxr-xr-x. 2 root root     38 May 26 15:34 bash_completion.d
-rw-r--r--. 1 root root   2777 Jun  7 19:59 bashrc
-rw-r--r--. 1 root root    535 Oct 29  2024 bindresvport.blacklist
drwxr-xr-x. 2 root root      6 May 12 05:45 binfmt.d
-rw-r--r--. 1 root root     36 Apr  7 05:45 centos-release
-rw-r--r--. 1 root root   1382 Nov 20  2025 chrony.conf
drwxr-xr-x. 2 root root     26 May 26 15:33 cifs-utils
drwx------. 2 root root      6 May 12 05:45 credstore
drwx------. 2 root root      6 May 12 05:45 credstore.encrypted
drwxr-xr-x. 2 root root     21 May 26 15:33 cron.d
drwxr-xr-x. 2 root root      6 Oct 29  2024 cron.daily
-rw-r--r--. 1 root root      0 Jan 14 05:45 cron.deny
drwxr-xr-x. 2 root root     22 Oct 29  2024 cron.hourly
drwxr-xr-x. 2 root root      6 Oct 29  2024 cron.monthly
-rw-r--r--. 1 root root    451 Oct 29  2024 crontab
drwxr-xr-x. 2 root root      6 Oct 29  2024 cron.weekly
drwxr-xr-x. 6 root root     81 May 26 15:33 crypto-policies
-rw-------. 1 root root      0 May 26 15:32 crypttab
-rw-r--r--. 1 root root   1401 Nov 29  2023 csh.cshrc
-rw-r--r--. 1 root root   1569 Nov 29  2023 csh.login
drwxr-xr-x. 4 root root     78 May 26 15:33 dbus-1
drwxr-xr-x. 3 root root     16 May 26 15:34 dconf
drwxr-xr-x. 2 root root     52 May 26 15:33 debuginfod
drwxr-xr-x. 2 root root     33 May 26 15:36 default
drwxr-xr-x. 2 root root     23 May 26 15:33 depmod.d
drwxr-xr-x. 3 root root     24 May 26 15:33 dhcp
-rw-r--r--. 1 root root   5923 Jan 15 05:45 DIR_COLORS
-rw-r--r--. 1 root root   6005 Jan 15 05:45 DIR_COLORS.lightbgcolor
drwxr-xr-x. 9 root root    163 May 26 15:33 dnf
-rw-r--r--. 1 root root    117 Jan 30 05:45 dracut.conf
drwxr-xr-x. 2 root root      6 Jan 30 05:45 dracut.conf.d
-rw-r--r--. 1 root root      0 Apr  8  2025 environment
-rw-r--r--. 1 root root   1362 Nov 29  2023 ethertypes
-rw-r--r--. 1 root root      0 Nov 29  2023 exports
-rw-r--r--. 1 root root     66 Nov 29  2023 filesystems
drwxr-x---. 8 root root    119 May 26 15:33 firewalld
drwxr-xr-x. 3 root root     20 May 26 15:33 fonts
-rw-r--r--. 1 root root    615 May 26 15:32 fstab
-rw-r--r--. 1 root root     94 Oct 29  2024 GREP_COLORS
drwxr-xr-x. 4 root root     40 May 26 15:33 groff
-rw-r--r--. 1 root root    518 May 26 15:36 group
-rw-r--r--. 1 root root    493 May 26 15:36 group-
lrwxrwxrwx. 1 root root     22 Feb 10 05:45 grub2.cfg -> ../boot/grub2/grub.cfg
drwx------. 2 root root   4096 May 26 15:33 grub.d
----------. 1 root root    412 May 26 15:36 gshadow
----------. 1 root root    391 May 26 15:36 gshadow-
drwxr-xr-x. 3 root root     20 May 26 15:33 gss
-rw-r--r--. 1 root root      9 Nov 29  2023 host.conf
-rw-r--r--. 1 root root     10 May 26 15:36 hostname
-rw-r--r--. 1 root root    384 Nov 29  2023 hosts
-rw-r--r--. 1 root root    490 May 12 05:45 inittab
-rw-r--r--. 1 root root    943 Nov 29  2023 inputrc
drwxr-xr-x. 2 root root     20 May 26 15:33 iproute2
-rw-r--r--. 1 root root     20 Apr  7 05:45 issue
drwxr-xr-x. 2 root root      6 Apr  7 05:45 issue.d
-rw-r--r--. 1 root root     19 Apr  7 05:45 issue.net
drwxr-xr-x. 4 root root     33 May 26 15:33 kdump
-rw-r--r--. 1 root root   8782 May 26 15:33 kdump.conf
drwxr-xr-x. 3 root root     38 May 26 15:36 kernel
drwxr-xr-x. 3 root root     17 May 26 15:33 keys
drwxr-xr-x. 2 root root      6 Oct 29  2024 keyutils
-rw-r--r--. 1 root root    880 Apr 28 05:45 krb5.conf
drwxr-xr-x. 2 root root     55 May 26 15:33 krb5.conf.d
-rw-r--r--. 1 root root  13567 Jun  8 16:27 ld.so.cache
-rw-r--r--. 1 root root     28 Feb  1  2024 ld.so.conf
drwxr-xr-x. 2 root root      6 May 11 05:45 ld.so.conf.d
-rw-r-----. 1 root root    191 Jan  6 05:45 libaudit.conf
drwxr-xr-x. 2 root root     35 May 26 15:33 libnl
drwxr-xr-x. 2 root root     62 May 26 15:33 libssh
-rw-r--r--. 1 root root     19 May 26 15:36 locale.conf
lrwxrwxrwx. 1 root root     36 May 26 15:36 localtime -> ../usr/share/zoneinfo/Asia/Kathmandu
-rw-r--r--. 1 root root   8888 Feb 23 05:45 login.defs
-rw-r--r--. 1 root root    493 Jul  5  2021 logrotate.conf
drwxr-xr-x. 2 root root    128 May 26 15:33 logrotate.d
drwxr-xr-x. 7 root root    115 May 26 15:33 lvm
-r--r--r--. 1 root root     33 May 26 15:33 machine-id
-rw-r--r--. 1 root root    110 May 11 05:45 magic
-rw-r--r--. 1 root root   5122 Nov  3  2025 makedumpfile.conf.sample
-rw-r--r--. 1 root root   5242 Jun 10  2025 man_db.conf
drwxr-xr-x. 3 root root     32 May 26 15:33 microcode_ctl
-rw-r--r--. 1 root root    782 Nov 19  2025 mke2fs.conf
drwxr-xr-x. 2 root root   4096 May 26 15:33 modprobe.d
drwxr-xr-x. 2 root root      6 May 12 05:45 modules-load.d
-rw-r--r--. 1 root root      0 Nov 29  2023 motd
drwxr-xr-x. 2 root root      6 Apr  8  2025 motd.d
lrwxrwxrwx. 1 root root     19 Mar  4 05:45 mtab -> ../proc/self/mounts
-rw-r--r--. 1 root root    767 Oct 29  2024 netconfig
drwxr-xr-x. 7 root root    134 May 26 15:33 NetworkManager
-rw-r--r--. 1 root root     58 Nov 29  2023 networks
drwx------. 3 root root     66 May 26 15:33 nftables
lrwxrwxrwx. 1 root root     29 May 26 15:34 nsswitch.conf -> /etc/authselect/nsswitch.conf
drwxr-xr-x. 2 root root     35 May 26 15:32 nvme
drwxr-xr-x. 3 root root     36 May 26 15:33 openldap
drwxr-xr-x. 2 root root      6 Apr  2  2025 opt
lrwxrwxrwx. 1 root root     21 Apr  7 05:45 os-release -> ../usr/lib/os-release
drwxr-xr-x. 2 root root   4096 May 26 15:34 pam.d
-rw-r--r--. 1 root root   1080 May 26 15:36 passwd
-rw-r--r--. 1 root root   1018 May 26 15:36 passwd-
drwxr-xr-x. 3 root root     21 May 26 15:33 pkcs11
drwxr-xr-x. 3 root root     27 Jun  7 18:08 pkgconfig
drwxr-xr-x. 7 root root     75 May 26 15:33 pki
drwxr-xr-x. 5 root root     52 May 26 15:33 pm
drwxr-xr-x. 2 root root      6 Oct 29  2024 popt.d
-rw-r--r--. 1 root root    233 Nov 29  2023 printcap
-rw-r--r--. 1 root root   1982 Nov 29  2023 profile
drwxr-xr-x. 2 root root   4096 Jun  7 18:08 profile.d
-rw-r--r--. 1 root root   6714 Nov 29  2023 protocols
drwxr-xr-x. 3 root root     36 May 26 15:33 rc.d
lrwxrwxrwx. 1 root root     13 May 12 05:45 rc.local -> rc.d/rc.local
lrwxrwxrwx. 1 root root     14 Apr  7 05:45 redhat-release -> centos-release
-rw-r--r--. 1 root root   1787 Oct 29  2024 request-key.conf
drwxr-xr-x. 2 root root      6 Oct 29  2024 request-key.d
-rw-r--r--. 1 root root     57 Jun  8 15:59 resolv.conf
-rw-r--r--. 1 root root   1634 Jan 31  2024 rpc
drwxr-xr-x. 2 root root      6 Feb  5 05:45 rpm
-rw-r--r--. 1 root root   3300 Apr 29 05:45 rsyslog.conf
drwxr-xr-x. 2 root root      6 Apr 29 05:45 rsyslog.d
drwxr-xr-x. 2 root root     35 May 26 15:33 rwtab.d
drwxr-xr-x. 2 root root     61 May 26 15:33 samba
drwxr-xr-x. 2 root root      6 Dec  4  2024 sasl2
drwxr-xr-x. 5 root root   4096 May 26 15:33 security
drwxr-xr-x. 3 root root     57 May 26 15:33 selinux
-rw-r--r--. 1 root root 701707 Nov 29  2023 services
-rw-r--r--. 1 root root    216 Apr  7 05:45 sestatus.conf
----------. 1 root root    669 May 26 15:37 shadow
----------. 1 root root    674 May 26 15:37 shadow-
-rw-r--r--. 1 root root     44 Nov 29  2023 shells
drwxr-xr-x. 2 root root     62 May 26 15:33 skel
drwxr-xr-x. 4 root root   4096 May 26 15:39 ssh
drwxr-xr-x. 2 root root     91 May 26 15:33 ssl
drwxr-x---. 4 root sssd     31 May 26 15:33 sssd
drwxr-xr-x. 2 root root      6 Apr  2  2025 statetab.d
-rw-r--r--. 1 root root     22 May 26 15:36 subgid
-rw-r--r--. 1 root root      0 Nov 29  2023 subgid-
-rw-r--r--. 1 root root     22 May 26 15:36 subuid
-rw-r--r--. 1 root root      0 Nov 29  2023 subuid-
-rw-r-----. 1 root root   4356 Apr 10 05:45 sudo.conf
-r--r-----. 1 root root   4328 Apr 10 05:45 sudoers
drwxr-x---. 2 root root      6 Apr 10 05:45 sudoers.d
-rw-r-----. 1 root root   3181 Apr 10 05:45 sudo-ldap.conf
drwxr-xr-x. 2 root root   4096 May 26 15:36 sysconfig
-rw-r--r--. 1 root root    449 May 12 05:45 sysctl.conf
drwxr-xr-x. 2 root root     28 May 26 15:33 sysctl.d
drwxr-xr-x. 5 root root     47 May 26 15:33 systemd
lrwxrwxrwx. 1 root root     14 Apr  7 05:45 system-release -> centos-release
-rw-r--r--. 1 root root     24 Apr  7 05:45 system-release-cpe
drwxr-xr-x. 2 root root      6 Mar 24 05:45 terminfo
drwxr-xr-x. 2 root root      6 May 12 05:45 tmpfiles.d
drwxr-xr-x. 3 root root     51 May 26 15:33 tpm2-tss
drwxr-xr-x. 4 root root     51 May 26 15:39 udev
-rw-r--r--. 1 root root     28 May 26 15:36 vconsole.conf
-rw-r--r--. 1 root root   4017 May 27 05:45 vimrc
-rw-r--r--. 1 root root   1183 May 27 05:45 virc
drwxr-xr-x. 6 root root     70 May 26 15:36 X11
-rw-r--r--. 1 root root    817 Oct 29  2024 xattr.conf
drwxr-xr-x. 4 root root     38 May 26 15:33 xdg
drwxr-xr-x. 2 root root     57 May 26 15:33 yum
lrwxrwxrwx. 1 root root     12 Mar 25 05:45 yum.conf -> dnf/dnf.conf
drwxr-xr-x. 2 root root     51 Apr  7 05:45 yum.repos.d
[aadarsha@labserver ~]$ 


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
[aadarsha@labserver ~]$ 

# using man command

[aadarsha@labserver ~]$ man ls

# search text in man page

# /text
  
# use n: next forward direction   and  N: backward direction

[aadarsha@labserver ~]$ man ls

[aadarsha@labserver ~]$ man ls

[aadarsha@labserver ~]$ ls -Sl /etc
total 1048
-rw-r--r--. 1 root root 701707 Nov 29  2023 services
...
...
-rw-r--r--. 1 root root      0 Nov 29  2023 subuid-
[aadarsha@labserver ~]$ 

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
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -S -l -r /etc
total 1048
-rw-r--r--. 1 root root      0 Nov 29  2023 subuid-
-rw-r--r--. 1 root root      0 Nov 29  2023 subgid-
-rw-r--r--. 1 root root      0 Nov 29  2023 motd
...
...
-rw-r--r--. 1 root root 701707 Nov 29  2023 services
[aadarsha@labserver ~]$ 

# man -k <key word related to task>

[aadarsha@labserver ~]$ man -k copy
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
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ man -k "copy file"
cp (1)               - copy files and directories
cpio (1)             - copy files to and from archives
install (1)          - copy files and set attributes
[aadarsha@labserver ~]$ 


# practice:  how to sort the output of ls -l /etc/   in descending order of files size

# getting online docs:
# docs.redhat.com
```