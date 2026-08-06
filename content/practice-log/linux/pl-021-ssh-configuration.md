---
title: "PL - 021 — Secure SSH Configuration & Remote Access Hardening"
date: 2026-08-06
draft: false
---

#### Concepts:
  > SSH access control, key-based authentication, sshd hardening, secure remote administration

#### Remote Access Methods (**Use when:**)
  - Telnet: legacy device access only; insecure due to plaintext communication
  - SSH: secure remote shell access and server administration
  - SSH tunneling: encrypted access to internal services through a trusted host
  - Jump host / Bastion host: controlled access path to private infrastructure
  - VPN: secure remote connectivity into protected networks
  - Console / Out-of-band access: emergency server recovery and troubleshooting

#### SSH Client Tools
  - OpenSSH Client (Linux/macOS/Windows CLI): native command-line SSH access for administrators and automation
  - PuTTY : lightweight Windows SSH client, commonly used for manual server access
  - MobaXterm: Windows terminal suite with SSH, SFTP, X11 forwarding, and remote administration features
  - Bitvise SSH Client: Windows SSH/SFTP client with advanced session and file transfer features
  - Git Bash: Windows Unix-like terminal environment that provides OpenSSH commands

#### Best Practices:
- Disable password authentication after key validation
- Disable direct root login
- Restrict SSH users and groups
- Validate `sshd_config` before restarting SSH service
- Monitor SSH authentication logs
- Disable unnecessary SSH features
- Maintain recovery access before applying lockout restrictions
 ---
#### Terminal Session 

```
aadarkdk@pop-os:~$ whoami
aadarkdk

aadarkdk@pop-os:~$ hostname
pop-os

aadarkdk@pop-os:~$ ping -c 2 192.168.254.2
PING 192.168.254.2 (192.168.254.2) 56(84) bytes of data.
64 bytes from 192.168.254.2: icmp_seq=1 ttl=64 time=0.486 ms
64 bytes from 192.168.254.2: icmp_seq=2 ttl=64 time=0.573 ms

--- 192.168.254.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1008ms
rtt min/avg/max/mdev = 0.486/0.529/0.573/0.043 ms
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Mon Aug  3 12:37:57 2026 from 192.168.254.32

[aadarsha@labserver ~]$ date
Mon Aug  3 01:48:08 PM +0545 2026

[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ hostname
labserver
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ hostname -I
192.168.254.3 2407:5200:404:17e1:a00:27nf:fec7:6ccb 
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Mon Aug  3 14:00:25 +0545 2026 on pts/0

[root@labserver ~]# whoami
root

[root@labserver ~]# yum update -y

[root@labserver ~]# rpm -q openssh-server
openssh-server-9.9p1-28.el10.x86_64

[root@labserver ~]# yum install -y openssh-server
Last metadata expiration check: 2:23:50 ago on Mon 03 Aug 2026 12:17:22 PM +0545.
Package openssh-server-9.9p1-28.el10.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
[root@labserver ~]# 

[root@labserver ~]# systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-08-03 14:20:57 +0545; 20min ago
...
[root@labserver ~]# 
 
[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@labserver ~]# 

[root@labserver ~]# exit
logout

[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.

aadarkdk@pop-os:~$ ssh root@192.168.254.2		# prevent direct root login in prod env
root@192.168.254.2's password: 
Last login: Mon Aug  3 14:03:33 2026

[root@labserver ~]# whoami
root

 # Configuring SSH Server

[root@labserver ~]# vi /etc/ssh/sshd_config
sshd_config    sshd_config.d/ 

[root@labserver ~]# vi /etc/ssh/sshd_config

[root@labserver ~]# netstat -tnl
-bash: netstat: command not found

[root@labserver ~]# yum whatprovides netstat
...
net-tools-2.0-0.72.20160912git.el10.x86_64 : Basic networking tools
Repo        : baseos
...
[root@labserver ~]#
 
[root@labserver ~]# yum -y install net-tools
...
[root@labserver ~]#

[root@labserver ~]# netstat -tnl | grep 5050
[root@labserver ~]# 

[root@labserver ~]# vi /etc/ssh/sshd_config

[root@labserver ~]# cat /etc/ssh/sshd_config
#	$OpenBSD: sshd_config,v 1.104 2021/07/02 05:11:21 dtucker Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.
...
# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change.
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
#Port 22
Port 5050
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
...
[root@labserver ~]# 

[root@labserver ~]# getenforce 
Enforcing

[root@labserver ~]# vi /etc/ssh/sshd_config
 
[root@labserver ~]# semanage port -a -t ssh_port_t -p tcp 5050
-bash: semanage: command not found

[root@labserver ~]# dnf provides '/*semanage'
... 

[root@labserver ~]# dnf search semanage
...

[root@labserver ~]# dnf install -y policycoreutils-python-utils
...
Installed:
  checkpolicy-3.11-1.el10.x86_64                policycoreutils-python-utils-3.11-1.el10.noarch        python3-audit-4.0.3-5.el10.x86_64          python3-distro-1.9.0-5.el10.noarch       
  python3-libsemanage-3.11-1.el10.x86_64        python3-policycoreutils-3.11-1.el10.noarch             python3-setools-4.7.0-1.el10.x86_64       
Complete!
[root@labserver ~]# 

[root@labserver ~]# which semanage
/usr/sbin/semanage

[root@labserver ~]# semanage port -a -t ssh_port_t -p tcp 5050
Port tcp/5050 already defined, modifying instead
[root@labserver ~]# 

[root@labserver ~]# semanage port -l | grep ssh
ssh_port_t                     tcp      5050, 22
[root@labserver ~]# 
[root@labserver ~]# systemctl status sshd
...

[root@labserver ~]# systemctl restart sshd

[root@labserver ~]# systemctl is-active sshd
active

[root@labserver ~]# systemctl reload sshd
...

[root@labserver ~]# systemctl reload sshd

[root@labserver ~]# exit
logout
Connection to 192.168.254.2 closed.

aadarkdk@pop-os:~$ ssh root@192.168.254.2
ssh: connect to host 192.168.254.2 port 22: Connection refused

aadarkdk@pop-os:~$ ssh -p 22 aadarsha@192.168.254.2
ssh: connect to host 192.168.254.2 port 22: Connection refused

aadarkdk@pop-os:~$ ssh -p 5050 aadarsha@192.168.254.2
ssh: connect to host 192.168.254.2 port 5050: No route to host

aadarkdk@pop-os:~$ ssh -p 5050 root@192.168.254.2
ssh: connect to host 192.168.254.2 port 5050: No route to host
aadarkdk@pop-os:~$ 

 # After configuring firewall:
 
 # <firewall-cmd --permanent --add-port=5050/tcp
 
 # <firewall-cmd --reload

aadarkdk@pop-os:~$ ssh -p 5050 root@192.168.254.2
root@192.168.254.2's password: 
Last login: Mon Aug  3 18:52:50 2026

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 5050/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@labserver ~]# 

[root@labserver ~]# ss -tlnp | grep sshd
LISTEN 0      128          0.0.0.0:5050      0.0.0.0:*    users:(("sshd",pid=19856,fd=7))
LISTEN 0      128             [::]:5050         [::]:*    users:(("sshd",pid=19856,fd=8))
[root@labserver ~]# 

 # Preventing root login

[root@labserver ~]# vim /etc/ssh/sshd_config

[root@labserver ~]# cat /etc/ssh/sshd_config
...
#LoginGraceTime 2m
#PermitRootLogin prohibit-password
PermitRootLogin no
...

[root@labserver ~]# systemctl reload sshd

[root@labserver ~]# exit
logout
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ ssh -p 5050 root@192.168.254.2
root@192.168.254.2's password: 
Last login: Mon Aug  3 19:53:47 2026 from 192.168.254.32

[root@labserver ~]# sshd -T | grep permitrootlogin
permitrootlogin yes

[root@labserver ~]# grep -R "PermitRootLogin" /etc/ssh/sshd_config.d/
/etc/ssh/sshd_config.d/01-permitrootlogin.conf:PermitRootLogin yes
 
[root@labserver ~]# ls /etc/ssh/sshd_config.d/
01-permitrootlogin.conf  40-redhat-crypto-policies.conf  50-redhat.conf

[root@labserver ~]# vim /etc/ssh/sshd_config.d/01-permitrootlogin.conf 

[root@labserver ~]# cat /etc/ssh/sshd_config.d/01-permitrootlogin.conf 
# This file has been generated by the Anaconda Installer.
# Allow root to log in using ssh. Remove this file to opt-out.
# PermitRootLogin yes
PermitRootLogin no		# --> added line

[root@labserver ~]# systemctl reload sshd

[root@labserver ~]# exit
logout
Connection to 192.168.254.2 closed.

aadarkdk@pop-os:~$ ssh -p 5050 root@192.168.254.2
root@192.168.254.2's password: 
Permission denied, please try again.
root@192.168.254.2's password: 
Permission denied, please try again.
root@192.168.254.2's password: 
root@192.168.254.2: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
aadarkdk@pop-os:~$ 

 # But if after logging in from normal user, then user can switch to root user 
 
aadarkdk@pop-os:~$ ssh -p 5050 aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Mon Aug  3 13:57:55 2026 from 192.168.254.32

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Mon Aug  3 19:54:14 +0545 2026 from 192.168.254.32 on pts/0
Last failed login: Mon Aug  3 20:01:15 +0545 2026 from 192.168.254.32 on ssh:notty
There were 3 failed login attempts since the last successful login.
[root@labserver ~]# 

[root@labserver ~]# whoami
root

 # login shells

[root@labserver ~]# grep bash /etc/passwd
root:x:0:0:Super User:/root:/bin/bash
aadarsha:x:1000:1000:Aadarsha Khadka:/home/aadarsha:/bin/bash
milan:x:1001:1004::/home/milan:/bin/bash
suman:x:1002:1005::/home/suman:/bin/bash

 # /bin/bash, /bin/sh, /bin/csh, /bin/tcsh

[root@labserver ~]# ls /home/
aadarsha  milan  suman

[root@labserver ~]# useradd -r -s /sbin/nologin appuser1

[root@labserver ~]# ls /home/
aadarsha  milan  suman

[root@labserver ~]# cat /etc/passwd | grep appuser1
appuser1:x:994:994::/home/appuser1:/sbin/nologin

 # Allow only the specific users to login using SSH
 
[root@labserver ~]# vim /etc/ssh/sshd_config

[root@labserver ~]# cat /etc/ssh/sshd_config
...
# allow the following users for ssh login
AllowUsers suman aadarsha
...
[root@labserver ~]# 

[root@labserver ~]# systemctl restart sshd
Job for sshd.service failed because the control process exited with error code.
See "systemctl status sshd.service" and "journalctl -xeu sshd.service" for details.

[root@labserver ~]# systemctl reload sshd
sshd.service is not active, cannot reload.

[root@labserver ~]# sshd -t
/etc/ssh/sshd_config line 45: unsupported option "n".

[root@labserver ~]# vi /etc/ssh/sshd_config

[root@labserver ~]# systemctl reload sshd

[root@labserver ~]# ls /home/
aadarsha  milan  suman

[root@labserver ~]# exit
logout

[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.

aadarkdk@pop-os:~$ ssh -p 5050 aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Mon Aug  3 20:01:42 2026 from 192.168.254.32

[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.

aadarkdk@pop-os:~$ ssh -p 5050 suman@192.168.254.2
suman@192.168.254.2's password: 
Permission denied, please try again.
suman@192.168.254.2's password: 
Permission denied, please try again.
suman@192.168.254.2's password: 
suman@192.168.254.2: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ ssh milan@192.168.254.2
ssh: connect to host 192.168.254.2 port 22: Connection refused

aadarkdk@pop-os:~$ ssh -p 5050 aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Mon Aug  3 20:28:16 2026 from 192.168.254.32

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Mon Aug  3 20:01:51 +0545 2026 on pts/0

[root@labserver ~]# vim /etc/ssh/sshd_config

[root@labserver ~]# cat /etc/ssh/sshd_config
...
# Deny the following users for ssh login
DenyUsers suman aadarsha
...
[root@labserver ~]# 

[root@labserver ~]# vim /etc/ssh/sshd_config

[root@labserver ~]# systemctl restart sshd
``` 

 ### Configuring OpenSSH and Restricting SSH Access Using Firewalld Rich Rules
 
```
 # The SSH server is configured to listen on TCP port 5050. Firewalld rich rules are then used to  demonstrate two access control scenarios:

   # Allow SSH connections from an entire subnet (192.168.254.0/24)
   # Allow SSH connections from a single host (192.168.254.2)
 
 # In Client machine
 
[root@labserver ~]# rpm -q openssh-clients
openssh-clients-9.9p1-28.el10.x86_64

 # Now we can use: ssh, scp, sftp

[root@labserver ~]# which ssh
/usr/bin/ssh
[root@labserver ~]# which scp 
/usr/bin/scp
[root@labserver ~]# which sftp
/usr/bin/sftp

[root@labserver ~]# rpm -qf /usr/bin/scp
openssh-clients-9.9p1-28.el10.x86_64

 # In Server machine 

[root@labserver ~]# rpm -q openssh-server
openssh-server-9.9p1-28.el10.x86_64

 # Allow SSH Login from Selected IPs/Networks Only

 # machine2: Client (192.168.254.2)

[root@labserver ~]# hostname -I
192.168.254.2

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 5050/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@labserver ~]# 

[root@labserver ~]# man firewalld.richlanguage

 # machine1: Server (192.168.254.1)

[aadarsha@labserver ~]$ hostname -I
192.168.254.1 
 
[aadarsha@labserver ~]$ su - root
Password: 

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@labserver ~]# 

[aadarsha@labserver ~]$ man firewalld.richlanguage

[aadarsha@labserver ~]$ firewalld-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.254.2/24" to-port="5050" protocol="tcp" accept'
-bash: firewalld-cmd: command not found
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Mon Aug  3 22:10:55 +0545 2026 on pts/0
[root@labserver ~]# 
 
  # Allow SSH from an Entire Network
 
[root@labserver ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.254.2/24" port port="5050" protocol="tcp" accept'
success
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --reload
success

[root@labserver ~]# hostname -I
192.168.254.1 2407:5200:401:17e1:a00:27ff:fe36:47e4 
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --permanent --remove-service=ssh
success

[root@labserver ~]# firewall-cmd --reload
success

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.2/24" port port="5050" protocol="tcp" accept
[root@labserver ~]# 

[root@labserver ~]# ss -tlnp | grep sshd
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=900,fd=7))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=900,fd=8))

[root@labserver ~]# systemctl status sshd

[root@labserver ~]# grep ^Port /etc/ssh/sshd_config
[root@labserver ~]# 

[root@labserver ~]# vi /etc/ssh/sshd_config

[root@labserver ~]# systemctl restart sshd

[root@labserver ~]# sshd -t
[root@labserver ~]# 
[root@labserver ~]# systemctl status sshd
...

[root@labserver ~]# journalctl -u sshd -n 20 --no-pager
...

[root@labserver ~]# getenforce
Enforcing

[root@labserver ~]# semanage port -l | grep ssh
-bash: semanage: command not found
 
[root@labserver ~]# dnf install policycoreutils-python-utils
...
[root@labserver ~]# 
[root@labserver ~]# semanage port -l | grep ssh
ssh_port_t                     tcp      22

[root@labserver ~]# semanage port -a -t ssh_port_t -p tcp 5050
Port tcp/5050 already defined, modifying instead

[root@labserver ~]# semanage port -l | grep ssh
ssh_port_t                     tcp      5050, 22

[root@labserver ~]# systemctl restart sshd

[root@labserver ~]# systemctl status sshd

[root@labserver ~]# ss -tlnp | grep sshd
LISTEN 0      128          0.0.0.0:5050      0.0.0.0:*    users:(("sshd",pid=3484,fd=7))
LISTEN 0      128             [::]:5050         [::]:*    users:(("sshd",pid=3484,fd=8))

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.2/24" port port="5050" protocol="tcp" accept

[root@labserver ~]# hostname -I
192.168.254.1 

 # Login from authorized Client:

   # from host machine on both: 
  
aadarkdk@pop-os:~$ ssh -p 5050 aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Tue Aug  4 06:43:15 2026 from 192.168.254.32

[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ hostname -I
192.168.254.2 

[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$ 
 
aadarkdk@pop-os:~$ ssh -p 5050 aadarsha@192.168.254.1
aadarsha@192.168.254.1's password: 
Last login: Tue Aug  4 06:47:53 2026 from 192.168.254.32

[aadarsha@labserver ~]$ whoami
aadarsha
[aadarsha@labserver ~]$ hostname -I
192.168.254.1 

[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.1 closed.

aadarkdk@pop-os:~$ hostname -I
192.168.254.32 
aadarkdk@pop-os:~$ 

 # from client machine: 
 
[root@labserver ~]# hostname -I
192.168.254.2 2407:5200:401:17e1:a00:27ff:fec7:6ccb 

[root@labserver ~]# ssh -p 5050 aadarsha@192.168.254.1
The authenticity of host '[192.168.254.1]:5050 ([192.168.254.1]:5050)' can't be established.
ED25519 key fingerprint is SHA256:YEx3XBATYe8oAk1bquc9zLHIWFe8pFYnfXJZqQMVCaE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[192.168.254.1]:5050' (ED25519) to the list of known hosts.
aadarsha@192.168.254.1's password: 
Last login: Tue Aug  4 09:42:44 2026 from 192.168.254.32

[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ hostname -I
192.168.254.1 2407:5200:401:17e1:a00:27ff:fe36:47e4 

[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.1 closed.
 
[root@labserver ~]# hostname -I
192.168.254.2 2407:5200:401:17e1:a00:27ff:fec7:6ccb 
[root@labserver ~]# 

 # Allowing only specific host only:
 
[root@labserver ~]# hostname -I
192.168.254.1 
 
[root@labserver ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.254.2" port port="5050" protocol="tcp" accept'
success

[root@labserver ~]# firewall-cmd --reload
success

[root@labserver ~]# firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.254.2/24" port port="5050" protocol="tcp" accept'
success

[root@labserver ~]# firewall-cmd --reload
success

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.2" port port="5050" protocol="tcp" accept
[root@labserver ~]# 

 # from client

[root@labserver ~]# hostname -I
192.168.254.2 

[root@labserver ~]# ssh -p 5050 aadarsha@192.168.254.1
aadarsha@192.168.254.1's password: 
Last login: Tue Aug  4 09:43:39 2026 from 192.168.254.2

[aadarsha@labserver ~]$ hostname -I
192.168.254.1
 
[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.1 closed.
[root@labserver ~]# 
 
aadarkdk@pop-os:~$ hostname -I
192.168.254.32

 # from host
 
aadarkdk@pop-os:~$ ssh -p 5050 aadarsha@192.168.254.1
ssh: connect to host 192.168.254.1 port 5050: No route to host			# ( Denied)
aadarkdk@pop-os:~$ 

[root@labserver ~]# hostname -I
192.168.254.1
[root@labserver ~]# firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.254.2" port port="5050" protocol="tcp" accept'
success
[root@labserver ~]# firewall-cmd --reload
success

 # Note:
  # The notation 192.168.254.2/24 matches the entire 192.168.254.0/24 subnet, not just the host 192.168.254.2

   # Network address:   192.168.254.0
   # Usable host range: 192.168.254.1 – 192.168.254.254
   # Broadcast address: 192.168.254.255

  # As a result, hosts such as 192.168.254.2, 192.168.254.32, and 192.168.254.100 all match the rule.
  # To allow only a single host, specify 192.168.254.2 or 192.168.254.2/32.
```
---
```
 # Allow Only Key-based Authentication for SSH Login ( PasswordLess Login )
 
                            SSH Key-Based Login


           Machine A (Client)                  Machine B (Server)
         (Initiates SSH Login)                (Accepts SSH Login)

         +--------------------+             +---------------------------+
         | Generate SSH Keys  |             | ~/.ssh/authorized_keys    |
         | ssh-keygen         |             |                           |
         |                    |             |                           |
         | Private Key 🔒     |             | Public Key 🔑             |
         | Public Key 🔑      |------------>| (Copied from Client)      |
         +--------------------+             +---------------------------+
                    |                                   ^
                    |                                   |
                    +----------- ssh user@server -------+
                               (Passwordless Login)
                          
[root@labserver ~]# vi /etc/ssh/sshd_config
 
[root@labserver ~]# cat /etc/ssh/sshd_config
...
#Port 22
# port changed to:
Port 5050
...

# Allow the following users for ssh login
AllowUsers suman aadarsha
...
#PubkeyAuthentication yes
# uncommented
PubkeyAuthentication yes
...
# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
# changed to no
PasswordAuthentication no
...
[root@labserver ~]# 

 # trying to login using ssh password based login
 
aadarkdk@pop-os:~$ ssh -p 5050 suman@192.168.254.2
suman@192.168.254.2: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).

aadarkdk@pop-os:~$ ssh -p 5050 aadarsha@192.168.254.2
aadarsha@192.168.254.2: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
aadarkdk@pop-os:~$ 


 # Generating SSH Key Pair
 
[root@labserver ~]# whoami
root

[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ whoami
aadarsha

 # Generating SSH key from aadarsha user

[aadarsha@labserver ~]$ hostname
labserver

[aadarsha@labserver ~]$ pwd
/home/aadarsha

[aadarsha@labserver ~]$ hostname -I
192.168.254.2 

[aadarsha@labserver ~]$ ls -a
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .lesshst  

 # In previous versions

 # id_rsa      ---->  Private key
 # id_rsa.pub  ---->  Public key

[aadarsha@labserver ~]$ ls -a
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .lesshst  

[aadarsha@labserver ~]$ ssh-keygen 
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/aadarsha/.ssh/id_ed25519): 
Created directory '/home/aadarsha/.ssh'.
...
+--[ED25519 256]--+
|    ++o          |
|   o.+           |
| .E =            |
|+.+B..           |
|=*.++o  S        |
|=+=+++o.         |
|o++ =*.o.        |
|. .+o+o.o        |
|   .=++o         |
+----[SHA256]-----+

[aadarsha@labserver ~]$ ls -a
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .lesshst  .ssh

[aadarsha@labserver ~]$ cd .ssh/

[aadarsha@labserver .ssh]$ ls
id_ed25519  id_ed25519.pub

[aadarsha@labserver .ssh]$ cat id_ed25519
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----

[aadarsha@labserver .ssh]$ cat id_ed25519.pub 
ssh-ed25519 AAAAC3NzaC1lZVI1NTE5AAAAILpf4TxhdlQ45+/z4RJeI/l1Q7LwbPpccIjcYm8xA/0b aadarsha@labserver

 # Client

[root@labserver ~]# hostname
labserver
[root@labserver ~]# hostname -I
192.168.254.2

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 5050/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@labserver ~]#

[root@labserver ~]# ls /home/
aadarsha  milan  suman

[root@labserver ~]# su - aadarsha
Last login: Wed Aug  5 00:51:52 +0545 2026 from 192.168.254.32 on pts/0

[aadarsha@labserver ~]$ ls -a
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .lesshst  .ssh
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cd .ssh/
[aadarsha@labserver .ssh]$ ls
id_ed25519  id_ed25519.pub

[aadarsha@labserver .ssh]$ scp id_ed25519.pub aadarsha@192.168.254.1 /home/aadarsha/lab-key.pub
/home/aadarsha/lab-key.pub: No such file or directory
[aadarsha@labserver .ssh]$ 

[aadarsha@labserver .ssh]$ scp id_ed25519.pub aadarsha@192.168.254.1:/home/aadarsha/lab-key.pub
The authenticity of host '192.168.254.1 (192.168.254.1)' can't be established.
ED25519 key fingerprint is SHA256:YEx3XBATYe8oAk1bquc9zLHIWFe8pFYnfXJZqQMVCaE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.254.1' (ED25519) to the list of known hosts.
aadarsha@192.168.254.1's password: 
id_ed25519.pub                                             100%  100    73.1KB/s   00:00    
[aadarsha@labserver .ssh]$ 
 
[aadarsha@labserver .ssh]$ # ssh-copy-id -p 22 aadarsha@192.168.254.1
 
[aadarsha@labserver .ssh]$ cd
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ hostname
labserver

[aadarsha@labserver ~]$ ssh -p 5050 -i /home/aadarsha/.ssh/id_ed25519.pub aadarsha@192.168.254.1
ssh: connect to host 192.168.254.1 port 5050: No route to host

[aadarsha@labserver ~]$ ssh -p 22 -i /home/aadarsha/.ssh/id_ed25519.pub aadarsha@192.168.254.1
aadarsha@192.168.254.1's password: 
Last login: Wed Aug  5 01:20:33 2026

[aadarsha@mainserver ~]$ hostname
mainserver

[aadarsha@mainserver ~]$ hostname -I
192.168.254.1 
[aadarsha@mainserver ~]$ exit
logout
Connection to 192.168.254.1 closed.

[aadarsha@labserver ~]$ ssh -p 22 -i /home/aadarsha/.ssh/id_ed25519.pub aadarsha@192.168.254.1
aadarsha@192.168.254.1's password: 
Last login: Wed Aug  5 01:36:46 2026 from 192.168.254.2
[aadarsha@mainserver ~]$ 

[aadarsha@mainserver ~]$ exit
logout
Connection to 192.168.254.1 closed.

[aadarsha@labserver ~]$ ssh -p 22 -i /home/aadarsha/.ssh/id_ed25519.pub aadarsha@192.168.254.1
aadarsha@192.168.254.1: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
[aadarsha@labserver ~]$ 
 
[aadarsha@labserver ~]$ ssh -p 22 -i /home/aadarsha/.ssh/id_ed25519 aadarsha@192.168.254.1
Last login: Wed Aug  5 01:39:05 2026 from 192.168.254.2
[aadarsha@mainserver ~]$
 
[aadarsha@mainserver ~]$ hostname
mainserver

[aadarsha@mainserver ~]$ hostname -I
192.168.254.1 
[aadarsha@mainserver ~]$ exit
logout
Connection to 192.168.254.1 closed.
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ hostname
labserver

[aadarsha@labserver ~]$ hostname -I
192.168.254.2 
 
[aadarsha@labserver ~]$ ssh aadarsha@192.168.254.1
Last login: Wed Aug  5 01:51:21 2026 from 192.168.254.2

[aadarsha@mainserver ~]$ exit
logout
Connection to 192.168.254.1 closed.

[aadarsha@labserver ~]$ ssh root@192.168.254.1
root@192.168.254.1: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
[aadarsha@labserver ~]$ 

 # Server

[root@mainserver ~]# hostname
mainserver
[root@mainserver ~]# hostname -I
192.168.254.1 

[root@mainserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@mainserver ~]# 
[root@mainserver ~]# ls /home/
aadarsha
[root@mainserver ~]# 

[root@mainserver ~]# su - aadarsha
Last login: Wed Aug  5 01:19:55 +0545 2026 on pts/0

[aadarsha@mainserver ~]$ ls -a
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .lesshst
[aadarsha@mainserver ~]$ pwd
/home/aadarsha

[aadarsha@mainserver ~]$ ls
lab-key.pub
 
[aadarsha@mainserver ~]$ ls -l
total 4
-rw-r--r--. 1 aadarsha aadarsha 100 Aug  5 01:23 lab-key.pub
[aadarsha@mainserver ~]$ 
[aadarsha@mainserver ~]$ ssh -p 5050 -i lab-key.pub aadarsha@192.168.254.2
The authenticity of host '[192.168.254.2]:5050 ([192.168.254.2]:5050)' can't be established.
ED25519 key fingerprint is SHA256:YEx3XBATYe8oAk1bquc9zLHIWFe8pFYnfXJZqQMVCaE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[192.168.254.2]:5050' (ED25519) to the list of known hosts.
aadarsha@192.168.254.2's password: 
Last login: Wed Aug  5 01:07:59 2026
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ hostname
labserver

[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.
[aadarsha@mainserver ~]$
 
[aadarsha@mainserver ~]$ hostname
mainserver

[aadarsha@mainserver ~]$ vi /etc/ssh/sshd_config

[aadarsha@mainserver ~]$ su - root
Password: 
Last login: Wed Aug  5 01:05:49 +0545 2026 on pts/0

[root@mainserver ~]# vi /etc/ssh/sshd_config

[root@mainserver ~]# cat /etc/ssh/sshd_config
...
# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.
...
# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change.
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
...
#PubkeyAuthentication yes
# un-commented
PubkeyAuthentication yes
...
# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
# yes --> no (don't allow password based authentication)
PasswordAuthentication no
#PermitEmptyPasswords no
...
[root@mainserver ~]# 

[root@mainserver ~]# systemctl restart sshd

[root@mainserver ~]# exit
logout

[aadarsha@mainserver ~]$ ls -a
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  lab-key.pub  .lesshst  .ssh
 
[aadarsha@mainserver ~]$ ls -la .ssh/
total 8
drwx------. 2 aadarsha aadarsha  48 Aug  5 01:27 .
drwx------. 3 aadarsha aadarsha 130 Aug  5 01:27 ..
-rw-------. 1 aadarsha aadarsha 858 Aug  5 01:27 known_hosts
-rw-r--r--. 1 aadarsha aadarsha 102 Aug  5 01:27 known_hosts.old

[aadarsha@mainserver ~]$ ls -ld .ssh/
drwx------. 2 aadarsha aadarsha 48 Aug  5 01:27 .ssh/

[aadarsha@mainserver ~]$ cat lab-key.pub >> .ssh/authorized_keys

[aadarsha@mainserver ~]$ ls .ssh/
authorized_keys  known_hosts  known_hosts.old

[aadarsha@mainserver ~]$ ls -l .ssh/authorized_keys 
-rw-r--r--. 1 aadarsha aadarsha 100 Aug  5 01:48 .ssh/authorized_keys

[aadarsha@mainserver ~]$ chmod 600 .ssh/authorized_keys 

[aadarsha@mainserver ~]$ ls -l .ssh/authorized_keys 
-rw-------. 1 aadarsha aadarsha 100 Aug  5 01:48 .ssh/authorized_keys

[aadarsha@mainserver ~]$ rm lab-key.pub 
[aadarsha@mainserver ~]$ 
```
```                        
                           Generate SSH Key Pair
                     (On the machine initiating SSH)

                          +----------------------+
                          |   Machine A          |
                          |   (Client/Admin)     |
                          |----------------------|
                          | Private Key          |
                          | Public Key           |
                          +----------+-----------+
                                     |
                Copy ONLY Public Key |
                                     |
                     +---------------+----------------+
                     |                                |
                     v                                v
          +----------------------+         +----------------------+
          |   Machine B          |         |   Machine C          |
          |   (Server 1)         |         |   (Server 2)         |
          |----------------------|         |----------------------|
          | ~/.ssh/              |         | ~/.ssh/              |
          | authorized_keys      |         | authorized_keys      |
          | (Public Key)         |         | (Public Key)         |
          +----------------------+         +----------------------+

                   SSH Login                     SSH Login
          Machine A  --------->  Machine B
          Machine A  --------->  Machine C
```
#### Authentication Process
```
        	                         SSH Login

	+---------------------------+                   +---------------------------+
	|     Machine A (Client)    |                   |     Machine B (Server)    |
	|---------------------------|                   |---------------------------|
	| Private Key               |                   | authorized_keys           |
	| id_ed25519                |                   | contains Public Key       |
	+-------------+-------------+                   +-------------+-------------+
	              |                                               ^
	              | 1. ssh user@server                            |
	              |---------------------------------------------->|
	              |                                               |
	              | 2. Server sends a challenge                   |
	              |<----------------------------------------------|
	              |                                               |
	              | 3. Client signs challenge                     |
	              |    using PRIVATE KEY                          |
	              |---------------------------------------------->|
	              |                                               |
	              | 4. Server verifies signature                  |
	              |    using PUBLIC KEY                           |
	              |                                               |
	              |<----------- Login Successful -----------------|


 # Generate Key Pair  --->  Client Machine

 # Private Key        --->  NEVER leaves the Client

 # Public Key         --->  Copy to Server

 # Server             --->  Stores Public Key in ~/.ssh/authorized_keys

 # SSH Login          --->  Client proves ownership of the Private Key
                            Server verifies using the Public Key
                            No password required
```

 #### Best Practices
 - Generate the SSH key pair only on the client machine (Machine A).
 - Copy only the public key (id_ed25519.pub) to each server's ~/.ssh/authorized_keys.
 - Never copy or share the private key (id_ed25519).
 - Use one key pair per administrator/user for better auditing and access management.
 - Set correct permissions:
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
 - Disable password authentication after verifying SSH key login (if required by your security policy).
 - Rotate SSH keys periodically and remove unused public keys from authorized_keys.

Remember: The private key never leaves the client machine. Only the public key is copied to the server's ~/.ssh/authorized_keys file.

## Summary note: 

### SSH Key-Based Authentication: Quick Reference

#### 1. Authentication Flow
1. **Client** generates a cryptographic key pair (private key and public key).
2. **Client** copies the public key to the **Server** (`~/.ssh/authorized_keys`).
3. **Client** initiates login using the private key.
4. **Server** issues a cryptographic challenge; **Client** signs it with the private key; **Server** verifies it against the stored public key.

#### 2. Standard Permissions Matrix
* `~/.ssh/` (Directory): `700` (`drwx------`)
* Private Key (Client): `600` (`-rw-------`)
* `~/.ssh/authorized_keys` (Server): `600` (`-rw-------`)
* Public Key (Client/Server): `644` (`-rw-r--r--`)

#### 3. Core Commands
* **Generate Key Pair (Client):** 
  `ssh-keygen -t ed25519`
* **Automated Setup (Client):** 
  `ssh-copy-id -i ~/.ssh/private_key.pub user@server_ip`
* **Manual Setup (Server):** 
  `mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat ~/public_key.pub >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys`
* **Connect (Client):** 
  `ssh -i ~/.ssh/private_key user@server_ip`

#### 4. Required Server Configuration (`/etc/ssh/sshd_config`)
```ini
PubkeyAuthentication yes
PasswordAuthentication no
```
- Run `systemctl restart sshd` after modifying.

---
 ### Secured File Transfer using scp 
```
  # machine1: labserver (192.168.254.2)

[aadarsha@labserver ~]$ hostname
labserver
[aadarsha@labserver ~]$ hostname -I
192.168.254.2 

  # File Transfer: Local --> Remote

[aadarsha@labserver ~]$ ls
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ vi labserver-data
[aadarsha@labserver ~]$ ls
labserver-data
[aadarsha@labserver ~]$ cat labserver-data 
this is labserver's data...
this file contain the confidential data...
 
[aadarsha@labserver ~]$ scp labserver-data aadarsha@192.168.254.1:/home/aadarsha/labserver-file
labserver-data                                             100%   71    62.3KB/s   00:00    
[aadarsha@labserver ~]$ 

 # File Transfer: Local <-- Remote

[aadarsha@labserver ~]$ ls
labserver-data

[aadarsha@labserver ~]$ pwd
/home/aadarsha

[aadarsha@labserver ~]$ # scp -r -P 22 aadarsha@192.168.254.1:/home/aadarsha/mainserver-dir .

[aadarsha@labserver ~]$ scp -r aadarsha@192.168.254.1:/home/aadarsha/mainserver-dir .
main-server-file                                           100%   38    27.1KB/s   00:00    

[aadarsha@labserver ~]$ ls
labserver-data  mainserver-dir
 
[aadarsha@labserver ~]$ ls mainserver-dir/
dir1  dir2  file1  main-server-file

[aadarsha@labserver ~]$ cat mainserver-dir/main-server-file 
this file is created on mainserver...
 
  # machine 2: mainserver (192.168.254.1 )

[aadarsha@mainserver ~]$ hostname -I
192.168.254.1 

[aadarsha@mainserver ~]$ hostname
mainserver

[aadarsha@mainserver ~]$ ls
[aadarsha@mainserver ~]$ 

[aadarsha@mainserver ~]$ su - root
...
[root@mainserver ~]# 

[root@mainserver ~]# systemctl status sshd
...
[root@mainserver ~]# systemctl start firewalld
 
[root@mainserver ~]# systemctl enable firewalld
 
[root@mainserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@mainserver ~]# 

[root@mainserver ~]# exit
logout

[aadarsha@mainserver ~]$ ls
[aadarsha@mainserver ~]$ pwd
/home/aadarsha

[aadarsha@mainserver ~]$ ls
labserver-file
[aadarsha@mainserver ~]$ cat labserver-file 
this is labserver's data...
this file contain the confidential data...

[aadarsha@mainserver ~]$ ls
labserver-file

[aadarsha@mainserver ~]$ mkdir mainserver-dir
[aadarsha@mainserver ~]$ cd mainserver-dir/

[aadarsha@mainserver mainserver-dir]$ touch file1 main-server-file
[aadarsha@mainserver mainserver-dir]$ ls
file1  main-server-file

[aadarsha@mainserver mainserver-dir]$ vi main-server-file 
[aadarsha@mainserver mainserver-dir]$ cd

[aadarsha@mainserver ~]$ ls
labserver-file  mainserver-dir

[aadarsha@mainserver ~]$ cd mainserver-dir/
[aadarsha@mainserver mainserver-dir]$ ls
file1  main-server-file
[aadarsha@mainserver mainserver-dir]$ mkdir dir1 dir2
[aadarsha@mainserver mainserver-dir]$ ls
dir1  dir2  file1  main-server-file
 
[aadarsha@mainserver mainserver-dir]$ cd
[aadarsha@mainserver ~]$ 
```
---
 ### Taking Network Backup
```
  # Main Server
 
[aadarsha@mainserver ~]$ hostname
mainserver

[aadarsha@mainserver ~]$ hostname -I
192.168.254.1

[aadarsha@mainserver ~]$ pwd
/home/aadarsha

[root@mainserver ~]# hostname
mainserver

[root@mainserver ~]# ls /home/
aadarsha

[root@mainserver ~]# rsync --rsh=ssh -r /home/ aadarsha@192.168.254.2:/home/aadarsha/backup
The authenticity of host '192.168.254.2 (192.168.254.2)' can't be established.
ED25519 key fingerprint is SHA256:YEx3XBATYe8oAk1bquc9zLHIWFe8pFYnfXJZqQMVCaE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.254.2' (ED25519) to the list of known hosts.
aadarsha@192.168.254.2's password: 
[root@mainserver ~]# 

[root@mainserver ~]# cd /home/
[root@mainserver home]# ls
aadarsha

[root@mainserver home]# vi mainserver-data

[root@mainserver home]# ls
aadarsha  mainserver-data

[root@mainserver home]# useradd newuser1

[root@mainserver home]# ls
aadarsha  mainserver-data  newuser1

[root@mainserver home]# cat mainserver-data 
this is the data from the main server...

[root@mainserver home]# rsync --rsh=ssh -r /home/ aadarsha@192.168.254.2:/home/aadarsha/backup
aadarsha@192.168.254.2's password: 
 
[root@mainserver home]# vi mainserver-data
 
[root@mainserver home]# cat mainserver-data
this is the data from the main server...
...more data about the main server is added 

[root@mainserver home]# rsync --rsh=ssh -r /home/ aadarsha@192.168.254.2:/home/aadarsha/backup
aadarsha@192.168.254.2's password: 
[root@mainserver home]# 
 
[root@mainserver home]# cd
[root@mainserver ~]# 
[root@mainserver ~]# ls -a
.   anaconda-ks.cfg  .bash_logout   .bashrc  .ssh
..  .bash_history    .bash_profile  .cshrc   .tcshrc
[root@mainserver ~]# 
[root@mainserver ~]# ls .ssh/
known_hosts  known_hosts.old
[root@mainserver ~]# 
[root@mainserver ~]# ssh-keygen
Generating public/private ed25519 key pair.
Enter file in which to save the key (/root/.ssh/id_ed25519): 
Enter passphrase for "/root/.ssh/id_ed25519" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_ed25519
Your public key has been saved in /root/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:Xb1ikv02WHJNqKtCGoisHQjx5ik+C3lvNEOKV0KRYBw root@mainserver
The key's randomart image is:
+--[ED25519 256]--+
|oEoo             |
|o.o          . . |
| +          . o .|
|. + o    . + . + |
|.* B .  S + * + .|
|+.O = . .  o O   |
|+=.o o +    o +  |
|o+... . .  . . . |
| .o..    ..      |
+----[SHA256]-----+
[root@mainserver ~]# 

[root@mainserver ~]# ls -a .ssh/
.  ..  id_ed25519  id_ed25519.pub  known_hosts  known_hosts.old
[root@mainserver ~]# 
 
[root@mainserver ~]# ssh-copy-id aadarsha@192.168.254.2
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
aadarsha@192.168.254.2's password: 

Number of key(s) added: 1

Now try logging into the machine, with: "ssh 'aadarsha@192.168.254.2'"
and check to make sure that only the key(s) you wanted were added.

[root@mainserver ~]# 

[root@mainserver ~]# ssh aadarsha@192.168.254.2
Last login: Wed Aug  5 19:35:32 2026 from 192.168.254.32

[aadarsha@backup ~]$ whoami
aadarsha

[aadarsha@backup ~]$ hostname
backup

[aadarsha@backup ~]$ exit
logout
Connection to 192.168.254.2 closed.

[root@mainserver ~]# hostname
mainserver

[root@mainserver ~]# ls
anaconda-ks.cfg
[root@mainserver ~]# ls /home/
aadarsha  mainserver-data  newuser1

[root@mainserver ~]# cd /home/
[root@mainserver home]# ls
aadarsha  mainserver-data  newuser1

[root@mainserver home]# touch newfile1
 
[root@mainserver home]# ls
aadarsha  mainserver-data  newfile1  newuser1

[root@mainserver home]# rsync --rsh=ssh -r /home/ aadarsha@192.168.254.2:/home/aadarsha/backup

[root@mainserver home]# rsync --rsh=ssh -r /home/ aadarsha@192.168.254.2:/home/aadarsha/backup
[root@mainserver home]# ls
aadarsha  mainserver-data  newfile1  newuser1

[root@mainserver home]# rsync -r -e "ssh -p 5050" /home/ aadarsha@192.168.254.2:/home/aadarsha/backup

[root@mainserver home]# ls
aadarsha  mainserver-data  newfile1  newuser1

[root@mainserver home]# useradd user2
[root@mainserver home]# ls
aadarsha  mainserver-data  newfile1  newuser1  user2

[root@mainserver home]# rsync -r -e "ssh -p 5050" /home/ aadarsha@192.168.254.2:/home/aadarsha/backup

[root@mainserver home]# userdel -r newuser1

[root@mainserver home]# useradd user1
 
[root@mainserver home]# rsync -az -e "ssh -p 5050" /home/ aadarsha@192.168.254.2:/home/aadarsha/backup

[root@mainserver home]# ls 
aadarsha  mainserver-data  newfile1  user1  user2

 # deleted files is not detected.

[root@mainserver home]# rsync --delete -az -e "ssh -p 5050" /home/ aadarsha@192.168.254.2:/home/aadarsha/backup

[root@mainserver home]# ls
aadarsha  mainserver-data  newfile1  user1  user2

[root@mainserver home]# userdel -r user1
[root@mainserver home]# userdel -r user2
 
[root@mainserver home]# rsync --delete -az -e "ssh -p 5050" /home/ aadarsha@192.168.254.2:/home/aadarsha/backup

 # adding crontab for regular automatic backup

[root@mainserver home]# crontab -e
no crontab for root - using an empty one
crontab: installing new crontab
"/tmp/crontab.qLVA1O":2: bad command
Invalid crontab file, can't install.
Do you want to retry the same edit? (Y/N) n
crontab: edits left in /tmp/crontab.qLVA1O
[root@mainserver home]# 

[root@mainserver home]# ls
aadarsha  mainserver-data  newfile1

[root@mainserver home]# cd
[root@mainserver ~]# 
 
[root@mainserver ~]# vi network-backup.sh

[root@mainserver ~]# ls
anaconda-ks.cfg  network-backup.sh

[root@mainserver ~]# pwd
/root

[root@mainserver ~]# crontab -e
no crontab for root - using an empty one
crontab: installing new crontab

[root@mainserver ~]# crontab -l
*/1   *   *   *   *  /root/network-backup.sh
[root@mainserver ~]# 

[root@mainserver ~]# chmod +x network-backup.sh 

[root@mainserver ~]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak

[root@mainserver ~]# crontab -l
*   *   *   *   *  /root/network-backup.sh
 
[root@mainserver ~]# cd /home/
[root@mainserver home]# ls
aadarsha  mainserver-data  newfile1
 
[root@mainserver home]# touch file2
 
[root@mainserver home]# useradd usera

[root@mainserver home]# useradd userb

[root@mainserver home]# ls 
aadarsha  file2  mainserver-data  newfile1  usera  userb

[root@mainserver home]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak
[root@mainserver home]# 

  # Backup Server

[aadarsha@labserver ~]$ hostname
labserver
[aadarsha@labserver ~]$ hostname -I
192.168.254.2 
 
[aadarsha@labserver ~]$ pwd
/home/aadarsha
 
[aadarsha@labserver ~]$ ls
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ mkdir backup

[aadarsha@labserver ~]$ ls
backup
[aadarsha@labserver ~]$ ls backup/
aadarsha
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls backup/
aadarsha  mainserver-data  newuser1
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ cat backup/mainserver-data 
this is the data from the main server...
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls backup/
aadarsha  mainserver-data  newuser1

[aadarsha@labserver ~]$ cat backup/mainserver-data 
this is the data from the main server...
...more data about the main server is added 
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ hostnamectl set hostname backup
Unknown command verb 'set', did you mean 'set-chassis'?
[aadarsha@labserver ~]$ hostnamectl set-hostname backup
Failed to execute /usr/bin/pkttyagent: No such file or directory
Could not set static hostname: Access denied
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ sudo hostnamectl set-hostname backup
[sudo] password for aadarsha: 

[aadarsha@labserver ~]$ hostname
backup

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Wed Aug  5 19:36:02 +0545 2026 on pts/0
[root@backup ~]# 
[root@backup ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 5050/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@backup ~]# 

[root@backup ~]# exit
logout
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ ls /backup/
6051512a1e194d8fa01bc1948bcb0427  dir3      journal   marketing  production  utempter
backup                            eventlog  keystore  newdir1    sales       write
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ cd /ls backup/
-bash: cd: too many arguments
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ ls backup/
aadarsha  mainserver-data  newuser1

[aadarsha@labserver ~]$ ls backup/
aadarsha  mainserver-data  newfile1  newuser1
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ su - root
Password: 
su: Authentication failure
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ su - root
Password: 
Last login: Wed Aug  5 21:05:53 +0545 2026 on pts/0
Last failed login: Wed Aug  5 21:15:52 +0545 2026 on pts/0
There was 1 failed login attempt since the last successful login.
[root@backup ~]# 

[root@backup ~]# vi /etc/ssh/sshd_config

[root@backup ~]# # added: Port 5050

[root@backup ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4 source address="192.168.5.172" port port="5050" protocol="tcp" accept'
Error: No closing quotation
[root@backup ~]# 
[root@backup ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.5.172" port port="5050" protocol="tcp" accept'
success
[root@backup ~]# 
[root@backup ~]# firewall-cmd --reload
success
[root@backup ~]# 
[root@backup ~]# firewall-cmd --list-rich-rules
rule family="ipv4" source address="192.168.5.172" port port="5050" protocol="tcp" accept
[root@backup ~]# 
[root@backup ~]# firewall-cmd --permanent --remove-service=ssh
success
[root@backup ~]# 
[root@backup ~]# firewall-cmd --reload
success
[root@backup ~]# 
[root@backup ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client
  ports: 5050/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.5.172" port port="5050" protocol="tcp" accept
[root@backup ~]# 

[root@backup ~]# vi /etc/ssh/sshd_config

[root@backup ~]# semanage port -a -t ssh_port_t -p tcp 5050
Port tcp/5050 already defined, modifying instead

[root@backup ~]# getenforce 
Enforcing

[root@backup ~]# semanage port -a -t ssh_port_t -p tcp 5050
Port tcp/5050 already defined, modifying instead
[root@backup ~]# 

[root@backup ~]# systemctl restart sshd

[root@backup ~]# netstat -tnl | grep 5050
tcp        0      0 0.0.0.0:5050            0.0.0.0:*               LISTEN     
tcp6       0      0 :::5050                 :::*                    LISTEN     
[root@backup ~]# 

[root@backup ~]# ls /home/aadarsha/backup/
aadarsha  mainserver-data  newfile1  newuser1  user2

[root@backup ~]# exit
logout
[aadarsha@labserver ~]$ ls
backup

[aadarsha@labserver ~]$ pwd
/home/aadarsha
[aadarsha@labserver ~]$ ls backup/
aadarsha  mainserver-data  newfile1  newuser1  user2

[aadarsha@labserver ~]$ ls backup/
aadarsha  mainserver-data  newfile1  newuser1  user1  user2
[aadarsha@labserver ~]$ 

 # deleted files is not deleted.
 
[aadarsha@labserver ~]$ ls backup/
aadarsha  mainserver-data  newfile1  user1  user2

[aadarsha@labserver ~]$ ls backup/
aadarsha  mainserver-data  newfile1

[aadarsha@labserver ~]$ ls backup/
aadarsha  mainserver-data  newfile1

[aadarsha@labserver ~]$ ls backup/
aadarsha  file2  mainserver-data  newfile1

[aadarsha@labserver ~]$ ls backup/
aadarsha  file2  mainserver-data  newfile1  usera  userb

[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$ 