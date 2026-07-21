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

```