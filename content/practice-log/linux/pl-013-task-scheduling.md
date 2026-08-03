---
title: "PL - 013 — Task Scheduling & Automation ( at Jobs / Cron Jobs)"
date: 2026-08-03
draft: false
---

#### Concepts:
  > at jobs, 
  > cron jobs

#### Task Scheduling ( **Use when:** )
> Run backups, Rotate logs, Clean temp files, Restart services, Send reports, Health checks, Database dumps, Automation scripts

 #### Two scheduling methods

| Tool   | Best For             |
|--------|----------------------|
| `cron` | Repeating jobs       |
| `at`   | One-time future jobs |

#### Best Practices:
 - Use absolute paths (/usr/bin/python3).
 - Make scripts executable: ```
  chmod +x script.sh ```
 - Test the script manually before scheduling.
 - Redirect output to a log file.
 - Use a dedicated service account for production jobs.
 - Keep cron jobs in version control when possible.
 - Add comments in crontab to describe each job.
 - Monitor logs and verify jobs actually completed.
 ---
 
#### Terminal Session 
```
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Wed Jul 29 13:46:18 2026

[aadarsha@labserver ~]$ date
Wed Jul 29 01:46:53 PM +0545 2026

[aadarsha@labserver ~]$ whoami
aadarsha

[root@labserver ~]# whoami
root

[root@labserver ~]# dnf -y install at

[root@labserver ~]# rpm -q at
at-3.2.5-14.el10.x86_64

[root@labserver ~]# systemctl status atd
● atd.service - Deferred execution scheduler
     Loaded: loaded (/usr/lib/systemd/system/atd.service; enabled; preset: enabled)
...
[root@labserver ~]# 

[root@labserver ~]# systemctl is-active atd
active

[root@labserver ~]# systemctl is-enabled atd
enabled

[root@labserver ~]# who
aadarsha pts/0        2026-07-29 20:20 (192.168.254.32)

 # Time format of at jobs

     # Time Only:
       - at 17:00

     # Relative Time:
       - at now + 10 minutes
       - at now + 2 hours
       - at now + 1 day / week / month / year

     # Time + Day of Week
       - at 09:00 Monday
       - at 17:00 Friday

     # Today / Tomorrow
       - at tomorrow
       - at 02:00 tomorrow

     # Month + Day
       - at 17:00 Nov 11

     # Month + Day + Year
       - at 17:00 Nov 11 2032

     # Ctrl+D -->  after entering commands to submit the job

[root@labserver ~]# at now +3 minutes
warning: commands will be executed using /bin/sh
at Wed Jul 29 21:58:00 2026
at> echo "Hello, World!" >/root/message
at> <EOT>
job 4 at Wed Jul 29 21:58:00 2026
[root@labserver ~]#

[root@labserver ~]# atq
4	Wed Jul 29 21:58:00 2026 a root

[root@labserver ~]# date
Wed Jul 29 09:57:39 PM +0545 2026

[root@labserver ~]# atq
4	Wed Jul 29 21:58:00 2026 a root

[root@labserver ~]# ls /root/
anaconda-ks.cfg

[root@labserver ~]# atq
[root@labserver ~]# 

[root@labserver ~]# ls /root/
anaconda-ks.cfg  message

[root@labserver ~]# pwd
/root
 
[root@labserver ~]# cat message 
Hello, World!

[root@labserver ~]# atq
5	Thu Jul 30 09:14:00 2026 a aadarsha
6	Fri Jul 31 05:00:00 2026 a root

 # format:
 
 # <job-id>   <scheduled-date-and-time>   <queue>   <user>
 
 #   a   -->  default queue used by the at command
 
[root@labserver ~]# pwd
/root
[root@labserver ~]# ls /var/spool/at
a0000501c60bd1  a0000601c61073  spool

[root@labserver ~]# cd /var/spool/at
[root@labserver at]# pwd
/var/spool/at
[root@labserver at]# ls
a0000501c60bd1  a0000601c61073  spool

 # file format : <queue> <encoded-job-identifier>
 #                  a --> queue name (default queue a)
 
[root@labserver at]# cat a0000501c60bd1 
#!/bin/sh
...
${SHELL:-/bin/sh} << 'marcinDELIMITER194b379f'
uptime > /aadarsha/uptime-output
marcinDELIMITER194b379f
[root@labserver at]# 

[root@labserver at]# cd -
/root

  # Remove an at job
 
[root@labserver ~]# atq
5	Thu Jul 30 09:14:00 2026 a aadarsha
6	Fri Jul 31 05:00:00 2026 a root
7	Fri Jul 30 05:00:00 2027 a root

[root@labserver ~]# atrm 6			    #  atq <job-id>

[root@labserver ~]# atq
5	Thu Jul 30 09:14:00 2026 a aadarsha
7	Fri Jul 30 05:00:00 2027 a root

  # By default, if an at job produces output (stdout or stderr) and the output is not redirected, at sends the output by email to the job owner's mail address.
  # A running mail transfer agent (MTA), such as Postfix, is required for email delivery.
 
[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ atq
 
[aadarsha@labserver ~]$ at now + 3 minutes
warning: commands will be executed using /bin/sh
at Thu Jul 30 09:22:00 2026
at> ps    
at> free -h
at> <EOT>
job 8 at Thu Jul 30 09:22:00 2026
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ systemctl status postfix
○ postfix.service - Postfix Mail Transport Agent
     Loaded: loaded (/usr/lib/systemd/system/postfix.service; disabled; preset: disabled)
     Active: inactive (dead)
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sudo systemctl start postfix
[sudo] password for aadarsha: 

[aadarsha@labserver ~]$ sudo systemctl enable postfix
Created symlink '/etc/systemd/system/multi-user.target.wants/postfix.service' → '/usr/lib/systemd/system/postfix.service'.
 
[aadarsha@labserver ~]$ systemctl status postfix
● postfix.service - Postfix Mail Transport Agent
     Loaded: loaded (/usr/lib/systemd/system/postfix.service; enabled; preset: disabled)
     Active: active (running) since Thu 2026-07-30 09:21:49 +0545; 32s ago
...
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ systemctl is-active postfix
active
You have new mail in /var/spool/mail/aadarsha
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat /var/spool/mail/aadarsha 
From aadarsha@labserver.localdomain  Thu Jul 30 09:22:00 2026
Return-Path: <aadarsha@labserver.localdomain>
X-Original-To: aadarsha
Delivered-To: aadarsha@labserver.localdomain
Received: by labserver.localdomain (Postfix, from userid 1000)
	id 1714B8C17; Thu, 30 Jul 2026 09:22:00 +0545 (+0545)
Subject: Output from your job        8
To: aadarsha@labserver.localdomain
Message-Id: <20260730033700.1714B8C17@labserver.localdomain>
Date: Thu, 30 Jul 2026 09:22:00 +0545 (+0545)
From: Aadarsha Khadka <aadarsha@labserver.localdomain>
    PID TTY          TIME CMD
   1905 ?        00:00:00 systemd
   1907 ?        00:00:00 (sd-pam)
   1960 ?        00:00:00 sshd-session
   2530 ?        00:00:00 sh
   2531 ?        00:00:00 bash
   2532 ?        00:00:00 ps
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       350Mi       1.2Gi       4.9Mi       240Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[aadarsha@labserver ~]$ 

 # A non-privileged user may schedule jobs only if authorized by the scheduler's access control configuration 
 # (for example, cron.allow, cron.deny, at.allow, or at.deny).
 # Scheduled jobs execute with the effective user ID and group permissions of the user who scheduled them and cannot perform operations beyond that user's assigned privileges.


 # running scripts by scheduling the job

[aadarsha@labserver ~]$ ls
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ mkdir scripts

[aadarsha@labserver ~]$ vi scripts/health_check.sh 

[aadarsha@labserver ~]$ cat scripts/health_check.sh 
#!/bin/bash

###############################################################################
# Script Name : health_check.sh
# Description : Basic Linux Health Check Script
# Author      : aadarsha
###############################################################################

echo "=========================================="
echo "        Linux Health Check Report"
echo "=========================================="
echo "Hostname : $(hostname)"
echo "Date     : $(date)"
echo

# System Uptime
echo "1. System Uptime"
uptime
echo

# CPU Load
echo "2. CPU Load Average"
uptime | awk -F'load average:' '{print $2}'
echo

# Memory Usage
echo "3. Memory Usage"
free -h
echo

# Disk Usage
echo "4. Disk Usage"
df -h
echo

# SSH Service Status
echo "5. SSH Service Status"

if systemctl is-active --quiet sshd; then
    echo "sshd service is running."
else
    echo "sshd service is NOT running."
fi

echo
echo "=========================================="
echo "Health Check Completed Successfully"
echo "=========================================="
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls -l scripts/health_check.sh 
-rw-r--r--. 1 aadarsha aadarsha 1047 Jul 30 21:50 scripts/health_check.sh
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ chmod +x scripts/health_check.sh 

[aadarsha@labserver ~]$ ls -l scripts/health_check.sh 
-rwxr-xr-x. 1 aadarsha aadarsha 1047 Jul 30 21:50 scripts/health_check.sh
[aadarsha@labserver ~]

[aadarsha@labserver ~]$ at now +3 minutes
warning: commands will be executed using /bin/sh
at Thu Jul 30 22:00:00 2026
at> /home/aadarsha/scripts/health_check.sh > /home/aadarsha/scripts_output
at> <EOT>
job 9 at Thu Jul 30 22:00:00 2026
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ atq
9	Thu Jul 30 22:00:00 2026 a aadarsha
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
scripts
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ pwd
/home/aadarsha
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ atq
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls
scripts  scripts_output
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ cat scripts_output 
==========================================
        Linux Health Check Report
==========================================
Hostname : labserver
Date     : Thu Jul 30 10:00:00 PM +0545 2026

1. System Uptime
 22:00:00 up 42 min,  2 users,  load average: 0.00, 0.00, 0.00

2. CPU Load Average
 0.00, 0.00, 0.00

3. Memory Usage
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       361Mi       1.2Gi       4.8Mi       239Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi

4. Disk Usage
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   20G  1.6G   19G   8% /
devtmpfs             830M     0  830M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            960M  386M  575M  41% /boot
/dev/mapper/cs-var   5.0G  220M  4.8G   5% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000

5. SSH Service Status
sshd service is running.

==========================================
Health Check Completed Successfully
==========================================
[aadarsha@labserver ~]$ 

 # Scheduling repeated tasks using cron
 
 # Common Cron Job Purposes (Notes)
   - Database backup – Run automatic backups daily or weekly.
   - Log rotation – Archive and remove old log files.
   - Temporary file cleanup – Delete unnecessary files periodically.
   - System monitoring – Check CPU, memory, disk, or server health.
   - Send emails/reports – Generate and email daily or weekly reports.
   - File synchronization – Sync files between servers or cloud storage.
   - Restart services – Restart applications or services at scheduled times.
   - Update software – Run package updates or security updates.
   - Execute scripts – Run automation scripts without manual intervention.
   - Data processing – Import, export, or transform data on a schedule.
 
[aadarsha@labserver ~]$ rpm -q cronie
cronie-1.7.0-14.el10.x86_64

[root@labserver ~]# man 4 crontabs

[root@labserver ~]# man crontab

[aadarsha@labserver ~]$ systemctl status crond
● crond.service - Command Scheduler
     Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; preset: enabled)
...
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ systemctl is-active crond
active

[aadarsha@labserver ~]$ systemctl is-enabled crond
enabled

 # Format of crontab:
 
           *  *  *  *  *   < command/script >
          │  │  │  │  │
          │  │  │  │  └── Day of week (0-7)
          │  │  │  └──── Month
          │  │  └────── Day of month
          │  └──────── Hour
          └────────── Minute

[root@labserver ~]# cat /etc/crontab 
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
#                                     .---------------- minute (0 - 59)
#                                     |  .------------- hour (0 - 23)
#                                     |  |  .---------- day of month (1 - 31)
#                                     |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
#                                     |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
#                                     |  |  |  |  |
#                                     *  *  *  *  * user-name  command to be executed
[root@labserver ~]#
 
 # Common special characters
| Symbol |    Meaning      | Example                    |
|--------|---------------- | -------------------------- |
|   *    | Every value     | * * * * * --> Every minute |
|  ,     | Multiple values |   1,15,30                  |
|  -     | Range           |   1-5                      |
|  /     | Step interval   |   */10 → Every 10 units    |

 # Cron's smallest unit is 1 minute,

 # Examples:
 
 *  *  *  *  *      -->   run every minute
 
 */5 * * * *        -->   every 5 minutes ( 00, 05, 10, ... , 55 )

 */15 * * * *       -->   every 15 minutes ( 00, 15, 30, 45 )
 
 0 * * * *          -->   every hour   ( 01:00, 02:00, ... )
 
 0 0 * * *          -->   every da at midnight ( 00:00 )
 
 30 2 * * *         -->   every day at 2:30 AM
 
 0 9 * * 1-5        -->   every week-day at 2:30 AM
 
 0 10 * * 6,0       -->   every weekend at 10:00 AM
 
 0 8 * * 1          -->   every Monday at 8:00 AM
 
 * * 1 * *          -->   first day of every month ( Jan 1, Feb 1, ... )
 
 0 0 1 1 *          -->   every January 1 ( once per year ) 
 
 0 3 * * 0          -->   every Sunday at 3:00 AM
 
 */10 9-17 * * 1-5   -->   every 10 minutes during business hours ( 9 AM to 5 PM, Mon - Fri )
 
 0 */2 * * *         -->   every 2 hours ( 00:00, 02:00, 04:00, .. )

 0 */6 * * *         -->   every 6 hours ( 00:00, 06:00, 12:00, 18:00 )
 
 0 0 */5 * *         -->   every 5 days
 
 0 8,20 * * *        -->   at 8:00 AM and 8:00 PM every day
 
 * 9-17 * * *        -->   every minute between 9 AM and 5 PM
 
 15 9 * * 1,3        -->   at 9:15 every Monday and Wednesday
 
 */5 * * * 1-5       -->   every 5 minutes on Weekdays
 
 * * * 1 *           -->   every minute in January
 
 0 12  * * *         -->   every day at noon
 
 30 5 * * 5          -->   every Friday at 5:30

[root@labserver ~]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak
 
[root@labserver ~]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

 *            *          *         *           *             ps
 */3          *          *         *           *             free -h
 30           5          *         *           *             df -h          
[root@labserver ~]# 

[root@labserver ~]# cat /var/spool/cron/root 
# Min         Hr         M         DoM         DoW         < command/script >

 *            *          *         *           *             ps
 */3          *          *         *           *             free -h
 30           5          *         *           *             df -h          
[root@labserver ~]# 

[root@labserver ~]# 
You have new mail in /var/spool/mail/root

[root@labserver ~]# cat /var/spool/mail/root 
From MAILER-DAEMON  Sat Aug  1 19:03:02 2026
Return-Path: <>
X-Original-To: root
Delivered-To: root@labserver.localdomain
Received: by labserver.localdomain (Postfix, from userid 0)
	id 397C38C10; Sat,  1 Aug 2026 19:03:02 +0545 (+0545)
...
[root@labserver ~]#

[root@labserver ~]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

 */3          *          *         *           *             free -h >>/root/mem-output
 30           5          *         *           *             df -h >>/root/disk-output
[root@labserver ~]# 

[root@labserver ~]# ls
anaconda-ks.cfg  message

[root@labserver ~]# ls
anaconda-ks.cfg  mem-output  message
 
[root@labserver ~]# cat mem-output 
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       338Mi       1.2Gi       6.3Mi       262Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[root@labserver ~]# 

[root@labserver ~]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

# */3          *          *         *           *             free -h >>/root/mem-output
# 30           5          *         *           *             df -h >>/root/disk-output
[root@labserver ~]# 

 # To edit or delete cron job

[root@labserver ~]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak

[root@labserver ~]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

 */2          *          *         *           *          free -h >>/root/mem-output
[root@labserver ~]# 

[root@labserver ~]# ls
anaconda-ks.cfg  mem-output  message

[root@labserver ~]# rm mem-output 
rm: remove regular file 'mem-output'? y

[root@labserver ~]# ls
anaconda-ks.cfg  message
[root@labserver ~]# 

[root@labserver ~]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

 */2          *          *         *           *          free -h >>/root/mem-output
[root@labserver ~]# 

[root@labserver ~]# crontab -r
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak              # Backup saved by the system

[root@labserver ~]# crontab -l
no crontab for root

[root@labserver ~]# cat /root/.cache/crontab/crontab.bak                                 # Backup saved by the system
# Min         Hr         M         DoM         DoW         < command/script >

 */2          *          *         *           *          free -h >>/root/mem-output
[root@labserver ~]# 

 # restore the backup 
 
[root@labserver ~]# crontab -l
no crontab for root

[root@labserver ~]# crontab /root/.cache/crontab/crontab.bak

[root@labserver ~]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

 */2          *          *         *           *          free -h >>/root/mem-output
[root@labserver ~]# 

[root@labserver ~]# systemctl restart crond

[root@labserver ~]# systemctl is-active crond
active
[root@labserver ~]# 

 # Viewing the logs of cronjobs
 
[root@labserver ~]# tail /var/log/cron
Aug  2 21:35:27 labserver crontab[2187]: (root) END EDIT (root)
Aug  2 21:35:34 labserver crontab[2201]: (root) LIST (root)
Aug  2 21:36:02 labserver crond[967]: (root) RELOAD (/var/spool/cron/root)
Aug  2 21:36:02 labserver CROND[2226]: (root) CMD (free -h >>/root/mem-output)
Aug  2 21:36:02 labserver CROND[2209]: (root) CMDEND (free -h >>/root/mem-output)
Aug  2 21:36:26 labserver crond[967]: (CRON) INFO (Shutting down)
Aug  2 21:36:26 labserver crond[2239]: (CRON) STARTUP (1.7.0)
Aug  2 21:36:26 labserver crond[2239]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 12% if used.)
Aug  2 21:36:26 labserver crond[2239]: (CRON) INFO (running with inotify support)
Aug  2 21:36:26 labserver crond[2239]: (CRON) INFO (@reboot jobs will be run at computer's startup.)
[root@labserver ~]# 

[root@labserver ~]# whoami
root
 
[root@labserver ~]# ls /var/spool/cron/
root

[root@labserver ~]# ls /var/spool/cron/root 
/var/spool/cron/root
 
[root@labserver ~]# cat /var/spool/cron/root 
# Min         Hr         M         DoM         DoW         < command/script >

 */2          *          *         *           *          free -h >>/root/mem-output
[root@labserver ~]# 

 # Controlling User Access to Cron
 
[root@labserver ~]# ls /home/
aadarsha  milan  suman
 
[root@labserver ~]# vi /etc/cron.deny 

[root@labserver ~]# cat /etc/cron.deny 

[root@labserver ~]# vi /etc/cron.deny 

[root@labserver ~]# cat /etc/cron.deny 
suman
ram
sita

[root@labserver ~]# su - milan
 
[milan@labserver ~]$ crontab -e
no crontab for milan - using an empty one
crontab: installing new crontab

[milan@labserver ~]$ crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

 */3          *          *         *           *          whoami >>/root/job1-output

[milan@labserver ~]$ exit
logout

[root@labserver ~]# su - suman

[suman@labserver ~]$ crontab -e
You (suman) are not allowed to use this program (crontab)
See crontab(1) for more information

[suman@labserver ~]$ exit
logout

[root@labserver ~]# vi /etc/cron.deny 
[root@labserver ~]# cat /etc/cron.deny 
[root@labserver ~]# 
[root@labserver ~]# vi /etc/cron.allow
[root@labserver ~]# 
[root@labserver ~]# cat /etc/cron.allow 
suman
[root@labserver ~]# 

[root@labserver ~]# su - suman
Last login: Mon Aug  3 11:07:07 +0545 2026 on pts/0

[suman@labserver ~]$ crontab -e
no crontab for suman - using an empty one
crontab: installing new crontab
[suman@labserver ~]$ 
[suman@labserver ~]$ crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

 */2          *          *         *           *          free -h >>/root/mem-output
[suman@labserver ~]$ 

[suman@labserver ~]$ exit
logout
[root@labserver ~]# 
 
[root@labserver ~]# ls /home/
aadarsha  milan  suman
[root@labserver ~]# 
[root@labserver ~]# su - milan
Last login: Mon Aug  3 11:05:29 +0545 2026 on pts/0
[milan@labserver ~]$ 
[milan@labserver ~]$ crontab -e
You (milan) are not allowed to use this program (crontab)
See crontab(1) for more information

[milan@labserver ~]$ exit
logout
[root@labserver ~]# 
[root@labserver ~]# su - aadarsha
Last login: Mon Aug  3 10:12:12 +0545 2026 from 192.168.254.32 on pts/0
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ crontab -e
You (aadarsha) are not allowed to use this program (crontab)
See crontab(1) for more information

[aadarsha@labserver ~]$ crontab -l
You (aadarsha) are not allowed to use this program (crontab)
See crontab(1) for more information

[aadarsha@labserver ~]$ exit
logout

[root@labserver ~]# vi /etc/cron.allow 

[root@labserver ~]# cat /etc/cron.allow 
[root@labserver ~]# 
[root@labserver ~]# cat /etc/cron.deny 
[root@labserver ~]# 

 # Now, any user can set the cronjobs

 # Similarly, for at jobs:
  # /etc/at.allow
  # /etc/at.deny

[root@labserver ~]# ls /var/spool/cron/
milan  root  suman

[root@labserver ~]# cat /var/spool/cron/milan 
# Min         Hr         M         DoM         DoW         < command/script >

 */3          *          *         *           *          whoami >>/root/job1-output

[root@labserver ~]# cat /var/spool/cron/suman 
# Min         Hr         M         DoM         DoW         < command/script >

 */2          *          *         *           *          free -h >>/root/mem-output

[root@labserver ~]# cat /var/spool/cron/root 
# Min         Hr         M         DoM         DoW         < command/script >

 */2          *          *         *           *          free -h >>/root/mem-output

[root@labserver ~]# rm /var/spool/cron/milan 
rm: remove regular file '/var/spool/cron/milan'? y

[root@labserver ~]# rm /var/spool/cron/suman 
rm: remove regular file '/var/spool/cron/suman'? y

[root@labserver ~]# ls /home
aadarsha  milan  suman
 
[root@labserver ~]# ls /var/spool/cron/
root
[root@labserver ~]# 

 # Creating a System-Wide Cron Job with /etc/cron.d
 
 # 7-Field Syntax:
 
 #  ┌───────────── minute (0 - 59)
 #  │  ┌─────────── hour (0 - 23)
 #  │  │  ┌───────── day of month (1 - 31)
 #  │  │  │  ┌─────── month (1 - 12)
 #  │  │  │  │  ┌───── day of week (0 - 7) (Sunday = 0 or 7)
 #  │  │  │  │  │  ┌─── user
 #  │  │  │  │  │  │
 #  *  *  *  *  *  *      <command>
 
[root@labserver ~]# cd /etc/cron.d

[root@labserver cron.d]# ls
0hourly

[root@labserver cron.d]# pwd
/etc/cron.d

[root@labserver cron.d]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak

[root@labserver cron.d]# vi mycronjob

[root@labserver cron.d]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

# */2          *          *         *           *          free -h >>/root/mem-output
[root@labserver cron.d]# 

[root@labserver cron.d]# ls -l /etc/cron.d/mycronjob
-rw-r--r--. 1 root root 236 Aug  3 11:52 /etc/cron.d/mycronjob
 
[root@labserver cron.d]# cat mycronjob 
# Min (0 - 59)    # Hr (0 - 23)    # DoM (1 - 31)    # Mo (1 - 12)    # DoW (0 - 7)    # user    # <command>

  */2                 *               *                    *               *            milan     date >>/home/milan/job1-op
[root@labserver cron.d]# 

[root@labserver cron.d]# cat -A mycronjob 
# Min (0 - 59)    # Hr (0 - 23)    # DoM (1 - 31)    # Mo (1 - 12)    # DoW (0 - 7)    # user    # <command>$
$
  */2                 *               *                    *               *            milan     date >>/home/milan/job1-op$
[root@labserver cron.d]# 

[root@labserver cron.d]# su - milan
Last login: Mon Aug  3 11:14:25 +0545 2026 on pts/0
[milan@labserver ~]$ 

[milan@labserver ~]$ ls
job1-op  myfile
[milan@labserver ~]$ 
[milan@labserver ~]$ cat job1-op 
Mon Aug  3 11:54:01 AM +0545 2026
Mon Aug  3 11:56:02 AM +0545 2026
Mon Aug  3 11:58:01 AM +0545 2026
Mon Aug  3 12:00:01 PM +0545 2026
Mon Aug  3 12:02:01 PM +0545 2026
Mon Aug  3 12:04:02 PM +0545 2026
Mon Aug  3 12:06:01 PM +0545 2026
[milan@labserver ~]$ 

[milan@labserver ~]$ exit
logout
[root@labserver cron.d]# 
 
[root@labserver cron.d]# vi mycronjob 

[root@labserver cron.d]# cat mycronjob 
# Min (0 - 59)    # Hr (0 - 23)    # DoM (1 - 31)    # Mo (1 - 12)    # DoW (0 - 7)    # user     # <command/script>

#  */2                 *               *                    *               *            milan     date >>/home/milan/job1-op
[root@labserver cron.d]# 

 # Common Locations for Cron Jobs:

 # User crontabs

[root@labserver cron.d]# cd
[root@labserver ~]# 

 # User crontabs

[root@labserver ~]# ls /var/spool/cron/
root

 # System crontab
 
[root@labserver ~]# ls /etc/crontab 
/etc/crontab

 # Additional system cron files

[root@labserver ~]# ls /etc/cron.d/
0hourly  mycronjob
 
 # Hourly jobs

[root@labserver ~]# ls /etc/cron.hourly/
0anacron

 # Daily jobs
 
[root@labserver ~]# ls /etc/cron.daily/
[root@labserver ~]#
 
 # Weekly jobs

[root@labserver ~]# ls /etc/cron.weekly/
[root@labserver ~]# 

 # Monthly jobs
 
[root@labserver ~]# ls /etc/cron.monthly/
[root@labserver ~]# 
 
 # System crontab

[root@labserver ~]# cat /etc/crontab 
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

[root@labserver ~]# 

[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$ 
