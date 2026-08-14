---
title: "PL - 015 — Bash Shell Scripting & Automation"
date: 2026-07-17
draft: false
---

### Automation

**Automation** is the practice of using software, scripts, or tools to perform tasks automatically with minimal or no manual intervention.

The primary goal of automation is to reduce repetitive manual work, improve consistency, minimize human error, and make processes more efficient and repeatable.

> **Bash scripting is one of the simplest and most widely used approaches to automation in Linux and Unix-like environments.**

#### For example:
**Manual process:**
Check disk space --> create backup --> compress files --> upload backup --> remove old files.

**Automated process:**
A script or automation workflow performs all of these steps automatically according to predefined rules or a schedule.

### Key Benefits of Automation

- **Consistency** — The same process can be executed the same way every time.
- **Efficiency** — Reduces repetitive manual work.
- **Reliability** — Reduces errors caused by manual execution.
- **Scalability** — Allows processes to be executed across many systems.
- **Repeatability** — Processes can be reproduced whenever required.
- **Time Savings** — Engineers can focus on higher-value tasks.
- **Auditability** — Automated processes can be stored as code and tracked through version control.

### Common Automation Tasks

Automation is commonly applied to:

- **Server provisioning and configuration**
- **Application deployment**
- **Database and file backups**
- **Log rotation and cleanup**
- **Disk-space monitoring**
- **User and permission management**
- **Package installation and updates**
- **Health checks and service monitoring**
- **Testing and build processes**
- **File processing and batch operations**
- **Scheduled maintenance**
- **Infrastructure provisioning**
- **Incident detection and remediation**

### Automation Techniques

Automation can be implemented using different techniques depending on the type of task, environment, and level of complexity.

| Technique | Purpose | Common Tools |
|---|---|---|
| **Scripting** | Automate commands, system tasks, and repetitive operations | Bash, Python, PowerShell |
| **Task Scheduling** | Execute tasks automatically at predefined times or intervals | `cron`, systemd timers, Windows Task Scheduler |
| **Workflow Automation** | Automate a sequence of dependent tasks | Apache Airflow, GitHub Actions, Jenkins |
| **Event-Driven Automation** | Trigger actions when a specific event occurs | AWS Lambda, Azure Functions, webhooks |
| **Configuration Management** | Automatically configure and maintain servers and systems | Ansible, Puppet, Chef |
| **Infrastructure as Code (IaC)** | Provision and manage infrastructure using code | Terraform, OpenTofu, AWS CloudFormation |
| **CI/CD Automation** | Automate building, testing, and deploying software | GitHub Actions, GitLab CI/CD, Jenkins, Argo CD |
| **Container Automation** | Build, manage, and deploy containerized applications | Docker, Kubernetes, Helm |
| **Monitoring & Remediation** | Detect issues and automatically perform corrective actions | Prometheus, Grafana, Ansible, Kubernetes |
| **Backup Automation** | Automatically create, verify, rotate, and retain backups | Bash, `rsync`, Restic, Velero |
| **Log Automation** | Collect, process, rotate, and analyze logs | `logrotate`, Fluent Bit, Logstash |
| **Security Automation** | Automate security checks, patching, and incident responses | Ansible, OpenSCAP, SIEM/SOAR platforms |

### Bash Scripting

**Bash** (Bourne Again SHell) is a command-line shell and scripting language commonly used on Linux and other Unix-like systems.

**Bash scripting** is the practice of writing Bash commands, logic, and functions in a script file so that a sequence of tasks can be executed consistently and repeatedly.

Instead of manually executing multiple commands:

```
mkdir backup
cp app.log backup/
tar -czf backup.tar.gz backup/
```
the commands can be placed in a script:
```text
#!/usr/bin/env bash

mkdir -p backup
cp app.log backup/
tar -czf backup.tar.gz backup/
```
The script can then be executed whenever the task needs to be performed:
```bash
./backup.sh
```

Bash is particularly useful for Linux administration, server management, DevOps workflows, deployments, backups, monitoring, and other command-line automation tasks.

Basic of bash scripting is [here](https://github.com/erkdk/systems-journey/tree/main/02-Scripting-Automation/scripts)

### Terminal Session
```
[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ mkdir scripts

[aadarsha@labserver ~]$ pwd
/home/aadarsha

[aadarsha@labserver ~]$ vi scripts/test.sh

[aadarsha@labserver ~]$ cat scripts/test.sh 
#!/bin/bash
echo "Hello! World"

[aadarsha@labserver ~]$ ls -l scripts/test.sh 
-rw-r--r--. 1 aadarsha aadarsha 34 Aug  9 16:17 scripts/test.sh

[aadarsha@labserver ~]$ chmod +x scripts/test.sh 

[aadarsha@labserver ~]$ ls -l scripts/test.sh 
-rwxr-xr-x. 1 aadarsha aadarsha 34 Aug  9 16:17 scripts/test.sh

[aadarsha@labserver ~]$ su - root

[root@labserver ~]# vi /etc/profile

 # added line: export PATH="$PATH:/home/aadarsha/scripts"

[root@labserver ~]# source /etc/profile

[root@labserver ~]# cd /home/aadarsha/scripts/

[root@labserver scripts]# pwd
/home/aadarsha/scripts

[root@labserver scripts]# ls
test.sh
 
[root@labserver scripts]# test.sh 
Hello! World

[root@labserver scripts]# exit
logout

[aadarsha@labserver scripts]$ test.sh
-bash: test.sh: command not found

[aadarsha@labserver scripts]$ source /etc/profile

[aadarsha@labserver scripts]$ test.sh
Hello! World

[aadarsha@labserver ~]$ scripts/test.sh
Hello! World
 
[aadarsha@labserver ~]$ ./scripts/test.sh 
Hello! World

[aadarsha@labserver ~]$ which test.sh
~/scripts/test.sh

[aadarsha@labserver ~]$ bash scripts/test.sh 
Hello! World

[aadarsha@labserver ~]$ umask
0022

	# shell development history

	  # sh -- csh -- ksh -- tcsh -- bash

 # String Operators
    
   string1 [operator] string2
   
   	     =  -->  True if string1 and string2 are same
   	     != -->  True if string1 and string2 are different
   	     
       -z  string1  -->  True if string1 is empty (zero)
       -n  string1  -->  True if string1 is Not Empty (not zero)
   
   
 # Numeric Operators
   
   num1 [operator] num2
        
        -eq  -->  True if num1 =  num2
        -ne  -->  True if num1 != num2
        -gt  -->  True if num1 >  num2
        -ge  -->  True if num1 >= num2
        -lt  -->  True if num1 <  num2
        -le  -->  True if num <=  num2
        
 # File Operators
 
        -e <file-name>  -->  True if the file exists
        -f <file-name>  -->  True if the file is a normal file
        -d <file-name>  -->  True if it is a directory
        -l <file-name>  -->  True if it is a soft link
        -r <file-name>  -->  True if the file is readable
        -w <file-name>  -->  True if the file is writable
        -x <file-name>  -->  True if it is executable
        -s <file-name>  -->  True if the file size is greater than zero (non-empty)
        
        
 # Logical Operators
           
           -a  -->  Logical AND
           -o  -->  Logical OR
           !   -->  Logical NOT
           
 # Positional Parameters
 
    $0  --> name of the scripts
 
    $1  -  $9
    
    $*  -->  all parameters
    
    $#  -->  count of parameters
 
 # example1
 
[aadarsha@labserver scripts]$ vi positional-parameters.sh

[aadarsha@labserver scripts]$ cat positional-parameters.sh 
#!/bin/bash

# This script demonstrates the use of positional parameters in bash scripting.

echo "First Name: $1"
echo "Middle Name: $2"
echo "Last Name: $3"
echo "Full Name: $*"

echo "Total number of arguments: $#"

[aadarsha@labserver scripts]$ chmod +x positional-parameters.sh 

[aadarsha@labserver scripts]$ positional-parameters.sh 
First Name: 
Middle Name: 
Last Name: 
Full Name: 
Total number of arguments: 0
 
[aadarsha@labserver scripts]$ positional-parameters.sh Ram Bdr. Khadka
First Name: Ram
Middle Name: Bdr.
Last Name: Khadka
Full Name: Ram Bdr. Khadka
Total number of arguments: 3

 # example2

[aadarsha@labserver scripts]$ vi backup.sh

[aadarsha@labserver scripts]$ cat backup.sh 
#!/bin/bash

# Take backup from source:$1 in destination:$2

echo "Taking backup of $1 in $2"

cp -r $1 $2

[aadarsha@labserver scripts]$ chmod +x backup.sh 

[aadarsha@labserver scripts]$ ls /tmp/

[aadarsha@labserver scripts]$ backup.sh /home/aadarsha/scripts /tmp/
Taking backup of /home/aadarsha/scripts in /tmp/

[aadarsha@labserver scripts]$ ls /tmp/
scripts                                                                    

[aadarsha@labserver scripts]$

 # example 3
 # Calling custom libraries functions
 
[aadarsha@labserver scripts]$ ls

[aadarsha@labserver scripts]$ vi sub-func.sh

[aadarsha@labserver scripts]$ ls
sub-func.sh

[aadarsha@labserver scripts]$ vi main-func.sh

[aadarsha@labserver scripts]$ ls
main-func.sh  sub-func.sh

[aadarsha@labserver scripts]$ cat main-func.sh 
#!/usr/bin/bash

# calls the custom library functions defined in sub-func.sh
# load the script that contains custom functions
source sub-func.sh

# call the functions from sub-func.sh
log_info "Deployment started!"

log_error "Failed !!!"
[aadarsha@labserver scripts]$ 
[aadarsha@labserver scripts]$ cat sub-func.sh 
#!/bin/bash

 # custom library function

log_info(){
	echo "[INFO] $1"
}

log_error(){
	echo "[ERROR] $1"
}

[aadarsha@labserver scripts]$ chmod +x sub-func.sh 
[aadarsha@labserver scripts]$ chmod +x main-func.sh 

[aadarsha@labserver scripts]$ ./main-func.sh 
[INFO] Deployment started!
[ERROR] Failed !!!
[aadarsha@labserver scripts]$
 
[aadarsha@labserver ~]$ pwd
/home/aadarsha
 
[aadarsha@labserver ~]$ ls
scripts
[aadarsha@labserver ~]$ ls scripts/
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ echo "current date and time is: date"
current date and time is: date

[aadarsha@labserver ~]$ echo "current date and time is: `date`"
current date and time is: Tue Aug 11 02:54:30 PM +0545 2026

[aadarsha@labserver ~]$ echo "current date and time is: $(date)"
current date and time is: Tue Aug 11 02:54:42 PM +0545 2026

[aadarsha@labserver ~]$ echo "current date and time is: $date"
current date and time is: 

 # Script to delete multiple users at once. 

[root@labserver ~]# useradd user1
[root@labserver ~]# useradd user2
[root@labserver ~]# useradd user3
[root@labserver ~]# useradd user4
[root@labserver ~]# useradd user5

[root@labserver ~]# tail -5 /etc/passwd | cut -f 1 -d :
user1
user2
user3
user4
user5

[root@labserver ~]# vi deleteusers.sh

[root@labserver ~]# cat deleteusers.sh 
```text
#!/usr/bin/bash

# Shell script to delete multiple (eg. last 5 created users) users

for user in $(tail -5 /etc/passwd | cut -f 1 -d :)
  do
	userdel -r $user
	echo "user $user is deleted."
  done
[root@labserver ~]# 

[root@labserver ~]# ls -l deleteusers.sh 
-rw-r--r--. 1 root root 197 Aug 11 15:16 deleteusers.sh

[root@labserver ~]# chmod u+x deleteusers.sh

[root@labserver ~]# tail -5 /etc/passwd | cut -f 1 -d :
user1
user2
user3
user4
user5

[root@labserver ~]# ./deleteusers.sh 
user user1 is deleted.
user user2 is deleted.
user user3 is deleted.
user user4 is deleted.
user user5 is deleted.

 # Script to add the multiple users at once

[root@labserver ~]# vi employees-list.txt
[root@labserver ~]# cat employees-list.txt 
ram
shyam
gita
sita
hari
krishna
shiva
bishnu

[root@labserver ~]# pwd
/root

[root@labserver ~]# cp deleteusers.sh addusers.sh

[root@labserver ~]# vi addusers.sh 
[root@labserver ~]# cat addusers.sh
```
```text
#!/usr/bin/bash

# Shell script to create multiple user accounts (eg. user accounts of employees)

for user in $(cat /root/employees-list.txt)
  do
	useradd $user
	echo "Nepal_123" | passwd --stdin $user
	chage -d 0 $user
	chage -m 1 -M 90 -W 3 -I 0 $user
	echo "user $user is added."
  done
```
```
[root@labserver ~]# ls /home/
aadarsha    

[root@labserver ~]# ls -l addusers.sh 
-rwxr--r--. 1 root root 292 Aug 11 15:56 addusers.sh

[root@labserver ~]# ./addusers.sh 
user ram is added.
user shyam is added.
user gita is added.
user sita is added.
user hari is added.
user krishna is added.
user shiva is added.
user bishnu is added.

[root@labserver ~]# ls /home/
aadarsha  bishnu  gita  hari  krishna  ram  shiva  shyam  sita

[root@labserver ~]# chage -l ram
Last password change					: password must be changed
Password expires					: password must be changed
Password inactive					: password must be changed
Account expires						: never
Minimum number of days between password change		: 1
Maximum number of days between password change		: 90
Number of days of warning before password expires	: 3

[root@labserver ~]# tail -8 /etc/passwd | cut -f 1 -d :
ram
shyam
gita
sita
hari
krishna
shiva
bishnu

[root@labserver ~]# vi deleteusers.sh 

[root@labserver ~]# ./deleteusers.sh 
user ram is deleted.
user shyam is deleted.
user gita is deleted.
user sita is deleted.
user hari is deleted.
user krishna is deleted.
user shiva is deleted.
user bishnu is deleted. 

 # Check services running or not
 
[root@labserver ~]# systemctl is-active sshd
active

[root@labserver ~]# systemctl is-active httpd
active

[root@labserver ~]# vi service-health-check.sh

[root@labserver ~]# ls -l service-health-check.sh 
-rw-r--r--. 1 root root 269 Aug 11 16:19 service-health-check.sh

[root@labserver ~]# chmod u+x service-health-check.sh 
 
[root@labserver ~]# ls -l service-health-check.sh 
-rwxr--r--. 1 root root 269 Aug 11 16:19 service-health-check.sh
 
[root@labserver ~]# cat service-health-check.sh 
```
```text
#!/usr/bin/bash

 # Shell script to check for the service status and start inactive service

for service in httpd sshd
  do
	systemctl is-active $service
        if [ $? -eq 0 ]
	then
		echo "$service is running"
	else
		systemctl start $service
	fi
done
```
```
[root@labserver ~]# ./service-health-check.sh 
active
httpd is running
active
sshd is running

[root@labserver ~]# systemctl stop httpd

[root@labserver ~]# systemctl is-active httpd
inactive

[root@labserver ~]# ./service-health-check.sh
inactive
active
sshd is running

[root@labserver ~]# systemctl is-active httpd
active

 # Automate the script to run automatically every 2 minutes

[root@labserver ~]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak
 
[root@labserver ~]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak

[root@labserver ~]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

# */2          *          *         *           *          free -h >>/root/mem-output

 */2          *          *         *           *           ./root/service-health-check.sh
You have new mail in /var/spool/mail/root


 # Email Notification

[root@labserver ~]# yum -y install postfix s-nail

[root@labserver ~]# systemctl is-active postfix
active

[root@labserver ~]# systemctl is-enabled postfix
enabled

[root@labserver ~]# useradd emailadmin

[root@labserver ~]# su - emailadmin

[emailadmin@labserver ~]$ whoami
emailadmin
 
[emailadmin@labserver ~]$ mail
s-nail: No mail for emailadmin at /var/spool/mail/emailadmin

[emailadmin@labserver ~]$ exit
logout
[root@labserver ~]# 

[root@labserver ~]# mail emailadmin@localhost
Subject: Test Mail
To: emailadmin@localhost
Hello,
This is the test mail.
^D
-------
(Preliminary) Envelope contains:
To: emailadmin@localhost
Subject: Test Mail
Send this message [yes/no, empty: recompose]? yes
[root@labserver ~]# 

[root@labserver ~]# echo "This is second test mail" | mail -s "Second Test" emailadmin@localhost

[root@labserver ~]# su - emailadmin
Last login: Tue Aug 11 18:20:29 +0545 2026 on pts/1
[emailadmin@labserver ~]$ 

[emailadmin@labserver ~]$ mail
s-nail version v14.9.24.  Type `?' for help
/var/spool/mail/emailadmin: 2 messages 2 new
▸N  1 Super User            2026-08-11 18:21   16/531   "Test Mail                          "
 N  2 Super User            2026-08-11 18:23   15/528   "Second Test                        "
& 
[-- Message  1 -- 16 lines, 531 bytes --]:
Date: Tue, 11 Aug 2026 18:21:59 +0545
To: emailadmin@localhost
Subject: Test Mail
Message-Id: <20260811123659.35BF0BC@labserver.localdomain>
From: Super User <root@labserver.localdomain>

Hello,
This is the test mail.

& 
[-- Message  2 -- 15 lines, 528 bytes --]:
Date: Tue, 11 Aug 2026 18:23:18 +0545
To: emailadmin@localhost
Subject: Second Test
Message-Id: <20260811123818.59653BC@labserver.localdomain>
From: Super User <root@labserver.localdomain>

This is second test mail

& q
Held 2 messages in /var/spool/mail/emailadmin
[emailadmin@labserver ~]$ 

[root@labserver ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   20G  1.8G   19G   9% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                342M  6.3M  335M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            960M  485M  476M  51% /boot
/dev/mapper/cs-var   5.0G  197M  4.8G   4% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
/dev/sdb              10G  228M  9.8G   3% /shared-storage/dev_data
[root@labserver ~]# 

[root@labserver ~]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak
 
[root@labserver ~]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

 */2          *          *          *           *          df -h >>/root/disk-info

  *           *          *           *          *          free -h >>/root/mem-output

[root@labserver ~]# ls /
afs     bin   dev  home  lib64  mnt  proc  run   shared-storage  sys  usr
backup  boot  etc  lib   media  opt  root  sbin  srv             tmp  var
[root@labserver ~]# 

[root@labserver ~]# ls /root/
addusers.sh      deleteusers.sh      mem-output
anaconda-ks.cfg  employees-list.txt  service-health-check.sh
[root@labserver ~]# 

[root@labserver ~]# cat mem-output 
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       404Mi       1.0Gi       6.6Mi       375Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[root@labserver ~]# 
 
[root@labserver ~]# cat disk-info 
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   20G  1.8G   19G   9% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                342M  6.3M  335M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            960M  485M  476M  51% /boot
/dev/mapper/cs-var   5.0G  197M  4.8G   4% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
/dev/sdb              10G  228M  9.8G   3% /shared-storage/dev_data
tmpfs                171M  4.0K  171M   1% /run/user/0
[root@labserver ~]# 

[root@labserver ~]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak
[root@labserver ~]# 

[root@labserver ~]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

 */2          *          *          *           *          df -h >>/root/disk-info

  *           *          *           *          *          free -h | mail -s "Memory Info" emailadmin@localhost

[root@labserver ~]# 

[root@labserver ~]# su - emailadmin
Last login: Tue Aug 11 18:23:26 +0545 2026 on pts/1
[emailadmin@labserver ~]$ 

[emailadmin@labserver ~]$ mail
s-nail version v14.9.24.  Type `?' for help
/var/spool/mail/emailadmin: 2 messages
▸   1 Super User            2026-08-11 18:21   17/542   "Test Mail                          "
    2 Super User            2026-08-11 18:23   16/539   "Second Test                        "
& q
New mail has arrived.
Held 2 messages in /var/spool/mail/emailadmin
[emailadmin@labserver ~]$ 

[emailadmin@labserver ~]$ mail
s-nail version v14.9.24.  Type `?' for help
/var/spool/mail/emailadmin: 3 messages 1 new
    1 Super User            2026-08-11 18:21   17/542   "Test Mail                          "
    2 Super User            2026-08-11 18:23   16/539   "Second Test                        "
▸N  3 Super User            2026-08-11 18:33   17/710   "Memory Info                        "
& 3
[-- Message  3 -- 17 lines, 710 bytes --]:
Date: Tue, 11 Aug 2026 18:33:01 +0545
To: emailadmin@localhost
Subject: Memory Info
Message-Id: <20260811124801.EA1C8BC@labserver.localdomain>
From: Super User <root@labserver.localdomain>

               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       400Mi       1.1Gi       6.6Mi       375Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi

& q
Held 3 messages in /var/spool/mail/emailadmin
[emailadmin@labserver ~]$ 

[emailadmin@labserver ~]$ 
You have mail in /var/spool/mail/emailadmin
[emailadmin@labserver ~]$ 

[emailadmin@labserver ~]$ exit
logout
[root@labserver ~]# 

[root@labserver ~]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak
[root@labserver ~]# 

[root@labserver ~]# cat disk_monitor.sh 
#!/bin/bash

# Shell script to monitor the disk usage

THRESHOLD=5    # for testing purpose use 5
USAGE=$(df -h / | grep / | awk '{print $5}' | sed 's/%//')

if [ $USAGE -ge $THRESHOLD ]; then
	echo "Disk usage is above $THRESHOLD%! Current: $USAGE%" | mail -s "Disk Alert! | space: $USAGE% used." emailadmin@localhost
fi
[root@labserver ~]#

[root@labserver ~]# df -h /
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   20G  2.0G   19G  10% /

[root@labserver ~]# df -h / | awk '{print $5}'
Use%
10%

[root@labserver ~]# df -h / | grep /
/dev/mapper/cs-root   20G  2.0G   19G  10% /

[root@labserver ~]# df -h / | grep / | awk '{print $5}'
10%

[root@labserver ~]# df -h / | grep / | awk '{print $5}' | sed 's/%//'
10

[root@labserver ~]# ls -l disk_monitor.sh 
-rw-r--r--. 1 root root 298 Aug 14 08:36 disk_monitor.sh

[root@labserver ~]# chmod a+x disk_monitor.sh 

[root@labserver ~]# ls -l disk_monitor.sh 
-rwxr-xr-x. 1 root root 298 Aug 14 08:36 disk_monitor.sh

[root@labserver ~]# su - emailadmin

[emailadmin@labserver ~]$ mail
s-nail: No mail for emailadmin at /var/spool/mail/emailadmin

[emailadmin@labserver ~]$ exit
logout

[root@labserver ~]# ./disk_monitor.sh

[root@labserver ~]# df -h /
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   20G  2.0G   19G  10% /

[root@labserver ~]# su - emailadmin
Last login: Fri Aug 14 08:49:44 +0545 2026 on pts/0

[emailadmin@labserver ~]$ mail
s-nail version v14.9.24.  Type `?' for help
/var/spool/mail/emailadmin: 1 message 1 new
▸N  1 Super User            2026-08-14 08:50   15/541   "Disk Alert                         "
& 1
[-- Message  1 -- 15 lines, 541 bytes --]:
Date: Fri, 14 Aug 2026 08:50:55 +0545
To: emailadmin@localhost
Subject: Disk Alert
Message-Id: <20260814030555.6C92E1F1@labserver.localdomain>
From: Super User <root@labserver.localdomain>

Disk usage is above 5%! Current: 10%

& q
Held 1 message in /var/spool/mail/emailadmin
[emailadmin@labserver ~]$ 

[emailadmin@labserver ~]$ cat /var/spool/mail/emailadmin 
From root@labserver.localdomain  Fri Aug 14 08:50:55 2026
Return-Path: <root@labserver.localdomain>
X-Original-To: emailadmin@localhost
Delivered-To: emailadmin@localhost
Received: by labserver.localdomain (Postfix, from userid 0)
	id 6C92E1F1; Fri, 14 Aug 2026 08:50:55 +0545 (+0545)
Date: Fri, 14 Aug 2026 08:50:55 +0545
To: emailadmin@localhost
Subject: Disk Alert
User-Agent: s-nail v14.9.24
Message-Id: <20260814030555.6C92E1F1@labserver.localdomain>
From: Super User <root@labserver.localdomain>
Status: RO

Disk usage is above 5%! Current: 10%

[emailadmin@labserver ~]$ exit
logout
[root@labserver ~]# 

 # To monitor the disk at regular interval and send mail, set the cronjob to run the script.

[root@labserver ~]# poweroff
[root@labserver ~]# Connection to 192.168.254.2 closed by remote host.
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$ 
```