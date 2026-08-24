---
title: "PL - 004 — Linux File System Links: Inode, Hard Links, and Soft Links"
date: 2026-06-08
draft: false
---
> In Unix-like systems such as Linux, a filename in a directory and the underlying file object are distinct concepts: directory entries associate names with inodes, while inodes hold file metadata and references to file data.

### Directory Entries (Dentries) and Inodes
```text
              +-------------------------------------------------------------+
              |                   Directory Entry (dentry)                  |
              | [ Name: "myfile" ] ---------------> [ Inode Number: 473651 ]|
              +-------------------------------------------------------------+
                                             |
                                             v
              +-------------------------------------------------------------+
              |            Inode (473651)                                   |
              |  - Permissions / Owner / Group                              |
              |  - File Size / Timestamps (atime, mtime, ctime)             |
              |  - Link Count (st_nlink = 2)                                |
              |  - Pointers to Data Blocks                                  |
              +-------------------------------------------------------------+
                                             |
                                             v
                             +-------------------------------+
                             |   Physical Data Blocks        |
                             |  "this is myfile data..."     |
                             +-------------------------------+
```
 - **Inode (Index Node):**
   A data structure on disk that holds file metadata (permissions, ownership, size, timestamps, block pointers). An inode does not store the file's name or its raw path.
 - **Dentry (Directory Entry):**
   A directory entry associates a filename with an inode number. In Linux, the VFS represents pathname components with dentry objects, while the filesystem stores directory information in its own on-disk format.
 - **Hard Link:**
   A new dentry created in a directory table pointing directly to an existing inode number.
 - **Soft Link (Symbolic Link):**
   A special file with its own unique inode whose data block contains a text string representing a target path.
```
# Soft Link and Hard Link

Hard link = another name for the same inode.
Soft link = a shortcut that stores a path.

Hard link --> “same file, different name”
Soft link --> “different file pointing to a name (path)”

| Feature                           | Hard Link                      | Soft Link (Symbolic Link)              |
| --------------------------------- | ------------------------------ | -------------------------------------- |
| Creates with                      | `ln source target`             | `ln -s source target`                  |
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
```

### Hard Link Allocation
When executing ```ln <source> <hard-link>```:
- Each hard-linked directory entry refers to the same inode.
- The VFS (Virtual File System) resolves source to its inode number (eg. 473651)
- A new entry is written inside the destination directory mapping ```hard_link -> 473651```
- The kernel increments ```st_nlink``` field in inode ```473651``` by 1.
- When ```rm source``` is executed, the kernel executes the ```unlink()``` system call. This removes the dentry mapping and decrements ```st_nlink```. The underlying data blocks are only freed when ```st_nlink == 0``` and no open file descriptors (```open()```) reference the inode.

### Soft Link (Fast Symlinks and Slow Symlinks)
When executing ```ln -s target soft_link```:
- The VFS allocates a brand-new inode for ```soft_link``` with file mode ```S_IFLNK```.
- **Fast Symlink**: If the target path string is shorter than 60 bytes (in Ext4), the filesystem stores the path string directly inside the inode's block pointers array instead of allocating an external disk block.
- **Slow Symlink**: If the target path exceeds 60 bytes, a dedicated data block is allocated to hold the path string.
- During path resolution ```(open(), stat())```, the kernel detects ```S_IFLNK``` and recursively traverses the stored path. Relative symlinks (e.g., ```../dir/file```) are resolved relative to the directory containing the symlink, not the current working directory of the process.

### Summary
- **Directories are just files containing mapping tables:** A directory maps names string to inode IDs. Hard linking simply adds a new mapping to that table.
- For a regular file, ```rm``` normally removes a directory entry by calling ```unlink()```. The inode and its data remain available while other hard links or open references still exist.
- **Fast Symlink Optimization:** Small symlinks (string length < 60 bytes) incur zero extra disk block allocations; the path string is packed directly into the inode table space reserved for block pointers.

### Terminal Session
```
[aadarsha@labserver ~]$ ls limited/
host.conf  locale.conf  logrotate.conf 

[aadarsha@labserver ~]$ ls
backup_conf  dir1  dira  limited  testcompany  testfile

[aadarsha@labserver ~]$ ls backup_conf/
chrony.conf  kdump.conf  ld.so.conf   mke2fs.conf    request-key.conf  rsyslog.conf   sysctl.conf    xattr.conf
dracut.conf  krb5.conf   man_db.conf  nsswitch.conf  resolv.conf       sestatus.conf  vconsole.conf  yum.conf
 
# command history and running already run history

[aadarsha@labserver ~]$ echo $HISTSIZE		            
1000                                               # Default history limit : 1000

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


[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile

[aadarsha@labserver ~]$ vim dir1/dir2/dir3/dir4/dir5/myfile

[aadarsha@labserver ~]$ cat dir1/dir2/dir3/dir4/dir5/myfile 
this is myfile created in dir5.

[aadarsha@labserver ~]$ ls
dir1  dira  testcompany  testfile

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

[aadarsha@labserver ~]$ ln -s dir1/dir2/dir3/dir4/dir5/myfile dira/

[aadarsha@labserver ~]$ ls -l dira/
total 0
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
lrwxrwxrwx. 1 aadarsha aadarsha 31 Jun  8 07:53 myfile -> dir1/dir2/dir3/dir4/dir5/myfile

[aadarsha@labserver ~]$ cat dira/myfile 
cat: dira/myfile: No such file or directory

[aadarsha@labserver ~]$ cd dira/

[aadarsha@labserver dira]$ ls
dirb  letfile  myfile

[aadarsha@labserver dira]$ cat myfile 
cat: myfile: No such file or directory 

[aadarsha@labserver dira]$ cat myfile 
cat: myfile: No such file or directory

[aadarsha@labserver dira]$ ls -l myfile 
lrwxrwxrwx. 1 aadarsha aadarsha 31 Jun  8 07:53 myfile -> dir1/dir2/dir3/dir4/dir5/myfile

[aadarsha@labserver dira]$ ls
dirb  letfile  myfile

[aadarsha@labserver dira]$ ls
dirb  letfile  myfile

[aadarsha@labserver dira]$ readlink myfile 
dir1/dir2/dir3/dir4/dir5/myfile

[aadarsha@labserver dira]$ cd

[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile

[aadarsha@labserver ~]$ ls dir1/
dir2  numfile

[aadarsha@labserver ~]$ ls dir1/
dir2/    numfile  

[aadarsha@labserver ~]$ ls dir1/dir2/dir3/dir4/dir5/
myfile

[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile

[aadarsha@labserver ~]$ cd dira/

[aadarsha@labserver dira]$ ls
dirb  letfile  myfile

[aadarsha@labserver dira]$ rm myfile 
rm: remove symbolic link 'myfile'? y

[aadarsha@labserver dira]$ cd

[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile

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

[aadarsha@labserver ~]$ ln -s ../dir1/dir2/dir3/dir4/dir5/myfile dira/myfile_soft

[aadarsha@labserver ~]$ ls dira/
dirb  letfile  myfile_soft

[aadarsha@labserver ~]$ ls -l dira/
total 0
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile

[aadarsha@labserver ~]$ cat dira/myfile_soft 
this is myfile created in dir5.

# OR using the absolute path is better approach

[aadarsha@labserver ~]$ vim dira/myfile_soft 

[aadarsha@labserver ~]$ cat dira/myfile_soft 
this is myfile created in dir5.
this line is added after creating soft link

[aadarsha@labserver ~]$ cat dir1/dir2/dir3/dir4/dir5/myfile 
this is myfile created in dir5.
this line is added after creating soft link 

# since soft link is like pointer, when I change the soft link, original file also got changed

[aadarsha@labserver ~]$ ls -l dir1/dir2/dir3/dir4/dir5/myfile 
-rw-r--r--. 1 aadarsha aadarsha 77 Jun  8 08:33 dir1/dir2/dir3/dir4/dir5/myfile

[aadarsha@labserver ~]$ ls -l dira/myfile_soft 
lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 dira/myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile

# size 77 bytes and softlink size 34 bytes  --->  size of a regular file represents the amount of data stored in the file.
#                                                 AND
#                                                 size of a symbolic (soft) link represents the number of bytes required to store the
#                                                 target pathname, not the size of the target file.

# creating hard link

[aadarsha@labserver ~]$ ln /home/aadarsha/dir1/dir2/dir3/dir4/dir5/myfile dira/myfile_hard

[aadarsha@labserver ~]$ ls -l dira/myfile_hard 
-rw-r--r--. 2 aadarsha aadarsha 77 Jun  8 08:33 dira/myfile_hard

[aadarsha@labserver ~]$ ls dira/
dirb  letfile  myfile_hard  myfile_soft

[aadarsha@labserver ~]$ ls -l dira/
total 4
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
-rw-r--r--. 2 aadarsha aadarsha 77 Jun  8 08:33 myfile_hard
lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile

[aadarsha@labserver ~]$ cat dira/myfile_hard 
this is myfile created in dir5.
this line is added after creating soft link

[aadarsha@labserver ~]$ vim dira/myfile_hard

[aadarsha@labserver ~]$ cat dira/myfile_hard 
this is myfile created in dir5.
this line is written from hard link file
 
[aadarsha@labserver ~]$ cat dir1/dir2/dir3/dir4/dir5/myfile 
this is myfile created in dir5.
this line is written from hard link file

[aadarsha@labserver ~]$ ls -l dir1/dir2/dir3/dir4/dir5/myfile 
-rw-r--r--. 2 aadarsha aadarsha 73 Jun  8 08:47 dir1/dir2/dir3/dir4/dir5/myfile

[aadarsha@labserver ~]$ ls -l dira/myfile_hard 
-rw-r--r--. 2 aadarsha aadarsha 73 Jun  8 08:47 dira/myfile_hard

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

[aadarsha@labserver ~]$ ls -i dir1/dir2/dir3/dir4/dir5/
473651 myfile

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
 
[aadarsha@labserver ~]$ ls -l dir1/dir2/dir3/dir4/dir5/
total 4
-rw-r--r--. 2 aadarsha aadarsha 73 Jun  8 08:47 myfile

[aadarsha@labserver ~]$ ls -l dira/
total 4
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
-rw-r--r--. 2 aadarsha aadarsha 73 Jun  8 08:47 myfile_hard
lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile

# now delete original source file myfile
 
[aadarsha@labserver ~]$ rm -r dir1/dir2/dir3/dir4/dir5/myfile 

[aadarsha@labserver ~]$ ls -l
total 4
drwxr-xr-x. 3 aadarsha aadarsha 33 Jun  8 06:47 dir1
drwxr-xr-x. 3 aadarsha aadarsha 71 Jun  8 08:47 dira
lrwxrwxrwx. 1 aadarsha aadarsha 31 Jun  8 07:53 myfile -> dir1/dir2/dir3/dir4/dir5/myfile
drwxr-xr-x. 5 aadarsha aadarsha 54 Jun  7 07:24 testcompany
-rw-r--r--. 1 aadarsha aadarsha 56 Jun  7 21:33 testfile

[aadarsha@labserver ~]$ cat myfile 
cat: myfile: No such file or directory

[aadarsha@labserver ~]$ ls -l dira/
total 4
drwxr-xr-x. 3 aadarsha aadarsha 18 Jun  8 07:50 dirb
-rw-r--r--. 1 aadarsha aadarsha  0 Jun  7 16:10 letfile
-rw-r--r--. 1 aadarsha aadarsha 73 Jun  8 08:47 myfile_hard
lrwxrwxrwx. 1 aadarsha aadarsha 34 Jun  8 08:25 myfile_soft -> ../dir1/dir2/dir3/dir4/dir5/myfile

[aadarsha@labserver ~]$ cat dira/myfile_soft 
cat: dira/myfile_soft: No such file or directory

# But for hard link

[aadarsha@labserver ~]$ cat dira/myfile_hard 
this is myfile created in dir5.
this line is written from hard link file

# Getting files/directory details

[aadarsha@labserver ~]$ ls
dir1  dira  myfile  testcompany  testfile 

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

  # Note on Symlink Permissions:
    Symlinks always show permissions ```0777 (lrwxrwxrwx)```. The kernel ignores these permissions during access checks; permissions of the target file are evaluated when dereferencing the symlink.


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

[aadarsha@labserver ~]$ rm myfile dira/myfile_soft 
rm: remove symbolic link 'myfile'? y
rm: remove symbolic link 'dira/myfile_soft'? y

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
```