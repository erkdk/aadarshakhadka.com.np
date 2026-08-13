---
title: "PL - 022 — Network Time Protocol (NTP) and Chrony Implementation"
date: 2026-08-13
draft: false
---
## 1. Network Time Protocol (NTP)

NTP is the standard protocol for synchronizing computer clocks over packet-switched networks to Coordinated Universal Time (UTC).

*   **Standard:** RFC 5905 (NTPv4).
*   **Protocol:** UDP 123.
*   **Time Basis:** UTC. Local time zones and DST are managed at the OS/Application layer.
*  **Stratum:** 
    A hierarchical level in the Network Time Protocol (NTP) that indicates how close a device is to an authoritative, high-precision time source.
*   **Stratum Hierarchy:**
    *   **Stratum 0:** Atomic/GPS hardware (not network-attached).
    *   **Stratum 1:** Servers directly connected to Stratum 0 devices.
    *   **Stratum 2:** Servers that sync via network with Stratum 1.
    *   **Stratum 3–15:** Downstream servers/clients.
    *   **Stratum 16:** Unsynchronized/Unreachable state.

### Security
*   **NTS (Network Time Security):** Uses TLS and AEAD (RFC 8915) to prevent spoofing and MitM attacks.
*   **Symmetric Keys:** Legacy authentication using pre-shared keys (SHA-256, etc.).

### Implementations
*   **`chrony`:** Optimized for modern environments (virtualization, intermittent connectivity).
*   **`ntpd`:** Legacy reference implementation for complex routing.
---

### Terminal Sessions
```
  # Configuring NTP Server
 
[root@labserver ~]# rpm -q chrony
chrony-4.8-2.el10.x86_64

[root@labserver ~]# systemctl status chronyd
● chronyd.service - NTP client/server
     Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-08-13 05:06:21 +0545; 36min ago
 Invocation: b51c5753d3184106adafe3c653dee688
       Docs: man:chronyd(8)
             man:chrony.conf(5)
   Main PID: 904 (chronyd)
      Tasks: 1 (limit: 10630)
     Memory: 8.3M (peak: 8.8M)
        CPU: 81ms
     CGroup: /system.slice/chronyd.service
             └─904 /usr/sbin/chronyd -n -F 2
...
[root@labserver ~]# 

  # after modifying /etc/chrony.conf

[root@labserver ~]# vim /etc/chrony.conf 

[root@labserver ~]# cat /etc/chrony.conf 
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (https://www.pool.ntp.org/join.html).
pool 2.centos.pool.ntp.org iburst
# added
pool 1.centos.pool.ntp.org iburst
pool 3.centos.pool.ntp.org iburst
...
# Allow NTP client access from local network.
#allow 192.168.0.0/16
#added--> allow the different network ( eg. department network)
allow 192.168.254.0/24
# allow 192.168.254.0/24        
...
# Serve time even if not synchronized to a time source.
# uncommented --> if above ntp server fails, use local time from BIOS
local stratum 10
...
[root@labserver ~]# 

[root@labserver ~]# systemctl start chronyd
[root@labserver ~]# systemctl is-active chronyd
active

[root@labserver ~]# systemctl is-enabled chronyd
enabled

[root@labserver ~]# systemctl restart chronyd

[root@labserver ~]# systemctl is-active chronyd
active

[root@labserver ~]# systemctl is-enabled chronyd
enabled

[root@labserver ~]# firewall-cmd --permanent --add-service=ntp
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
  services: cockpit dhcpv6-client http mountd nfs ntp rpc-bind ssh
  ports: 5050/tcp 8080/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@labserver ~]# 

 # adding rich rule

[root@labserver ~]# man firewalld.richlanguage

[root@labserver ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.254.0/24" service name="ntp" accept'
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
  services: cockpit dhcpv6-client http mountd nfs ntp rpc-bind ssh
  ports: 5050/tcp 8080/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.0/24" service name="ntp" accept
 
[root@labserver ~]# firewall-cmd --permanent --remove-service=ntp
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
  services: cockpit dhcpv6-client http mountd nfs rpc-bind ssh
  ports: 5050/tcp 8080/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.0/24" service name="ntp" accept
[root@labserver ~]# 
```
### Verification & Diagnostics

| Task | Command |
| :--- | :--- |
| Check Connected Clients (Server) | `chronyc clients` |
| Verify Sync Sources (Client) | `chronyc sources -v` |
| View Sync Tracking | `chronyc tracking` |
| Check System Time Status | `timedatectl` |
| Force Time Step (Immediate) | `chronyc makestep` |

---
``` 
 # Configuring NTP Client (Synchronize Time with the NTP Server Over the Network)
 
[root@client1 ~]# hostname
client1
 
[root@client1 ~]# rpm -q chrony
chrony-4.8-2.el10.x86_64

[root@client1 ~]# systemctl start chronyd

[root@client1 ~]# systemctl is-active chronyd
active

[root@client1 ~]# systemctl is-enabled chronyd
enabled

[root@client1 ~]# vi /etc/chrony.conf 

[root@client1 ~]# cat /etc/chrony.conf 
...
# pool 2.centos.pool.ntp.org iburst
  server 192.168.254.2 iburst
...
[root@client1 ~]# 

[root@client1 ~]# systemctl restart chronyd

[root@client1 ~]# systemctl status chronyd
● chronyd.service - NTP client/server
     Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-08-13 13:34:11 +0545; 6s ago
 Invocation: 0b9937e673bc43d6b65d3738558d0116
       Docs: man:chronyd(8)
             man:chrony.conf(5)
   Main PID: 3161 (chronyd)
      Tasks: 1 (limit: 10630)
     Memory: 1M (peak: 2.9M)
        CPU: 23ms
     CGroup: /system.slice/chronyd.service
             └─3161 /usr/sbin/chronyd -n -F 2
...
[root@client1 ~]#

 # server

[root@labserver ~]# hostname -I
192.168.254.2 
[root@labserver ~]# chronyc clients
Hostname                      NTP   Drop Int IntL Last     Cmd   Drop Int  Last
===============================================================================
192.168.254.3                  11      0   6   -    26       0      0   -     -
[root@labserver ~]#

 # client/s
 
 [root@client1 ~]# chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* 192.168.254.2                 4   6    17    53    -12us[ -109us] +/-   86ms
[root@client1 ~]# 

[root@client1 ~]# timedatectl
               Local time: Thu 2026-08-13 13:37:10 +0545
           Universal time: Thu 2026-08-13 07:52:10 UTC
                 RTC time: Thu 2026-08-13 07:52:10
                Time zone: Asia/Kathmandu (+0545, +0545)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
[root@client1 ~]# 

 # Testing time synchronization with NTP server by changing the date and time on client

[root@client1 ~]# date --set=2001-11-01
Thu Nov  1 12:00:00 AM +0545 2001

[root@client1 ~]# date
Thu Nov  1 12:00:05 AM +0545 2001

[root@client1 ~]# date
Thu Nov  1 12:02:54 AM +0545 2001
 
[root@client1 ~]# systemctl restart chronyd
 
[root@client1 ~]# date
Thu Nov  1 12:03:00 AM +0545 2001
 
[root@client1 ~]# chronyc makestep
200 OK

[root@client1 ~]# date
Thu Aug 13 01:42:24 PM +0545 2026

[root@client1 ~]# chronyc tracking
Reference ID    : C0A8FE02 (192.168.254.2)
Stratum         : 5
Ref time (UTC)  : Thu Aug 13 07:56:49 2026
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +0.000017753 seconds
RMS offset      : 0.000017753 seconds
Frequency       : 1.516 ppm fast
Residual freq   : +3.910 ppm
Skew            : 2.462 ppm
Root delay      : 0.164002657 seconds
Root dispersion : 0.003660986 seconds
Update interval : 2.0 seconds
Leap status     : Normal
[root@client1 ~]#
 
[root@client1 ~]# timedatectl
               Local time: Thu 2026-08-13 13:42:59 +0545
           Universal time: Thu 2026-08-13 07:57:59 UTC
                 RTC time: Thu 2026-08-13 07:57:59
                Time zone: Asia/Kathmandu (+0545, +0545)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
[root@client1 ~]# 

[root@client1 ~]# chronyc sources -v

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* 192.168.254.2                 4   6    37    54  -3085ns[  -66us] +/-   85ms
[root@client1 ~]#

[root@client1 ~]# exit
logout
[aadarkdk@client1 ~]$ exit
logout
Connection to 192.168.254.3 closed.
aadarkdk@pop-os:~$ 
```
  

