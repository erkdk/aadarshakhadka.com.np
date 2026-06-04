---
title: "Linux Practice 001"
date: 2026-06-04
draft: false
---


```


aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.67
aadarsha@192.168.254.67's password: 
Last login: Thu Jun  4 06:34:06 2026
[aadarsha@labserver ~]$ whoami
aadarsha
[aadarsha@labserver ~]$ # Switching User's account
[aadarsha@labserver ~]$ # su - <username>
[aadarsha@labserver ~]$ su - root
Password: 
Last login: Tue Jun  2 16:55:19 +0545 2026 on pts/1
[root@labserver ~]# whoami
root
[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ whoami
aadarsha
[aadarsha@labserver ~]$ sudo -i
[sudo] password for aadarsha: 
Sorry, try again.
[sudo] password for aadarsha: 
Sorry, try again.
[sudo] password for aadarsha: 
sudo: 3 incorrect password attempts
[aadarsha@labserver ~]$ sudo -i
[sudo] password for aadarsha: 
Sorry, try again.
[sudo] password for aadarsha: 
[root@labserver ~]# whoami
root
[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ sudo su
[root@labserver aadarsha]# whoami
root
[root@labserver aadarsha]# exit
exit
[aadarsha@labserver ~]$ sudo su
[root@labserver aadarsha]# cd
[root@labserver ~]# ls
anaconda-ks.cfg
[root@labserver ~]# useradd myuser1
[root@labserver ~]# passwd myuser1
New password: 
Retype new password: 
passwd: password updated successfully
[root@labserver ~]# su - myuser1
[myuser1@labserver ~]$ exit
logout
[root@labserver ~]# exit
exit
[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.67 closed.
aadarkdk@pop-os:~$ ssh myuser1@192.168.254.67
myuser1@192.168.254.67's password: 
Last login: Thu Jun  4 08:04:35 2026 from 192.168.254.69
[myuser1@labserver ~]$ su - aadarsha
Password: 
Last login: Thu Jun  4 06:35:22 +0545 2026 from 192.168.254.69 on pts/0
[aadarsha@labserver ~]$ su - root
Password: 
Last login: Thu Jun  4 07:55:36 +0545 2026 on pts/1
[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ pwd
/home/aadarsha
[aadarsha@labserver ~]$ exit
logout
[myuser1@labserver ~]$ pwd
/home/myuser1
[myuser1@labserver ~]$ su - aadarsha
Password: 
Last login: Thu Jun  4 08:05:42 +0545 2026 on pts/0
[aadarsha@labserver ~]$ exit
logout
[myuser1@labserver ~]$ whoami
myuser1
[myuser1@labserver ~]$ passwd myuser1
Current password: 
New password: 
Retype new password: 
passwd: password updated successfully
[myuser1@labserver ~]$ passwd
Current password: 
New password: 
Retype new password: 
passwd: password updated successfully
[myuser1@labserver ~]$ su - aadarsha
Password: 
Last login: Thu Jun  4 08:06:15 +0545 2026 on pts/0
[aadarsha@labserver ~]$ date
Thu Jun  4 08:09:21 AM +0545 2026
[aadarsha@labserver ~]$ su - root
Password: 
Last login: Thu Jun  4 08:05:52 +0545 2026 on pts/0
[root@labserver ~]# date --set=2026-06-01
Mon Jun  1 12:00:00 AM +0545 2026
[root@labserver ~]# date
Mon Jun  1 12:00:03 AM +0545 2026
[root@labserver ~]# date --set=2026-06-04
Thu Jun  4 12:00:00 AM +0545 2026
[root@labserver ~]# date --set=08:13:00
Thu Jun  4 08:13:00 AM +0545 2026
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
                    
[aadarsha@labserver ~]$ cal 2026
                               2026                               

       January               February                 March       
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
             1  2  3    1  2  3  4  5  6  7    1  2  3  4  5  6  7
 4  5  6  7  8  9 10    8  9 10 11 12 13 14    8  9 10 11 12 13 14
11 12 13 14 15 16 17   15 16 17 18 19 20 21   15 16 17 18 19 20 21
18 19 20 21 22 23 24   22 23 24 25 26 27 28   22 23 24 25 26 27 28
25 26 27 28 29 30 31                          29 30 31            
                                                                  
        April                   May                   June        
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
          1  2  3  4                   1  2       1  2  3  4  5  6
 5  6  7  8  9 10 11    3  4  5  6  7  8  9    7  8  9 10 11 12 13
12 13 14 15 16 17 18   10 11 12 13 14 15 16   14 15 16 17 18 19 20
19 20 21 22 23 24 25   17 18 19 20 21 22 23   21 22 23 24 25 26 27
26 27 28 29 30         24 25 26 27 28 29 30   28 29 30            
                       31                                         
        July                  August                September     
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
          1  2  3  4                      1          1  2  3  4  5
 5  6  7  8  9 10 11    2  3  4  5  6  7  8    6  7  8  9 10 11 12
12 13 14 15 16 17 18    9 10 11 12 13 14 15   13 14 15 16 17 18 19
19 20 21 22 23 24 25   16 17 18 19 20 21 22   20 21 22 23 24 25 26
26 27 28 29 30 31      23 24 25 26 27 28 29   27 28 29 30         
                       30 31                                      
       October               November               December      
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
             1  2  3    1  2  3  4  5  6  7          1  2  3  4  5
 4  5  6  7  8  9 10    8  9 10 11 12 13 14    6  7  8  9 10 11 12
11 12 13 14 15 16 17   15 16 17 18 19 20 21   13 14 15 16 17 18 19
18 19 20 21 22 23 24   22 23 24 25 26 27 28   20 21 22 23 24 25 26
25 26 27 28 29 30 31   29 30                  27 28 29 30 31      
                                                                  
[aadarsha@labserver ~]$ cd
[aadarsha@labserver ~]$ pwd
/home/aadarsha
[aadarsha@labserver ~]$ su - myuser1
Password: 
Last login: Thu Jun  4 08:05:22 +0545 2026 from 192.168.254.69 on pts/0
[myuser1@labserver ~]$ pwd
/home/myuser1
[myuser1@labserver ~]$ cd /
[myuser1@labserver /]$ pwd
/
[myuser1@labserver /]$ cd
[myuser1@labserver ~]$ exit
logout
[aadarsha@labserver ~]$ # Creating a directory 
[aadarsha@labserver ~]$ mkdir mydir1
[aadarsha@labserver ~]$ # Creating a File
[aadarsha@labserver ~]$ # using 'cat' command
[aadarsha@labserver ~]$ cat > file1
This is my first file...     
[aadarsha@labserver ~]$ # CTRL + D --> Save and exit 
[aadarsha@labserver ~]$ ls
file1  mydir1
[aadarsha@labserver ~]$ cat file1
This is my first file...
[aadarsha@labserver ~]$ # using 'vi' editor
[aadarsha@labserver ~]$ vi file2
[aadarsha@labserver ~]$ hostname
labserver
[aadarsha@labserver ~]$ pwd
/home/aadarsha
[aadarsha@labserver ~]$ # home directory of the current user =>  /home/<user-name>
[aadarsha@labserver ~]$ # Viewing the contents of the file
[aadarsha@labserver ~]$ cat file1
This is my first file...
[aadarsha@labserver ~]$ cat file2 
This is my second file...
[aadarsha@labserver ~]$ # Modifying the content of the file
[aadarsha@labserver ~]$ vi file2
[aadarsha@labserver ~]$ cat file2
This is my second file...

This is added later
[aadarsha@labserver ~]$ vi file2
[aadarsha@labserver ~]$ cat file2
This is my second file...

This is added later

[aadarsha@labserver ~]$ ls
file1  file2  mydir1
[aadarsha@labserver ~]$ cd mydir1/
[aadarsha@labserver mydir1]$ pwd
/home/aadarsha/mydir1
[aadarsha@labserver mydir1]$ cd ~
[aadarsha@labserver ~]$ cd /
[aadarsha@labserver /]$ cd ~
[aadarsha@labserver ~]$ pwd
/home/aadarsha
[aadarsha@labserver ~]$ 

```