---
title: "PL - 007 — User and Group Administration"
date: 2026-07-29
draft: false
---

#### Concepts:
  > User Management, 
  > Group Management, 
  > Special Permission Bits,
  > Password Policy,  
  > ACL (Access Control List)
  

### Terminal Session
```
 # User Management Configuration files:

 #-----------------------------------------------------------------------------------------------------
 # File              | Purpose                             |  Key Contents
 #-----------------------------------------------------------------------------------------------------
 # /etc/passwd       | User account registry               |  Username, UID, GID, home, shell
 #-----------------------------------------------------------------------------------------------------
 # /etc/shadow       | Password storage & policy           |  Encrypted passwords, expiry rules
 #-----------------------------------------------------------------------------------------------------
 # /etc/login.defs   | Default account creation settings   |  UID/GID ranges, password defaults
 #-----------------------------------------------------------------------------------------------------
 # /etc/group        | Group definitions & memberships     |  Group name, GID, members
 #-----------------------------------------------------------------------------------------------------
 # /etc/gshadow      | Group password security             |  Encrypted group passwords, admins
 #-----------------------------------------------------------------------------------------------------
 # /etc/sudoers      | Privilege escalation control        |  Sudo rules for users/groups
 #-----------------------------------------------------------------------------------------------------
 # /etc/skel/        | New user home template              |  Default config files (.bashrc, .profile)
 #-----------------------------------------------------------------------------------------------------
 # /var/log/secure   | authenticate & authorization events |  Logins, sudo, authentication attempts
 #-----------------------------------------------------------------------------------------------------

 # /var/log/secure (RHEL-based systems)  &  /var/log/auth.log (Debian/Ubuntu)
 
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Thu Jul 23 20:04:25 2026

[aadarsha@labserver ~]$ whoami
aadarsha
 
[aadarsha@labserver ~]$ su - root
Password: 
Last login: Wed Jul 22 19:07:43 +0545 2026 on pts/0
 
[root@labserver ~]# cat /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
bin:x:1:1:bin:/bin:/usr/sbin/nologin
...
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin		# --> nologin shell generally created for service accounts
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
[root@labserver ~]#  

 # Format
  
 # <user-name> : <password> : <UID> : <GID> : <full-name or comment > : <home-directory> : <shell>
 
 # aadarsha : x : 1000 : 1000 : Aadarsha Khadka : /home/aadarsha : /bin/bash
 #    |       |    |      |        |                  |              |
 #    |       |    |      |        |                  |              +------- Shell (login shell)
 #    |       |    |      |        |                  +---------------------- Home Directory
 #    |       |    |      |        +----------------------------------------- GECOS (full name / comment)
 #    |       |    |      +-------------------------------------------------- Primary GID
 #    |       |    +--------------------------------------------------------- UID
 #    |       +-------------------------------------------------------------- Password (x means in /etc/shadow)
 #    +---------------------------------------------------------------------- Username
 
[root@labserver ~]# cat /etc/group
root:x:0:
...
systemd-coredump:x:995:
aadarsha:x:1000:
[root@labserver ~]# 

 # Format
  
 # <group-name> : <password> : <GID>
 
 # aadarsha : x : 1000 :
 #    |       |     |
 #    |       |     +---- GID (Group ID) - matches user's primary GID in /etc/passwd
 #    |       +---------- Password ('x' = stored in /etc/gshadow, empty = no password)
 #    +------------------ Group Name

 # Primary group is automatically created by default when a user is created, unless we specify an existing group using -g option.
 
[root@labserver ~]# cat /etc/shadow
root:$y$j9T$bSL.DYlX2bFQqj9MhuK1eNmo$lHY07CmxyKDGoHy3x9mjzRCV9/JicXeG0/Iovox9ZW1::0:99999:7:::
bin:*:20186:0:99999:7:::
...
systemd-coredump:!*:20647::::::
aadarsha:$y$j9T$hknYweHgxIvveqhAzwjsY4P4$f2Uowb.VrCDXEpbQo9Oj3sFsbQXfGaWHjtxwTCfUks4::0:99999:7:::
[root@labserver ~]# 
 
 # Format
 
 # <username> : <password-hash> : <last-change> : <min-age> : <max-age> : <warning> : <password-inactivity> : <account-expire> : <reserved>
 
 # aadarsha : $y$j9T$hknYweHgxIvveqhAzwjsY4P4$f2Uowb.VrCDXEpbQo9Oj3sFsbQXfGaWHjtxwTCfUks4 :   : 0 : 99999 : 7 :   :   :
 #    |                           |                                                         |   |     |     |   |   |   |
 #    |                           |                                                         |   |     |     |   |   |   +--- Reserved (currently unused)
 #    |                           |                                                         |   |     |     |   |   +------- Account expiration date (days since Unix epoch; empty = never expires)
 #    |                           |                                                         |   |     |     |   +----------- Password inactivity period (days after password expires before account is disabled; empty = no inactivity limit)
 #    |                           |                                                         |   |     |     +--------------- Warning period before password expiration (days; empty = no warning)
 #    |                           |                                                         |   |     +--------------------- Maximum password age in days (99999 commonly means effectively never expires)
 #    |                           |                                                         |   +--------------------------- Minimum password age in days (0 = password may be changed immediately)
 #    |                           |                                                         +------------------------------- Last password change (days since Unix epoch, Jan 1, 1970)
 #    |                           +------------------ Salted, hashed password (! --> means account is locked and cannot be used for password login.)
 #    +---------------------------------------------- Username
 
[root@labserver ~]# cat /etc/gshadow
root:::
...
systemd-coredump:!*::
aadarsha:!::
[root@labserver ~]#

 # Format
 
 # <group_name> : <password> : <administrators> : <members>
 
 # aadarsha : ! :   :
 #    |       |   |   |
 #    |       |   |   +---- Group members (comma-separated; empty = no members)
 #    |       |   +-------- Group administrators (comma-separated; empty = no administrators)
 #    |       +------------ Group password or status:
 #    |                       !  = password is locked (cannot be used)
 #    |                       *  = no valid password (password login disabled)
 #    |                       <hash> = encrypted group password
 #    |                       empty = no password
 #    |                      
 #    +--------------------- Group name
 
[root@labserver ~]# cat /etc/sudoers
## Sudoers allows particular users to run various commands as
## the root user, without needing the root password.
...
## Allows people in group wheel to run all commands
%wheel	ALL=(ALL)	ALL
...
[root@labserver ~]#

 # Meaning of:
 
 # %wheel ALL=(ALL) ALL
 
 # %wheel    ALL    =    (ALL)    ALL
 #   |        |            |        |
 #   |        |            |        +---- May run all commands
 #   |        |            +------------- May run commands as any user
 #   |        +-------------------------- On all hosts
 #   +----------------------------------- Members of the "wheel" group (% indicates a group)
 
 # ---> Any member of the wheel group can run any command as any user on any host (after authenticating with their password).

  # NOTE:
 
  | Rule                           | Meaning                                                                                                                 |
  | -------------------------------|-------------------------------------------------------------------------------------------------------------------------|
  | %wheel ALL=(ALL) ALL           | Members of the wheel group may run any command as any user on any host, after authenticating.                           |
  | %wheel ALL=(ALL) NOPASSWD: ALL | Members of the wheel group may run any command as any user on any host **without entering a password** (if uncommented).|
  | root ALL=(ALL) ALL             | The root user may run any command as any user on any host.                                                              |

[root@labserver ~]# cd /etc/skel/
[root@labserver skel]# ls
[root@labserver skel]# ls -la
total 24
drwxr-xr-x.  2 root root   62 Jul 13 14:29 .
drwxr-xr-x. 82 root root 8192 Jul 24 12:06 ..
-rw-r--r--.  1 root root   18 Oct 29  2024 .bash_logout           # ---> Runs when a Bash login shell exits.
-rw-r--r--.  1 root root  144 Oct 29  2024 .bash_profile          # ---> Runs when a Bash login shell starts.( Typically used to set environment variables and initialize the user's login environment )
-rw-r--r--.  1 root root  522 Oct 29  2024 .bashrc                # ---> Runs for every interactive non-login Bash shell.( Typically used for aliases, shell options, functions, and prompt customization )
[root@labserver skel]# 

 # Note: On most Linux systems, .bash_profile sources .bashrc so interactive login shells also inherit aliases, functions, and prompt settings.
 
[root@labserver]# cat /var/log/secure 
...
Jul 23 20:04:10 labserver sshd[943]: Server listening on 0.0.0.0 port 22.
Jul 23 20:04:10 labserver sshd[943]: Server listening on :: port 22.
Jul 23 20:04:24 labserver (systemd)[4818]: pam_unix(systemd-user:session): session opened for user aadarsha(uid=1000) by aadarsha(uid=0)
Jul 23 20:04:25 labserver login[956]: pam_unix(login:session): session opened for user aadarsha(uid=1000) by aadarsha(uid=0)
Jul 23 20:04:25 labserver login[956]: LOGIN ON tty1 BY aadarsha
Jul 23 20:04:57 labserver sshd-session[6619]: Accepted password for aadarsha from 192.168.254.32 port 35038 ssh2
Jul 23 20:04:57 labserver sshd-session[6619]: pam_unix(sshd:session): session opened for user aadarsha(uid=1000) by aadarsha(uid=0)
Jul 23 20:05:48 labserver su[6666]: pam_unix(su-l:session): session opened for user root(uid=0) by aadarsha(uid=1000)
...
[root@labserver]#  
  
 # Default settings used by useradd:
 
[root@labserver ~]# useradd -D
GROUP=100
GROUPS=
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
USRSKEL=/usr/etc/skel
CREATE_MAIL_SPOOL=yes
LOG_INIT=yes
[root@labserver ~]# 

[root@labserver ~]# cat /etc/default/useradd 
# useradd defaults file
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
[root@labserver ~]#

 # By default, when a user is created using useradd, the following changes happen internally:

[root@labserver ~]# ls /home/
aadarsha

[root@labserver ~]# useradd a-user

[root@labserver ~]# ls /home
aadarsha  a-user

[root@labserver ~]# ls -l /home/
total 0
drwx------. 2 aadarsha aadarsha 83 Jul 13 14:46 aadarsha
drwx------. 2 a-user   a-user   62 Jul 24 12:06 a-user       # user ownership and group ownership --> user-a         
[root@labserver ~]#
 
[root@labserver ~]# cat /etc/passwd
...
a-user:x:1001:1001::/home/a-user:/bin/bash               
[root@labserver ~]#

[root@labserver ~]# grep a-user /etc/passwd
a-user:x:1001:1001::/home/a-user:/bin/bash
[root@labserver ~]# 

[root@labserver ~]# grep a-user /etc/group
a-user:x:1001:
[root@labserver ~]# 

[root@labserver ~]# grep a-user /etc/shadow
a-user:!:20658:0:99999:7:::				   # ! --> Password authentication is locked because no password has been assigned yet
[root@labserver ~]# 

[root@labserver ~]# pwd
/root
[root@labserver ~]# cd /home/a-user/
[root@labserver a-user]# pwd
/home/a-user
[root@labserver a-user]# ls
[root@labserver a-user]# ls -a
.  ..  .bash_logout  .bash_profile  .bashrc
[root@labserver a-user]# 
[root@labserver a-user]# cd
[root@labserver ~]# pwd
/root
[root@labserver ~]# 
 
 # useradd a-user
 #       |
 #       +--> /etc/passwd   --> User account information added
 #       |
 #       +--> /etc/group    --> Primary group created
 #       |
 #       +--> /etc/shadow   --> Password entry created (locked initially)
 #       |
 #       +--> /etc/gshadow  --> Secure group information created/updated
 #       |
 #       +--> /home/a-user  --> User home directory created
 #       |
 #       +--> /etc/skel/*   --> Default shell files copied to user's home directory

 # The files placed in /etc/skel/ are automatically copied to the newly created user's home directory.
 # Application:
 # Organization-specific files (company rules, login instructions, scripts,
 # configuration files, or user guidance documents) can be placed in /etc/skel/
 # so they are automatically provided to new users during account creation.

[root@labserver ~]# cd /etc/skel/
[root@labserver skel]# ls
[root@labserver skel]# vi myfile
[root@labserver skel]# ls
myfile
[root@labserver skel]# ls -a
.  ..  .bash_logout  .bash_profile  .bashrc  myfile
[root@labserver skel]# cd
[root@labserver ~]# 
[root@labserver ~]# useradd b-user
[root@labserver ~]# cd /home/b-user/
[root@labserver b-user]# ls
myfile
[root@labserver b-user]# ls -a
.  ..  .bash_logout  .bash_profile  .bashrc  myfile
[root@labserver b-user]# cd
[root@labserver ~]# 

 # Changes made to /etc/skel/ affect only users created after the change.
 # Existing users' home directories are not updated automatically.

[root@labserver ~]# cd /home/a-user/
[root@labserver a-user]# ls
[root@labserver a-user]# ls -a
.  ..  .bash_logout  .bash_profile  .bashrc
[root@labserver a-user]# 

 # /var/spool/mail/ --> User Mail Spool Directory

 # Stores local mail spool files for users on the system.
 # Each user has a mailbox file named after the username.
 # When a new user is created with useradd (CREATE_MAIL_SPOOL=yes), a mail spool file is automatically created:

[root@labserver ~]# cd /var/spool/mail/
[root@labserver mail]# ls
aadarsha  a-user  b-user
[root@labserver mail]# ls -la
total 0
drwxrwxr-x. 2 root     mail 50 Jul 24 14:26 .
drwxr-xr-x. 6 root     root 56 Jul 13 14:30 ..
-rw-rw----. 1 aadarsha mail  0 Jul 13 14:32 aadarsha
-rw-rw----. 1 a-user   mail  0 Jul 24 12:06 a-user
-rw-rw----. 1 b-user   mail  0 Jul 24 14:26 b-user

[root@labserver mail]# ls /var/mail
aadarsha  a-user  b-user

[root@labserver mail]# ls -l /var/mail
lrwxrwxrwx. 1 root root 10 Apr  2  2025 /var/mail -> spool/mail              # /var/mail is normally a symbolic link to /var/spool/mail
[root@labserver mail]# 

 # add password

[root@labserver ~]# ls /home/
aadarsha  a-user  b-user
[root@labserver ~]# grep a-user /etc/shadow
a-user:!:20658:0:99999:7:::

[root@labserver ~]# passwd a-user
New password: 
Retype new password: 
passwd: password updated successfully

[root@labserver ~]# grep a-user /etc/shadow
a-user:$y$j9T$09jS8Aj4NajTvf3D9DItu1$ywEo4KJbYr64Y439Y7SmP3ZsWbFuKguoKuM8ex/GwI6:20658:0:99999:7:::
[root@labserver ~]# 

 # Get password age info
 
[root@labserver ~]# chage -l a-user
Last password change					: Jul 24, 2026
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
[root@labserver ~]# 

[root@labserver ~]# vi /etc/login.defs                # --> contains the default settings used by tools such as useradd, passwd, and other shadow password utilities.

[root@labserver ~]# whoami
root

[root@labserver ~]# ps
    PID TTY          TIME CMD
   1985 pts/0    00:00:00 su
   1991 pts/0    00:00:00 bash
   4043 pts/0    00:00:00 ps
[root@labserver ~]# 

[root@labserver ~]# su - a-user
[a-user@labserver ~]$ whoami
a-user
[a-user@labserver ~]$ ps
    PID TTY          TIME CMD
   4058 pts/0    00:00:00 bash
   4088 pts/0    00:00:00 ps
[a-user@labserver ~]$ 

 # To create user accounts with non-default settings
 
 # useradd | usermod [options] <username>
  # [options]

  #  -c  -->  <comment>
  #  -d  -->  <home dir>
  #  -e  -->  <a/c expiry date>
  #  -g  -->  <primary group>
  #  -G  -->  <secondary groups>
  #  -u  -->  <UID >
  #  -s  -->  <shell> 
  #  -l  -->  <new login name>
  #  -L  -->  <Lock user's account>
  #  -U  -->  <Unlock>
  
[root@labserver ~]# useradd --help

[root@labserver ~]# usermod -h

[root@labserver ~]# # useradd -u 3001 -s /bin/csh -c "Mission Karki, KTM-32, Tinkune, +977-9860397731" -d /opt/milan -e 2027-01-01 -g employee -G IT,admin mission     # example 

[root@labserver ~]# ls /home
aadarsha  a-user  b-user
 
[root@labserver ~]# groups aadarsha
aadarsha : aadarsha wheel			     # first is primary group & remaining are secondary groups
        
[root@labserver ~]# groups b-user
b-user : b-user
[root@labserver ~]# 

[root@labserver ~]# grep b-user /etc/passwd
b-user:x:1002:1002::/home/b-user:/bin/bash
 
[root@labserver ~]# passwd b-user
New password: 
Retype new password: 
passwd: password updated successfully

[root@labserver ~]# grep b-user /etc/passwd
b-user:x:1002:1002::/home/b-user:/bin/bash

 # add dummy groups

[root@labserver ~]# groupadd managers
[root@labserver ~]# groupadd developers
[root@labserver ~]# groupadd operations

[root@labserver ~]# chage -l b-user
Last password change					: Jul 24, 2026
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7

[root@labserver ~]# grep b-user /etc/group
b-user:x:1002:

[root@labserver ~]# usermod -c "Bb User, Lalitpur, 9798457832" -e 2027-01-01 -g managers -G developers,operations -u 2002 -s /bin/ksh b-user
usermod: Warning: missing or non-executable shell '/bin/ksh'

[root@labserver ~]# chage -l b-user
Last password change					: Jul 24, 2026
Password expires					: never
Password inactive					: never
Account expires						: Dec 31, 2026
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7

[root@labserver ~]# grep b-user /etc/passwd
b-user:x:2002:1003:Bb User, Lalitpur, 9798457832:/home/b-user:/bin/ksh
 
[root@labserver ~]# grep b-user /etc/group
b-user:x:1002:
developers:x:1004:b-user
operations:x:1005:b-user

[root@labserver ~]# groups b-user
b-user : managers developers operations
[root@labserver ~]# 

[root@labserver ~]# su - b-user
su: failed to execute /bin/ksh: No such file or directory

[root@labserver ~]# usermod -s /bin/bash b-user
[root@labserver ~]# su - b-user
Last login: Fri Jul 24 18:57:23 +0545 2026 on pts/0
[b-user@labserver ~]$ whoami
b-user
[b-user@labserver ~]$ pwd
/home/b-user

[b-user@labserver ~]$ exit
logout
[root@labserver ~]# usermod -l bishal b-user
[root@labserver ~]# grep b-user /etc/passwd
bishal:x:2002:1003:Bb User, Lalitpur, 9798457832:/home/b-user:/bin/bash            # Home directory remains /home/b-user

[root@labserver ~]# grep b-user /etc/shadow

[root@labserver ~]# grep bishal /etc/shadow
bishal:$y$j9T$BOtZEsmKfliEvI/9VLki21$2GdWuL3462NjklMJaqok1Uy2rwtn/SpJRPTOpKjZrl9:20658:0:99999:7::20818:

[root@labserver ~]# groups bishal
bishal : managers developers operations

 # Lock and Unlock the user

[root@labserver ~]# grep bishal /etc/shadow
bishal:$y$j9T$BOtZEsmKfliEvI/9VLki21$2GdWuL3462NjklMJaqok1Uy2rwtn/SpJRPTOpKjZrl9:20658:0:99999:7::20818:
 
[root@labserver ~]# usermod -L bishal

[root@labserver ~]# grep bishal /etc/shadow
bishal:!$y$j9T$BOtZEsmKfliEvI/9VLki21$2GdWuL3462NjklMJaqok1Uy2rwtn/SpJRPTOpKjZrl9:20658:0:99999:7::20818:	   # Lock --> !
 
[root@labserver ~]# usermod -U bishal                                           # Unlock

[root@labserver ~]# grep bishal /etc/shadow
bishal:$y$j9T$BOtZEsmKfliEvI/9VLki21$2GdWuL3462NjklMJaqok1Uy2rwtn/SpJRPTOpKjZrl9:20658:0:99999:7::20818:

[root@labserver ~]# tail -2 /var/log/secure 					# View Logs
Jul 24 08:36:21 labserver usermod[2079]: lock user 'bishal' password
Jul 24 08:37:16 labserver usermod[2088]: unlock user 'bishal' password

 # Setting User's Password Policy
 
  # chage [options] <username/login name>
 
  # options
 
   # -l  -->  <list policy>
   # -m  -->  <min no of days between password change>
   # -M  -->  <max no. of days between password change>
   # -W  -->  <warning days before password expires>
   # -I  -->  <grace period before locking password after the password expires>
   # -E  -->  <a/c expiray date>
   
[root@labserver ~]# chage --h

[root@labserver ~]# chage -l bishal
Last password change					: Jul 24, 2026
Password expires					: never
Password inactive					: never
Account expires						: Dec 31, 2026
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
 
[root@labserver ~]# vi /etc/login.defs 

[root@labserver ~]# vi /etc/default/useradd 

[root@labserver ~]# chage -l bishal
Last password change					: Jul 24, 2026
Password expires					: never
Password inactive					: never
Account expires						: Dec 31, 2026
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7

[root@labserver ~]# chage -m 5 -M 365 -W 10 -E 2028-01-01 -I 90 bishal
 
[root@labserver ~]# chage -l bishal
Last password change					: Jul 24, 2026
Password expires					: Jul 24, 2027
Password inactive					: Oct 22, 2027
Account expires						: Dec 31, 2027
Minimum number of days between password change		: 5
Maximum number of days between password change		: 365
Number of days of warning before password expires	: 10


 # Enforcing a user to change their password on First Login ( production use case )

[root@labserver ~]# chage -h

[root@labserver ~]# chage -d 0 bishal          # Force the user bishal to change their password at the next login by setting the last password change date to 0.

 # --> chage -d 0 <username> sets the last password change date to January 1, 1970 (day 0), causing the password to be treated as expired immediately.
 
[root@labserver ~]# chage -l bishal
Last password change					: password must be changed
Password expires					: password must be changed
Password inactive					: password must be changed
Account expires						: Dec 31, 2027
Minimum number of days between password change		: 5
Maximum number of days between password change		: 365
Number of days of warning before password expires	: 10

[root@labserver ~]# exit
logout

[aadarsha@labserver ~]$ su - bishal
Password: 
You are required to change your password immediately (administrator enforced).
Current password: 
New password: 
Retype new password: 
Last login: Fri Jul 24 18:58:31 +0545 2026 on pts/0
Last failed login: Sat Jul 25 09:36:45 +0545 2026 on pts/0
There was 1 failed login attempt since the last successful login.

[bishal@labserver ~]$ whoami
bishal

[bishal@labserver ~]$ exit
logout

[aadarsha@labserver ~]$ su - bishal
Password: 
Last login: Sat Jul 25 09:37:40 +0545 2026 on pts/0
[bishal@labserver ~]$ 

[bishal@labserver ~]$ exit
logout

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Sat Jul 25 08:30:02 +0545 2026 on pts/0
[root@labserver ~]# 

 # PAM (Pluggable Authentication Modules) 
 # is the Linux authentication framework that enforces authentication, account, password, and session policies for services such as login, sshd, su, and sudo.
 
 # Primary PAM configuration: /etc/pam.d/   ---> Contains PAM configuration files for individual services (recommended location).
 # Supporting policy files: /etc/security/   ---> Contains supporting configuration files used by PAM modules (e.g., limits.conf, access.conf, faillock.conf).

[root@labserver ~]# ls /etc/pam.d/
chfn  config-util  fingerprint-auth  other   password-auth  remote   runuser-l       sshd              su    sudo-i  switchable-auth  vlock
chsh  crond        login             passwd  postlogin      runuser  smartcard-auth  sssd-shadowutils  sudo  su-l    system-auth
[root@labserver ~]# 

 # Deleting User's Account
 
 # userdel -r <username> ---> To delete the user completely (along with home dir and mail spool)
 # userdel <username>    ---> To delete the user completely (except home dir and mail spool)

[root@labserver ~]# ls /home
aadarsha  a-user  b-user
 
[root@labserver ~]# tail -2 /etc/passwd
a-user:x:1001:1001::/home/a-user:/bin/bash
bishal:x:2002:1003:Bb User, Lalitpur, 9798457832:/home/b-user:/bin/bash

[root@labserver ~]# grep bishal /etc/shadow
bishal:$y$j9T$VlRZkORrgRGKkAJ.bJ2XJ.$GDc9S3J3VIp1ibU3tBO82T4Fdn5M4UZsgLQXqI5vr6.:20659:5:365:10:90:21183:

[root@labserver ~]# grep bishal /etc/group
developers:x:1004:bishal
operations:x:1005:bishal
 
[root@labserver ~]# groups bishal
bishal : managers developers operations
 
[root@labserver ~]# ls /var/spool/mail/
aadarsha  a-user  bishal

[root@labserver ~]# userdel -r bishal
 
[root@labserver ~]# ls /home/
aadarsha  a-user
[root@labserver ~]# grep bishal /etc/passwd
[root@labserver ~]# grep bishal /etc/shadow
[root@labserver ~]# grep bishal /etc/group
[root@labserver ~]# groups bishal
groups: ‘bishal’: no such user
[root@labserver ~]# ls /var/spool/mail/
aadarsha  a-user

[root@labserver ~]# useradd ram
[root@labserver ~]# ls /home/
aadarsha  a-user  ram
[root@labserver ~]# grep ram /etc/passwd
ram:x:1002:1006::/home/ram:/bin/bash
[root@labserver ~]# grep ram /etc/shadow
ram:!:20659:0:99999:7:::
[root@labserver ~]# passwd ram
New password: 
Retype new password: 
passwd: password updated successfully
[root@labserver ~]# grep ram /etc/shadow
ram:$y$j9T$kD5ld9mfTMku4r7MevImP1$GJWefZ6k9FpdExJke41Xl8WucFUPSUujue4wNsCr0Z2:20659:0:99999:7:::
[root@labserver ~]#
[root@labserver ~]# grep ram /etc/group
ram:x:1006:
[root@labserver ~]# ls -l /var/spool/mail/ram 
-rw-rw----. 1 ram mail 0 Jul 25 13:53 /var/spool/mail/ram
[root@labserver ~]# ls -l /home/ram/
total 4
-rw-r--r--. 1 ram ram 19 Jul 24 14:25 myfile
[root@labserver ~]# ls -ld /home/ram/
drwx------. 2 ram ram 76 Jul 25 13:53 /home/ram/
[root@labserver ~]# su - ram
[ram@labserver ~]$ pwd
/home/ram
[ram@labserver ~]$ ls
myfile
[ram@labserver ~]$ vi ram-file
[ram@labserver ~]$ ls -l
total 8
-rw-r--r--. 1 ram ram 19 Jul 24 14:25 myfile
-rw-r--r--. 1 ram ram 35 Jul 25 13:59 ram-file
[ram@labserver ~]$ 
 
[ram@labserver ~]$ userdel ram
userdel: user ram is currently used by process 4240
[ram@labserver ~]$ 
[ram@labserver ~]$ ps
    PID TTY          TIME CMD
   4240 pts/0    00:00:00 bash
   4339 pts/0    00:00:00 ps
[ram@labserver ~]$ 
[ram@labserver ~]$ exit
logout
[root@labserver ~]# userdel ram
[root@labserver ~]# ls /home/
aadarsha  a-user  ram
[root@labserver ~]# 
[root@labserver ~]# ls -ld /home/ram
drwx------. 2 1002 1006 113 Jul 25 14:07 /home/ram
[root@labserver ~]# 
[root@labserver ~]# ls -l /home/ram/
total 8
-rw-r--r--. 1 1002 1006 19 Jul 24 14:25 myfile
-rw-r--r--. 1 1002 1006 35 Jul 25 13:59 ram-file
[root@labserver ~]# 
[root@labserver ~]# grep ram /etc/passwd
[root@labserver ~]# grep ram /etc/shadow
[root@labserver ~]# grep ram /etc/group
[root@labserver ~]# 
[root@labserver ~]# ls -ld /var/spool/mail/ram 
-rw-rw----. 1 1002 mail 0 Jul 25 13:53 /var/spool/mail/ram
[root@labserver ~]# 

 # When user ram is deleted, it's home directory and mail directory remains, now giving new user permission to use those left directories
 # ( use case: When an employees leaves company, the resources used by them are handed to other employee )
 
[root@labserver ~]# useradd -u 1002 -d /home/ram sita               # assign same UID and Home Directory to new user sita
useradd: warning: the home directory /home/ram already exists.
useradd: Not copying any file from skel directory into it.
[root@labserver ~]# 
[root@labserver ~]# passwd sita
New password: 
Retype new password: 
passwd: password updated successfully
[root@labserver ~]# ls -ld /home/ram
drwx------. 2 sita sita 113 Jul 25 14:07 /home/ram
[root@labserver ~]# 
[root@labserver ~]# ls -ld /var/spool/mail/ram 
-rw-rw----. 1 sita mail 0 Jul 25 13:53 /var/spool/mail/ram
[root@labserver ~]# 
[root@labserver ~]# ls -l /home/ram/
total 8
-rw-r--r--. 1 sita sita 19 Jul 24 14:25 myfile
-rw-r--r--. 1 sita sita 35 Jul 25 13:59 ram-file
[root@labserver ~]# 
[root@labserver ~]# su - sita
[sita@labserver ~]$ whoami
sita
[sita@labserver ~]$ pwd
/home/ram
[sita@labserver ~]$ ls -l
total 8
-rw-r--r--. 1 sita sita 19 Jul 24 14:25 myfile
-rw-r--r--. 1 sita sita 35 Jul 25 13:59 ram-file
[sita@labserver ~]$ 
[sita@labserver ~]$ exit
logout
[root@labserver ~]# grep sita /etc/passwd
sita:x:1002:1006::/home/ram:/bin/bash
[root@labserver ~]# 
[root@labserver ~]# cd /home/
[root@labserver home]# ls
aadarsha  a-user  ram
[root@labserver home]# mv ram sita
[root@labserver home]# ls
aadarsha  a-user  sita
[root@labserver home]#
[root@labserver home]# ls -l
total 0
drwx------. 2 aadarsha aadarsha  83 Jul 13 14:46 aadarsha
drwx------. 2 a-user   a-user    83 Jul 24 17:28 a-user
drwx------. 2 sita     sita     113 Jul 25 14:07 sita
[root@labserver home]# 
[root@labserver ~]# grep sita /etc/passwd
sita:x:1002:1006::/home/ram:/bin/bash
[root@labserver ~]# usermod -d /home/sita sita
[root@labserver ~]# grep sita /etc/passwd
sita:x:1002:1006::/home/sita:/bin/bash
[root@labserver ~]# 
[root@labserver ~]# cd /var/spool/mail/
[root@labserver mail]# ls
aadarsha  a-user  ram  sita
[root@labserver mail]# mv ram sita 
mv: overwrite 'sita'? y
[root@labserver mail]# ls -l
total 0
-rw-rw----. 1 aadarsha mail 0 Jul 13 14:32 aadarsha
-rw-rw----. 1 a-user   mail 0 Jul 24 12:06 a-user
-rw-rw----. 1 sita     mail 0 Jul 25 13:53 sita
[root@labserver mail]# 

 # Managing groups
 
[root@labserver ~]# groupadd friends

[root@labserver ~]# grep friends /etc/group
friends:x:1007:
[root@labserver ~]# tail -9 /etc/group
systemd-coredump:x:995:
aadarsha:x:1000:
a-user:x:1001:
b-user:x:1002:
managers:x:1003:
developers:x:1004:
operations:x:1005:
sita:x:1006:
friends:x:1007:
[root@labserver ~]# 

[root@labserver ~]# groupmod -g 1008 friends
[root@labserver ~]# grep friends /etc/group
friends:x:1008:
[root@labserver ~]#

[root@labserver ~]# groupmod -n newfriends friends
[root@labserver ~]# grep friends /etc/group
newfriends:x:1008:
[root@labserver ~]# 

[root@labserver ~]# groupdel newfriends
[root@labserver ~]# grep newfriends /etc/group
[root@labserver ~]#

 # Linux User and Group Management: Organizational Structure

		Boss
		│
		└── Manager
		    ├── Production
		    │   ├── prod-user1
		    │   └── prod-user2
		    ├── Marketing
		    │   ├── market-user1
		    │   └── market-user2
		    └── Sales
		        ├── sale-user1
		        └── sale-user2

 # 1. Create departments (groups)
 
[root@labserver ~]# groupadd production
[root@labserver ~]# groupadd marketing
[root@labserver ~]# groupadd sales

[root@labserver ~]# getent group

[root@labserver ~]# tail -4 /etc/group
aadarsha:x:1000:
production:x:1001:
marketing:x:1002:
sales:x:1003:

 # 2. Create users in each department (group)
 
[root@labserver ~]# useradd --help

 # production
 
[root@labserver ~]# useradd -G production prod-user1
[root@labserver ~]# useradd -G production prod-user2

[root@labserver ~]# grep production /etc/group
production:x:1001:prod-user1,prod-user2

 # marketing

[root@labserver ~]# useradd -G marketing market-user1
[root@labserver ~]# useradd -G marketing market-user2

[root@labserver ~]# grep marketing /etc/group
marketing:x:1002:market-user1,market-user2

 # sales 

[root@labserver ~]# useradd -G sales sale-user1
[root@labserver ~]# useradd -G sales sale-user2

[root@labserver ~]# grep sales /etc/group
sales:x:1003:sale-user1,sale-user2
 
[root@labserver ~]# tail /etc/group
aadarsha:x:1000:
production:x:1001:prod-user1,prod-user2
marketing:x:1002:market-user1,market-user2
sales:x:1003:sale-user1,sale-user2
prod-user1:x:1004:
prod-user2:x:1005:
market-user1:x:1006:
market-user2:x:1007:
sale-user1:x:1008:
sale-user2:x:1009:
[root@labserver ~]# 

 # 3. Create the manager user and assign supplementary groups

[root@labserver ~]# useradd --help

[root@labserver ~]# useradd -m -G production,marketing,sales manager

[root@labserver ~]# grep manager /etc/group
production:x:1001:prod-user1,prod-user2,manager
marketing:x:1002:market-user1,market-user2,manager
sales:x:1003:sale-user1,sale-user2,manager
manager:x:1010:

[root@labserver ~]# id manager
uid=1007(manager) gid=1010(manager) groups=1010(manager),1001(production),1002(marketing),1003(sales)

[root@labserver ~]# useradd -m -G production,marketing,sales boss
[root@labserver ~]# id boss
uid=1009(boss) gid=1012(boss) groups=1012(boss),1001(production),1002(marketing),1003(sales)

[root@labserver ~]# tail -13 /etc/group
aadarsha:x:1000:
production:x:1001:prod-user1,prod-user2,manager,new-user1,boss
marketing:x:1002:market-user1,market-user2,manager,new-user1,boss
sales:x:1003:sale-user1,sale-user2,manager,new-user1,boss
prod-user1:x:1004:
prod-user2:x:1005:
market-user1:x:1006:
market-user2:x:1007:
sale-user1:x:1008:
sale-user2:x:1009:
manager:x:1010:
new-user1:x:1011:
boss:x:1012:

 # add new-user1

[root@labserver ~]# useradd new-user1

[root@labserver ~]# grep new-user1 /etc/group
new-user1:x:1011:

 # To add new-user1 in production group (department)

[root@labserver ~]# usermod -G production new-user1
 
[root@labserver ~]# grep new-user1 /etc/group
production:x:1001:prod-user1,prod-user2,manager,new-user1
new-user1:x:1011:

[root@labserver ~]# usermod -G marketing,sales new-user1                  # Replaces supplementary groups

[root@labserver ~]# grep new-user1 /etc/group
marketing:x:1002:market-user1,market-user2,manager,new-user1              # --> Verify that production membership was removed
sales:x:1003:sale-user1,sale-user2,manager,new-user1
new-user1:x:1011:

 # Append production without removing existing groups

[root@labserver ~]# usermod -aG production new-user1

[root@labserver ~]# grep new-user1 /etc/group
production:x:1001:prod-user1,prod-user2,manager,new-user1
marketing:x:1002:market-user1,market-user2,manager,new-user1
sales:x:1003:sale-user1,sale-user2,manager,new-user1
new-user1:x:1011:

[root@labserver ~]# groups new-user1
new-user1 : new-user1 production marketing sales

 # Assign a temporary password for testing purposes only. ( not used in production environment ) 
 
[root@labserver ~]# grep new-user1 /etc/shadow
new-user1:!:20659:0:99999:7:::
[root@labserver ~]# grep prod-user1 /etc/shadow
prod-user1:!:20659:0:99999:7:::
[root@labserver ~]# grep prod-user2 /etc/shadow
prod-user2:!:20659:0:99999:7:::

 # Read the password from standard input (stdin)
 
[root@labserver ~]# echo "Nepal-123" | passwd --stdin new-user1
[root@labserver ~]# echo "Nepal-123" | passwd --stdin prod-user1
[root@labserver ~]# echo "Nepal-123" | passwd --stdin prod-user2
[root@labserver ~]# echo "Nepal-123" | passwd --stdin sale-user1
[root@labserver ~]# echo "Nepal-123" | passwd --stdin sale-user2

[root@labserver ~]# grep new-user1 /etc/shadow
new-user1:$y$j9T$9g9Oov3zZEAi7LzA.X44u/$ZPuULJkBpBATIzw0.H..YVZ5gjGVt/38Y1ZcyJOO6m4:20659:0:99999:7:::
[root@labserver ~]# grep prod-user1 /etc/shadow
prod-user1:$y$j9T$Iagb2/1WX0DC4TB/aOdmp1$0Mop5VZb6qeDRGOj71bUbrov0tihMQ24Vh01LU1qLSD:20659:0:99999:7:::
[root@labserver ~]# grep prod-user2 /etc/shadow
prod-user2:$y$j9T$dRE4E7v0xEkEZBrhEbd.a1$ojJFePa/3H0vIuLsZ3MuJbWkmfv/TRw31Wpds8qGddA:20659:0:99999:7:::

[root@labserver ~]# echo "market-user1:Nepal-123" | chpasswd 
[root@labserver ~]# echo "market-user2:Nepal-123" | chpasswd 

[root@labserver ~]# grep market-user1 /etc/shadow
market-user1:$y$j9T$yCakDMvFpwR1s7yf9KwPs1$Q67N9G/x/i1RUhMjiY/rzXxK1Ov26cfrW1KM3JMb3BA:20660:0:99999:7:::
[root@labserver ~]# grep market-user2 /etc/shadow
market-user2:$y$j9T$wFAi7QKerJUzEHmeVYrpY/$JJcCxEuT/HnX3okVbjAcYVpqqqTHMupf00zi5jsPdf1:20660:0:99999:7:::

 # Note:
   # Linux does not support nested groups.
   # The company hierarchy is represented by supplementary group memberships
   # rather than parent-child relationships between groups.

 # 3. Create departmental directories/folders

[root@labserver ~]# cd /
[root@labserver /]# pwd
/
[root@labserver /]# ls -ld
dr-xr-xr-x. 19 root root 250 Jul 22 13:23 .

[root@labserver /]# ls
abcbank  afs  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

[root@labserver /]# mkdir production marketing sales
[root@labserver /]# ls
abcbank  afs  bin  boot  dev  etc  home  lib  lib64  marketing  media  mnt  opt  proc  production  root  run  sales  sbin  srv  sys  tmp  usr  var

[root@labserver /]# ls -ld production marketing sales
drwxr-xr-x. 2 root root 6 Jul 25 21:44 marketing
drwxr-xr-x. 2 root root 6 Jul 25 21:44 production
drwxr-xr-x. 2 root root 6 Jul 25 21:44 sales

[root@labserver /]# whoami
root

 # 4. Change the ownership of the departmental directories 

 # chown -R <new owner>:<new group> <dir/file>		  # -R --> recursively apply inside the directories
 
[root@labserver /]# chown -R boss:marketing marketing
[root@labserver /]# chown -R boss:production production
[root@labserver /]# chown -R boss:sales sales

[root@labserver /]# ls -ld marketing production sales
drwxr-xr-x. 2 boss marketing  6 Jul 25 21:44 marketing
drwxr-xr-x. 2 boss production 6 Jul 25 21:44 production
drwxr-xr-x. 2 boss sales      6 Jul 25 21:44 sales

[root@labserver ~]# su - prod-user1
[prod-user1@labserver ~]$ whoami
prod-user1
[prod-user1@labserver ~]$ ls -ld /production
drwxr-xr-x. 2 boss production 6 Jul 25 21:44 /production

[prod-user1@labserver ~]$ cd /production
[prod-user1@labserver production]$ ls
[prod-user1@labserver production]$ mkdir dir1
mkdir: cannot create directory ‘dir1’: Permission denied
 
[prod-user1@labserver production]$ cd
[prod-user1@labserver ~]$ 

 # 5. Set the appropriate required persmissions on the departmental directories
 
[prod-user1@labserver ~]$ exit
logout

[root@labserver ~]# chmod 770 /production /marketing /sales

[root@labserver ~]# ls -ld /production /marketing /sales
drwxrwx---. 2 boss marketing  6 Jul 25 21:44 /marketing
drwxrwx---. 2 boss production 6 Jul 25 21:44 /production
drwxrwx---. 2 boss sales      6 Jul 25 21:44 /sales

[root@labserver ~]# su - prod-user1
Last login: Sun Jul 26 08:07:10 +0545 2026 on pts/0
[prod-user1@labserver ~]$ whoami
prod-user1
[prod-user1@labserver ~]$ cd /production

[prod-user1@labserver production]$ ls -ld
drwxrwx---. 3 boss production 39 Jul 26 08:19 .			     # on current directory ownership: <user = boss> : <group = production>

[prod-user1@labserver production]$ vi testfile1
[prod-user1@labserver production]$ mkdir testdir1

[prod-user1@labserver production]$ ls -l
total 4
drwxr-xr-x. 2 prod-user1 prod-user1  6 Jul 26 08:19 testdir1         # on current directory newly created files ownership: <user = prod-user1> : <group = prod-user1> BUT
-rw-r--r--. 1 prod-user1 prod-user1 25 Jul 26 08:19 testfile1        # By default we want: <user = prod-user1> : <group = production>

[prod-user1@labserver production]$ cd
[prod-user1@labserver ~]$ 

[prod-user1@labserver ~]$ cd /sales
-bash: cd: /sales: Permission denied

 # remove user from the particular group
 
[root@labserver ~]# groups new-user1
new-user1 : new-user1 production marketing sales

[root@labserver ~]# groups new-user1
new-user1 : new-user1 production marketing sales

[root@labserver ~]# vi /etc/group                                  # remove new-user from production group in /etc/group file
 
[root@labserver ~]# groups new-user1
new-user1 : new-user1 marketing sales

[root@labserver ~]# man usermod

[root@labserver ~]# usermod -rG marketing new-user1                # remove from marketing group 

[root@labserver ~]# groups new-user1
new-user1 : new-user1 sales

[root@labserver ~]# su - sale-user1

[sale-user1@labserver ~]$ cd /sales/

[sale-user1@labserver sales]$ ls -ld
drwxrwx---. 2 boss sales 6 Jul 25 21:44 .

[sale-user1@labserver sales]$ mkdir dir1
[sale-user1@labserver sales]$ ls -ld dir1
drwxr-xr-x. 2 sale-user1 sale-user1 6 Jul 26 08:48 dir1           # By default group should be sales, so use SGID to inherit the parent directory's group ownership
 
[sale-user1@labserver sales]$ exit
logout
[root@labserver ~]# 

 # Special permission bits : SUID (4), SGID (2), Sticky Bit (1)

 # SGID Bit - Set Group ID Bit
 # Octal Value: 2
 # SGID ensures that new files and directories inherit the parent directory's group ownership instead of the creator's primary group. 
 # If SGID bit is set on a directory then any files/dirs created inside the folder inherits group ownership of that group
 # Use Case: Shared team directories where multiple users need automatic, collaborative read/write access to newly created files.

[root@labserver ~]# whoami
root
[root@labserver ~]# cd /
[root@labserver /]# pwd
/
[root@labserver /]# ls -ld production marketing sales
drwxrwx---. 2 boss marketing   6 Jul 25 21:44 marketing
drwxrwx---. 3 boss production 39 Jul 26 08:19 production
drwxrwx---. 3 boss sales      18 Jul 26 08:48 sales
 
[root@labserver /]# chmod g+s marketing production sales

[root@labserver /]# ls -ld production marketing sales
drwxrws---. 2 boss marketing   6 Jul 25 21:44 marketing
drwxrws---. 3 boss production 39 Jul 26 08:19 production
drwxrws---. 3 boss sales      18 Jul 26 08:48 sales

 # 'x' (execute) permission of group member is replaced by 's'

  #  s -->  SGID bit + Execute is set
  #  S -->  SGID bit only set
  #  x -->  Execute only set

[root@labserver /]# chmod g-x marketing

[root@labserver /]# ls -ld marketing production sales
drwxrwS---. 2 boss marketing   6 Jul 25 21:44 marketing
drwxrws---. 3 boss production 39 Jul 26 08:19 production
drwxrws---. 3 boss sales      18 Jul 26 08:48 sales

[root@labserver /]# chmod g+x marketing

[root@labserver /]# su - sale-user1
Last login: Sun Jul 26 08:47:31 +0545 2026 on pts/0

[sale-user1@labserver ~]$ whoami
sale-user1

[sale-user1@labserver ~]$ pwd
/home/sale-user1

[sale-user1@labserver ~]$ cd /sales

[sale-user1@labserver sales]$ pwd
/sales
[sale-user1@labserver sales]$ ls -l
total 0
drwxr-xr-x. 2 sale-user1 sale-user1 6 Jul 26 08:48 dir1

[sale-user1@labserver sales]$ ls -ld
drwxrws---. 3 boss sales 18 Jul 26 08:48 .

[sale-user1@labserver sales]$ mkdir dir2

[sale-user1@labserver sales]$ touch sales-report-2026

[sale-user1@labserver sales]$ ls -l
total 0
drwxr-xr-x. 2 sale-user1 sale-user1 6 Jul 26 08:48 dir1                      # --> Created before setting SGID bit
drwxr-sr-x. 2 sale-user1 sales      6 Jul 26 16:17 dir2
-rw-r--r--. 1 sale-user1 sales      0 Jul 26 16:18 sales-report-2026

 # SGID is not retroactive. It affects only new files and directories created after the SGID bit is set.
 
 # So, manual change is required to change to sales group for dir1

[sale-user1@labserver sales]$ chgrp -R sales /sales/dir1

[sale-user1@labserver sales]$ ls -l
total 0
drwxr-xr-x. 2 sale-user1 sales 6 Jul 26 08:48 dir1
drwxr-sr-x. 2 sale-user1 sales 6 Jul 26 16:17 dir2
-rw-r--r--. 1 sale-user1 sales 0 Jul 26 16:18 sales-report-2026

[sale-user1@labserver sales]$ cd
[sale-user1@labserver ~]$ pwd
/home/sale-user1

[sale-user1@labserver ~]$ ls
[sale-user1@labserver ~]$ mkdir dir1
[sale-user1@labserver ~]$ vi report1
[sale-user1@labserver ~]$ ls -l
total 4
drwxr-xr-x. 2 sale-user1 sale-user1  6 Jul 26 16:37 dir1
-rw-r--r--. 1 sale-user1 sale-user1 43 Jul 26 16:37 report1

[sale-user1@labserver ~]$ ls -ld /home/sale-user1
drwx------. 3 sale-user1 sale-user1 110 Jul 26 16:37 /home/sale-user1

[sale-user1@labserver ~]$ exit
logout
[root@labserver /]# su - sale-user2
[sale-user2@labserver ~]$ pwd
/home/sale-user2
[sale-user2@labserver ~]$ cd /home/sale-user1
-bash: cd: /home/sale-user1: Permission denied

[sale-user2@labserver ~]$ ls -ld /home/sale-user1
drwx------. 3 sale-user1 sale-user1 110 Jul 26 16:37 /home/sale-user1

[sale-user2@labserver ~]$ cd /sales/
[sale-user2@labserver sales]$ ls
dir1  dir2  sales-report-2026
[sale-user2@labserver sales]$ ls -l
total 0
drwxr-xr-x. 2 sale-user1 sales 6 Jul 26 08:48 dir1
drwxr-sr-x. 2 sale-user1 sales 6 Jul 26 16:17 dir2
-rw-r--r--. 1 sale-user1 sales 0 Jul 26 16:18 sales-report-2026

[sale-user1@labserver sales]$ chmod g+s /sales/dir1

[sale-user1@labserver sales]$ ls -l
total 0
drwxr-sr-x. 2 sale-user1 sales 6 Jul 26 08:48 dir1
drwxr-sr-x. 2 sale-user1 sales 6 Jul 26 16:17 dir2
-rw-r--r--. 1 sale-user1 sales 0 Jul 26 17:12 sales-report-2026

[sale-user1@labserver sales]$ exit
logout
[root@labserver /]# su - sale-user2
Last login: Sun Jul 26 16:40:57 +0545 2026 on pts/0
[sale-user2@labserver ~]$ cd /sales/
[sale-user2@labserver sales]$ ls -l
total 0
drwxr-sr-x. 2 sale-user1 sales 6 Jul 26 08:48 dir1
drwxr-sr-x. 2 sale-user1 sales 6 Jul 26 16:17 dir2
-rw-r--r--. 1 sale-user1 sales 0 Jul 26 17:12 sales-report-2026

[sale-user2@labserver sales]$ touch report2
[sale-user2@labserver sales]$ mkdir dir3
[sale-user2@labserver sales]$ ls -l
total 0
drwxr-sr-x. 2 sale-user1 sales 6 Jul 26 08:48 dir1
drwxr-sr-x. 2 sale-user1 sales 6 Jul 26 16:17 dir2
drwxr-sr-x. 2 sale-user2 sales 6 Jul 26 17:39 dir3
-rw-r--r--. 1 sale-user2 sales 0 Jul 26 17:39 report2
-rw-r--r--. 1 sale-user1 sales 0 Jul 26 17:12 sales-report-2026

[sale-user2@labserver sales]$ echo "sale-user2 can not modify dir/files created by sale-user1" >> sales-report-2026 
-bash: sales-report-2026: Permission denied
[sale-user2@labserver sales]$ 

[sale-user2@labserver sales]$ umask
0022
 
 # SGID does not grant write access. It only controls group ownership inheritance.
 
 # Add umask 007 in /etc/bashrc 
 
 # For files:
 
  # 666 - 007 = 660
  # -rw-rw----
 
 # For directories:
 
  # 777 - 007 = 770
  # drwxrwx---

[root@labserver /]# vi /etc/bashrc 
[root@labserver /]# 
 
 # --- added line ---
 umask 007

[root@labserver /]# umask
0022

[root@labserver /]# source /etc/bashrc 

[root@labserver /]# umask
0007

[root@labserver /]# su - sale-user1
Last login: Sun Jul 26 17:11:09 +0545 2026 on pts/0
 
[sale-user1@labserver ~]$ whoami
sale-user1
[sale-user1@labserver ~]$ cd /sales/
[sale-user1@labserver sales]$ pwd
/sales
[sale-user1@labserver sales]$ ls
dir1  dir2  dir3  report2  sales-report-2026

[sale-user1@labserver sales]$ mkdir newdir1
[sale-user1@labserver sales]$ vi newsales-2026

[sale-user1@labserver sales]$ cat newsales-2026 
this is new sales data... added by sale-user1

[sale-user1@labserver sales]$ ls -l
total 4
drwxr-sr-x. 2 sale-user1 sales  6 Jul 26 08:48 dir1
drwxr-sr-x. 2 sale-user1 sales  6 Jul 26 16:17 dir2
drwxr-sr-x. 2 sale-user2 sales  6 Jul 26 17:39 dir3
drwxrws---. 2 sale-user1 sales  6 Jul 26 19:32 newdir1
-rw-rw----. 1 sale-user1 sales 26 Jul 26 19:32 newsales-2026
-rw-r--r--. 1 sale-user2 sales  0 Jul 26 17:39 report2
-rw-r--r--. 1 sale-user1 sales  0 Jul 26 17:12 sales-report-2026
[sale-user1@labserver sales]$ 

[sale-user1@labserver sales]$ exit
logout
[root@labserver /]# su - sale-user2
Last login: Sun Jul 26 19:35:49 +0545 2026 on pts/0
[sale-user2@labserver ~]$ cd /sales/
[sale-user2@labserver sales]$ ls
dir1  dir2  dir3  newdir1  newsales-2026  report2  sales-report-2026

[sale-user2@labserver sales]$ vi newsales-2026 
[sale-user2@labserver sales]$ cat newsales-2026 
this is new sales data... added by sale-user1
this is new sales data... added by sale-user2
[sale-user2@labserver sales]$ 
[sale-user2@labserver sales]$ cd newdir1/
[sale-user2@labserver newdir1]$ touch newreport2
[sale-user2@labserver newdir1]$ ls -l
total 0
-rw-rw----. 1 sale-user2 sales 0 Jul 26 19:43 newreport2
[sale-user2@labserver newdir1]$ cd ..
[sale-user2@labserver sales]$ ls -l
total 4
drwxr-sr-x. 2 sale-user1 sales  6 Jul 26 08:48 dir1
drwxr-sr-x. 2 sale-user1 sales  6 Jul 26 16:17 dir2
drwxr-sr-x. 2 sale-user2 sales  6 Jul 26 17:39 dir3
drwxrws---. 2 sale-user1 sales 24 Jul 26 19:43 newdir1
-rw-rw----. 1 sale-user1 sales 92 Jul 26 19:42 newsales-2026
-rw-r--r--. 1 sale-user2 sales  0 Jul 26 17:39 report2
-rw-r--r--. 1 sale-user1 sales  0 Jul 26 17:12 sales-report-2026
[sale-user2@labserver sales]$ 

 # Sticky Bit
 # Octal Value: 1
 # Restricts file deletion within a directory. Only the file owner, directory owner, or root can delete or rename files.
 # Use Case: Publicly accessible directories like /tmp, preventing users from accidentally or maliciously deleting each other's work.

[sale-user2@labserver sales]$ ls 
dir1  dir2  dir3  newdir1  newsales-2026  report2  sales-report-2026
 
[sale-user2@labserver sales]$ ls newdir1/
newreport2

[sale-user2@labserver sales]$ ls -l newdir1/newreport2 
-rw-rw----. 1 sale-user2 sales 0 Jul 26 19:43 newdir1/newreport2

[sale-user2@labserver sales]$ exit
logout
[root@labserver /]# su - sale-user1
Last login: Sun Jul 26 19:38:33 +0545 2026 on pts/0

[sale-user1@labserver ~]$ cd /sales/
[sale-user1@labserver sales]$ ls -l newdir1/newreport2 
-rw-rw----. 1 sale-user2 sales 0 Jul 26 19:43 newdir1/newreport2

[sale-user1@labserver sales]$ rm newdir1/newreport2 		# newreport2 was created by sale-user2 and removed/deleted by sale-user1

[sale-user1@labserver sales]$ ls newdir1/
[sale-user1@labserver sales]$ 

[sale-user1@labserver sales]$ ls -ld /sales
drwxrws---. 6 boss sales 118 Jul 26 19:42 /sales

 # apply Stick Bit

[sale-user1@labserver sales]$ exit
logout
[root@labserver ~]# 

[root@labserver ~]# ls -ld /production /marketing /sales
drwxrws---. 2 boss marketing    6 Jul 25 21:44 /marketing
drwxrws---. 3 boss production  39 Jul 26 08:19 /production
drwxrws---. 6 boss sales      118 Jul 26 19:42 /sales
 
[root@labserver ~]# chmod o+t /production /marketing /sales

[root@labserver ~]# ls -ld /production /marketing /sales
drwxrws--T. 2 boss marketing    6 Jul 25 21:44 /marketing
drwxrws--T. 3 boss production  39 Jul 26 08:19 /production
drwxrws--T. 6 boss sales      118 Jul 26 19:42 /sales

 # T  --> Sticky bit
 # x  --> execute
 # t  --> execute + Sticky bit

[root@labserver ~]# chmod o+x /production

[root@labserver ~]# ls -ld /production /marketing /sales
drwxrws--T. 2 boss marketing    6 Jul 25 21:44 /marketing
drwxrws--t. 3 boss production  39 Jul 26 08:19 /production
drwxrws--T. 6 boss sales      118 Jul 26 19:42 /sales
[root@labserver ~]# 

[root@labserver ~]# su - sale-user1
Last login: Mon Jul 27 06:30:29 +0545 2026 on pts/0

[sale-user1@labserver ~]$ cd /marketing	                 # others don't have execute permissions on marketing
-bash: cd: /marketing: Permission denied			

[sale-user1@labserver ~]$ cd /production		 # others have execute permissions on production
[sale-user1@labserver production]$ pwd
/production

[sale-user1@labserver production]$ cd -
/home/sale-user1

[sale-user1@labserver ~]$ exit
logout
[root@labserver ~]# chmod o-x /production/
[root@labserver ~]# ls -ld /production
drwxrws--T. 3 boss production 39 Jul 26 08:19 /production

[root@labserver ~]# su - sale-user1
Last login: Mon Jul 27 06:45:15 +0545 2026 on pts/0

[sale-user1@labserver ~]$ cd /sales/

[sale-user1@labserver sales]$ ls
dir1  dir2  dir3  newdir1  newsales-2026  report2  sales-report-2026

[sale-user1@labserver sales]$ ls -l 
total 4
drwxr-sr-x. 2 sale-user1 sales  6 Jul 26 08:48 dir1
drwxr-sr-x. 2 sale-user1 sales  6 Jul 26 16:17 dir2
drwxr-sr-x. 2 sale-user2 sales  6 Jul 26 17:39 dir3
drwxrws---. 2 sale-user1 sales  6 Jul 26 19:57 newdir1
-rw-rw----. 1 sale-user1 sales 92 Jul 26 19:42 newsales-2026
-rw-r--r--. 1 sale-user2 sales  0 Jul 26 17:39 report2
-rw-r--r--. 1 sale-user1 sales  0 Jul 26 17:12 sales-report-2026

[sale-user1@labserver sales]$ rm sales-report-2026 

[sale-user1@labserver sales]$ rmdir dir1 dir2
[sale-user1@labserver sales]$ ls -l
total 4
drwxr-sr-x. 2 sale-user2 sales  6 Jul 26 17:39 dir3
drwxrws---. 2 sale-user1 sales  6 Jul 26 19:57 newdir1
-rw-rw----. 1 sale-user1 sales 92 Jul 26 19:42 newsales-2026
-rw-r--r--. 1 sale-user2 sales  0 Jul 26 17:39 report2

[sale-user1@labserver ~]$ exit
logout
[root@labserver ~]# 

[root@labserver ~]# su - sale-user2
Last login: Sun Jul 26 19:38:55 +0545 2026 on pts/0
[sale-user2@labserver ~]$ 

[sale-user2@labserver ~]$ whoami
sale-user2
[sale-user2@labserver ~]$ pwd
/home/sale-user2

[sale-user2@labserver ~]$ cd /sales
[sale-user2@labserver sales]$ ls -ld
drwxrws--T. 4 boss sales 69 Jul 27 08:11 .

[sale-user2@labserver sales]$ ls -l
total 4
drwxr-sr-x. 2 sale-user2 sales  6 Jul 26 17:39 dir3
drwxrws---. 2 sale-user1 sales 24 Jul 27 08:13 newdir1
-rw-rw----. 1 sale-user1 sales 92 Jul 26 19:42 newsales-2026
-rw-r--r--. 1 sale-user2 sales  0 Jul 26 17:39 report2

[sale-user2@labserver sales]$ cd newdir1/
[sale-user2@labserver newdir1]$ ls -l 
total 4
-rw-rw----. 1 sale-user1 sales 54 Jul 27 08:13 newreport1

[sale-user2@labserver newdir1]$ cat newreport1 
this is new sales report 1 added by the sale-user1...

[sale-user2@labserver newdir1]$ vi newreport1 
[sale-user2@labserver newdir1]$ cat newreport1 
this is new sales report 1 added by the sale-user1...
this is new sales report 2 added by the sale-user2...

 # Sticky bit is not set on /newdir1 so deletion of newreport1 created by previous user is possible
 
[sale-user2@labserver newdir1]$ rm newreport1         
[sale-user2@labserver newdir1]$ ls -l
total 0
[sale-user2@labserver newdir1]$ 

[sale-user2@labserver newdir1]$ su - root
Password: 
Last login: Mon Jul 27 08:15:23 +0545 2026 on pts/0

[root@labserver ~]# cd /sales
[root@labserver sales]# ls -l
total 4
drwxr-sr-x. 2 sale-user2 sales  6 Jul 26 17:39 dir3
drwxrws---. 2 sale-user1 sales  6 Jul 27 08:18 newdir1
-rw-rw----. 1 sale-user1 sales 92 Jul 26 19:42 newsales-2026
-rw-r--r--. 1 sale-user2 sales  0 Jul 26 17:39 report2

[root@labserver sales]# ls -ld
drwxrws--T. 4 boss sales 69 Jul 27 08:11 .

[root@labserver sales]# chmod +t newdir1
[root@labserver sales]# ls -l
total 4
drwxr-sr-x. 2 sale-user2 sales  6 Jul 26 17:39 dir3
drwxrws--T. 2 sale-user1 sales  6 Jul 27 08:18 newdir1
-rw-rw----. 1 sale-user1 sales 92 Jul 26 19:42 newsales-2026
-rw-r--r--. 1 sale-user2 sales  0 Jul 26 17:39 report2

[root@labserver sales]# exit
logout

[sale-user2@labserver newdir1]$ ls -ld
drwxrws--T. 2 sale-user1 sales 6 Jul 27 08:18 .

   # A file in a sticky directory can be deleted only by:
      # the file owner
      # the directory owner
      # root

[sale-user2@labserver newdir1]$ ls

[sale-user2@labserver newdir1]$ vi file2

[sale-user2@labserver newdir1]$ ls -l
total 4
-rw-rw----. 1 sale-user2 sales 42 Jul 27 08:57 file2
[sale-user2@labserver newdir1]$ cat file2 
this is sales data added by sale-user2...

[sale-user2@labserver newdir1]$ exit
logout 
[root@labserver ~]# 

[root@labserver ~]# su - sale-user1
Last login: Mon Jul 27 08:14:53 +0545 2026 on pts/0
[sale-user1@labserver ~]$ 

[sale-user1@labserver ~]$ cd /sales
[sale-user1@labserver sales]$ 

[sale-user1@labserver sales]$ cd newdir1/

[sale-user1@labserver newdir1]$ ls -ld
drwxrws--T. 2 sale-user1 sales 19 Jul 27 08:57 .

[sale-user1@labserver newdir1]$ ls -l
total 4
-rw-rw----. 1 sale-user2 sales 42 Jul 27 08:57 file2

[sale-user1@labserver newdir1]$ vi file2 
[sale-user1@labserver newdir1]$ cat file2 
this is sales data added by sale-user2...
this is sales data added by sale-user1...

[sale-user1@labserver newdir1]$ rm file2           # sale-user2 created file2 but removed by sale-user1 is possible because newdir1 is owned by sale-user1
[sale-user1@labserver newdir1]$ ls
[sale-user1@labserver newdir1]$ 

[sale-user1@labserver newdir1]$ ls -ld
drwxrws--T. 2 sale-user1 sales 6 Jul 27 08:59 .

[sale-user1@labserver newdir1]$ vi file1
[sale-user1@labserver newdir1]$ ls -l
total 4
-rw-rw----. 1 sale-user1 sales 42 Jul 27 09:08 file1
[sale-user1@labserver newdir1]$ cat file1 
this sale data is added by the sale-user1

[sale-user1@labserver newdir1]$ exit
logout
[root@labserver ~]# 
[root@labserver ~]# su - sale-user2
Last login: Mon Jul 27 08:15:40 +0545 2026 on pts/0

[sale-user2@labserver ~]$ cd /sales/newdir1/
[sale-user2@labserver newdir1]$ ls -ld
drwxrws--T. 2 sale-user1 sales 19 Jul 27 09:08 .

[sale-user2@labserver newdir1]$ vi file1 
[sale-user2@labserver newdir1]$ cat file1 
this sale data is added by the sale-user1
this sales data is added by the sale-user2

[sale-user2@labserver newdir1]$ ls -l
total 4
-rw-rw----. 1 sale-user1 sales 85 Jul 27 09:09 file1
[sale-user2@labserver newdir1]$ ls -ld
drwxrws--T. 2 sale-user1 sales 19 Jul 27 09:09 .

 # file1 can be modified by sale-user2 as it is the member of sales but it can't be deleted by sale-user2 because it is under the directory owned by sale-user1
 
[sale-user2@labserver newdir1]$ rm file1 	                 
rm: cannot remove 'file1': Operation not permitted

[sale-user2@labserver newdir1]$ cd
[sale-user2@labserver ~]$ 
[sale-user2@labserver ~]$ exit
logout
[root@labserver ~]#

 # SUID (Set User ID) 
   # Octal Value: 4
   # Executes a file using the privileges of the file owner, not the user running it.
   # If SUID bit is set on an executable file/program file then it executes under the security context of owner rather than the user.
   # Use Case: The passwd command, which allows users to update their password by writing to root-restricted files.
   
[root@labserver ~]# useradd raman

[root@labserver ~]# passwd raman
New password: 
Retype new password: 
passwd: password updated successfully
[root@labserver ~]# 

 [root@labserver ~]# which passwd
/usr/bin/passwd

[root@labserver ~]# echo $PATH
/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
 
 # < passwd raman >  is equivalent to  < /usr/bin/passwd >
[root@labserver ~]#

[root@labserver ~]# ls -l /usr/bin/passwd 
-rwsr-xr-x. 1 root root 91424 Feb 23 05:45 /usr/bin/passwd

 # /usr/bin/passwd has user owner: root and group owner: root and others has execute permission.
 
[root@labserver ~]# ls -l /etc/shadow
----------. 1 root root 1554 Jul 27 09:46 /etc/shadow

 # root is SUPER user so can see modify any files eg. /etc/shadow

[root@labserver ~]# su - raman

[raman@labserver ~]$ whoami
raman

[raman@labserver ~]$ passwd 
Current password: 
New password: 
Retype new password: 
passwd: password updated successfully

 # by observing --> ----------. 1 root root 1554 Jul 27 09:46 /etc/shadow, raman should not be allowed to change the password
 
[raman@labserver ~]$ ls -l /usr/bin/passwd 
-rwsr-xr-x. 1 root root 91424 Feb 23 05:45 /usr/bin/passwd
[raman@labserver ~]$ 
 
 # it is possible because in /user/bin/passwd file, there is SUID bit (s)
 
   #    -     --s     ---     ---
   
   #    -       s  -->  represents SUID bit, so although user raman runs the program /usr/bin/passwd it runs under the security context of root
   #		s  --> SUID + execute
	
 # find all the files with SUID bit
 
[root@labserver ~]# find / -perm /4000

[root@labserver ~]# find / -perm -4000 -type f

[root@labserver ~]# find / -perm /4000 -type f

[root@labserver ~]# find / -perm /4000 | wc -l

[root@labserver ~]# find / -perm  /4000 -exec ls -ld {} \;       # files with SUID bits only
...

[root@labserver ~]# find / -perm /2000 -exec ls -ld {} \;        # files with SGID bits only
...

[root@labserver ~]# find / -perm /2000 -and -user sale-user1 2>/dev/null 
/sales/newdir1
/backup/sales/newdir1
/backup/newdir1
[root@labserver ~]# 
 
 # Copy all files and directories with the SGID bit to /backup
 
[root@labserver ~]# find / -perm /2000 -exec cp -pr {} /backup \; 2>/dev/null 

[root@labserver ~]# cd /backup/

[root@labserver backup]# ls -l
total 40
drwxr-s---+ 2 root       systemd-journal    28 Jul 27 06:23 6051512a1e194d8fa01bc1948bcb0427
drwxr-sr-x+ 3 root       systemd-journal    24 Jul 27 12:56 backup
drwxr-sr-x+ 2 sale-user2 sales               6 Jul 26 17:39 dir3
drwxrwsr-x+ 2 tss        tss                 6 Jul 27 06:23 eventlog
drwxr-sr-x+ 3 root       systemd-journal    46 Jul 27 06:23 journal
drwxrwsr-x+ 2 tss        tss                 6 Jul 13 14:32 keystore
drwxrws--T+ 2 boss       marketing           6 Jul 25 21:44 marketing
drwxrws--T+ 2 sale-user1 sales              19 Jul 27 09:09 newdir1
drwxrws--T+ 3 boss       production         39 Jul 26 08:19 production
drwxrws--T+ 4 boss       sales              69 Jul 27 08:11 sales
-rwx--s--x+ 1 root       utmp            15976 Oct 29  2024 utempter
-rwxr-sr-x+ 1 root       tty             24152 Mar  4 05:45 write
[root@labserver backup]# 

      # NOTE:

	# SGID = Group inheritance (new files/directories inherit the parent directory's group)
	# g+w  = Group collaboration (group members can modify shared files)
	# umask 0002 = Future files are created with group write permission
	# Sticky bit = Prevents users from deleting or renaming files owned by other users

	# Shared production directory:
	# chmod 3770 <directory-name>     or    chmod 2770 <directory-name>    as required

	# 3 --> SGID + Sticky bit
	# 7 --> Owner: read/write/execute
	# 7 --> Group: read/write/execute
	# 0 --> Others: no access

	# SGID (2)     --> Group inheritance
	# Sticky (1)   --> Only file owner, directory owner, or root can delete/rename files


 # ACL ( Access Control List )

   # Extends standard Linux permissions by allowing fine-grained access control for specific users and groups on files and directories.
   
[root@labserver ~]# yum whatprovides setfacl
...
acl-2.3.2-3.el10.x86_64 : Access control list utilities
...
[root@labserver ~]# rpm -q acl
package acl is not installed
[root@labserver ~]# yum install -y acl
...
Installed:
  acl-2.3.2-4.el10.x86_64                                                                                                                                                                    
Complete!
[root@labserver ~]# 

[raman@labserver ~]$ whoami
raman
[raman@labserver ~]$ pwd
/home/raman
[raman@labserver ~]$ ls -ld
drwxrwx--- raman raman 97 Jul 27 11:57 .

[raman@labserver ~]$ setfacl -m u:sale-user1:rwx /home/raman

[raman@labserver ~]$ setfacl -m u:sale-user2:rx /home/raman
 
[raman@labserver ~]$ setfacl -m u:market-user1:x /home/raman

[raman@labserver ~]$ setfacl -m g:production:rx /home/raman
 
[raman@labserver ~]$ ls -ld /home/raman/
drwxrwx---+ 2 raman raman 97 Jul 27 11:57 /home/raman/

[raman@labserver ~]$ getfacl /home/raman
getfacl: Removing leading '/' from absolute path names
# file: home/raman
# owner: raman
# group: raman
user::rwx
user:market-user1:--x
user:sale-user1:rwx
user:sale-user2:r-x
group::---
group:production:r-x
mask::rwx
other::---

[raman@labserver ~]$ exit
logout
[root@labserver ~]# su - sale-user1
Last login: Mon Jul 27 08:58:15 +0545 2026 on pts/0

[sale-user1@labserver ~]$ whoami
sale-user1

[sale-user1@labserver ~]$ ls -ld /home/raman
drwxrwx---+ 2 raman raman 97 Jul 27 11:57 /home/raman

[sale-user1@labserver ~]$ cd /home/raman/
[sale-user1@labserver raman]$ pwd
/home/raman

[sale-user1@labserver raman]$ ls
myfile
[sale-user1@labserver raman]$ rm myfile 
rm: remove write-protected regular file 'myfile'? y
[sale-user1@labserver raman]$ ls
[sale-user1@labserver raman]$ 

[sale-user1@labserver raman]$ vi newfile1
[sale-user1@labserver raman]$ cat newfile1
this is new file created by sale-user1

[sale-user1@labserver raman]$ ls -l newfile1 
-rw-rw----. 1 sale-user1 sale-user1 39 Jul 27 14:33 newfile1
[sale-user1@labserver raman]$ 

[sale-user1@labserver raman]$ exit
logout
[root@labserver ~]# su - raman
Last login: Mon Jul 27 14:10:45 +0545 2026 on pts/0

[raman@labserver ~]$ ls -l
total 4
-rw-rw----. 1 sale-user1 sale-user1 39 Jul 27 14:33 newfile1

[raman@labserver ~]$ cat newfile1 
cat: newfile1: Permission denied
[raman@labserver ~]$
```
 > File ownership in Linux:
 >> New files are owned by the user (process) that creates them, not by the owner of the directory.
    A directory's owner does not automatically gain ownership or access to files created by others.
    Ownership or access can only be changed explicitly (e.g., with chown, shared groups with setgid, or ACLs).
```
 # To remove ACL

[raman@labserver ~]$ setfacl -x u:sale-user1 /home/raman

[raman@labserver ~]$ getfacl /home/raman/ | grep sale-user1

[raman@labserver ~]$ getfacl /home/raman

[raman@labserver ~]$ setfacl -m d:u:sale-user1:rwx /home/raman    # -d --> Default ACL (applies to newly created files and subdirectories)

 # Granting Administrative Privileges to a Normal User
 
[raman@labserver ~]$ whoami
raman

[raman@labserver ~]$ useradd testuser
useradd: Permission denied.
useradd: cannot lock /etc/passwd; try again later.

[raman@labserver ~]$ sudo useradd testuser

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

For security reasons, the password you type will not be visible.

[sudo] password for raman: 
raman is not in the sudoers file.

[raman@labserver ~]$ exit
logout

[root@labserver ~]# cat /etc/sudoers

 # visudo --> edit of /etc/sudoers file with syntax checking capability in-built
 
[root@labserver ~]# visudo /etc/sudoers
/etc/sudoers:122:44: Cmnd_Alias "AL" referenced but not defined
[root@labserver ~]# visudo /etc/sudoers
[root@labserver ~]# 
[root@labserver ~]# grep raman /etc/sudoers
raman        ALL=(ALL)           ALL        
[root@labserver ~]# 

 # add raman in wheel group or
 # add this line in /etc/sudoers: raman        ALL=(ALL)           ALL 

[root@labserver ~]# grep wheel /etc/group
wheel:x:10:aadarsha

[root@labserver ~]# su - raman
Last login: Mon Jul 27 14:43:04 +0545 2026 on pts/0
[raman@labserver ~]$ 

[raman@labserver ~]$ useradd testuser
useradd: Permission denied.
useradd: cannot lock /etc/passwd; try again later.
[raman@labserver ~]$ 

[raman@labserver ~]$ sudo useradd testuser
[sudo] password for raman: 
[raman@labserver ~]$ 

[raman@labserver ~]$ passwd testuser
passwd: You may not view or modify password information for testuser.

[raman@labserver ~]$ sudo passwd testuser
[sudo] password for raman: 
New password: 
Retype new password: 
passwd: password updated successfully

[raman@labserver ~]$ su - testuser
Password: 
Last login: Mon Jul 27 16:35:30 +0545 2026 on pts/0

[testuser@labserver ~]$ whoami
testuser
[testuser@labserver ~]$ 

[testuser@labserver ~]$ cat /etc/group | grep wheel
wheel:x:10:aadarsha

[testuser@labserver ~]$ grep wheel /etc/group
wheel:x:10:aadarsha

[testuser@labserver ~]$ grep testuser /etc/group
testuser:x:1014:

 # add testuser in wheel group to give sudo privileges

[testuser@labserver ~]$ sudo useradd newtest
..
[sudo] password for testuser: 
testuser is not in the sudoers file.
[testuser@labserver ~]$ 

[testuser@labserver ~]$ grep wheel /etc/group
wheel:x:10:aadarsha

[testuser@labserver ~]$ grep testuser /etc/group
testuser:x:1014:

[testuser@labserver ~]$ exit
logout

[root@labserver ~]# usermod -aG wheel testuser

[root@labserver ~]# grep wheel /etc/group
wheel:x:10:aadarsha,testuser

[root@labserver ~]# grep testuser /etc/group
wheel:x:10:aadarsha,testuser
testuser:x:1014:

[root@labserver ~]# su - testuser
Last login: Tue Jul 28 09:59:16 +0545 2026 on pts/0
 
[testuser@labserver ~]$ sudo useradd newuser
[sudo] password for testuser: 

[testuser@labserver ~]$ grep newuser /etc/passwd
newuser:x:1012:1015::/home/newuser:/bin/bash

[testuser@labserver ~]$ sudo userdel -r newuser

[testuser@labserver ~]$ grep newuser /etc/passwd
[testuser@labserver ~]$ 

[testuser@labserver ~]$ sudo systemctl is-active sshd
active

[testuser@labserver ~]$ sudo systemctl is-enabled sshd
enabled

 # Provide production group sudo previliges 

[root@labserver ~]# visudo 

 # added lines in (/etc/:
  
  User_Alias   TRUSTED=sale-user1,prod-user1,prod-user2
  TRUSTED      ALL=(ALL)           NOPASSWD:ALL
 %production  ALL=(ALL)           ALL
  
[root@labserver ~]# su - prod-user1
Last login: Sun Jul 26 08:18:35 +0545 2026 on pts/0

[prod-user1@labserver ~]$ sudo cat /etc/sudoers
...
raman        ALL=(ALL)           ALL        

User_Alias   TRUSTED=sale-user1,prod-user1,prod-user2
TRUSTED      ALL=(ALL)           NOPASSWD:ALL
%production  ALL=(ALL)           ALL
...
[prod-user1@labserver ~]$

[prod-user1@labserver ~]$ sudo systemctl is-active sshd

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

For security reasons, the password you type will not be visible.

[sudo] password for prod-user1: 
active
[prod-user1@labserver ~]$ 

[prod-user1@labserver ~]$ sudo systemctl is-active sshd
active

 # When a command is executed using sudo, it runs with the privileges of the target user (by default, root). 
 # As a result, any files created by that command are owned by root unless the command explicitly preserves or changes ownership.

[prod-user1@labserver ~]$ ls -l
total 4
-rw-r--r--. 1 prod-user1 prod-user1 19 Jul 24 14:25 myfile

[prod-user1@labserver ~]$ sudo vi newfile1
[sudo] password for prod-user1: 
[prod-user1@labserver ~]$ ls -l
total 8
-rw-r--r--. 1 prod-user1 prod-user1 19 Jul 24 14:25 myfile
-rw-r-----. 1 root       root       20 Jul 28 11:14 newfile1               # --> using sudo newly created file has owner user: root   and  group: root
[prod-user1@labserver ~]$ 

[prod-user1@labserver ~]$ sudo cat newfile1 
[sudo] password for prod-user1: 
this is newfile1...
[prod-user1@labserver ~]$ 

[prod-user1@labserver ~]$ vi mynewfile

[prod-user1@labserver ~]$ cat mynewfile 
this is the file created by prod-user1

[prod-user1@labserver ~]$ cat newfile1 
cat: newfile1: Permission denied
[prod-user1@labserver ~]$ 

[prod-user1@labserver ~]$ cat myfile 
this is my file...

[prod-user1@labserver ~]$ ls -l
total 12
-rw-r--r--. 1 prod-user1 prod-user1 19 Jul 24 14:25 myfile
-rw-rw----. 1 prod-user1 prod-user1 39 Jul 28 11:21 mynewfile
-rw-r-----. 1 root       root       20 Jul 28 11:14 newfile1
[prod-user1@labserver ~]$ 

 # The shell umask configured in /etc/bashrc applies only to interactive Bash sessions and does not affect commands executed through sudo. 
 # sudo executes privileged commands with its own effective umask (022 by default when not overridden by sudoers configuration), resulting in different default file permissions.
 
[prod-user1@labserver ~]$ umask
0007

[prod-user1@labserver ~]$ exit
logout

[root@labserver ~]# umask
0007

[prod-user1@labserver ~]$ touch testfile1

[prod-user1@labserver ~]$ sudo touch testfile2
[sudo] password for prod-user1: 
[prod-user1@labserver ~]$ 

[prod-user1@labserver ~]$ ls -l
total 12
-rw-r--r--. 1 prod-user1 prod-user1 19 Jul 24 14:25 myfile
-rw-rw----. 1 prod-user1 prod-user1 39 Jul 28 11:21 mynewfile
-rw-r-----. 1 root       root       20 Jul 28 11:14 newfile1
-rw-rw----. 1 prod-user1 prod-user1  0 Jul 28 11:30 testfile1
-rw-r-----. 1 root       root        0 Jul 28 11:31 testfile2
[prod-user1@labserver ~]$ 
 
[prod-user1@labserver ~]$ sudo sudo -V | grep -i umask
Umask to use or 0777 to use user's: 022
[prod-user1@labserver ~]$ 

[prod-user1@labserver ~]$ sudo sudo -V | grep -i umask
Umask to use or 0777 to use user's: 022
[prod-user1@labserver ~]$ 
[prod-user1@labserver ~]$ sudo cat /etc/sudoers | grep umask
[prod-user1@labserver ~]$ 
[prod-user1@labserver ~]$ sudo cat /etc/sudoers.d/ | grep umask
cat: /etc/sudoers.d/: Is a directory
[prod-user1@labserver ~]$ 
 
[prod-user1@labserver ~]$ sudo grep -i include /etc/sudoers
#includedir /etc/sudoers.d

[prod-user1@labserver ~]$ sudo sudo -V | grep -i umask
Umask to use or 0777 to use user's: 022
[prod-user1@labserver ~]$ 

  # giving permission to use only specific commands
  
[root@labserver ~]# which systemctl 
/usr/bin/systemctl
 
[root@labserver ~]# which usermod
/usr/sbin/usermod

[root@labserver ~]# visudo
[root@labserver ~]# cat /etc/sudoers
...
raman        ALL=(ALL)       /usr/bin/systemctl,/usr/sbin/usermod
...
[root@labserver ~]# 

[root@labserver ~]# su - raman
Last login: Tue Jul 28 09:41:44 +0545 2026 on pts/0
[raman@labserver ~]$ 

[raman@labserver ~]$ sudo systemctl is-active sshd
active

[raman@labserver ~]$ sudo systemctl is-enabled sshd
enabled

[raman@labserver ~]$ sudo useradd thistestuser
Sorry, user raman is not allowed to execute '/sbin/useradd thistestuser' as root on labserver.

 # But can modify existing user

[raman@labserver ~]$ sudo cat /etc/passwd
Sorry, user raman is not allowed to execute '/bin/cat /etc/passwd' as root on labserver.

[raman@labserver ~]$ cat /etc/passwd | grep testuser
testuser:x:1011:1014::/home/testuser:/bin/bash

[raman@labserver ~]$ sudo usermod -u 2022 -s /bin/csh testuser
usermod: Warning: missing or non-executable shell '/bin/csh'

[raman@labserver ~]$ cat /etc/passwd | grep testuser
testuser:x:2022:1014::/home/testuser:/bin/csh
[raman@labserver ~]$ 

[raman@labserver ~]$ sudo -i		   #  start an interactive login shell as the root user 
Sorry, user raman is not allowed to execute '/bin/bash' as root on labserver.
[raman@labserver ~]$

[raman@labserver ~]$ sudo -l
Matching Defaults entries for raman on labserver:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR
    USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User raman may run the following commands on labserver:
    (ALL) /usr/bin/systemctl, /usr/sbin/usermod
[raman@labserver ~]$ 

 # User Security 
 
 # Tracking Admin Activities

[root@labserver ~]# su - testuser
Last login: Tue Jul 28 13:48:03 +0545 2026 on pts/0
[testuser@labserver ~]$ 

[testuser@labserver ~]$ sudo useradd u1

[testuser@labserver ~]$ sudo userdel -r u1

[testuser@labserver ~]$ exit
logout

[root@labserver ~]# ls /var/log/secure 
/var/log/secure

[root@labserver ~]# tail /var/log/secure 
Jul 28 13:49:38 labserver useradd[4277]: new group: name=u1, GID=2023
Jul 28 13:49:38 labserver useradd[4277]: new user: name=u1, UID=2023, GID=2023, home=/home/u1, shell=/bin/bash, from=/dev/pts/1
Jul 28 13:49:38 labserver sudo[4274]: pam_unix(sudo:session): session closed for user root
Jul 28 13:49:57 labserver sudo[4280]: testuser : TTY=pts/0 ; PWD=/home/testuser ; USER=root ; COMMAND=/sbin/userdel -r u1
Jul 28 13:49:57 labserver sudo[4280]: pam_unix(sudo:session): session opened for user root(uid=0) by aadarsha(uid=2022)
Jul 28 13:49:57 labserver userdel[4283]: delete user 'u1'
Jul 28 13:49:57 labserver userdel[4283]: removed group 'u1' owned by 'u1'
Jul 28 13:49:57 labserver userdel[4283]: removed shadow group 'u1' owned by 'u1'
Jul 28 13:49:58 labserver sudo[4280]: pam_unix(sudo:session): session closed for user root
Jul 28 13:50:25 labserver su[4234]: pam_unix(su-l:session): session closed for user testuser
[root@labserver ~]# 

[root@labserver ~]# visudo

[root@labserver ~]# cat /etc/sudoers
...
Defaults logfile = "/var/log/useractivity.log"                 # <-- this line is added in /etc/sudoers file
...
[root@labserver ~]#

[root@labserver ~]# su - testuser
Last login: Tue Jul 28 13:48:31 +0545 2026 on pts/0

[testuser@labserver ~]$ sudo useradd testu1

[testuser@labserver ~]$ sudo userdel -r testu1

[testuser@labserver ~]$ exit
logout

[root@labserver ~]# su - sale-user1
Last login: Mon Jul 27 14:28:53 +0545 2026 on pts/0

[sale-user1@labserver ~]$ sudo cat /etc/passwd | grep testuser
...
[sudo] password for sale-user1: 
testuser:x:2022:1014::/home/testuser:/bin/bash

[sale-user1@labserver ~]$ exit
logout

[root@labserver ~]# tail /var/log/useractivity.log 
Jul 28 13:56:59 : root : TTY=pts/0 ; PWD=/root ; USER=root ;
    COMMAND=/sbin/useradd testu1
Jul 28 13:57:38 : testuser : TTY=pts/0 ; PWD=/home/testuser ; USER=root ;
    COMMAND=/sbin/useradd testu1
Jul 28 13:57:55 : testuser : TTY=pts/0 ; PWD=/home/testuser ; USER=root ;
    COMMAND=/sbin/userdel -r testu1
Jul 28 13:58:54 : sale-user1 : TTY=pts/0 ; PWD=/home/sale-user1 ; USER=root ;
    COMMAND=/bin/cat /etc/passwd
[root@labserver ~]# 

[root@labserver ~]# su - sale-user2
Last login: Mon Jul 27 09:09:01 +0545 2026 on pts/0

[sale-user2@labserver ~]$ sudo userdel - testuser
...
[sudo] password for sale-user2: 
sale-user2 is not in the sudoers file.

[sale-user2@labserver ~]$ sudo useradd testu2
[sudo] password for sale-user2: 
sale-user2 is not in the sudoers file.

[sale-user2@labserver ~]$ exit
logout

[root@labserver ~]# tail /var/log/useractivity.log 
Jul 28 13:57:38 : testuser : TTY=pts/0 ; PWD=/home/testuser ; USER=root ;
    COMMAND=/sbin/useradd testu1
Jul 28 13:57:55 : testuser : TTY=pts/0 ; PWD=/home/testuser ; USER=root ;
    COMMAND=/sbin/userdel -r testu1
Jul 28 13:58:54 : sale-user1 : TTY=pts/0 ; PWD=/home/sale-user1 ; USER=root ;
    COMMAND=/bin/cat /etc/passwd
Jul 28 14:03:06 : sale-user2 : user NOT in sudoers ; TTY=pts/0 ;
    PWD=/home/sale-user2 ; USER=root ; COMMAND=/sbin/userdel - testuser
Jul 28 14:04:06 : sale-user2 : user NOT in sudoers ; TTY=pts/0 ;
    PWD=/home/sale-user2 ; USER=root ; COMMAND=/sbin/useradd testu2
[root@labserver ~]# 

   # Enforce custom strict password policy
   
   # A negative value means "minimum required count.
   
   # minlen  = n          --> minimum password length of n characters
   # dcredit = -n         --> require at least n digit
   # ucredit = -n         --> require at least n uppercase letter
   # lcredit = -n         --> require at least n lowercase letter
   # ocredit = -n         --> require at least n special/other character
   # maxrepeat = n        --> allow a maximum of n consecutive identical characters
   # difok = n            --> require at least n characters different from the previous password
   # enforce_for_root     --> apply the policy to the root user as well
   
[root@labserver ~]# vi /etc/security/pwquality.conf
 
[root@labserver ~]# cat /etc/security/pwquality.conf
...
# added password policy
 minlen = 10					     			
 dcredit = -1
 ucredit = -1
 lcredit = -1
 ocredit = -1
 maxrepeat = 2
 difok = 4
 enforce_for_root
[root@labserver ~]# 

[root@labserver ~]# echo "Nepal-123" | passwd --stdin testuser
BAD PASSWORD: The password is shorter than 10 characters
passwd: (user testuser) pam_chauthtok() failed, error:
Authentication token manipulation error

[root@labserver ~]# echo "Nepal-12345" | passwd --stdin testuser

[root@labserver ~]# echo "Nepal-1234555" | passwd --stdin testuser
BAD PASSWORD: The password contains more than 2 same characters consecutively
passwd: (user testuser) pam_chauthtok() failed, error:
Authentication token manipulation error

[root@labserver ~]# echo "Nepal-12345" | passwd --stdin testuser

 
  # Lock the account after a certain number of failed authentication attempts

   # deny = 3               --> lock the account after 3 consecutive failed login attempts
   # unlock_time = 900      --> automatically unlock the account after 900 seconds (15 minutes)
   # fail_interval = 600    --> count failed login attempts within a 600-second (10-minute) window
   # even_deny_root         --> apply the account lockout policy to the root user as well


[root@labserver ~]# su - raman
Last login: Tue Jul 28 15:44:52 +0545 2026 on pts/0

[raman@labserver ~]$ su - testuser
Password: 
su: Authentication failure
[raman@labserver ~]$ su - testuser
Password: 
su: Authentication failure
[raman@labserver ~]$ su - testuser
Password: 
su: Authentication failure

[raman@labserver ~]$ su - testuser
Password: 
Last login: Tue Jul 28 13:57:27 +0545 2026 on pts/0
Last failed login: Tue Jul 28 15:49:16 +0545 2026 on pts/0
There were 7 failed login attempts since the last successful login.

[testuser@labserver ~]$ vi /etc/security/faillock.conf 
...
# added lines

deny = 3
unlock_time = 900
fail_interval = 600
even_deny_root
...
[root@labserver ~]#

 # PAM (Pluggable Authentication Modules)

 # pam_faillock.so is used to protect the system from brute-force attacks by tracking failed login attempts and
   temporarily locking user accounts after a specified number of consecutive authentication failures.
   
 # add the below PAM in /etc/pam.d/system-auth  and  /etc/pam.d/password-auth 

   auth required pam_faillock.so preauth silent audit deny=3 unlock_time=900
   auth required pam_faillock.so authfail audit deny=3 unlock_time=900
   account required pam_faillock.so

[root@labserver ~]# vi /etc/pam.d/system-auth 

[root@labserver ~]# cat /etc/pam.d/system-auth
# Generated by authselect
# Do not modify this file manually, use authselect instead. Any user changes will be overwritten.
# You can stop authselect from managing your configuration by calling 'authselect opt-out'.
# See authselect(8) for more details.

# added PAM
auth required pam_faillock.so preauth silent audit deny=3 unlock_time=900
auth required pam_faillock.so authfail audit deny=3 unlock_time=900
account required pam_faillock.so

auth        required                                     pam_env.so
auth        required                                     pam_faildelay.so delay=2000000
auth        sufficient                                   pam_unix.so nullok
auth        required                                     pam_deny.so

account     required                                     pam_unix.so

password    requisite                                    pam_pwquality.so
password    sufficient                                   pam_unix.so yescrypt shadow nullok use_authtok
password    required                                     pam_deny.so

session     optional                                     pam_keyinit.so revoke
session     required                                     pam_limits.so
-session    optional                                     pam_systemd.so
session     [success=1 default=ignore]                   pam_succeed_if.so service in crond quiet use_uid
session     required                                     pam_unix.so
[root@labserver ~]# 

[root@labserver ~]# vi /etc/pam.d/password-auth 
[root@labserver ~]# cat /etc/pam.d/password-auth
...
# See authselect(8) for more details.

# added PAM
auth required pam_faillock.so preauth silent audit deny=3 unlock_time=900
auth required pam_faillock.so authfail audit deny=3 unlock_time=900
account required pam_faillock.so

auth        required                                     pam_env.so
...
[root@labserver ~]# 

  # testing

[root@labserver ~]# su - raman
Last login: Wed Jul 29 05:53:41 +0545 2026 on pts/0
[raman@labserver ~]$ 
[raman@labserver ~]$ su - testuser
Password: 
su: Authentication failure
[raman@labserver ~]$ su - testuser
Password: 
su: Authentication failure
[raman@labserver ~]$ su - testuser
Password: 
Last login: Wed Jul 29 11:30:13 +0545 2026 on pts/0
Last failed login: Wed Jul 29 11:31:22 +0545 2026 on pts/0
There were 2 failed login attempts since the last successful login.
[testuser@labserver ~]$ 

[testuser@labserver ~]$ exit
logout
[raman@labserver ~]$ 
[raman@labserver ~]$ su - testuser
Password: 
su: Authentication failure
[raman@labserver ~]$ su - testuser
Password: 
su: Authentication failure
[raman@labserver ~]$ su - testuser
Password: 
su: Authentication failure
[raman@labserver ~]$ 

 # wrong password is entered 3 times thus account is locked for 900s (15 min)
 
[raman@labserver ~]$ su - testuser
Password: 
su: Authentication failure                # --> correct password was entered but login failed, because account is locked.

[raman@labserver ~]$ exit
logout
[root@labserver ~]# 

 # Check failed login history and account lockout status

[root@labserver ~]# faillock --user raman
raman:
When                Type  Source                                           Valid
[root@labserver ~]# 

[root@labserver ~]# faillock --user testuser
testuser:
When                Type  Source                                           Valid
2026-07-29 11:31:41 TTY   /dev/pts/0                                           V
2026-07-29 11:32:01 TTY   /dev/pts/0                                           V
2026-07-29 11:32:08 TTY   /dev/pts/0                                           V
[root@labserver ~]# 

[root@labserver ~]# tail /var/log/secure
Jul 29 11:31:32 labserver su[2580]: pam_unix(su-l:session): session opened for user testuser(uid=2022) by aadarsha(uid=1010)
...
Jul 29 11:32:03 labserver su[2618]: pam_unix(su-l:auth): authentication failure; logname=aadarsha uid=1010 euid=0 tty=/dev/pts/0 ruser=raman rhost=  user=testuser
Jul 29 11:32:08 labserver su[2624]: pam_faillock(su-l:auth): Consecutive login failures for user testuser account temporarily locked					
                                                                                                 # --> Account temporarily locked after consecutive failed authentication attempts.
...
[root@labserver ~]# 

 # Reset the faillock counter immediately (instead of waiting for the unlock time)

[root@labserver ~]# faillock --user testuser --reset

[root@labserver ~]# su - raman
Last login: Wed Jul 29 11:29:59 +0545 2026 on pts/0

[raman@labserver ~]$ su - testuser
Password: 
Last login: Wed Jul 29 11:31:32 +0545 2026 on pts/0
Last failed login: Wed Jul 29 11:32:24 +0545 2026 on pts/0
There were 4 failed login attempts since the last successful login.
[testuser@labserver ~]$ 
[testuser@labserver ~]$ whoami
testuser
[testuser@labserver ~]$

 # Auto Disable Inactive Users 
 
 # Eg : If the users do not login for last 30 days, diable their accounts
 
 # and
 
 # Track User's Command 

 # 1. Enable Audit Logging

[root@labserver ~]# man auditctl

 [root@labserver ~]# auditctl -w /etc/passwd -p wa -k user_changes
Old style watch rules are slower

[root@labserver ~]# ausearch -k user_changes
----
time->Wed Jul 29 12:00:54 2026
...
type=CONFIG_CHANGE msg=audit(1785305754.715:181): auid=1000 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 op=add_rule key="user_changes" list=4 res=1
[root@labserver ~]# 

 # Configure the system to force users to logout automatically after certain idle time
 
[root@labserver ~]# vi /etc/profile

[root@labserver ~]# cat /etc/profile
# /etc/profile
...
# add timeout

TMOUT=300
[root@labserver ~]#

[root@labserver ~]# source /etc/profile

[root@labserver ~]# source /etc/profile

[root@labserver ~]# su - testuser
Last login: Wed Jul 29 11:44:04 +0545 2026 on pts/0
[testuser@labserver ~]$ 

[testuser@labserver ~]$ ps 
    PID TTY          TIME CMD
   3022 pts/0    00:00:00 bash
   3053 pts/0    00:00:00 ps

[testuser@labserver ~]$ whoami
testuser
[testuser@labserver ~]$ 
timed out waiting for input: auto-logout              # --> automatically logged out
timed out waiting for input: auto-logout
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ whoami
aadarsha
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$ 
```

<<
