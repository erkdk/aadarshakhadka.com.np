---
title: "Practice 004 — Creating Soft Link & Hard Link "
date: 2026-06-08
draft: false
---

### Terminal Session

#### Command History

```
[aadarsha@labserver ~]$ ls limited/
host.conf  locale.conf  logrotate.conf
[aadarsha@labserver ~]$ 

aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.72
aadarsha@192.168.254.72's password: 
Last login: Mon Jun  8 06:45:39 2026 from 192.168.254.152
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
backup_conf  dir1  dira  limited  testcompany  testfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls backup_conf/
chrony.conf  kdump.conf  ld.so.conf   mke2fs.conf    request-key.conf  rsyslog.conf   sysctl.conf    xattr.conf
dracut.conf  krb5.conf   man_db.conf  nsswitch.conf  resolv.conf       sestatus.conf  vconsole.conf  yum.conf
[aadarsha@labserver ~]$ 

# command history and running already run history

[aadarsha@labserver ~]$ echo $HISTSIZE		            
1000                                               # ---> Default history limit : 1000
[aadarsha@labserver ~]$

# FIFO --> first entered command removes at first after reaching history limit number
 
[aadarsha@labserver ~]$ history
    1  ip a
    2  sudo -i
    3  clear
    4  ip a
    5  clear
    6  whoami
    7  ls
    8  clear
    9  ls
   10  clear
   11  hostname -I
   ...
[aadarsha@labserver ~]$ 
```

### Soft Link and Hard Link

```
Hard link = another name for the same inode.
Soft link = a shortcut that stores a path.


Hard link → “same file, different name”
Soft link → “different file pointing to a name (path)”


| Feature                           | Hard Link                      | Soft Link (Symbolic Link)              |
| --------------------------------- | ------------------------------ | -------------------------------------- |
| Creates with                      |  `ln source target`            | `ln -s source target`                  |
| Points to                         |  Same inode as original file   |  Pathname of target file               |
| Has its own inode?                |  No (shares inode with target) |  Yes (separate inode)                  |
| Inode number (`ls -i`)            |  Same as original              |  Different from target                 |
| If original file is deleted       |  Still works                   |  Breaks                                |
| If original file is renamed       |  Still works                   |  Breaks                                |
| If original file is moved         |  Still works                   |  Usually breaks                        |
| Can link directories?             |  Generally not allowed         |  Yes                                   |
| Can cross filesystems/partitions? |  No                            |  Yes                                   |
| Size of link                      |  Same file data                |  Small file containing path            |
| Permissions                       |  Same underlying file          |  Symlink permissions usually ignored   |
| Broken/Dangling possible?         |  No                            |  Yes                                   |
| Uses inode count                  |  Increases link count          |  Does not increase target's link count |

[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vim dir1/dir2/dir3/dir4/dir5/myfile

[aadarsha@labserver ~]$ cat dir1/dir2/dir3/dir4/dir5/myfile 
this is myfile created in dir5.
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile
[aadarsha@labserver ~]$ 

# creating soft link
 
[aadarsha@labserver ~]$ ln -s dir1/dir2/dir3/dir4/dir5/myfile .

[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile
[aadarsha@labserver ~]$ ls -l
total 4
drwxr-xr-x. 3 aadarsha aadarsha 33 Jun  8 06:47 dir1
drwxr-xr-x. 3 aadarsha aadarsha 33 Jun  8 06:47 dira
lrwxrwxrwx. 1 aadarsha aadarsha 31 Jun  8 07:53 myfile -> dir1/dir2/dir3/dir4/dir5/myfile
drwxr-xr-x. 5 aadarsha aadarsha 54 Jun  7 07:24 testcompany
-rw-r--r--. 1 aadarsha aadarsha 56 Jun  7 21:33 testfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ln -s dir1/dir2/dir3/dir4/dir5/myfile dira/

[aadarsha@labserver ~]$ ls -l dira/
total 0
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
lrwxrwxrwx. 1 aadarsha aadarsha 31 Jun  8 07:53 myfile -> dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat dira/myfile 
cat: dira/myfile: No such file or directory
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cd dira/
[aadarsha@labserver dira]$ ls
dirb  letfile  myfile
[aadarsha@labserver dira]$ cat myfile 
cat: myfile: No such file or directory
[aadarsha@labserver dira]$ 

[aadarsha@labserver dira]$ cat myfile 
cat: myfile: No such file or directory
[aadarsha@labserver dira]$ 

[aadarsha@labserver dira]$ ls -l myfile 
lrwxrwxrwx. 1 aadarsha aadarsha 31 Jun  8 07:53 myfile -> dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver dira]$ 

[aadarsha@labserver dira]$ ls
dirb  letfile  myfile
[aadarsha@labserver dira]$ 

[aadarsha@labserver dira]$ ls
dirb  letfile  myfile
[aadarsha@labserver dira]$ readlink myfile 
dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver dira]$ 

[aadarsha@labserver dira]$ cd
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile
[aadarsha@labserver ~]$ ls dir1/
dir2  numfile
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls dir1/
dir2/    numfile  
[aadarsha@labserver ~]$ ls dir1/dir2/dir3/dir4/dir5/
myfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile
[aadarsha@labserver ~]$ cd dira/
[aadarsha@labserver dira]$ ls
dirb  letfile  myfile
[aadarsha@labserver dira]$ rm myfile 
rm: remove symbolic link 'myfile'? y
[aadarsha@labserver dira]$ 

[aadarsha@labserver dira]$ cd
[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile
[aadarsha@labserver ~]$ tree dir1/dir2/dir3/dir4/dir5/
dir1/dir2/dir3/dir4/dir5/
└── myfile

1 directory, 1 file
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile
[aadarsha@labserver ~]$ ls dira/
dirb  letfile
[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ln -s ../dir1/dir2/dir3/dir4/dir5/myfile dira/myfile_soft
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls dira/
dirb  letfile  myfile_soft
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls -l dira/
total 0
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver ~]$ cat dira/myfile_soft 
this is myfile created in dir5.
[aadarsha@labserver ~]$ 

# OR using the absolute path is better approach

[aadarsha@labserver ~]$ vim dira/myfile_soft 
[aadarsha@labserver ~]$ cat dira/myfile_soft 
this is myfile created in dir5.
this line is added after creating soft link
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat dir1/dir2/dir3/dir4/dir5/myfile 
this is myfile created in dir5.
this line is added after creating soft link
[aadarsha@labserver ~]$ 

# since soft link is like pointer, when I change the soft link, original file also got changed

[aadarsha@labserver ~]$ ls -l dir1/dir2/dir3/dir4/dir5/myfile 
-rw-r--r--. 1 aadarsha aadarsha 77 Jun  8 08:33 dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -l dira/myfile_soft 
lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 dira/myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver ~]$ 

# size 77 bytes and softlink size 34 bytes  ---> size of a regular file represents the amount of data stored in the file.
#                                                 AND
#                                                 size of a symbolic (soft) link represents the number of bytes required to store the target pathname, not the size of the target file.


# creating hard link

[aadarsha@labserver ~]$ ln /home/aadarsha/dir1/dir2/dir3/dir4/dir5/myfile dira/myfile_hard

[aadarsha@labserver ~]$ ls -l dira/myfile_hard 
-rw-r--r--. 2 aadarsha aadarsha 77 Jun  8 08:33 dira/myfile_hard
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls dira/
dirb  letfile  myfile_hard  myfile_soft
[aadarsha@labserver ~]$ ls -l dira/
total 4
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
-rw-r--r--. 2 aadarsha aadarsha 77 Jun  8 08:33 myfile_hard
lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat dira/myfile_hard 
this is myfile created in dir5.
this line is added after creating soft link
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ vim dira/myfile_hard 
[aadarsha@labserver ~]$ cat dira/myfile_hard 
this is myfile created in dir5.
this line is written from hard link file
[aadarsha@labserver ~]$
 
[aadarsha@labserver ~]$ cat dir1/dir2/dir3/dir4/dir5/myfile 
this is myfile created in dir5.
this line is written from hard link file
[aadarsha@labserver ~]$

[aadarsha@labserver ~]$ ls -l dir1/dir2/dir3/dir4/dir5/myfile 
-rw-r--r--. 2 aadarsha aadarsha 73 Jun  8 08:47 dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -l dira/myfile_hard 
-rw-r--r--. 2 aadarsha aadarsha 73 Jun  8 08:47 dira/myfile_hard
[aadarsha@labserver ~]$ 

# same size for original file and hard link ---> A hard link has the same file size as the original file because 
#                                                both are directory entries pointing to the same inode and share the same underlying file data.

[aadarsha@labserver ~]$ ls -i dira/
  473657 dirb    526555 letfile    473651 myfile_hard  25447628 myfile_soft
[aadarsha@labserver ~]$ ls -il dira/
total 4
  473657 drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
  526555 -rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
  473651 -rw-r--r--. 2 aadarsha aadarsha 73 Jun  8 08:47 myfile_hard
25447628 lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -i dir1/dir2/dir3/dir4/dir5/
473651 myfile
[aadarsha@labserver ~]$ 

# Hard-linked files have the same inode number as the original file.
# Changes made through either the original file or the hard link are reflected in both, as they reference the same underlying file.
# If the original file is deleted, the hard link continues to provide access to the file data because it still references the same inode.
# If the original file is deleted, the symbolic (soft) link becomes a broken (dangling) link because it only stores the pathname to the original file.

[aadarsha@labserver ~]$ ls -l
total 4
drwxr-xr-x. 3 aadarsha aadarsha 33 Jun  8 06:47 dir1
drwxr-xr-x. 3 aadarsha aadarsha 71 Jun  8 08:47 dira
lrwxrwxrwx. 1 aadarsha aadarsha 31 Jun  8 07:53 myfile -> dir1/dir2/dir3/dir4/dir5/myfile
drwxr-xr-x. 5 aadarsha aadarsha 54 Jun  7 07:24 testcompany
-rw-r--r--. 1 aadarsha aadarsha 56 Jun  7 21:33 testfile
[aadarsha@labserver ~]$
 
[aadarsha@labserver ~]$ ls -l dir1/dir2/dir3/dir4/dir5/
total 4
-rw-r--r--. 2 aadarsha aadarsha 73 Jun  8 08:47 myfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -l dira/
total 4
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
-rw-r--r--. 2 aadarsha aadarsha 73 Jun  8 08:47 myfile_hard
lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver ~]$ 

# now delete original source file myfile

[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ rm -r dir1/dir2/dir3/dir4/dir5/myfile 
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -l
total 4
drwxr-xr-x. 3 aadarsha aadarsha 33 Jun  8 06:47 dir1
drwxr-xr-x. 3 aadarsha aadarsha 71 Jun  8 08:47 dira
lrwxrwxrwx. 1 aadarsha aadarsha 31 Jun  8 07:53 myfile -> dir1/dir2/dir3/dir4/dir5/myfile
drwxr-xr-x. 5 aadarsha aadarsha 54 Jun  7 07:24 testcompany
-rw-r--r--. 1 aadarsha aadarsha 56 Jun  7 21:33 testfile

[aadarsha@labserver ~]$ cat myfile 
cat: myfile: No such file or directory
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -l dira/
total 4
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
-rw-r--r--. 1 aadarsha aadarsha 73 Jun  8 08:47 myfile_hard
lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat dira/myfile_soft 
cat: dira/myfile_soft: No such file or directory
[aadarsha@labserver ~]$ 

# But for hard link

[aadarsha@labserver ~]$ cat dira/myfile_hard 
this is myfile created in dir5.
this line is written from hard link file
[aadarsha@labserver ~]$ 

# Getting files/directory details

[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ stat myfile 
  File: myfile -> dir1/dir2/dir3/dir4/dir5/myfile
  Size: 31        	Blocks: 0          IO Block: 4096   symbolic link
Device: 253,0	Inode: 473659      Links: 1
Access: (0777/lrwxrwxrwx)  Uid: ( 1000/aadarsha)   Gid: ( 1000/aadarsha)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2026-06-08 07:53:09.245051929 +0545
Modify: 2026-06-08 07:53:07.765053104 +0545
Change: 2026-06-08 07:53:07.765053104 +0545
 Birth: 2026-06-08 07:53:07.765053104 +0545
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ stat dira/myfile_hard 
  File: dira/myfile_hard
  Size: 73        	Blocks: 8          IO Block: 4096   regular file
Device: 253,0	Inode: 473651      Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/aadarsha)   Gid: ( 1000/aadarsha)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2026-06-08 09:02:10.463769531 +0545
Modify: 2026-06-08 08:47:34.586697736 +0545
Change: 2026-06-08 08:57:30.271064335 +0545
 Birth: 2026-06-08 07:52:05.533102327 +0545
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ rm myfile dira/myfile_soft 
rm: remove symbolic link 'myfile'? y
rm: remove symbolic link 'dira/myfile_soft'? y
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ stat testcompany/
  File: testcompany/
  Size: 54        	Blocks: 0          IO Block: 4096   directory
Device: 253,0	Inode: 8398100     Links: 5
Access: (0755/drwxr-xr-x)  Uid: ( 1000/aadarsha)   Gid: ( 1000/aadarsha)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2026-06-08 07:50:40.423169663 +0545
Modify: 2026-06-07 07:24:23.317639366 +0545
Change: 2026-06-07 07:24:23.317639366 +0545
 Birth: 2026-06-07 07:24:23.317639366 +0545
[aadarsha@labserver ~]$ 
```