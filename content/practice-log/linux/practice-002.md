---
title: "Practice 002 — Linux Filesystem"
date: 2026-06-06
draft: false
---

### Linux Filesystem

The Linux filesystem hierarchy generally follows the **Filesystem Hierarchy Standard (FHS)**.
Some distributions may organize files differently (e.g., merged `/usr`, `systemd`-based layouts), but the purpose of each directory remains largely the same.

> **"everything is represented as a file interface whenever practical."**

### Terminal Session

```
[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ pwd
/home/aadarsha
```

```
[aadarsha@labserver ~]$ cd /
[aadarsha@labserver /]$ 

# / ---> ( root directory )
```
The top-level directory of the Linux filesystem.
Everything in Linux exists under `/`, including:
- system files
- user files
- devices
- mounted filesystems
- virtual filesystems
Every absolute path starts from `/`.

```
[aadarsha@labserver /]$ ls
afs  boot  etc   lib    media  opt   root  sbin  sys  usr
bin  dev   home  lib64  mnt    proc  run   srv   tmp  var

# /boot ---> contains files required during the boot process, for example: bootloader files (GRUB), kernel configuration, Initial RAM filesystem ('initramfs')

[aadarsha@labserver /]$ ls /boot/
config-6.12.0-228.el10.x86_64
efi
grub2
initramfs-0-rescue-8bc5a82f16be4f5cb41e8587e279ce1e.img
initramfs-6.12.0-228.el10.x86_64.img
initramfs-6.12.0-228.el10.x86_64kdump.img
loader
symvers-6.12.0-228.el10.x86_64.xz
System.map-6.12.0-228.el10.x86_64
vmlinuz-0-rescue-8bc5a82f16be4f5cb41e8587e279ce1e
vmlinuz-6.12.0-228.el10.x86_64
[aadarsha@labserver /]$ 

# /dev  ---> contains device nodes managed by the kernel (usually through undev)
Examples: Hard disks, SSDs, USB devices, Terminals, Random number generators
 
[aadarsha@labserver /]$ ls /dev
autofs           hugepages     rtc0      tty14  tty33  tty52    uhid         vcsu
block            hwrng         sda       tty15  tty34  tty53    uinput       vcsu1
bsg              initctl       sda1      tty16  tty35  tty54    urandom      vcsu2
bus              input         sda2      tty17  tty36  tty55    usbmon0      vcsu3
cdrom            kmsg          sda3      tty18  tty37  tty56    usbmon1      vcsu4
char             log           sg0       tty19  tty38  tty57    usbmon2      vcsu5
console          loop-control  sg1       tty2   tty39  tty58    userfaultfd  vcsu6
core             mapper        shm       tty20  tty4   tty59    vcs          vfio
cpu              mcelog        snapshot  tty21  tty40  tty6     vcs1         vga_arbiter
cpu_dma_latency  mem           snd       tty22  tty41  tty60    vcs2         vhci
cs               mqueue        sr0       tty23  tty42  tty61    vcs3         vhost-net
cuse             net           stderr    tty24  tty43  tty62    vcs4         vhost-vsock
disk             null          stdin     tty25  tty44  tty63    vcs5         vmci
dm-0             nvram         stdout    tty26  tty45  tty7     vcs6         vsock
dm-1             port          tty       tty27  tty46  tty8     vcsa         zero
dma_heap         ppp           tty0      tty28  tty47  tty9     vcsa1
dri              ptmx          tty1      tty29  tty48  ttyS0    vcsa2
fd               pts           tty10     tty3   tty49  ttyS1    vcsa3
full             random        tty11     tty30  tty5   ttyS2    vcsa4
fuse             rfkill        tty12     tty31  tty50  ttyS3    vcsa5
hpet             rtc           tty13     tty32  tty51  udmabuf  vcsa6
[aadarsha@labserver /]$ 

# /etc ---> contains host-specific configuration files, example: Network configuration, SSH configuration, User authentication, DNS configuration, Package manager configuration, configuration files for ( mail server, web server, database server, proxy server, ... )

[aadarsha@labserver /]$ ls /etc
adjtime                  firewalld       magic                     samba
aliases                  fonts           makedumpfile.conf.sample  sasl2
alternatives             fstab           man_db.conf               security
anacrontab               GREP_COLORS     microcode_ctl             selinux
audit                    groff           mke2fs.conf               services
authselect               group           modprobe.d                sestatus.conf
bash_completion.d        group-          modules-load.d            shadow
bashrc                   grub2.cfg       motd                      shadow-
...
[aadarsha@labserver /]$ 

# /home  ---> Home directories of regular users, i.e. all normal users created on the system

[aadarsha@labserver /]$ ls /home/
aadarsha
[aadarsha@labserver /]$ 

# /media ---> Mount point for removable storage automatically mounted by the operating system. used to define mount point. to access storage device (read/write).
Examples: USB flash drives, DVDs, External hard drives

[aadarsha@labserver /]$ ls /media
[aadarsha@labserver /]$ 

# /mnt ---> Temporary mount point intended for manually mounted filesystems, typical mount point to used for network mounting
Examples: Additional disks, ISO images, NFS shares

[aadarsha@labserver /]$ ls /mnt
[aadarsha@labserver /]$ 

# /opt  ---> Optional application software packages. Typically used by third-party vendors.
Examples: /opt/google/, /opt/vmware/, /opt/oracle/

[aadarsha@labserver /]$ ls /opt
[aadarsha@labserver /]$ 

# /proc ---> A virtual filesystem (procfs) created by the Linux kernel.
Files do not physically exist on disk.
Provides runtime information about
- processes
- CPU
- memory
- kernel
- devices
- networking

files that contains the information of the this system. eg: details of the CPU, memory, swap details, hardware details, process id of the process

# /run
Temporary runtime state information.
Examples:
- PID files
- UNIX sockets
- Locks
- Runtime service data
Contents are recreated after every boot.

[aadarsha@labserver /]$ ls /proc
1     1937  22   41   481  579  762  895         devices        kpagecgroup   slabinfo
11    1984  23   42   49   580  763  896         diskstats      kpagecount    softirqs
12    1995  24   43   5    581  784  90          dma            kpageflags    stat
13    1996  242  439  51   582  8    931         driver         loadavg       swaps
14    1997  25   44   52   583  80   96          dynamic_debug  locks         sys
15    2     28   440  53   6    800  97          execdomains    mdstat        sysrq-trigger
16    20    29   441  535  665  81   98          filesystems    meminfo       sysvipc
17    2014  3    442  54   7    814  acpi        fs             misc          thread-self
18    2022  32   443  541  701  815  asound      interrupts     modules       timer_list
1818  2028  33   45   55   709  817  bootconfig  iomem          mounts        tty
1820  2029  34   46   56   75   818  buddyinfo   ioports        mtrr          uptime
1829  2030  35   463  57   754  82   bus         irq            net           version
1866  2031  36   464  574  755  820  cgroups     kallsyms       pagetypeinfo  vmallocinfo
1870  2035  38   466  575  756  84   cmdline     kcore          partitions    vmstat
1871  2039  39   467  576  759  85   consoles    keys           schedstat     zoneinfo
19    21    4    47   577  760  872  cpuinfo     key-users      scsi
1918  216   40   48   578  761  882  crypto      kmsg           self

# In above digits/numbers are process id of the currently running processes
 
# /proc is the runtime that contains has file when the system is running and the files are temporary, where files are cleared on reboot or restart. runtime files like in /run, dynamically created during runtime
[aadarsha@labserver /]$ 

[aadarsha@labserver /]$ cat /proc/meminfo 
MemTotal:        1745836 kB
MemFree:         1283196 kB
MemAvailable:    1379928 kB
Buffers:            4224 kB
Cached:           220764 kB
SwapCached:            0 kB
Active:           190912 kB
Inactive:         115060 kB
...
[aadarsha@labserver /]$ 

# /tmp ---> Temporary files.
Applications store short-lived data here.
Files may be deleted automatically.
Many systems clear `/tmp` during boot or by scheduled cleanup.

[aadarsha@labserver /]$ ls /tmp
systemd-private-955f6b4f4799448883321b61974cd287-chronyd.service-xq1Bov
systemd-private-955f6b4f4799448883321b61974cd287-dbus-broker.service-iaPvYS
systemd-private-955f6b4f4799448883321b61974cd287-irqbalance.service-1VK3Pg
systemd-private-955f6b4f4799448883321b61974cd287-systemd-logind.service-4nT2TT
[aadarsha@labserver /]$ 

# /root ---> Home directory of the root (superuser).
Different from the root directory (`/`).

[aadarsha@labserver /]$ ls /root
ls: cannot open directory '/root': Permission denied
[aadarsha@labserver /]$ su - root
Password: 
Last login: Fri Jun  5 20:48:37 +0545 2026 on pts/0

[root@labserver ~]# cd /root
[root@labserver ~]# ls
anaconda-ks.cfg
[root@labserver ~]# pwd
/root
[root@labserver ~]# whoami 
root
[root@labserver ~]# exit
logout
[aadarsha@labserver /]$ 

# /srv  ---> contains data served by system services
Examples:
/srv/www
/srv/ftp
/srv/git
Not all systems use this directory.

[aadarsha@labserver /]$ 
[aadarsha@labserver /]$ ls /srv
[aadarsha@labserver /]$ 

# /sys  ---> Virtual filesystem (sysfs)
Exports information about
- hardware
- kernel
- drivers
- devices
Also allows certain kernel parameters to be configured.

[aadarsha@labserver /]$ ls /sys
block  bus  class  dev  devices  firmware  fs  hypervisor  kernel  module  power
[aadarsha@labserver /]$ 

# /var ---> Variable data that changes while the system is running.

[aadarsha@labserver /]$ ls /var
adm    crash  empty  games     lib    lock  mail  opt       run    tmp
cache  db     ftp    kerberos  local  log   nis   preserve  spool  yp
[aadarsha@labserver /]$ 
 
[aadarsha@labserver /]$ ls /var/log
anaconda  chrony           dnf.log      hawkey.log  messages  secure   wtmp
audit     cron             dnf.rpm.log  lastlog     private   spooler
btmp      dnf.librepo.log  firewalld    maillog     samba     sssd
[aadarsha@labserver /]$

# /usr ---> Contains user applications, libraries and shared read-only data.
 contains different types of files such as binary, library. and contains the contains the adminstrative commands 

[aadarsha@labserver /]$ ls /usr/sbin/
accessdb                     lvreduce
addpart                      lvremove
adduser                      lvrename
agetty                       lvresize
...
[aadarsha@labserver /]$ 
 
[aadarsha@labserver /]$ ls /usr
bin  games  include  lib  lib64  libexec  local  sbin  share  src  tmp
[aadarsha@labserver /]$ 

# /usr --->  contains different types of files such as binary, library. and contains the contains the adminstrative commands and also non adminstrative commands 

[aadarsha@labserver /]$ ls /usr/bin/
[aadarsha@labserver /]$ 

# /usr/local:
Software installed locally by the system administrator.
Usually unaffected by the package manager.
Examples:
/usr/local/bin
/usr/local/lib

# /bin:
Traditionally contained essential user commands.
On most modern Linux distributions,
`/bin` is a symbolic link to `/usr/bin`
(UsrMerge).

[aadarsha@labserver /]$ ls
afs  boot  etc   lib    media  opt   root  sbin  sys  usr
bin  dev   home  lib64  mnt    proc  run   srv   tmp  var
[aadarsha@labserver /]$ ls -l
total 24
dr-xr-xr-x.   2 root root    6 Apr  2  2025 afs
lrwxrwxrwx.   1 root root    7 Apr  2  2025 bin -> usr/bin
dr-xr-xr-x.   5 root root 4096 May 26 15:40 boot
drwxr-xr-x.  20 root root 3280 Jun  4 21:25 dev
drwxr-xr-x.  82 root root 8192 Jun  4 21:25 etc
drwxr-xr-x.   3 root root   22 May 26 15:36 home
lrwxrwxrwx.   1 root root    7 Apr  2  2025 lib -> usr/lib
lrwxrwxrwx.   1 root root    9 Apr  2  2025 lib64 -> usr/lib64
drwxr-xr-x.   2 root root    6 Apr  2  2025 media
drwxr-xr-x.   2 root root    6 Apr  2  2025 mnt
drwxr-xr-x.   2 root root    6 Apr  2  2025 opt
dr-xr-xr-x. 185 root root    0 Jun  4 21:25 proc
dr-xr-x---.   3 root root  147 May 26 15:44 root
drwxr-xr-x.  32 root root  880 Jun  4 21:25 run
lrwxrwxrwx.   1 root root    8 Apr  2  2025 sbin -> usr/sbin
drwxr-xr-x.   2 root root    6 Apr  2  2025 srv
dr-xr-xr-x.  13 root root    0 Jun  4 21:25 sys
drwxrwxrwt.  10 root root 4096 Jun  4 22:24 tmp
drwxr-xr-x.  12 root root  144 May 26 15:33 usr
drwxr-xr-x.  19 root root 4096 May 26 15:39 var
[aadarsha@labserver /]$ 

# here : bin -> usr/bin,   lib -> usr/lib,  lib64 -> usr/lib64, sbin -> usr/sbin  are soft links 

[aadarsha@labserver /]$ 
[aadarsha@labserver /]$ ls /lib
binfmt.d       firewalld  kdump       modules-load.d  pam.d       sysimage
credstore      firmware   kernel      motd            pcrlock.d   systemd
debug          games      locale      motd.d          python3.12  sysusers.d
dracut         grub       modprobe.d  NetworkManager  rpm         tmpfiles.d
environment.d  kbd        modules     os-release      sysctl.d    udev
[aadarsha@labserver /]$ 

[aadarsha@labserver /]$ ls /lib64
...
...
[aadarsha@labserver /]$ 
```
### Virtual Filesystems
These filesystems exist only in memory.
Examples:
- /proc
- /sys
- /run

Generated dynamically by the kernel.

Nothing here is permanently stored on disk.
