---
title: "Practice 005 — Compression and Archiving "
date: 2026-06-09
draft: false
---

### Terminal Session

#### Compression and Archiving

> compression and archiving are different concepts

- Archiving packages files together.
- Compression reduces the size of the archive.

```
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile
[aadarsha@labserver ~]$ 

# Compression

[root@labserver ~]# yum whatprovides /usr/share/dict/words
...

[root@labserver ~]# yum install -y words
Last metadata expiration check: 0:00:56 ago on Mon 08 Jun 2026 09:51:15 AM +0545.
Dependencies resolved.
...
Installed:
  words-3.0-47.el10.noarch                                                                                                                  
Complete!
[root@labserver ~]# ls /usr/share/dict/
linux.words  words
[root@labserver ~]# 

[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ cp /usr/share/dict/words .

[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls -l words 
-rw-r--r--. 1 aadarsha aadarsha 4953598 Jun  8 09:52 words
[aadarsha@labserver ~]$ ls -lh words 
-rw-r--r--. 1 aadarsha aadarsha 4.8M Jun  8 09:52 words
[aadarsha@labserver ~]$ 

# we will use words to practice compression

# compression tools
    # 1. gzip (.gz)    -->  gunzip
    # 2. bzip2 (.)     -->  bunzip2
    # 2. bzip2 (.bz2)  -->  bunzip2
    # 3. zip (.zip)    -->  unzip2           # compatible to cross platform

    # Some history commands:

    ls -lh words 
    rpm -q gzip
    rpm -q bgzip2
    rpm -q bzip2
    rpm -q zip
    which gzip
    which gunzip
    which bzip2
    which zip
    su - root
    ls
    ll words 
    ll -h words 
    gzip words 
    ls
    ls -lh words.gz 
    gunzip words.gz 
    ls
    ls -lh words 
    gzip -v words 
    ls
    ls -lh words.gz 
    gunzip words.gz 
    ls
    bzip2 -v words 
    ls 
    ls -lh words.bz2 
    bunzip2 words.bz2 
    ls
    ls -lh words 
    ls
    zip -o compressed-words.zip words 
    ls
    ls -lh compressed-words.zip 
    unzip compressed-words.zip 
    ls
    unzip -l compressed-words.zip 
    unzip -v compressed-words.zip
    unzip -p compressed-words.zip
    history

[aadarsha@labserver ~]$ unzip -p compressed-words.zip words | head
1080
10-point
10th
11-point
12-point
16-point
18-point
1st
2
20-point
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ unzip -p compressed-words.zip words | head
1080
10-point
10th
11-point
12-point
16-point
18-point
1st
2
20-point
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
compressed-words.zip  dir1  dira  testcompany  testfile  words
[aadarsha@labserver ~]$ rm -r compressed-words.zip 
rm: remove regular file 'compressed-words.zip'? y
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -lh words 
-rw-r--r--. 1 aadarsha aadarsha 4.8M Jun  8 09:52 words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ which gz
gzexe  gzip   
[aadarsha@labserver ~]$ which gzip 
/usr/bin/gzip
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ gzip words 
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words.gz

[aadarsha@labserver ~]$ ll -lh words.gz 
-rw-r--r--. 1 aadarsha aadarsha 1.5M Jun  8 09:52 words.gz
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ gunzip words.gz words.gz 
gzip: words.gz: No such file or directory
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words
[aadarsha@labserver ~]$ gzip words 

[aadarsha@labserver ~]$ ls -lh words.gz 
-rw-r--r--. 1 aadarsha aadarsha 1.5M Jun  8 09:52 words.gz
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ gunzip words.gz 
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words
[aadarsha@labserver ~]$ 
 
[aadarsha@labserver ~]$ gzip -v words 
words:	 70.2% -- replaced with words.gz
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -lh words.gz 
-rw-r--r--. 1 aadarsha aadarsha 1.5M Jun  8 09:52 words.gz
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ gunzip words.gz 
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words
[aadarsha@labserver ~]$

[aadarsha@labserver ~]$ bzip2 -v words 
  words:    2.893:1,  2.766 bits/byte, 65.43% saved, 4953598 in, 1712421 out.
[aadarsha@labserver ~]$ ls -lh words.bz2 
-rw-r--r--. 1 aadarsha aadarsha 1.7M Jun  8 09:52 words.bz2
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words.bz2
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -lh words.bz2 
-rw-r--r--. 1 aadarsha aadarsha 1.7M Jun  8 09:52 words.bz2
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ bunzip2 words.bz2 
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -lh words 
-rw-r--r--. 1 aadarsha aadarsha 4.8M Jun  8 09:52 words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ zip -o words.zip words 
  adding: words (deflated 70%)
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -l
total 6288
drwxr-xr-x. 3 aadarsha aadarsha      33 Jun  8 06:47 dir1
drwxr-xr-x. 3 aadarsha aadarsha      52 Jun  8 09:39 dira
drwxr-xr-x. 5 aadarsha aadarsha      54 Jun  7 07:24 testcompany
-rw-r--r--. 1 aadarsha aadarsha      56 Jun  7 21:33 testfile
-rw-r--r--. 1 aadarsha aadarsha 4953598 Jun  8 09:52 words
-rw-r--r--. 1 aadarsha aadarsha 1476203 Jun  8 09:52 words.zip
[aadarsha@labserver ~]$ ls -lh words
-rw-r--r--. 1 aadarsha aadarsha 4.8M Jun  8 09:52 words
[aadarsha@labserver ~]$

[aadarsha@labserver ~]$ ls -lh words.zip 
-rw-r--r--. 1 aadarsha aadarsha 1.5M Jun  8 09:52 words.zip
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ unzip words.zip 
Archive:  words.zip
replace words? [y]es, [n]o, [A]ll, [N]one, [r]ename: N
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words  words.zip
[aadarsha@labserver ~]$ unzip words.zip dira/
Archive:  words.zip
caution: filename not matched:  dira/
[aadarsha@labserver ~]$ unzip words.zip dira/words
Archive:  words.zip
caution: filename not matched:  dira/words
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words  words.zip
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ unzip -l words.zip 
Archive:  words.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
  4953598  06-08-2026 09:52   words
---------                     -------
  4953598                     1 file
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ unzip -v compressed-words.zip
unzip:  cannot find or open compressed-words.zip, compressed-words.zip.zip or compressed-words.zip.ZIP.
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ unzip -v words.zip
Archive:  words.zip
 Length   Method    Size  Cmpr    Date    Time   CRC-32   Name
--------  ------  ------- ---- ---------- ----- --------  ----
 4953598  Defl:N  1476043  70% 06-08-2026 09:52 b81d644b  words
--------          -------  ---                            -------
 4953598          1476043  70%                            1 file
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ unzip words.zip -d dira
Archive:  words.zip
  inflating: dira/words              
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words  words.zip
[aadarsha@labserver ~]$  
[aadarsha@labserver ~]$ ls dira/
dirb  letfile  myfile_hard  words
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words  words.zip
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ rm -r words.zip 
rm: remove regular file 'words.zip'? 
[aadarsha@labserver ~]$ rm -r words.zip 
rm: remove regular file 'words.zip'? y
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ gzip -vc words >words.gz
words:	 70.2% -- replaced with stdout
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls 
dir1  dira  testcompany  testfile  words  words.gz
[aadarsha@labserver ~]$ 
```

#### Archiving

```
# Archiving Files/Dirs (tar)

# what is archiving?
# --> 

# tar [options] <files/dirs to be archived>

#    -c                     -->  create archive
#    -v                     -->  verbose (show details)
#    -f <archive filename>  -->  to define archive file 
#    -t                     -->  list contents of archive
#    -r                     -->  add new files into an archive 
#    -z                     -->  to compress archive using gzip ( use .tar.gz or .tgz filename extension )
#    -j                     -->  to compress archive using bgzip2 ( use .tar.gz2 or .tbz filename extension )

aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.170
aadarsha@192.168.254.170's password: 
Last login: Mon Jun  8 16:00:16 2026 from 192.168.254.152
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words  words.gz
[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile  words  words.gz
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Mon Jun  8 11:21:38 +0545 2026 on pts/1
[root@labserver ~]# 

# tar -cvf impfiles.tar              # here -f must be at last after which we can write the file name

[root@labserver ~]# ls
anaconda-ks.cfg
[root@labserver ~]# cd /home/aadarsha/
[root@labserver aadarsha]# ls
dir1  dira  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# 


[root@labserver aadarsha]# #  tar -cvf impfiles.tar /etc/*.conf /var/log /etc/hosts        # archive all the files of .conf extension from /etc/, /var/log and /etc/hosts 
[root@labserver aadarsha]# 

[root@labserver aadarsha]# tar -cvf impfiles.tar /etc/*.conf /var/log /etc/hosts 
-bash: tar: command not found
[root@labserver aadarsha]# 

[root@labserver aadarsha]# tar -cvf impfiles.tar /etc/*.conf /var/log /etc/hosts 
-bash: tar: command not found
[root@labserver aadarsha]# 

[root@labserver aadarsha]# which tar
/usr/bin/which: no tar in (/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin)
[root@labserver aadarsha]# 

[root@labserver aadarsha]# rpm -q tar
package tar is not installed
[root@labserver aadarsha]# 

[root@labserver aadarsha]# yum install -y tar
Last metadata expiration check: 4:20:18 ago on Mon 08 Jun 2026 12:06:47 PM +0545.
Complete!
[root@labserver aadarsha]# 

[root@labserver aadarsha]# tar --version
tar (GNU tar) 1.35
Copyright (C) 2023 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by John Gilmore and Jay Fenlason.
[root@labserver aadarsha]# 

[root@labserver aadarsha]# tar -cvf impfiles.tar /etc/*.conf /var/log /etc/hosts 
tar: Removing leading `/' from member names
/etc/chrony.conf
tar: Removing leading `/' from hard link targets
/etc/dracut.conf
/etc/host.conf
/etc/kdump.conf
/etc/krb5.conf
/etc/ld.so.conf
...
/var/log/firewalld
/var/log/cron
/var/log/dnf.log
/var/log/dnf.librepo.log
/var/log/dnf.rpm.log
/var/log/hawkey.log
/etc/hosts
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls 
dir1  dira  impfiles.tar  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls -lh impfiles.tar 
-rw-r--r--. 1 root root 6.5M Jun  8 16:27 impfiles.tar
[root@labserver aadarsha]# 
 
[root@labserver aadarsha]# gzip impfiles.tar 
[root@labserver aadarsha]# 
[root@labserver aadarsha]# ls
dir1  dira  impfiles.tar.gz  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls -lh impfiles.tar.gz 
-rw-r--r--. 1 root root 767K Jun  8 16:27 impfiles.tar.gz
[root@labserver aadarsha]# 

[root@labserver aadarsha]# 
[root@labserver aadarsha]# # better compress file while creating archive using -z
[root@labserver aadarsha]# 

[root@labserver aadarsha]# 
[root@labserver aadarsha]# tar -cvzf newimpfiles.tar /etc/*.conf /var/log /etc/hosts 
tar: Removing leading `/' from member names
/etc/chrony.conf
tar: Removing leading `/' from hard link targets
/etc/dracut.conf
/etc/host.conf
...
/var/log/firewalld
/var/log/cron
/var/log/dnf.log
/var/log/dnf.librepo.log
/var/log/dnf.rpm.log
/var/log/hawkey.log
/etc/hosts
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls
dir1  dira  impfiles.tar.gz  newimpfiles.tar  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls -lh newimpfiles.tar 
-rw-r--r--. 1 root root 767K Jun  8 16:30 newimpfiles.tar
[root@labserver aadarsha]#

[root@labserver aadarsha]# rm -r newimpfiles.tar 
rm: remove regular file 'newimpfiles.tar'? 
[root@labserver aadarsha]# rm -r newimpfiles.tar 
rm: remove regular file 'newimpfiles.tar'? y
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls
dir1  dira  impfiles.tar.gz  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# 

# tar -cvzf newimpfiles.tar.gz /etc/*.conf /var/log /etc/hosts    (  use extension .tar.gz   or .tgz   not to confuse )

[root@labserver aadarsha]# tar -cvzf newimpfiles.tar.gz /etc/*.conf /var/log /etc/hosts 
tar: Removing leading `/' from member names
/etc/chrony.conf
tar: Removing leading `/' from hard link targets
/etc/dracut.conf
/etc/host.conf
/etc/kdump.conf
...
/etc/hosts
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls 
dir1  dira  impfiles.tar.gz  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls -lh newimpfiles.tar.gz 
-rw-r--r--. 1 root root 767K Jun  8 16:33 newimpfiles.tar.gz
[root@labserver aadarsha]# 

[root@labserver aadarsha]# gunzip impfiles.tar.gz 
[root@labserver aadarsha]# ls
dir1  dira  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# ls -lh 
total 14M
drwxr-xr-x. 3 aadarsha aadarsha   33 Jun  8 06:47 dir1
drwxr-xr-x. 3 aadarsha aadarsha   65 Jun  8 12:07 dira
-rw-r--r--. 1 root     root     6.5M Jun  8 16:27 impfiles.tar
-rw-r--r--. 1 root     root     767K Jun  8 16:33 newimpfiles.tar.gz
drwxr-xr-x. 5 aadarsha aadarsha   54 Jun  7 07:24 testcompany
-rw-r--r--. 1 aadarsha aadarsha   56 Jun  7 21:33 testfile
-rw-r--r--. 1 aadarsha aadarsha 4.8M Jun  8 09:52 words
-rw-r--r--. 1 aadarsha aadarsha 1.5M Jun  8 12:09 words.gz
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls -lh impfiles.tar 
-rw-r--r--. 1 root root 6.5M Jun  8 16:27 impfiles.tar
[root@labserver aadarsha]# 

[root@labserver aadarsha]# tar -t impfiles.tar
tar: Refusing to read archive contents from terminal (missing -f option?)
tar: Error is not recoverable: exiting now
[root@labserver aadarsha]# 

[root@labserver aadarsha]# tar -tf impfiles.tar
etc/chrony.conf
...
etc/hosts
[root@labserver aadarsha]# 
 
[root@labserver aadarsha]# tar -tvf impfiles.tar 
-rw-r--r-- root/root      1382 2025-11-20 05:45 etc/chrony.conf
-rw-r--r-- root/root       117 2026-01-30 05:45 etc/dracut.conf
...
-rw-r--r-- root/root        28 2024-02-01 03:47 etc/ld.so.conf
...
-rw-r--r-- root/root      2844 2026-06-08 16:27 var/log/dnf.rpm.log
-rw-r--r-- root/root      1380 2026-06-08 16:27 var/log/hawkey.log
-rw-r--r-- root/root       384 2023-11-29 16:19 etc/hosts
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls
dir1  dira  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# 

# add another file in archive .tar use -r  but not cannot add in .tar.gz

[root@labserver aadarsha]# tar -rvf impfiles.tar /etc/passwd dira
tar: Removing leading `/' from member names
/etc/passwd
tar: Removing leading `/' from hard link targets
dira/
dira/dirb/
dira/dirb/dirc/
dira/dirb/dirc/dird/
dira/dirb/dirc/dird/dire/
dira/letfile
dira/myfile_hard
dira/words
[root@labserver aadarsha]# 

# -x --> to extract contents from archive 

# previous archive

[aadarsha@labserver ~]$ ls
dir1  dira  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[aadarsha@labserver ~]$

[aadarsha@labserver ~]$ su - root
Password: 
su: Authentication failure
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Mon Jun  8 16:19:02 +0545 2026 on pts/0
[root@labserver ~]# 

[root@labserver ~]# ls
anaconda-ks.cfg
[root@labserver ~]# cd /home/aadarsha/
[root@labserver aadarsha]# ls
dir1  dira  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls
dir1  dira  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# 

[root@labserver aadarsha]# mkdir extracted

[root@labserver aadarsha]# ls
dir1  dira  extracted  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# cd extracted/
[root@labserver extracted]# 

[root@labserver extracted]# ls
[root@labserver extracted]# pwd
/home/aadarsha/extracted
[root@labserver extracted]# 
[root@labserver extracted]# ls -lh ../impfiles.tar 
-rw-r--r--. 1 root root 12M Jun  8 16:38 ../impfiles.tar
[root@labserver extracted]# 
[root@labserver extracted]# exit
logout
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  extracted  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[aadarsha@labserver ~]$ cd extracted/
[aadarsha@labserver extracted]$ 

[aadarsha@labserver extracted]$ pwd
/home/aadarsha/extracted
[aadarsha@labserver extracted]$ 

[aadarsha@labserver extracted]$ tar -xvf ../impfiles.tar 
etc/chrony.conf
tar: etc: Cannot mkdir: Permission denied
tar: etc/chrony.conf: Cannot open: No such file or directory
etc/dracut.conf
tar: etc: Cannot mkdir: Permission denied
tar: etc/dracut.conf: Cannot open: No such file or directory
etc/host.conf
...
tar: dira: Cannot mkdir: Permission denied
tar: dira/words: Cannot open: No such file or directory
tar: Exiting with failure status due to previous errors
[aadarsha@labserver extracted]$ 

[aadarsha@labserver extracted]$ ls
[aadarsha@labserver extracted]$ 
[aadarsha@labserver extracted]$ cd ..
[aadarsha@labserver ~]$ ls -ld extracted/
drwxr-xr-x. 2 root root 6 Jun  8 20:07 extracted/
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sudo su
[sudo] password for aadarsha: 
[root@labserver aadarsha]# 

[root@labserver aadarsha]# ls
dir1  dira  extracted  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# cd extracted/
[root@labserver extracted]# 

[root@labserver extracted]# tar -xvf ../impfiles.tar 
etc/chrony.conf
etc/dracut.conf
etc/host.conf
...
dira/myfile_hard
dira/words
[root@labserver extracted]# 

[root@labserver extracted]# ls
dira  etc  var
[root@labserver extracted]# cd etc/
[root@labserver etc]# ls
chrony.conf  host.conf  kdump.conf  ld.so.conf     locale.conf     man_db.conf  nsswitch.conf  request-key.conf  rsyslog.conf   sudo.conf       sysctl.conf    xattr.conf
dracut.conf  hosts      krb5.conf   libaudit.conf  logrotate.conf  mke2fs.conf  passwd         resolv.conf       sestatus.conf  sudo-ldap.conf  vconsole.conf  yum.conf
[root@labserver etc]# cd ../var/

[root@labserver var]# ls
log
[root@labserver var]# cd log/
[root@labserver log]# ls
anaconda  audit  btmp  chrony  cron  dnf.librepo.log  dnf.log  dnf.rpm.log  firewalld  hawkey.log  lastlog  maillog  messages  private  samba  secure  spooler  sssd  wtmp
[root@labserver log]# cd ../../dira/
[root@labserver dira]# ls
dirb  letfile  myfile_hard  words
[root@labserver dira]# cd -
/home/aadarsha/extracted/var/log
[root@labserver log]# cd ../../
[root@labserver extracted]# ls
dira  etc  var
[root@labserver extracted]# 

[root@labserver extracted]# ls
dira  etc  var
[root@labserver extracted]# ls -lh
total 4.0K
drwxr-xr-x. 3 aadarsha aadarsha   65 Jun  8 12:07 dira
drwxr-xr-x. 2 root     root     4.0K Jun  8 20:10 etc
drwxr-xr-x. 3 root     root       17 Jun  8 20:10 var
[root@labserver extracted]# 

[root@labserver extracted]# ls var/log/
anaconda  audit  btmp  chrony  cron  dnf.librepo.log  dnf.log  dnf.rpm.log  firewalld  hawkey.log  lastlog  maillog  messages  private  samba  secure  spooler  sssd  wtmp
[root@labserver extracted]# 

# to extract compressed files use -z

[root@labserver extracted]# cd 
[root@labserver ~]# 
[root@labserver ~]# ls
anaconda-ks.cfg
[root@labserver ~]# cd /home/aadarsha/
[root@labserver aadarsha]# ls
dir1  dira  extracted  impfiles.tar  newimpfiles.tar.gz  testcompany  testfile  words  words.gz
[root@labserver aadarsha]# 

[root@labserver aadarsha]# cd extracted/
[root@labserver extracted]# 
[root@labserver extracted]# tar -xvf ../newimpfiles.tar.gz 
etc/chrony.conf
etc/dracut.conf
etc/host.conf
etc/kdump.conf
...
etc/hosts
[root@labserver extracted]# 

[root@labserver extracted]# ls
dira  etc  var
[root@labserver extracted]# ls -la
total 8
drwxr-xr-x. 5 root     root       40 Jun  8 20:10 .
drwx------. 6 aadarsha aadarsha 4096 Jun  8 20:07 ..
drwxr-xr-x. 3 aadarsha aadarsha   65 Jun  8 12:07 dira
drwxr-xr-x. 2 root     root     4096 Jun  8 20:13 etc
drwxr-xr-x. 3 root     root       17 Jun  8 20:10 var
[root@labserver extracted]# 

[root@labserver extracted]# tar -zxvf ../newimpfiles.tar.gz 
etc/chrony.conf
etc/dracut.conf
...
etc/hosts
[root@labserver extracted]#
[root@labserver extracted]# ls
dira  etc  var
[root@labserver extracted]# 

[root@labserver extracted]# ls
dira  etc  var
[root@labserver extracted]# 

[root@labserver extracted]# rm -r dira/ etc/ var/
...
[root@labserver extracted]# 

[root@labserver extracted]# ls

[root@labserver extracted]# tar -zxvf ../newimpfiles.tar.gz 
etc/chrony.conf
...
var/log/hawkey.log
etc/hosts
[root@labserver extracted]# ls
etc  var
[root@labserver extracted]# 
```