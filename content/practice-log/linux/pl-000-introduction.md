---
title: "Why I Chose Linux — Resources Worth Exploring"
date: 2026-06-03
draft: false
---

### Why I Chose Linux

> I first started using Linux in the first year of my bachelor's degree out of curiosity. What kept me using it was the terminal.

I've never enjoyed searching through multiple menus and buttons for simple tasks. With Linux, I can tell the system exactly what I want, and it does exactly that. It feels precise, efficient, and predictable.

As I explored further, I discovered that the Linux kernel is the foundation of an ecosystem built on open-source collaboration, transparency, user freedom, and continuous learning. I was also fascinated to learn how extensively Linux is used to power servers and modern computing infrastructure, making it one of the most important technologies in modern computing.

Over time, every command I typed became more than just a command—it became an opportunity to understand operating system concepts, such as processes, filesystems, permissions, networking, storage, security, and how computers work behind the scenes.

### Resources Worth Exploring
Here are some resources for exploring Linux further.

- [Linux Filesystem Hierarchy Standard (FHS)](https://refspecs.linuxfoundation.org/fhs.shtml)
- [Linux Kernel Architecture Map](https://makelinux.github.io/kernel/map/)

### Exploring Linux from the Terminal
> Let's check the system information.
```
aadarkdk@pop-os:~$ uname -a
 Linux pop-os 7.0.11-76070011-generic #202606011647~1780583630~22.04~70ad774 SMP PREEMPT_DYNAMIC Thu J x86_64 x86_64 x86_64 GNU/Linux
   |     |    |                       |                                                                 |      |      |        |
   |     |    |                       |                                                                 |      |      |        +-------- operating system (-o)
   |     |    |                       |                                                                 |      |      +----------------- hardware platform (-i)
   |     |    |                       |                                                                 |      +------------------------ processor type (-p)
   |     |    |                       |                                                                 +------------------------------- machine hardware name (-m)
   |     |    |                       +----------------------------------------- kernel version (-v)
   |     |    +---------------------------------------- kernel release (-r)
   |     +----------------- nodename / hostname (-n)
   +---kernel name (-s)

aadarkdk@pop-os:~$ cat /etc/os-release
NAME="Pop!_OS"
VERSION="22.04 LTS"
ID=pop
ID_LIKE="ubuntu debian"
PRETTY_NAME="Pop!_OS 22.04 LTS"
VERSION_ID="22.04"
HOME_URL="https://pop.system76.com"
SUPPORT_URL="https://support.system76.com"
BUG_REPORT_URL="https://github.com/pop-os/pop/issues"
PRIVACY_POLICY_URL="https://system76.com/privacy"
VERSION_CODENAME=jammy
UBUNTU_CODENAME=jammy
LOGO=distributor-logo-pop-os
aadarkdk@pop-os:~$
```
#### Basic Linux Commands
```
aadarkdk@pop-os:~$ whoami
aadarkdk

aadarkdk@pop-os:~$ hostname
pop-os

 # Connect to a remote system using SSH

aadarkdk@pop-os:~$ ssh aadarsha@192.0.2.67
aadarsha@192.0.2.67's password: 
Last login: Thu Jun  4 06:34:06 2026

[aadarsha@labserver ~]$ hostname
labserver

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

[aadarsha@labserver ~]$ pwd
/home/aadarsha

[aadarsha@labserver ~]$ whoami
aadarsha

 # Switch to another user's login shell: su - <username>

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Tue Jun  2 16:55:19 +0545 2026 on pts/1

[root@labserver ~]# whoami
root

[root@labserver ~]# exit
logout

 # Start a root login shell using sudo

[aadarsha@labserver ~]$ sudo -i
[sudo] password for aadarsha: 

[root@labserver ~]# whoami
root

 # Create a new user account with a home directory

[root@labserver ~]# useradd -m myuser1

[root@labserver ~]# passwd myuser1
New password: 
Retype new password: 
passwd: password updated successfully

[root@labserver ~]# su - myuser1

[myuser1@labserver ~]$ whoami
myuser1

[myuser1@labserver ~]$ passwd
Current password: 
New password: 
Retype new password: 
passwd: password updated successfully

[myuser1@labserver ~]$ pwd
/home/myuser1                            # home directory of the current user  -->  /home/<user-name>

[myuser1@labserver ~]$ su - aadarsha
Password: 
Last login: Thu Jun  4 06:35:22 +0545 2026 from 192.0.2.69 on pts/0

[aadarsha@labserver ~]$ date
Thu Jun  4 08:09:21 AM +0545 2026

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Thu Jun  4 08:05:52 +0545 2026 on pts/0

[root@labserver ~]# exit
logout

[aadarsha@labserver ~]$ cal
      June 2026     
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30            
                    
[aadarsha@labserver ~]$ cal 2026             # entire 12 months of 2026
...

[aadarsha@labserver ~]$ pwd
/home/aadarsha

[aadarsha@labserver ~]$ cd /                 # Change to the root directory

[aadarsha@labserver /]$ pwd
/

[aadarsha@labserver ~]$ cd                   # Change to the current user's home directory

[aadarsha@labserver ~]$ pwd
/home/aadarsha
 
[aadarsha@labserver ~]$ mkdir mydir1         # Creating a directory

[aadarsha@labserver ~]$ cat > file1          # Create a file and write text to it
This is my first file...     

 # Press Ctrl+D to send EOF and finish entering the file

[aadarsha@labserver ~]$ ls
file1  mydir1

[aadarsha@labserver ~]$ cat file1
This is my first file...

 # Using the vi editor

[aadarsha@labserver ~]$ vi file2

 # View the contents of a file

[aadarsha@labserver ~]$ cat file1
This is my first file...

[aadarsha@labserver ~]$ cat file2 
This is my second file...

 # Modify the contents of the file

[aadarsha@labserver ~]$ vi file2

[aadarsha@labserver ~]$ cat file2
This is my second file...
This is added later

[aadarsha@labserver ~]$ ls
file1  file2  mydir1

[aadarsha@labserver ~]$ cd mydir1/

[aadarsha@labserver mydir1]$ pwd
/home/aadarsha/mydir1

[aadarsha@labserver ~]$ cd /

[aadarsha@labserver /]$ pwd
/

[aadarsha@labserver /]$ cd ~                 # ~ means the current user's home directory

[aadarsha@labserver ~]$ pwd
/home/aadarsha

[aadarsha@labserver ~]$ exit
logout
Connection to 192.0.2.67 closed.
aadarkdk@pop-os:~$
```