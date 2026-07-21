---
title: "Practice 009 —  File Search, Redirection & Output Control "
date: 2026-06-13
draft: false
---

### Terminal Session

```

# File Search, Redirection & Output Control

# Standard Input/Output Devices

# Keyboard --> Standard Input device (ch 0)
# Terminal Window --> Standard Output device (ch 1)
# Terminal Window --> Standard Error device (ch 2)

# Re-direction operators
# > filename --> redirects the standard output to the given file (ovewrite)

# >> filename --> appends the standard output to the given file (no ovewrite)

# 2> filename --> appends the standard error to the given file ( ovewrite)

# 2>> filename --> appends the standard error to the given file ( no ovewrite)

# 2> filename --> redirects the standard error to the given file ( ovewrite)

# something left above see recording jun 12

[aadarsha@labserver ~]$ ls
cut_result  extracted         passwd           sort_result   testfile     words
dir1        multiple_changes  passwd_YESLOGIN  testcompany   testthis
dira        newfile1          secret_data      testdatafile  uniq_result
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

# This above output is standard output on terminal window

[aadarsha@labserver ~]$ grep bash -r /etc
grep: /etc/crypttab: Permission denied
...
grep: /etc/sudo-ldap.conf: Permission denied
grep: /etc/udev/hwdb.bin: binary file matches
grep: /etc/NetworkManager/system-connections/enp0s3.nmconnection: Permission denied
grep: /etc/sssd: Permission denied
[aadarsha@labserver ~]$ 

# permission denied, error message, .... ---> standard error

# results ---> standard output

[aadarsha@labserver ~]$ # grep bash -r /etc > bash_output
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
cut_result  extracted         passwd           sort_result   testfile     words
dir1        multiple_changes  passwd_YESLOGIN  testcompany   testthis
dira        newfile1          secret_data      testdatafile  uniq_result
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ rm -r cut_result extracted/ passwd sort_result multiple_changes passwd testthis secret_data testdatafile uniq_result 
...
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash -r /etc > bash_output
grep: /etc/crypttab: Permission denied
...
grep: /etc/sssd: Permission denied
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
bash_output  dir1  dira  extracted  newfile1  passwd_YESLOGIN  testcompany  testfile  words
[aadarsha@labserver ~]$
 
[aadarsha@labserver ~]$ cat bash_output 
/etc/profile.d/which2.sh:# Initialization script for bash, sh, mksh and ksh
/etc/profile.d/which2.sh:bash|sh)
...
/etc/crontab:SHELL=/bin/bash
/etc/sestatus.conf:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash -r /etc 2>bash_error
/etc/profile.d/which2.sh:# Initialization script for bash, sh, mksh and ksh
/etc/profile.d/which2.sh:bash|sh)
/etc/profile.d/bash_completion.sh:# Check for interactive bash and that we haven't already been sourced.
...
/etc/sestatus.conf:/bin/bash
[aadarsha@labserver ~]$ 

# only error exists in bash_error

[aadarsha@labserver ~]$ ls
bash_error   dir1  extracted  passwd_YESLOGIN  testfile
bash_output  dira  newfile1   testcompany      words
[aadarsha@labserver ~]$ 

# only output exists in bash_output

[aadarsha@labserver ~]$ cat bash_error 
grep: /etc/crypttab: Permission denied
...
grep: /etc/udev/hwdb.bin: binary file matches
grep: /etc/NetworkManager/system-connections/enp0s3.nmconnection: Permission denied
grep: /etc/sssd: Permission denied
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash -r /etc >output 2>error

# output in output file and error in error file

[aadarsha@labserver ~]$ ls
bash_error   dir1  error      newfile1  passwd_YESLOGIN  testfile
bash_output  dira  extracted  output    testcompany      words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash -r /etc >all_output_error 2>&1

[aadarsha@labserver ~]$ ls
all_output_error  bash_output  dira   extracted  output           testcompany  words
bash_error        dir1         error  newfile1   passwd_YESLOGIN  testfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi all_output_error 

[aadarsha@labserver ~]$ cat all_output_error 
grep: /etc/crypttab: Permission denied
...
grep: /etc/udev/hwdb.bin: binary file matches
grep: /etc/NetworkManager/system-connections/enp0s3.nmconnection: Permission denied
grep: /etc/sssd: Permission denied
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 7 -d : /etc/passwd | sort | uniq -c
      2 /bin/bash
      1 /bin/sync
      1 /sbin/halt
      3 /sbin/nologin
      1 /sbin/shutdown
     13 /usr/sbin/nologin
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 7 -d : /etc/passwd | tee cutout | sort | tee sort_out | uniq -c | tee uniq_out
      2 /bin/bash
      1 /bin/sync
      1 /sbin/halt
      3 /sbin/nologin
      1 /sbin/shutdown
     13 /usr/sbin/nologin
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
all_output_error  cutout  error      output           testcompany  words
bash_error        dir1    extracted  passwd_YESLOGIN  testfile
bash_output       dira    newfile1   sort_out         uniq_out
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
all_output_error  cutout  error      output           testcompany  words
bash_error        dir1    extracted  passwd_YESLOGIN  testfile
bash_output       dira    newfile1   sort_out         uniq_out
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi imp_output

[aadarsha@labserver ~]$ cat imp_output 
this file contains the imp data.....
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash -r /etc >imp_output 
grep: /etc/crypttab: Permission denied
...
grep: /etc/NetworkManager/system-connections/enp0s3.nmconnection: Permission denied
grep: /etc/sssd: Permission denied
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat imp_output 
/etc/profile.d/which2.sh:# Initialization script for bash, sh, mksh and ksh
...
/etc/dhcp/dhclient.d/chrony.sh:#!/usr/bin/bash
/etc/crontab:SHELL=/bin/bash
/etc/sestatus.conf:/bin/bash
[aadarsha@labserver ~]$ 

# file is overwritten

[aadarsha@labserver ~]$ vi new_output

[aadarsha@labserver ~]$ cat new
cat: new: No such file or directory
[aadarsha@labserver ~]$ cat new_output 
this is important data file which is new....

[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash -r /etc/passwd >> new_output 

[aadarsha@labserver ~]$ cat new_output 
this is important data file which is new....

root:x:0:0:Super User:/root:/bin/bash
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash

# appended on previous file

# look the recording about lsattr and immutable and prevent overwrite

[aadarsha@labserver ~]$ # chattr +a <filename>

# to destroy error
 
[aadarsha@labserver ~]$ grep bash -r /etc 2>/dev/null 
/etc/profile.d/which2.sh:# Initialization script for bash, sh, mksh and ksh
...
/etc/dhcp/dhclient.d/chrony.sh:#!/usr/bin/bash
/etc/crontab:SHELL=/bin/bash
/etc/sestatus.conf:/bin/bash
[aadarsha@labserver ~]$ 

# all errors are destroyed or removed

# tr - Character Translator

[aadarsha@labserver ~]$ ls
all_output_error  cutout  error       newfile1    passwd_YESLOGIN  testfile
bash_error        dir1    extracted   new_output  sort_out         uniq_out
bash_output       dira    imp_output  output      testcompany      words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat output 
/etc/profile.d/which2.sh:# Initialization script for bash, sh, mksh and ksh
...
/etc/selinux/targeted/contexts/files/file_contexts:/usr/bin/bash2	--	system_u:object_r:shell_exec_t:s0
/etc/cron.d/0hourly:SHELL=/bin/bash
/etc/dhcp/dhclient.d/chrony.sh:#!/usr/bin/bash
/etc/crontab:SHELL=/bin/bash
/etc/sestatus.conf:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
all_output_error  cutout  error       newfile1    passwd_YESLOGIN  testfile
bash_error        dir1    extracted   new_output  sort_out         uniq_out
bash_output       dira    imp_output  output      testcompany      words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ tr 'a-z' 'A-Z' <cutout 
/BIN/BASH
/USR/SBIN/NOLOGIN
...
/USR/SBIN/NOLOGIN
/BIN/BASH
[aadarsha@labserver ~]$ 
 
[aadarsha@labserver ~]$ cat cutout 
/bin/bash
/usr/sbin/nologin
...
/usr/sbin/nologin
/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ tr 'a-z' 'A-Z' <cutout >CUTOUT

[aadarsha@labserver ~]$ ls
all_output_error  cutout  dira       imp_output  output           testcompany  words
bash_error        CUTOUT  error      newfile1    passwd_YESLOGIN  testfile
bash_output       dir1    extracted  new_output  sort_out         uniq_out
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat CUTOUT 
/BIN/BASH
/USR/SBIN/NOLOGIN
...
/BIN/BASH
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ 

# redirection can be used for any commands

# Searching Files using 'find' Command
   	# find [path] [option] <argument>
   	# options:
   		# -atime <+N/-N/N>  --> performs access-time bases search ( N --> Day)
   		# -mtime <+N/-N/N>  --> performs modification-time bases search ( N --> Day)
   		# -ntime <+N/-N/N>  --> performs name-based search ( case sensitive )
   		# -intime <+N/-N/N>  --> performs name-based search ( case in-sensitive )
   		# -name <name>  --> performs name-based search ( case sensitive )
   		# -iname <name>  --> performs name-based search ( case insensitive )
   		# -size <+N/-N/N>  --> performs size-based search (N --> KB, MB, GB, .. )
   		# -user <owner> --> performs user-ownership based search
   		# -group <group> --> perform group-ownership based search
   		# -type <type> --> perform file-type based search
  			 # types:
  			 # f  -->  search normal files only
  			 # d --> search directories
   			 # l --> search soft links
 
[aadarsha@labserver ~]$ date
Mon Jun 15 04:59:06 PM +0545 2026
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Mon Jun 15 16:48:46 +0545 2026 on pts/0
[root@labserver ~]# 

[root@labserver ~]# find /etc -name passwd
/etc/passwd
/etc/pam.d/passwd
[root@labserver ~]# find /etc -name passwd 2>/dev/null
/etc/passwd
/etc/pam.d/passwd
[root@labserver ~]# 
[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /etc -name passwd 
find: ‘/etc/lvm/devices’: Permission denied
...
find: ‘/etc/audit’: Permission denied
find: ‘/etc/sssd’: Permission denied
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /etc -name passwd 2>/dev/null 
/etc/passwd
/etc/pam.d/passwd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ touch /tmp/passwd
[aadarsha@labserver ~]$ find /etc -name passwd 2>/dev/null
/etc/passwd
/etc/pam.d/passwd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find / -name passwd 2>/dev/null
/sys/fs/selinux/class/passwd
/sys/fs/selinux/class/passwd/perms/passwd
/tmp/passwd
/etc/passwd
/etc/pam.d/passwd
/usr/bin/passwd
/usr/share/bash-completion/completions/passwd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find / -iname pAsswd 2>/dev/null
/sys/fs/selinux/class/passwd
/sys/fs/selinux/class/passwd/perms/passwd
/tmp/passwd
/tmp/Passwd
/tmp/PassWD
/etc/passwd
/etc/pam.d/passwd
/usr/bin/passwd
/usr/share/bash-completion/completions/passwd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ touch /tmp/PassWD
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ find / -iname passwd 2>/dev/null
/sys/fs/selinux/class/passwd
/sys/fs/selinux/class/passwd/perms/passwd
/tmp/passwd
/tmp/Passwd
/tmp/PassWD
/etc/passwd
/etc/pam.d/passwd
/usr/bin/passwd
/usr/share/bash-completion/completions/passwd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find / -name passwd 2>/dev/null | wc -l
7
[aadarsha@labserver ~]$ find / -iname passwd 2>/dev/null | wc -l
9
[aadarsha@labserver ~]$ 

# access time

[aadarsha@labserver ~]$ find / -atime -2
...
...

/tmp/PassWD
/etc
/etc/fstab
/etc/lvm
/etc/lvm/devices
find: ‘/etc/lvm/devices’: Permission denied
/etc/lvm/archive
find: ‘/etc/lvm/archive’: Permission denied
/etc/lvm/backup
find: ‘/etc/lvm/backup’: Permission denied
/etc/lvm/cache
find: ‘/etc/lvm/cache’: Permission denied
/etc/lvm/profile
...
...

[aadarsha@labserver ~]$ find / -atime -2 2>/dev/null | wc -l
83501
[aadarsha@labserver ~]$ find / -mtime -2 2>/dev/null | wc -l
76970
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ find / -atime -2 2>/dev/null | wc -l
83268
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ find / -atime -2 | wc -l
find: ‘/boot/efi/EFI/centos’: Permission denied
find: ‘/boot/grub2’: Permission denied


[aadarsha@labserver ~]$ find / -atime -2 2>/dev/null | wc -l
83501
[aadarsha@labserver ~]$ find / -mtime -2 2>/dev/null | wc -l
76970
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ find /etc -mtime -2 2>/dev/null | wc -l
2
[aadarsha@labserver ~]$ find / -mtime -2 2>/dev/null | wc -l
76970
[aadarsha@labserver ~]$ find /etc -atime -2 2>/dev/null | wc -l
323
[aadarsha@labserver ~]$ 


[aadarsha@labserver ~]$ find / -atime -2 2>/dev/null | wc -l
83501
[aadarsha@labserver ~]$ find / -mtime -2 2>/dev/null | wc -l
76970
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ find /etc -mtime -2 2>/dev/null | wc -l
2
[aadarsha@labserver ~]$ find / -mtime -2 2>/dev/null | wc -l
76970
[aadarsha@labserver ~]$ find /etc -atime -2 2>/dev/null | wc -l
323
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ find /etc -atime -2 2>/dev/null | wc -l
323
[aadarsha@labserver ~]$ find /etc -mtime -2 2>/dev/null | wc -l
2
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ find /var/l
lib/   local/ lock/  log/   
[aadarsha@labserver ~]$ find /var -size +10M 2>/dev/null 
/var/cache/dnf/appstream-25519c512d836b42/repodata/28fcbe99d124870c057e25d821ae6d6d4c8fa1e58c8609ffe3511cdf0fd53b66-filelists.xml.gz
/var/cache/dnf/appstream-filenames.solvx
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /var -size +10M
find: ‘/var/lib/selinux/targeted/active’: Permission denied
...
find: ‘/var/tmp/systemd-private-e670512019ad43ffb843c29fe99f0b06-systemd-logind.service-9Mcv0n’: Permission denied
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /var -size +10M 2>/dev/null 
/var/cache/dnf/appstream-25519c512d836b42/repodata/28fcbe99d124870c057e25d821ae6d6d4c8fa1e58c8609ffe3511cdf0fd53b66-filelists.xml.gz
/var/cache/dnf/appstream-filenames.solvx
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /var -size +10M 2>/dev/null 
/var/cache/dnf/appstream-25519c512d836b42/repodata/28fcbe99d124870c057e25d821ae6d6d4c8fa1e58c8609ffe3511cdf0fd53b66-filelists.xml.gz
/var/cache/dnf/appstream-filenames.solvx
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Mon Jun 15 16:59:23 +0545 2026 on pts/0
[root@labserver ~]# 

[root@labserver ~]# find /var/log -size +1M
/var/log/audit/audit.log
/var/log/anaconda/journal.log
/var/log/messages-20260611
[root@labserver ~]# 

[root@labserver ~]# find /var/log -size -1M
/var/log/anaconda/dnf.librepo.log
/var/log/firewalld
/var/log/maillog-20260611
/var/log/maillog
/var/log/spooler-20260611
/var/log/spooler
[root@labserver ~]# 

# Finding and processing

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Mon Jun 15 17:44:27 +0545 2026 on pts/0
[root@labserver ~]# 
[root@labserver ~]# find /var/log -size +1M -exec ls -ldh {} \;
-rw-------. 1 root root 1.1M Jun 15 18:03 /var/log/audit/audit.log
-rw-------. 1 root root 2.7M May 26 15:37 /var/log/anaconda/journal.log
-rw-------. 1 root root 1.4M Jun 11 05:39 /var/log/messages-20260611
[root@labserver ~]# 

[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /var/log -size +1M -exec ls -ldh {} \;
find: ‘/var/log/private’: Permission denied
find: ‘/var/log/samba’: Permission denied
find: ‘/var/log/audit’: Permission denied
find: ‘/var/log/sssd’: Permission denied
find: ‘/var/log/chrony’: Permission denied
-rw-------. 1 root root 2.7M May 26 15:37 /var/log/anaconda/journal.log
-rw-------. 1 root root 1.4M Jun 11 05:39 /var/log/messages-20260611
[aadarsha@labserver ~]$ 
 
[aadarsha@labserver ~]$ # find /var/log -size +1M -exec ls -ldh {} \;
[aadarsha@labserver ~]$ # --->finding-part<------ ---->processing-part<--

[aadarsha@labserver ~]$ # --->finding-part<------|---->processing-part<--

# exec --> execute action on the found files without asking for confirmation
# ok   --> execute action on the found files after asking for confirmation

# ls -ldh  -->  It represents the action to be taken on the found files

# {}  -->  place holder of the found files

# \;  --> escape delimeter
# {}  -->  place holder of the found files (buffer)


[root@labserver ~]# find /var/log -size +1M -exec ls -ldh {} \;
-rw-------. 1 root root 1.1M Jun 15 18:12 /var/log/audit/audit.log
-rw-------. 1 root root 2.7M May 26 15:37 /var/log/anaconda/journal.log
-rw-------. 1 root root 1.4M Jun 11 05:39 /var/log/messages-20260611
[root@labserver ~]# 

[root@labserver ~]# find /var/log -size +1M -ok ls -ldh {} \;
< ls ... /var/log/audit/audit.log > ? y
-rw-------. 1 root root 1.1M Jun 15 18:12 /var/log/audit/audit.log
< ls ... /var/log/anaconda/journal.log > ? n
< ls ... /var/log/messages-20260611 > ? y
-rw-------. 1 root root 1.4M Jun 11 05:39 /var/log/messages-20260611
[root@labserver ~]# 

[root@labserver ~]# find /var/log -size +1M -ok ls -ldh {} \;
< ls ... /var/log/audit/audit.log > ? y
-rw-------. 1 root root 1.1M Jun 15 18:12 /var/log/audit/audit.log
< ls ... /var/log/anaconda/journal.log > ? y
-rw-------. 1 root root 2.7M May 26 15:37 /var/log/anaconda/journal.log
< ls ... /var/log/messages-20260611 > ? y
-rw-------. 1 root root 1.4M Jun 11 05:39 /var/log/messages-20260611
[root@labserver ~]# 

[root@labserver ~]# find /var/log -size +1M -ok ls -ldh {} \;
< ls ... /var/log/audit/audit.log > ? n
< ls ... /var/log/anaconda/journal.log > ? n
< ls ... /var/log/messages-20260611 > ? n
[root@labserver ~]# 

[root@labserver ~]# mkdir -p /root/largefiles

[root@labserver ~]# ls /root/largefiles/
 
[root@labserver ~]# # find /var/log -size +1M -exec cp -pr {}(acts as the source for found files) /root/largefiles/ (destination) \;

[root@labserver ~]# # find /var/log -size +1M -exec cp -pr {} /root/largefiles/ \;

[root@labserver ~]# ls /root/largefiles/

[root@labserver ~]# # find /var/log -size +1M -exec cp -pr {} /root/largefiles  \;

[root@labserver ~]# find /var/log -size +1M -exec cp -pr {} /root/largefiles  \;
 
[root@labserver ~]# ls /root/largefiles/
audit.log  journal.log  messages-20260611
[root@labserver ~]# 

[root@labserver ~]# ls -ldh /root/largefiles/
drwxr-xr-x. 2 root root 67 Jun 15 18:17 /root/largefiles/
[root@labserver ~]# 

[root@labserver ~]# ls -lh /root/largefiles/
total 5.1M
-rw-------. 1 root root 1.1M Jun 15 18:12 audit.log
-rw-------. 1 root root 2.7M May 26 15:37 journal.log
-rw-------. 1 root root 1.4M Jun 11 05:39 messages-20260611
[root@labserver ~]# 

[root@labserver ~]# # eg: back of .conf type of files 

[root@labserver ~]# mkdir -p /root/confbkp

[root@labserver ~]# ls -ldh /root/confbkp
drwxr-xr-x. 2 root root 6 Jun 15 18:18 /root/confbkp
[root@labserver ~]# 

[root@labserver ~]# find /etc -name *.conf -exec cp -p {} /root/confbkp \;

[root@labserver ~]# ls -ldh /root/confbkp
drwxr-xr-x. 2 root root 4.0K Jun 15 18:22 /root/confbkp
[root@labserver ~]#
 
[root@labserver ~]# rm -r /root/confbkp/*
rm: remove regular file '/root/confbkp/00-keyboard.conf'? ^C
[root@labserver ~]# 

[root@labserver ~]# unalias rm
[root@labserver ~]# 
[root@labserver ~]# rm -r /root/confbkp/*
[root@labserver ~]# 
[root@labserver ~]# ls -ldh /root/confbkp
drwxr-xr-x. 2 root root 6 Jun 15 18:51 /root/confbkp
[root@labserver ~]# 

[root@labserver ~]# ls /root/confbkp | wc -l
0
[root@labserver ~]# 
[root@labserver ~]# find /etc -name *.conf -exec cp -p {} /root/confbkp \;
[root@labserver ~]# 
[root@labserver ~]# ls /root/confbkp | wc -l
75
[root@labserver ~]# 
[root@labserver ~]# ls -ldh /root/confbkp
drwxr-xr-x. 2 root root 4.0K Jun 15 18:53 /root/confbkp
[root@labserver ~]# 

# Using Logical Operators with find
[root@labserver ~]# 
[root@labserver ~]# # -and --> logical AND
[root@labserver ~]# # -o   -->  logical OR
[root@labserver ~]# # !  -->  logical NOT

[root@labserver ~]# find /var/log --size +1M -and -size -3M
find: unknown predicate `--size'
[root@labserver ~]# 
[root@labserver ~]# find /var/log -size +1M -and -size -3M
/var/log/audit/audit.log
/var/log/messages-20260611
[root@labserver ~]# 

[root@labserver ~]# find /var/log -size +1M -and -size -3M -exec ls -lh {} \;
-rw-------. 1 root root 1.1M Jun 15 18:32 /var/log/audit/audit.log
-rw-------. 1 root root 1.4M Jun 11 05:39 /var/log/messages-20260611
[root@labserver ~]# 

[root@labserver ~]# find /var/log -size +1k -and -size -3k -exec ls -lh {} \;
-rw-------. 1 root root 1.2K May 26 15:37 /var/log/anaconda/program.log
-rw-r--r--. 1 root root 1.7K Jun 10 05:29 /var/log/hawkey.log-20260611
[root@labserver ~]# 

[root@labserver ~]# find /etc/ -size 100c -exec ls -lhd {} \;
lrwxrwxrwx. 1 root root 100 May 26 15:34 /etc/pki/tls/certs/9b5697b0.0 -> /etc/pki/ca-trust/extracted/pem/directory-hash/Trustwave_Global_ECC_P256_Certification_Authority.pem
lrwxrwxrwx. 1 root root 100 May 26 15:34 /etc/pki/tls/certs/1ae85e5e.0 -> /etc/pki/ca-trust/extracted/pem/directory-hash/Trustwave_Global_ECC_P256_Certification_Authority.pem
lrwxrwxrwx. 1 root root 100 May 26 15:34 /etc/pki/tls/certs/d887a5bb.0 -> /etc/pki/ca-trust/extracted/pem/directory-hash/Trustwave_Global_ECC_P384_Certification_Authority.pem
lrwxrwxrwx. 1 root root 100 May 26 15:34 /etc/pki/tls/certs/9aef356c.0 -> /etc/pki/ca-trust/extracted/pem/directory-hash/Trustwave_Global_ECC_P384_Certification_Authority.pem
drwxr-x---. 4 root root 100 May 26 15:33 /etc/audit
[root@labserver ~]# 
 
[root@labserver ~]# find /var/log -size +10M -o -size -10K -exec ls -lh {} \;
find: invalid -size type `K'
[root@labserver ~]# 
[root@labserver ~]# find /var/log -size +10M -o -size -10k -exec ls -lh {} \;
total 2.2M
drwxr-xr-x. 2 root   root   4.0K May 26 15:37 anaconda
drwx------. 2 root   root     23 May 26 15:39 audit
-rw-rw----. 1 root   utmp   4.5K Jun 15 16:31 btmp
drwxr-x---. 2 chrony chrony    6 Nov 20  2025 chrony
-rw-------. 1 root   root   4.9K Jun 15 19:01 cron
...
... 
[root@labserver ~]# 
[root@labserver ~]# find /var/log -size +10M -o -size -10k -exec ls -lh {} \; | wc -l
61
[root@labserver ~]# 

[root@labserver ~]# ls /home/
aadarsha  milan  suman  user1  user2
[root@labserver ~]# 

[root@labserver ~]# su - user2
[user2@labserver ~]$ 
[user2@labserver ~]$ touch f1 f2 f3
[user2@labserver ~]$ ls
f1  f2  f3
[user2@labserver ~]$ ls -l
total 0
-rw-r--r--. 1 user2 user2 0 Jun 15 19:12 f1
-rw-r--r--. 1 user2 user2 0 Jun 15 19:12 f2
-rw-r--r--. 1 user2 user2 0 Jun 15 19:12 f3
[user2@labserver ~]$ 
[user2@labserver ~]$ exit
logout
[root@labserver ~]# 
[root@labserver ~]# su - user1
[user1@labserver ~]$ 
[user1@labserver ~]$ pwd
/home/user1
[user1@labserver ~]$ 
[user1@labserver ~]$ touch abc0 abc1 abc2 abc3
[user1@labserver ~]$ ls
abc0  abc1  abc2  abc3
[user1@labserver ~]$ ls -l
total 0
-rw-r--r--. 1 user1 user1 0 Jun 15 19:14 abc0
-rw-r--r--. 1 user1 user1 0 Jun 15 19:14 abc1
-rw-r--r--. 1 user1 user1 0 Jun 15 19:14 abc2
-rw-r--r--. 1 user1 user1 0 Jun 15 19:14 abc3
[user1@labserver ~]$ 

[user1@labserver ~]$ exit
logout
[root@labserver ~]# 
[root@labserver ~]# find /home -user user1
/home/user1
/home/user1/.bash_logout
/home/user1/.bash_profile
/home/user1/.bashrc
/home/user1/abc0
/home/user1/abc1
/home/user1/abc2
/home/user1/abc3
/home/user1/.bash_history
[root@labserver ~]# 

[root@labserver ~]# find /home -user user1 -o -user user2
/home/user1
/home/user1/.bash_logout
/home/user1/.bash_profile
/home/user1/.bashrc
/home/user1/abc0
/home/user1/abc1
/home/user1/abc2
/home/user1/abc3
/home/user1/.bash_history
/home/user2
/home/user2/.bash_logout
/home/user2/.bash_profile
/home/user2/.bashrc
/home/user2/f1
/home/user2/f2
/home/user2/f3
/home/user2/.bash_history
[root@labserver ~]# 

[root@labserver ~]# mkdir -p /bkp

[root@labserver ~]# find /home -user user1 -o user user2 -exec cp -p {} /bkp \;
find: paths must precede expression: `user'
[root@labserver ~]# 
[root@labserver ~]# ls -l /bkp
total 0
[root@labserver ~]# 

[root@labserver ~]# find /home /(-user user1 -o user user2\) -exec cp -p {} /bkp \;
-bash: syntax error near unexpected token `('
[root@labserver ~]# find /home /(-user user1 -o user user2/) -exec cp -p {} /bkp \;
-bash: syntax error near unexpected token `('
[root@labserver ~]# 

[root@labserver ~]# find /home \(-user user1 -o user user2\) -exec cp -p {} /bkp \;
find: paths must precede expression: `user'
[root@labserver ~]# 

[root@labserver ~]# find /home \(-user user1 -o -user user2\) -exec cp -p {} /bkp \;
find: invalid user name or UID argument to -user: ‘user2)’
[root@labserver ~]# 

[root@labserver ~]# find /home \( -user user1 -o -user user2 \) -exec cp -p {} /bkp \;
cp: -r not specified; omitting directory '/home/user1'
cp: -r not specified; omitting directory '/home/user2'
[root@labserver ~]# 

[root@labserver ~]# ls /bkp
abc0  abc1  abc2  abc3  f1  f2  f3
[root@labserver ~]# 
 
[root@labserver ~]# find /home \( -user user1 -o -user user2 \) -exec cp -p {} /bkp \;
cp: -r not specified; omitting directory '/home/user1'
cp: -r not specified; omitting directory '/home/user2'
[root@labserver ~]# 

[root@labserver ~]# ls /bkp
abc0  abc1  abc2  abc3  f1  f2  f3
[root@labserver ~]# 

[root@labserver ~]# find / -group marketting
find: invalid group name or GID argument to -group: ‘marketting’
[root@labserver ~]# 
[root@labserver ~]# # find / -group marketting

[root@labserver ~]# find /etc -type l | wc -l
707
[root@labserver ~]# 
[root@labserver ~]# find /etc -type d | wc -l
195
[root@labserver ~]# 
[root@labserver ~]# find /etc -type f | wc -l
488
[root@labserver ~]# 
[root@labserver ~]# 
[root@labserver ~]# find /home -user aadarsha | wc -l
41
[root@labserver ~]# 
[root@labserver ~]# find /home ! c-user aadarsha | wc -l
find: paths must precede expression: `c-user'
0
[root@labserver ~]# 
 
[root@labserver ~]# find /home ! -user aadarsha | wc -l
85
[root@labserver ~]# 
 
[root@labserver ~]# find /home \( -user user1 -o -user user2 \) -exec ll {} /bkp \;
find: ‘ll’: No such file or directory
find: ‘ll’: No such file or directory
find: ‘ll’: No such file or directory
find: ‘ll’: No such file or directory
[root@labserver ~]# 

[root@labserver ~]# find /home \( -user user1 -o -user user2 \) -exec ls -l {} /bkp \;
/bkp:

[root@labserver ~]# touch emp1 emp2 emp3 emp4
[root@labserver ~]# ls
anaconda-ks.cfg  confbkp  emp1  emp2  emp3  emp4  largefiles
[root@labserver ~]# 

[root@labserver ~]# pwd
/root
[root@labserver ~]# 
[root@labserver ~]# find -size 0
./emp1
./emp2
./emp3
./emp4
[root@labserver ~]# 

[root@labserver ~]# find -size 0
./emp1
./emp2
./emp3
./emp4
[root@labserver ~]# 

[root@labserver ~]# find -size 0 -exec rm -f {} \;

[root@labserver ~]# find -size 0

[root@labserver ~]# ls
anaconda-ks.cfg  confbkp  largefiles
[root@labserver ~]# 

[root@labserver ~]# ls /root/confbkp/
00-keyboard.conf                  grub2-pc.conf                pwhistory.conf
01-permitrootlogin.conf           grub2-tools-minimal.conf     pwquality.conf
40-redhat-crypto-policies.conf    host.conf                    request-key.conf
50-redhat.conf                    kdump.conf                   resolv.conf
56-google-noto-sans-mono-vf.conf  krb5.conf                    rsyslog.conf
56-google-noto-sans-vf.conf       l2tp_eth-blacklist.conf      sctp-blacklist.conf
56-google-noto-serif-vf.conf      l2tp_ip6-blacklist.conf      sctp_diag-blacklist.conf
64-redhat-mono-vf.conf            l2tp_ip-blacklist.conf       selinux-policy-targeted.conf
64-redhat-text-vf.conf            l2tp_netlink-blacklist.conf  semanage.conf
99-sysctl.conf                    l2tp_ppp-blacklist.conf      sepermit.conf
access.conf                       ldap.conf                    session.conf
auditd.conf                       ld.so.conf                   sestatus.conf
authselect.conf                   libaudit.conf                setrans.conf
ca-legacy.conf                    limits.conf                  setup.conf
chrony.conf                       locale.conf                  smb.conf
chroot.conf                       logrotate.conf               sudo.conf
copr.conf                         lvm.conf                     sudo-ldap.conf
debuginfo-install.conf            lvmlocal.conf                sysctl.conf
dist.conf                         man_db.conf                  system.conf
dnf.conf                          mke2fs.conf                  systemd.conf
dracut.conf                       namespace.conf               time.conf
faillock.conf                     NetworkManager.conf          tipc_diag-blacklist.conf
firewalld.conf                    nftables.conf                vconsole.conf
firewalld-sysctls.conf            nsswitch.conf                xattr.conf
group.conf                        pam_env.conf                 yum.conf
[root@labserver ~]# 

[root@labserver ~]# cd /root/confbkp/
[root@labserver confbkp]# 
[root@labserver confbkp]# touch f1 f2 f3
[root@labserver confbkp]# 
[root@labserver confbkp]# cd
[root@labserver ~]# 

[root@labserver ~]# find /root/confbkp/ -name *.conf -exec mv {} {}.bak \;
[root@labserver ~]# 
[root@labserver ~]# ls /root/confbkp
00-keyboard.conf.bak.conf.bak                  ld.so.conf.bak.conf.bak
01-permitrootlogin.conf.bak.conf.bak           libaudit.conf.bak.conf.bak
40-redhat-crypto-policies.conf.bak.conf.bak    limits.conf.bak.conf.bak
50-redhat.conf.bak.conf.bak                    locale.conf.bak.conf.bak
56-google-noto-sans-mono-vf.conf.bak.conf.bak  logrotate.conf.bak.conf.bak
56-google-noto-sans-vf.conf.bak.conf.bak       lvm.conf.bak.conf.bak
56-google-noto-serif-vf.conf.bak.conf.bak      lvmlocal.conf.bak.conf.bak
64-redhat-mono-vf.conf.bak.conf.bak            man_db.conf.bak.conf.bak
64-redhat-text-vf.conf.bak.conf.bak            mke2fs.conf.bak.conf.bak
99-sysctl.conf.bak.conf.bak                    namespace.conf.bak.conf.bak
access.conf.bak.conf.bak                       NetworkManager.conf.bak.conf.bak
auditd.conf.bak.conf.bak                       nftables.conf.bak.conf.bak
authselect.conf.bak.conf.bak                   nsswitch.conf.bak.conf.bak
ca-legacy.conf.bak.conf.bak                    pam_env.conf.bak.conf.bak
chrony.conf.bak.conf.bak                       pwhistory.conf.bak.conf.bak
chroot.conf.bak.conf.bak                       pwquality.conf.bak.conf.bak
copr.conf.bak.conf.bak                         request-key.conf.bak.conf.bak
debuginfo-install.conf.bak.conf.bak            resolv.conf.bak.conf.bak
dist.conf.bak.conf.bak                         rsyslog.conf.bak.conf.bak
dnf.conf.bak.conf.bak                          sctp-blacklist.conf.bak.conf.bak
dracut.conf.bak.conf.bak                       sctp_diag-blacklist.conf.bak.conf.bak
f1                                             selinux-policy-targeted.conf.bak.conf.bak
f2                                             semanage.conf.bak.conf.bak
f3                                             sepermit.conf.bak.conf.bak
faillock.conf.bak.conf.bak                     session.conf.bak.conf.bak
firewalld.conf.bak.conf.bak                    sestatus.conf.bak.conf.bak
firewalld-sysctls.conf.bak.conf.bak            setrans.conf.bak.conf.bak
group.conf.bak.conf.bak                        setup.conf.bak.conf.bak
grub2-pc.conf.bak.conf.bak                     smb.conf.bak.conf.bak
grub2-tools-minimal.conf.bak.conf.bak          sudo.conf.bak.conf.bak
host.conf.bak.conf.bak                         sudo-ldap.conf.bak.conf.bak
kdump.conf.bak.conf.bak                        sysctl.conf.bak.conf.bak
krb5.conf.bak.conf.bak                         system.conf.bak.conf.bak
l2tp_eth-blacklist.conf.bak.conf.bak           systemd.conf.bak.conf.bak
l2tp_ip6-blacklist.conf.bak.conf.bak           time.conf.bak.conf.bak
l2tp_ip-blacklist.conf.bak.conf.bak            tipc_diag-blacklist.conf.bak.conf.bak
l2tp_netlink-blacklist.conf.bak.conf.bak       vconsole.conf.bak.conf.bak
l2tp_ppp-blacklist.conf.bak.conf.bak           xattr.conf.bak.conf.bak
ldap.conf.bak.conf.bak                         yum.conf.bak.conf.bak
[root@labserver ~]# 

# log processing

[root@labserver ~]# ls
anaconda-ks.cfg  confbkp  f1  f2  f3  largefiles
[root@labserver ~]# 

[root@labserver ~]# mkdir logdir
[root@labserver ~]# ls
anaconda-ks.cfg  confbkp  f1  f2  f3  largefiles  logdir
[root@labserver ~]# cd logdir/
[root@labserver logdir]# 

[root@labserver logdir]# vi app.log
[root@labserver logdir]# cat app.log 
INFO Application started successfully
INFO User Alice logged in
ERROR Payment Gateway timeout
WARNING Memory usage reached 75%
INFO User bob logged in
ERROR Database connection lost ...
INFO Backup completed successfully
WARNING Disk space reached 85%
ERROR Invalid API request received
INFO User charlie logged out
ERROR Payment gateway timeout
ERROR Database connection lost
INFO User Alice logged out
[root@labserver logdir]# 

[root@labserver logdir]# vi employees.csv
[root@labserver logdir]# cat employees.csv 
empid,name,destination,department,salary
E001,Ram Bahadur Thapa,Manager,Administration,85000
E002,Sita Kumari Sharma,Officer,Finance,65000
E003,Hari Prasad Adhikari,Engineer,IT,78000
E004,Gita Devi Koirala,Assistant,Human Resources,52000
E005,Binod Kumar Nepal,Supervisor,Operations,60000
E006,Manisha Rai,Analyst,Marketing,70000
E007,Rajan Gurung,Coordinator,Procurement,58000
E008,Sujan Shrestha,Developer,IT,75000
E009,Kavita Bhattarai,Accountant,Finance,62000
E010,Prakash Tamang,Technician,Maintenance,55000
[root@labserver logdir]# 

[root@labserver logdir]# ls
app.log  employees.csv
[root@labserver logdir]# 

[root@labserver logdir]# sed 's/ERROR/CRITICAL/g' app.log 
INFO Application started successfully
INFO User Alice logged in
CRITICAL Payment Gateway timeout
WARNING Memory usage reached 75%
INFO User bob logged in
CRITICAL Database connection lost ...
INFO Backup completed successfully
WARNING Disk space reached 85%
CRITICAL Invalid API request received
INFO User charlie logged out
CRITICAL Payment gateway timeout
CRITICAL Database connection lost
INFO User Alice logged out
[root@labserver logdir]# 

[root@labserver logdir]# sed 's/ERROR/CRITICAL/g' app.log >criticallogs
[root@labserver logdir]# ls
app.log  criticallogs  employees.csv
[root@labserver logdir]# cat criticallogs 
INFO Application started successfully
INFO User Alice logged in
CRITICAL Payment Gateway timeout
WARNING Memory usage reached 75%
INFO User bob logged in
CRITICAL Database connection lost ...
INFO Backup completed successfully
WARNING Disk space reached 85%
CRITICAL Invalid API request received
INFO User charlie logged out
CRITICAL Payment gateway timeout
CRITICAL Database connection lost
INFO User Alice logged out
[root@labserver logdir]# 

[root@labserver logdir]# sed '/INFO/d' criticallogs 
CRITICAL Payment Gateway timeout
WARNING Memory usage reached 75%
CRITICAL Database connection lost ...
WARNING Disk space reached 85%
CRITICAL Invalid API request received
CRITICAL Payment gateway timeout
CRITICAL Database connection lost
[root@labserver logdir]# 

[root@labserver logdir]# cat criticallogs 
INFO Application started successfully
INFO User Alice logged in
CRITICAL Payment Gateway timeout
WARNING Memory usage reached 75%
INFO User bob logged in
CRITICAL Database connection lost ...
INFO Backup completed successfully
WARNING Disk space reached 85%
CRITICAL Invalid API request received
INFO User charlie logged out
CRITICAL Payment gateway timeout
CRITICAL Database connection lost
INFO User Alice logged out
[root@labserver logdir]# 

[root@labserver logdir]# sed '/INFO/d' criticallogs >noinfo.log
[root@labserver logdir]# ls
app.log  criticallogs  employees.csv  noinfo.log
[root@labserver logdir]# cat noinfo.log 
CRITICAL Payment Gateway timeout
WARNING Memory usage reached 75%
CRITICAL Database connection lost ...
WARNING Disk space reached 85%
CRITICAL Invalid API request received
CRITICAL Payment gateway timeout
CRITICAL Database connection lost
[root@labserver logdir]# 

[root@labserver logdir]# sed 's/Database/Postgres Database' app.log 
sed: -e expression #1, char 28: unterminated `s' command
[root@labserver logdir]# 

[root@labserver logdir]# sed 's/Database/Postgres Database/g' app.log 
INFO Application started successfully
INFO User Alice logged in
ERROR Payment Gateway timeout
WARNING Memory usage reached 75%
INFO User bob logged in
ERROR Postgres Database connection lost ...
INFO Backup completed successfully
WARNING Disk space reached 85%
ERROR Invalid API request received
INFO User charlie logged out
ERROR Payment gateway timeout
ERROR Postgres Database connection lost
INFO User Alice logged out
[root@labserver logdir]# 

[root@labserver logdir]# ls
app.log  criticallogs  employees.csv  noinfo.log
[root@labserver logdir]# 

[root@labserver logdir]# sed '/INFO/d' app.log 
ERROR Payment Gateway timeout
WARNING Memory usage reached 75%
ERROR Database connection lost ...
WARNING Disk space reached 85%
ERROR Invalid API request received
ERROR Payment gateway timeout
ERROR Database connection lost
[root@labserver logdir]#
 
[root@labserver logdir]# cat app.log 
INFO Application started successfully
INFO User Alice logged in
ERROR Payment Gateway timeout
WARNING Memory usage reached 75%
INFO User bob logged in
ERROR Database connection lost ...
INFO Backup completed successfully
WARNING Disk space reached 85%
ERROR Invalid API request received
INFO User charlie logged out
ERROR Payment gateway timeout
ERROR Database connection lost
INFO User Alice logged out
[root@labserver logdir]# 

[root@labserver logdir]# sed -i '/INFO/d' app.log 

[root@labserver logdir]# cat app.log 
ERROR Payment Gateway timeout
WARNING Memory usage reached 75%
ERROR Database connection lost ...
WARNING Disk space reached 85%
ERROR Invalid API request received
ERROR Payment gateway timeout
ERROR Database connection lost
[root@labserver logdir]# 

[root@labserver logdir]# ls
app.log  criticallogs  employees.csv  noinfo.log
[root@labserver logdir]# cat criticallogs 
INFO Application started successfully
INFO User Alice logged in
CRITICAL Payment Gateway timeout
WARNING Memory usage reached 75%
INFO User bob logged in
CRITICAL Database connection lost ...
INFO Backup completed successfully
WARNING Disk space reached 85%
CRITICAL Invalid API request received
INFO User charlie logged out
CRITICAL Payment gateway timeout
CRITICAL Database connection lost
INFO User Alice logged out
[root@labserver logdir]# 

[root@labserver logdir]# ls
app.log  criticallogs  employees.csv  noinfo.log
[root@labserver logdir]# 

[root@labserver logdir]# man sed
 
[root@labserver logdir]# sed -n '/CRITICAL/p' app.log 

[root@labserver logdir]# sed -n '/CRITICAL/p' criticallogs 
CRITICAL Payment Gateway timeout
CRITICAL Database connection lost ...
CRITICAL Invalid API request received
CRITICAL Payment gateway timeout
CRITICAL Database connection lost
[root@labserver logdir]# 

# Using awk

[root@labserver logdir]# ls
app.log  criticallogs  employees.csv  noinfo.log
[root@labserver logdir]# 

[root@labserver logdir]# cat employees.csv 
empid,name,destination,department,salary
E001,Ram Bahadur Thapa,Manager,Administration,85000
E002,Sita Kumari Sharma,Officer,Finance,65000
E003,Hari Prasad Adhikari,Engineer,IT,78000
E004,Gita Devi Koirala,Assistant,Human Resources,52000
E005,Binod Kumar Nepal,Supervisor,Operations,60000
E006,Manisha Rai,Analyst,Marketing,70000
E007,Rajan Gurung,Coordinator,Procurement,58000
E008,Sujan Shrestha,Developer,IT,75000
E009,Kavita Bhattarai,Accountant,Finance,62000
E010,Prakash Tamang,Technician,Maintenance,55000
[root@labserver logdir]# 

[root@labserver logdir]# cut -f 2 -d, employees.csv 
name
Ram Bahadur Thapa
Sita Kumari Sharma
Hari Prasad Adhikari
Gita Devi Koirala
Binod Kumar Nepal
Manisha Rai
Rajan Gurung
Sujan Shrestha
Kavita Bhattarai
Prakash Tamang
[root@labserver logdir]# 

[root@labserver logdir]# awk -F ',' '{print $2}' employees.csv 
name
Ram Bahadur Thapa
Sita Kumari Sharma
Hari Prasad Adhikari
Gita Devi Koirala
Binod Kumar Nepal
Manisha Rai
Rajan Gurung
Sujan Shrestha
Kavita Bhattarai
Prakash Tamang
[root@labserver logdir]# 

[root@labserver logdir]# awk -F ',' '{print $2, $5}' employees.csv 
name salary
Ram Bahadur Thapa 85000
Sita Kumari Sharma 65000
Hari Prasad Adhikari 78000
Gita Devi Koirala 52000
Binod Kumar Nepal 60000
Manisha Rai 70000
Rajan Gurung 58000
Sujan Shrestha 75000
Kavita Bhattarai 62000
Prakash Tamang 55000
[root@labserver logdir]# 

[root@labserver logdir]# awk -F ',' '$5 >70000' '{print $2, $5}' employees.csv 
awk: fatal: cannot open file `{print $2, $5}' for reading: No such file or directory
[root@labserver logdir]# 

[root@labserver logdir]# awk -F ',' '$5 >70000 {print $2, $5}' employees.csv 
name salary
Ram Bahadur Thapa 85000
Hari Prasad Adhikari 78000
Sujan Shrestha 75000
[root@labserver logdir]# 
```