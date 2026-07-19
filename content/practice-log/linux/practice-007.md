---
title: "Practice 006 — Working with Vi Editor"
date: 2026-06-11
draft: false
---

### Terminal Session

```
# Modes of vi editor
# 1. Insert Mode
# 2. Command Mode
# 3. Ex Mode
# 4. Visual Mode


# 1. Insert Mode
# i
# ESC
# :wq   OR   :x     -->  Save & Exit


# 2. Command Mode
# cursor movement

# G --> move to the bottom of the file
# gg --> Move to the top 

# 'N'G --> Move to the n(th) line

[aadarsha@labserver ~]$ ls
dir1  extracted     newimpfiles.tar.gz  testfile  words.gz
dira  impfiles.tar  testcompany         words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cp /etc/passwd .

[aadarsha@labserver ~]$ ls
dir1  extracted     newimpfiles.tar.gz  testcompany  words
dira  impfiles.tar  passwd              testfile     words.gz
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi passwd
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ cd
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi .vimrc
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ source .vimrc 
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sudo su
[sudo] password for aadarsha: 
[root@labserver aadarsha]# 
 
# vi
# vim = vi + extra features
[root@labserver aadarsha]# 

[root@labserver aadarsha]# yum list vim*
...
[root@labserver aadarsha]# 

[root@labserver aadarsha]# exit
exit
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  extracted     newimpfiles.tar.gz  testcompany  words
dira  impfiles.tar  passwd              testfile     words.gz

[aadarsha@labserver ~]$ ls -la
total 18544
drwx------. 6 aadarsha aadarsha     4096 Jun  9 05:19 .
drwxr-xr-x. 3 root     root           22 May 26 15:36 ..
-rw-------. 1 aadarsha aadarsha    16303 Jun  8 22:34 .bash_history
-rw-r--r--. 1 aadarsha aadarsha       18 Oct 29  2024 .bash_logout
-rw-r--r--. 1 aadarsha aadarsha      144 Oct 29  2024 .bash_profile
-rw-r--r--. 1 aadarsha aadarsha      609 Jun  8 07:20 .bashrc
drwxr-xr-x. 3 aadarsha aadarsha       33 Jun  8 06:47 dir1
drwxr-xr-x. 3 aadarsha aadarsha       65 Jun  8 12:07 dira
drwxr-xr-x. 4 root     root           28 Jun  8 20:16 extracted
-rw-r--r--. 1 root     root     11704320 Jun  8 16:38 impfiles.tar
-rw-------. 1 aadarsha aadarsha       51 Jun  8 21:54 .lesshst
-rw-r--r--. 1 root     root       784897 Jun  8 16:33 newimpfiles.tar.gz
-rw-r--r--. 1 aadarsha aadarsha     1080 Jun  9 05:17 passwd
-rw-r--r--. 1 aadarsha aadarsha       24 Jun  7 07:54 .secretdata
drwxr-xr-x. 5 aadarsha aadarsha       54 Jun  7 07:24 testcompany
-rw-r--r--. 1 aadarsha aadarsha       56 Jun  7 21:33 testfile
-rw-------. 1 aadarsha aadarsha     7727 Jun  8 08:47 .viminfo
-rw-r--r--. 1 aadarsha aadarsha       11 Jun  9 05:18 .vimrc
-rw-r--r--. 1 aadarsha aadarsha  4953598 Jun  8 09:52 words
-rw-r--r--. 1 aadarsha aadarsha  1476067 Jun  8 12:09 words.gz
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vim .vimrc 

[aadarsha@labserver ~]$ vi passwd 

[aadarsha@labserver ~]$ cat .vimrc 
set number
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -a
.              .bash_logout   dir1       impfiles.tar        passwd       testfile  words
..             .bash_profile  dira       .lesshst            .secretdata  .viminfo  words.gz
.bash_history  .bashrc        extracted  newimpfiles.tar.gz  testcompany  .vimrc
[aadarsha@labserver ~]$ vi .vimrc 

[aadarsha@labserver ~]$ vi passwd 

[aadarsha@labserver ~]$ vi .vimrc 

[aadarsha@labserver ~]$ vi .vimrc

[aadarsha@labserver ~]$ cat .vimrc 
set nu
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi passwd 

# not working above --> set number permanently 

[aadarsha@labserver ~]$ vi +10 passwd          # set cursor in line number 10

[aadarsha@labserver ~]$ vi +18 passwd          # set cursor in line number 18

 
# Delete, Copy and Paste

# yy --> copy the current line

# 'N'yy --> copy the next N lines

# dd --> Delete the current line

# 'N'dd --> Delete the next N lines

# p --> To paste the copied/deleted text

# u --> To undo the last change

# CTRL + r --> To redo the last change

[aadarsha@labserver ~]$ vi passwd 


# Ex Mode

# Searching Text

# /<text>

# n --> next 

# N --> backward next

# same as in man page

# Search and Replace Text

# :%s/current text/new text/c  --> search and replace in the whole file with confirmation

# :%s/current text/new text/g  --> search and replace in the whole file without confirmation

# :%5,15s/current text/new text/ c or g  --> search and replace in the whole file with or without confirmation in the given lines

[aadarsha@labserver ~]$ vi passwd 

# Visual Mode

[aadarsha@labserver ~]$ vi passwd 

# v --> line oriented mode
# V --> Block oriented visual mode

# y --> 
# d --> 
# p -->

[aadarsha@labserver ~]$ vi passwd 

# V --> line oriented visual mode

# v --> line oriented mode

[aadarsha@labserver ~]$ vi passwd 

# Password Protect a file

[aadarsha@labserver ~]$ vi secret_data

[aadarsha@labserver ~]$ ls
dir1  extracted     newimpfiles.tar.gz  secret_data  testfile  words.gz
dira  impfiles.tar  passwd              testcompany  words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat secret_data 
this is secret file
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ vim passwd 

[aadarsha@labserver ~]$ vim passwd 

[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ alias vi='vim'

[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ vi secret_data 
 
[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ vi secret_data 

[aadarsha@labserver ~]$ vim .bashrc 

[aadarsha@labserver ~]$ sudo su
[sudo] password for aadarsha: 
[root@labserver aadarsha]# 

[root@labserver aadarsha]# vim .bashrc 
[root@labserver aadarsha]# exit
exit
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vi secret_data 
[aadarsha@labserver ~]$ 

# File Recovery after crash

[aadarsha@labserver ~]$ whoami
aadarsha
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  extracted     newimpfiles.tar.gz  secret_data  testfile  words.gz
dira  impfiles.tar  passwd              testcompany  words
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ rm -r newimpfiles.tar.gz words.gz impfiles.tar 
rm: remove write-protected regular file 'newimpfiles.tar.gz'? 
rm: remove regular file 'words.gz'? y
rm: remove write-protected regular file 'impfiles.tar'? y
[aadarsha@labserver ~]$ y
-bash: y: command not found
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ rm -r newimpfiles.tar.gz words.gz impfiles.tar 
rm: remove write-protected regular file 'newimpfiles.tar.gz'? y
rm: cannot remove 'words.gz': No such file or directory
rm: cannot remove 'impfiles.tar': No such file or directory
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  extracted  passwd  secret_data  testcompany  testfile  words
[aadarsha@labserver ~]$ 
 
[aadarsha@labserver ~]$ ls
dir1  dira  extracted  passwd  secret_data  testcompany  testfile  words
[aadarsha@labserver ~]$ 

# swap file or guard file --> created to protect the currently opening file when sudden incident like poweroff or terminate 

# recover from .swp file

# vir -r passwd.swp

[aadarsha@labserver ~]$ 
```
