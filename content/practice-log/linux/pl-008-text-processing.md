---
title: "PL - 008 —  Text Processing, Data Manipulation, Pipelines, and File Searching"
date: 2026-06-12
draft: false
---
### Linux Text Processing & File Auditing
 Text processing and file auditing are treated as high-performance streaming pipelines based on three core principles:
* **The Unix Philosophy:** Every utility is single-purpose, optimized for memory efficiency, and processes text streams line-by-line via standard input (`stdin`) and standard output (`stdout`).
* **The Stream Pipeline (`|`):** Data manipulation avoids expensive disk I/O operations by processing transformations entirely in volatile memory (RAM) as data moves between tools.
* **Decoupled Automation:** Combining discovery (`find`), stream editing (`sed`), and pattern filtration (`grep`) allows sysadmins to parse gigabytes of system log telemetry or apply configuration patches across thousands of nodes simultaneously without interactive UI overhead.

---
| Command / Flag |  Syntax  |  Purpose  |
| :--- | :--- | :--- |
| **`head`** | `head -n <N> <file>` | Views the first $N$ lines of a text stream (Default: 10). |
| **`tail`** | `tail -n <N> <file>` | Views the last $N$ lines of a text stream (Default: 10). |
| **`tail -f`** | `tail -f <file>` | Live-streams appending data entries (e.g., tracking active logs). |
| **`sort`** | `sort -nrk <idx> -t '<delim>'` | Orders files numerically (`-n`), in reverse (`-r`), targeting a key field (`-k`). |
| **`cut`** | `cut -d '<delim>' -f <fields>` | Extracts explicit columns or field index ranges from a data matrix. |
| **`\|` (Pipeline)** | `command1 \| command2` | Binds standard output (`stdout`) of one process to input (`stdin`) of the next. |
| **`uniq`** | `uniq -c` | Deduplicates text layers. `-c` appends hit counts. **Requires pre-sorted input.** |
| **`tee`** | `tee <output_file>` | Splits data streams. Mirrors input to both `stdout` and a disk file. |
| **`wc`** | `wc -l` | Counts structural text metrics. `-l` isolates line totals. |
| **`grep`** | `grep '<pattern>' <file>` | Filters data streams for specific text string patterns or regular expressions. |
| **`egrep`** | `egrep 'A\|B' <file>` | Extended pattern engine. Uses logical OR criteria structures natively. |
| **`sed`** | `sed 's/old/new/g' <file>` | Stream editor for non-destructive string transformations. |
| **`sed -i`** | `sed -i 's/old/new/g' <file>` | In-place stream modification. Overwrites disk files directly. |
| **`find`** | `find <path> <expressions>` | Scans system trees to find files by ownership, timestamps, name, or size metrics. |
| **`2>/dev/null`** | `command 2>/dev/null` | Shell error redirector. Suppresses access permission warnings from standard error. |
---
### Terminal Session
```
# Text processing and Data Manipulation

# commands: head, tail, cat, less, wc, sort, cut, tee, uniq
# pipeline
# grep, sed

[aadarsha@labserver ~]$ cat passwd 
root:x:0:0:Super User:/root:/bin/bash
...
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
daemon:x:2:2:daemon:/sbin:/usr/sbin/nologin
adm:x:3:4:adm:/var/adm:/usr/sbin/nologin
...
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
[aadarsha@labserver ~]$ 
 
# viewing First/Last few lines of a file

[aadarsha@labserver ~]$ head passwd 
root:x:0:0:Super User:/root:/bin/bash
...
operator:x:11:0:operator:/root:/usr/sbin/nologin
[aadarsha@labserver ~]$ 

# By default 10 lines

[aadarsha@labserver ~]$ head -2 passwd 
root:x:0:0:Super User:/root:/bin/bash
bin:x:1:1:bin:/bin:/usr/sbin/nologin

[aadarsha@labserver ~]$ head -10 passwd 
root:x:0:0:Super User:/root:/bin/bash
bin:x:1:1:bin:/bin:/usr/sbin/nologin
daemon:x:2:2:daemon:/sbin:/usr/sbin/nologin
adm:x:3:4:adm:/var/adm:/usr/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/usr/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/usr/sbin/nologin
operator:x:11:0:operator:/root:/usr/sbin/nologin
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ tail passwd 
chrony:x:997:996:chrony system user:/var/lib/chrony:/sbin/nologin
...
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi passwd 

[aadarsha@labserver ~]$ tail -5 /etc/passwd
sssd:x:998:997:User for sssd:/run/sssd/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
chrony:x:997:996:chrony system user:/var/lib/chrony:/sbin/nologin
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi passwd 

[aadarsha@labserver ~]$ tail -5 /etc/passwd
sssd:x:998:997:User for sssd:/run/sssd/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
chrony:x:997:996:chrony system user:/var/lib/chrony:/sbin/nologin
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ tail /var/log/secure 
tail: cannot open '/var/log/secure' for reading: Permission denied
[aadarsha@labserver ~]$ 
 
[aadarsha@labserver ~]$ sudo su
[sudo] password for aadarsha: 
[root@labserver aadarsha]# 

[root@labserver aadarsha]# tail /var/log/secure 
Jun  9 05:24:00 labserver su[2184]: pam_unix(su:session): session closed for user root
...
Jun  9 06:20:51 labserver sudo[2991]: pam_unix(sudo:session): session opened for user root(uid=0) by aadarsha(uid=1000)
Jun  9 06:20:51 labserver su[2996]: pam_unix(su:session): session opened for user root(uid=0) by aadarsha(uid=0)
[root@labserver aadarsha]# 

[root@labserver aadarsha]# tail -f /var/log/secure 
Jun  9 05:24:00 labserver su[2184]: pam_unix(su:session): session closed for user root
Jun  9 05:24:00 labserver sudo[2179]: pam_unix(sudo:session): session closed for user root
Jun  9 06:04:24 labserver sudo[2754]: aadarsha : TTY=pts/0 ; PWD=/home/aadarsha ; USER=root ; COMMAND=/bin/su
Jun  9 06:04:24 labserver sudo[2754]: pam_unix(sudo:session): session opened for user root(uid=0) by aadarsha(uid=1000)
...
[root@labserver aadarsha]# 

# checking the realtime live activities using tail -f /var/log/secure   and   use ----------------(line)

# using sort

[aadarsha@labserver ~]$ cat /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
bin:x:1:1:bin:/bin:/usr/sbin/nologin
daemon:x:2:2:daemon:/sbin:/usr/sbin/nologin
...
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Wed Jun 10 05:48:53 +0545 2026 on pts/0
[root@labserver ~]# 

[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sort /etc/passwd
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
...
systemd-oom:x:999:999:systemd Userspace OOM Killer:/:/sbin/nologin
tss:x:59:59:Account used for TPM access:/:/usr/sbin/nologin
[aadarsha@labserver ~]$ 

# sort [option] <argument>

#  -n  --> numeric sort

#  -r  --> reverse sort

# -k -->
 
# -t --> 

[aadarsha@labserver ~]$ sort -n -k 3 -t: /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
bin:x:1:1:bin:/bin:/usr/sbin/nologin
daemon:x:2:2:daemon:/sbin:/usr/sbin/nologin
...
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
nobody:x:65534:65534:Kernel Overflow User:/:/usr/sbin/nologin
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sort -n -k 3 -t: -r /etc/passwd
nobody:x:65534:65534:Kernel Overflow User:/:/usr/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
systemd-oom:x:999:999:systemd Userspace OOM Killer:/:/sbin/nologin
sssd:x:998:997:User for sssd:/run/sssd/:/sbin/nologin
...
root:x:0:0:Super User:/root:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sort -nrk 3 -t: /etc/passwd
nobody:x:65534:65534:Kernel Overflow User:/:/usr/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
systemd-oom:x:999:999:systemd Userspace OOM Killer:/:/sbin/nologin
...
bin:x:1:1:bin:/bin:/usr/sbin/nologin
root:x:0:0:Super User:/root:/bin/bash
[aadarsha@labserver ~]$ 

# using cut

[aadarsha@labserver ~]$ cat /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
bin:x:1:1:bin:/bin:/usr/sbin/nologin
daemon:x:2:2:daemon:/sbin:/usr/sbin/nologin
...
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

# cut [option] <argument>

#   -f <field no>
#   -d <delimeter>

[aadarsha@labserver ~]$ cut -f 1 -d : /etc/passwd
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
operator
games
ftp
nobody
tss
systemd-oom
dbus
sssd
sshd
chrony
systemd-coredump
aadarsha
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 1,3,6 -d : /etc/passwd
root:0:/root
bin:1:/bin
daemon:2:/sbin
...
aadarsha:1000:/home/aadarsha
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 1,3,6,7 -d : /etc/passwd
root:0:/root:/bin/bash
bin:1:/bin:/usr/sbin/nologin
daemon:2:/sbin:/usr/sbin/nologin
...
systemd-coredump:995:/:/usr/sbin/nologin
aadarsha:1000:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 
 
[aadarsha@labserver ~]$ cut -f 1-4 -d : /etc/passwd
root:x:0:0
bin:x:1:1
daemon:x:2:2
...
systemd-coredump:x:995:995
aadarsha:x:1000:1000
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f -4 -d : /etc/passwd
root:x:0:0
...
systemd-coredump:x:995:995
aadarsha:x:1000:1000
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 4- -d : /etc/passwd
0:Super User:/root:/bin/bash
1:bin:/bin:/usr/sbin/nologin
2:daemon:/sbin:/usr/sbin/nologin
...
1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

# Linux principles

# small single purpose commands
# small commands can be combined

# small commands can be combined using pipelines

[aadarsha@labserver ~]$ cut -f 1 d: /etc/passwd
cut: 'd:': No such file or directory
root:x:0:0:Super User:/root:/bin/bash
bin:x:1:1:bin:/bin:/usr/sbin/nologin
...
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 1 d : /etc/passwd
cut: d: No such file or directory
cut: ':': No such file or directory
root:x:0:0:Super User:/root:/bin/bash
bin:x:1:1:bin:/bin:/usr/sbin/nologin
...
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 1 -d : /etc/passwd
root
...
aadarsha
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 1 -d : /etc/passwd | sort
aadarsha
...
tss
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 1 -d : /etc/passwd | sort
aadarsha
adm
...
systemd-oom
tss
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sort /etc/passwd | cut -f 1 -d : 
aadarsha
...
systemd-oom
tss
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sort /etc/passwd | cut -f 1 -d : | less

# using uniq

[aadarsha@labserver ~]$ cut -f 7 d : /etc/passwd
cut: d: No such file or directory
cut: ':': No such file or directory
root:x:0:0:Super User:/root:/bin/bash
...
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 7 -d : /etc/passwd
/bin/bash
/usr/sbin/nologin
...
/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 7 -d : /etc/passwd | uniq
/bin/bash
/usr/sbin/nologin
...
/bin/bash
[aadarsha@labserver ~]$ 

# uniq --> continuous repeating is replaced by single 

[aadarsha@labserver ~]$ cut -f 7 -d : /etc/passwd | sort | uniq
/bin/bash
/bin/sync
/sbin/halt
/sbin/nologin
/sbin/shutdown
/usr/sbin/nologin
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 7 -d : /etc/passwd | sort | uniq -c
      2 /bin/bash
      1 /bin/sync
      1 /sbin/halt
      3 /sbin/nologin
      1 /sbin/shutdown
     13 /usr/sbin/nologin
[aadarsha@labserver ~]$ 

# tee --> it saves intermediate results of a pipeline
 
[aadarsha@labserver ~]$ ls 
dir1  dira  extracted  newfile1  passwd  secret_data  testcompany  testfile  words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cut -f 7 -d : /etc/passwd | tee cut_result | sort | tee sort_result | uniq -c | tee uniq_result
      2 /bin/bash
      1 /bin/sync
      1 /sbin/halt
      3 /sbin/nologin
      1 /sbin/shutdown
     13 /usr/sbin/nologin
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
cut_result  dira       newfile1  secret_data  testcompany  uniq_result
dir1        extracted  passwd    sort_result  testfile     words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat cut_result 
/bin/bash
...
/usr/sbin/nologin
/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat uniq_result 
      2 /bin/bash
      1 /bin/sync
      1 /sbin/halt
      3 /sbin/nologin
      1 /sbin/shutdown
     13 /usr/sbin/nologin
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat sort_result 
...
/usr/sbin/nologin
[aadarsha@labserver ~]$ 

# grep
# grep --> it searches for a pattern in a file and displays only those lines that have a matching pattern

# grep <pattern> [option] <argument>

# -v --> displays non-matching lines

# -V --> displays non-matching lines

# -c --> displays total count of lines

# -i --> ignores line

# -r --> searches resursive

[aadarsha@labserver ~]$ grep aadarsha /etc/passwd
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep aadar /etc/passwd
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep a /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
daemon:x:2:2:daemon:/sbin:/usr/sbin/nologin
...
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
chrony:x:997:996:chrony system user:/var/lib/chrony:/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

# matches pattern in all lines by default 

[aadarsha@labserver ~]$ grep iv /etc/passwd
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash -c /etc/passwd
2
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash /etc/passwd | wc
      2       4     100
[aadarsha@labserver ~]$ grep bash /etc/passwd | wc -l
2
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat /etc/passwd | grep bash
root:x:0:0:Super User:/root:/bin/bash
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat /etc/passwd | grep bash | wc -l
2
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat /etc/passwd | grep bash | wc
      2       4     100
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash -c /etc/passwd
2
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash -r /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash -v /etc/passwd
bin:x:1:1:bin:/bin:/usr/sbin/nologin
daemon:x:2:2:daemon:/sbin:/usr/sbin/nologin
...
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi testdatafile

[aadarsha@labserver ~]$ grep boy testdatafile 
i'm a good boy
are you a boy or girl?
[aadarsha@labserver ~]$

[aadarsha@labserver ~]$ vi testfile 

[aadarsha@labserver ~]$ vi testdatafile 

[aadarsha@labserver ~]$ grep ^boy testdatafile 
 
[aadarsha@labserver ~]$ grep boy$ testdatafile 
i'm a good boy
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
...
chrony:x:997:996:chrony system user:/var/lib/chrony:/sbin/nologin
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep ^s /etc/passwd
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
...
chrony:x:997:996:chrony system user:/var/lib/chrony:/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep bash$ /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi testdatafile 

[aadarsha@labserver ~]$ grep boy testdatafile 
i'm a good boy
is this boy who is the best man in the world
are you a boy or girl?
Boy is running and you boy?
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep Boy testdatafile 
Boy is running and you boy?
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep -i Boy testdatafile 
i'm a good boy
is this boy who is the best man in the world
are you a boy or girl?
Boy is running and you boy?
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep -i Boy|girl testdatafile 
-bash: girl: command not found
^Z  
[1]+  Stopped                 grep --color=auto -i Boy | girl testdatafile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ egrep 'boy|girl' testdatafile 
i'm a good boy
this is the beautiful girl 
is this boy who is the best man in the world
are you a boy or girl?
Boy is running and you boy?
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep -e boy -e girl testdatafile 
i'm a good boy
this is the beautiful girl 
is this boy who is the best man in the world
are you a boy or girl?
Boy is running and you boy?
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi testthis 

[aadarsha@labserver ~]$ whoami
aadarsha
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep -r good .
./dira/words:aftergood
./dira/words:a-good
./dira/words:Allgood
...
./testdatafile:good morning 
./testthis:are you good or bad
./testthis:I want good
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep -r good 
dira/words:aftergood
dira/words:a-good
dira/words:Allgood
...
grep: extracted/var/log/cron: Permission denied
testdatafile:i'm a good boy
testdatafile:good morning 
testthis:are you good or bad
testthis:I want good
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ su - root 
Password: 
Last login: Thu Jun 11 05:12:59 +0545 2026 on pts/0
[root@labserver ~]# 

[root@labserver ~]# grep -r good /etc/
/etc/lvm/lvm.conf:	# to have good deduplication rates but compression is still desired.
/etc/lvm/lvm.conf:	#     device if there is only one remaining good copy.
/etc/nftables/osf/pf.os:#   A fairly good method is to simply round the observed TTL up to
/etc/bashrc:# It's NOT a good idea to change this file unless you know what you
/etc/profile:# It's NOT a good idea to change this file unless you know what you
/etc/firewalld/firewalld.conf:# time that is needed to apply changes and to start the daemon, but is good for
grep: /etc/udev/hwdb.bin: binary file matches
[root@labserver ~]# 

[root@labserver ~]# cd 

[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep -r boy .
./words:boycottage
...
grep: ./extracted/var/log/cron: Permission denied
./testdatafile:i'm a good boy
./testdatafile:is this boy who is the best man in the world
./testdatafile:are you a boy or girl?
./testdatafile:Boy is running and you boy?
./testthis:boy and girl are beautiful
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ grep -r boy . | wc -l
grep: ./extracted/etc/libaudit.conf: Permission denied
...
grep: ./extracted/var/log/firewalld: Permission denied
grep: ./extracted/var/log/cron: Permission denied
481
[aadarsha@labserver ~]$ 

# sed

# using 'sed'

# sed --> perfroms search and replace operations

[aadarsha@labserver ~]$ ls
cut_result  dira       newfile1  secret_data  testcompany   testfile  uniq_result
dir1        extracted  passwd    sort_result  testdatafile  testthis  words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat passwd 
root:x:0:0:Super User:/root:/bin/bash
...
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 
 
# sed

[aadarsha@labserver ~]$ sed 's/nologin/YESLOGIN/g' /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
...
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/YESLOGIN
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sed 's/nologin/YESLOGIN/g' /etc/passwd > passwd_YESLOGIN

[aadarsha@labserver ~]$ ls
cut_result  extracted  passwd_YESLOGIN  testcompany   testthis
dir1        newfile1   secret_data      testdatafile  uniq_result
dira        passwd     sort_result      testfile      words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat passwd_YESLOGIN 
root:x:0:0:Super User:/root:/bin/bash
...
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/YESLOGIN
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat passwd
root:x:0:0:Super User:/root:/bin/bash
...
lp:x:4:7:lp:/var/spool/lpd:/usr/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sed -e 's/bash/CASH/g' -e 's/nologin/YESLOGIN/g' /etc/passwd > multiple_changes

[aadarsha@labserver ~]$ ls
cut_result  extracted         passwd           sort_result   testfile     words
dir1        multiple_changes  passwd_YESLOGIN  testcompany   testthis
dira        newfile1          secret_data      testdatafile  uniq_result
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat multiple_changes 
root:x:0:0:Super User:/root:/bin/CASH
...
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/YESLOGIN
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/CASH
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat testdatafile 
i'm a good boy
this is the beautiful girl 
how are you 
is this your baby boy?
boy understand fundamental concepts
good morning 
why are visiting the USA?
Boy is running?
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sed -i 's/boy/BOY/g' testdatafile 

[aadarsha@labserver ~]$ cat testdatafile 
i'm a good BOY
this is the beautiful girl 
how are you 
is this your baby BOY?
BOY understand the fundamental concepts
good morning 
why are visiting the USA?
Boy is running?
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ 

# To search whole line

[aadarsha@labserver ~]$ grep 'boy and' testdatafile 
[aadarsha@labserver ~]$ 
```
---
### File Searching and finding
```
# File Search

      # Searching Files using 'find' command
      
      # find [path] [options] <argument>

      # -atime <+N/-N/N> ---> performs access-time bases search (N-->Day)

      # -mtime <+N/-N/N> ---> performs modification-time bases search (N-->Day)

      # -ntime <name> ---> performs name-based bases search (case sensitive)
      
      # -intime <name> ---> performs name-based bases search (case insensitive)

      # -size <+N/-N/N> --> performs size-based search

      # -user <owner> --> performs user-ownership based search

      # -group <group> --> performs group-ownership based search

      # -type <type> --> 

[aadarsha@labserver ~]$ find / -name passwd 2>/dev/null 
/sys/fs/selinux/class/passwd
/sys/fs/selinux/class/passwd/perms/passwd
/etc/passwd
/etc/pam.d/passwd
/usr/bin/passwd
/usr/share/bash-completion/completions/passwd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /etc -name passwd 2>/dev/null 
/etc/passwd
/etc/pam.d/passwd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ touch /tmp/passwd

[aadarsha@labserver ~]$ find / -name passwd 2>/dev/null 
/sys/fs/selinux/class/passwd
/sys/fs/selinux/class/passwd/perms/passwd
/tmp/passwd
/etc/passwd
/etc/pam.d/passwd
/usr/bin/passwd
/usr/share/bash-completion/completions/passwd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ touch /tmp/Passwd
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

[aadarsha@labserver ~]$ find / -iname passwd 2>/dev/null 
/sys/fs/selinux/class/passwd
/sys/fs/selinux/class/passwd/perms/passwd
/tmp/passwd
/tmp/Passwd
/etc/passwd
/etc/pam.d/passwd
/usr/bin/passwd
/usr/share/bash-completion/completions/passwd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find / -iname passwd 2>/dev/null | wc -l
8
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find / -name passwd 2>/dev/null  | wc -l
7
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find / -atime -7 | wc -l
find: ‘/boot/efi/EFI/centos’: Permission denied
find: ‘/boot/grub2’: Permission denied
find: ‘/boot/loader/entries’: Permission denied
find: ‘/proc/tty/driver’: Permission denied
find: ‘/proc/1/task/1/fd’: Permission denied
...
find: ‘/usr/libexec/initscripts/legacy-actions/auditd’: Permission denied
find: ‘/home/aadarsha/extracted/var/log/private’: Permission denied
find: ‘/home/aadarsha/extracted/var/log/samba’: Permission denied
find: ‘/home/aadarsha/extracted/var/log/audit’: Permission denied
find: ‘/home/aadarsha/extracted/var/log/sssd’: Permission denied
find: ‘/home/aadarsha/extracted/var/log/chrony’: Permission denied
87254
[aadarsha@labserver ~]$

[aadarsha@labserver ~]$ find / -atime -7 2>/dev/null | wc -l
87254
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find / -mtime -7 2>/dev/null | wc -l            # modified
77799
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /etc -mtime -7 2>/dev/null | wc -l         # modified
7
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /etc -atime -7 2>/dev/null | wc -l         # accessed
685
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /etc -mtime -7 2>/dev/null                 # modified
/etc
/etc/resolv.conf
/etc/profile.d
/etc/bashrc
/etc/profile
/etc/ld.so.cache
/etc/pkgconfig
[aadarsha@labserver ~]$ 

# focused auditing and non focused auditing in security auditing

# focused auditing and non-focused auditing in security auditing

# safety and performance can not be achieved at a time

# finding based on size

[aadarsha@labserver ~]$ ls /var/l
lib/   local/ lock/  log/  

[aadarsha@labserver ~]$ find /var -size +10M >/dev/null 
find: ‘/var/lib/selinux/targeted/active’: Permission denied
find: ‘/var/lib/selinux/final’: Permission denied
find: ‘/var/lib/private’: Permission denied
...
find: ‘/var/tmp/systemd-private-6d5f834212da46da9320190dce0efa12-systemd-logind.service-WsqjBK’: Permission denied
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ find /var -size +10M 2>/dev/null 
/var/cache/dnf/appstream-25519c512d836b42/repodata/28fcbe99d124870c057e25d821ae6d6d4c8fa1e58c8609ffe3511cdf0fd53b66-filelists.xml.gz
/var/cache/dnf/appstream-filenames.solvx

[aadarsha@labserver ~]$ find /var -size +10M 2>/dev/null 
/var/cache/dnf/appstream-25519c512d836b42/repodata/28fcbe99d124870c057e25d821ae6d6d4c8fa1e58c8609ffe3511cdf0fd53b66-filelists.xml.gz
/var/cache/dnf/appstream-filenames.solvx

[aadarsha@labserver ~]$ ls -lh /var/cache/dnf/appstream-25519c512d836b42/repodata/28fcbe99d124870c057e25d821ae6d6d4c8fa1e58c8609ffe3511cdf0fd53b66-filelists.xml.gz 
-rw-r--r--. 1 root root 16M Jun  8 09:51 /var/cache/dnf/appstream-25519c512d836b42/repodata/28fcbe99d124870c057e25d821ae6d6d4c8fa1e58c8609ffe3511cdf0fd53b66-filelists.xml.gz
```
---