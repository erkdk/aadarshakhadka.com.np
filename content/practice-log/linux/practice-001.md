---
title: "Practice 001 — Basic Shell Commands"
date: 2026-06-05
draft: false
---

### Terminal Session
```
# General Syntax of Commands

# command_name [options] <arguments>

[aadarsha@labserver ~]$ touch file1 file2 file3
[aadarsha@labserver ~]$ ls
file1  file2  file3

[aadarsha@labserver ~]$ pwd
/home/aadarsha
[aadarsha@labserver ~]$ ls /
afs  boot  etc   lib    media  opt   root  sbin  sys  usr
bin  dev   home  lib64  mnt    proc  run   srv   tmp  var

[aadarsha@labserver ~]$ ls /etc/
adjtime                  firewalld       magic                     samba
aliases                  fonts           makedumpfile.conf.sample  sasl2
alternatives             fstab           man_db.conf               security
anacrontab               GREP_COLORS     microcode_ctl             selinux
audit                    groff           mke2fs.conf               services
authselect               group           modprobe.d                sestatus.conf
bash_completion.d        group-          modules-load.d            shadow
bashrc                   grub2.cfg       motd                      shadow-
bindresvport.blacklist   grub.d          motd.d                    shells
binfmt.d                 gshadow         mtab                      skel
centos-release           gshadow-        netconfig                 ssh
chrony.conf              gss             NetworkManager            ssl
cifs-utils               host.conf       networks                  sssd
credstore                hostname        nftables                  statetab.d
credstore.encrypted      hosts           nsswitch.conf             subgid
cron.d                   inittab         nvme                      subgid-
cron.daily               inputrc         openldap                  subuid
cron.deny                iproute2        opt                       subuid-
cron.hourly              issue           os-release                sudo.conf
cron.monthly             issue.d         pam.d                     sudoers
crontab                  issue.net       passwd                    sudoers.d
cron.weekly              kdump           passwd-                   sudo-ldap.conf
crypto-policies          kdump.conf      pkcs11                    sysconfig
crypttab                 kernel          pki                       sysctl.conf
csh.cshrc                keys            pm                        sysctl.d
csh.login                keyutils        popt.d                    systemd
dbus-1                   krb5.conf       printcap                  system-release
dconf                    krb5.conf.d     profile                   system-release-cpe
debuginfod               ld.so.cache     profile.d                 terminfo
default                  ld.so.conf      protocols                 tmpfiles.d
depmod.d                 ld.so.conf.d    rc.d                      tpm2-tss
dhcp                     libaudit.conf   rc.local                  udev
DIR_COLORS               libnl           redhat-release            vconsole.conf
DIR_COLORS.lightbgcolor  libssh          request-key.conf          virc
dnf                      locale.conf     request-key.d             X11
dracut.conf              localtime       resolv.conf               xattr.conf
dracut.conf.d            login.defs      rpc                       xdg
environment              logrotate.conf  rpm                       yum
ethertypes               logrotate.d     rsyslog.conf              yum.conf
exports                  lvm             rsyslog.d                 yum.repos.d
filesystems              machine-id      rwtab.d

[aadarsha@labserver ~]$ ls -l /etc
total 1044
-rw-r--r--. 1 root root     16 May 26 15:36 adjtime
-rw-r--r--. 1 root root   1529 Nov 29  2023 aliases
drwxr-xr-x. 2 root root   4096 May 26 15:34 alternatives
-rw-r--r--. 1 root root    541 Jan 14 05:45 anacrontab
drwxr-x---. 4 root root    100 May 26 15:33 audit
drwxr-xr-x. 3 root root   4096 May 26 15:34 authselect
drwxr-xr-x. 2 root root     38 May 26 15:34 bash_completion.d
-rw-r--r--. 1 root root   2709 Nov 29  2023 bashrc
...
...
lrwxrwxrwx. 1 root root     12 Mar 25 05:45 yum.conf -> dnf/dnf.conf
drwxr-xr-x. 2 root root     51 Apr  7 05:45 yum.repos.d

[aadarsha@labserver ~]$ ls -l
total 0
-rw-r--r--. 1 aadarsha aadarsha 0 Jun  4 21:29 file1
-rw-r--r--. 1 aadarsha aadarsha 0 Jun  4 21:29 file2
-rw-r--r--. 1 aadarsha aadarsha 0 Jun  4 21:29 file3
[aadarsha@labserver ~]$ 
 
[aadarsha@labserver ~]$ whoami
aadarsha
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ date
Thu Jun  4 09:33:38 PM +0545 2026

[aadarsha@labserver ~]$ cal
      June 2026     
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30            
                    
[aadarsha@labserver ~]$ ls
file1  file2  file3
[aadarsha@labserver ~]$ mkdir dir1 dir2 dir3

[aadarsha@labserver ~]$ ls
dir1  dir2  dir3  file1  file2  file3

# running multiple commands at a time

[aadarsha@labserver ~]$ date; cal; ls
Thu Jun  4 09:34:10 PM +0545 2026
      June 2026     
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30            
                    
dir1  dir2  dir3  file1  file2  file3

[aadarsha@labserver ~]$ date && cal && ls
Thu Jun  4 09:34:31 PM +0545 2026
      June 2026     
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30            
                    
dir1  dir2  dir3  file1  file2  file3
```
Note:
```
# in case of ; --> each command runs independently
# in case of && --> if previous command doesn't run successfully then the following command doesn't run
```
```
[aadarsha@labserver ~]$ date && call && ls && whoami
Thu Jun  4 09:36:56 PM +0545 2026
-bash: call: command not found


# check success or failure status of the last run command or script
[aadarsha@labserver ~]$ echo $?
0

# if o/p of "echo $?" --> 0, the command or script run successfully
# if o/p of "echo $?" --> other than 0, the command or script don't run successfully

[aadarsha@labserver ~]$ who
aadarsha tty1         2026-06-04 21:25
aadarsha pts/0        2026-06-04 21:26 (192.168.254.152)
 
[aadarsha@labserver ~]$ wrongcommand
-bash: wrongcommand: command not found
[aadarsha@labserver ~]$ echo $?
127
[aadarsha@labserver ~]$ 
```