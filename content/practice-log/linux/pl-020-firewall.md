---
title: "PL - 020 — Firewall & Network Security Management"
date: 2026-08-22
draft: false
---

### Firewall:
A security control that permits, denies, or logs network traffic according
to defined rules/policy.

**Primary objective:**
- Allow required communication.
- Block unnecessary or unauthorized communication.

**Core principles:**
- Default deny: Deny unless explicitly required.
- Least privilege: Allow only the minimum required access.
- Specificity: Restrict source, destination, protocol, port, interface, direction.
- Segmentation: Separate systems according to trust/function.
- Stateful filtering: Track connection state.
- Ingress + egress: Consider both inbound and outbound traffic.
- Visibility: Log important traffic appropriately.
- Persistence: Ensure intended rules survive reload/restart.
- Testing: Test both allowed and denied traffic.
- Documentation: Record purpose, owner, and change reference.
- Recovery: Protect management access before remote firewall changes.

### Default policy:
```
    DENY → unless explicitly allowed
```
Example:
```
    Internet → Server:443/TCP → ALLOW
    Internet → Server:22/TCP  → DENY
```
Avoid:
```
    ANY → ANY → ANY → ALLOW
```

### Least Privilege
Restrict traffic to the narrowest practical:
```
    Source
    Destination
    Protocol
    Port
    Interface
    Direction
    Connection state
```
Example:
```
    10.10.10.0/24
        ↓
    10.20.20.50
        ↓
     TCP/443
        ↓
     INGRESS
        ↓
      ALLOW
```

### Stateful Firewall

A stateful firewall tracks connection state.

Common states:
```
    NEW
    ESTABLISHED
    RELATED
```
Purpose:

    Allow legitimate return traffic without requiring broad
    reverse-direction rules.
Exact state handling depends on the firewall implementation.

### External Firewall:
- Controls untrusted/external traffic.
- Common example: ```Internet --> organization.```
- Primary purpose: Reduce external exposure.

### Internal Firewall:
- Controls traffic between internal security zones.
- Examples:
 ```   
    Users
    Applications
    Databases
    Management
```
- Primary purpose: Limit lateral movement.

Typical architecture:
```
   Internet
       ↓
  External Firewall
       ↓
      DMZ
       ↓
  Internal Firewall
       ↓
  Internal Network
```

### DMZ (Demilitarized Zone)

A separate security/network segment for systems requiring controlled
exposure to less-trusted networks.

Common DMZ systems:
- Public web servers
- Reverse proxies
- Mail gateways
- Public-facing services

Purpose:

    Reduce the chance that compromise of a public-facing system
    directly exposes the internal network.


### Firewall Layers

L3 — Network Layer:
- Source IP
- Destination IP
- Network/subnet

L4 — Transport Layer:
- TCP/UDP
- Source/destination port
- Connection state

L7 — Application Layer:
- HTTP methods
- URLs
- Headers
- Hostnames
- Application identity
- Application-level requests

Note:

    L3 = WHERE?
    L4 = WHICH CONNECTION/PORT?
    L7 = WHAT APPLICATION REQUEST?


### Packet Filtering

Allows or blocks traffic primarily using network and transport attributes.

Typical conditions:
```
    SOURCE
      ↓
    DESTINATION
      ↓
    PROTOCOL/PORT
      ↓
    INTERFACE
      ↓
    STATE
      ↓
    ACTION
```
eg:
```
    10.10.10.0/24 → 10.20.20.50 → TCP/443 → ALLOW
```
Strength:
- Efficient network connectivity control.

Limitation:
- Does not provide full application-level understanding.


### Firewall Rule Structure

Think of every rule as:
```
    SOURCE
      →
    DESTINATION
      →
    PROTOCOL/PORT
      →
    DIRECTION
      →
    ACTION
      →
    REASON
```
eg:
```
    10.10.10.0/24
      →
    10.20.20.50
      →
    TCP/443
      →
    INGRESS
      →
    ALLOW
      →
    Application access
```
Prefer:

    Specific rules

Avoid:

    ANY → ANY → ANY → ALLOW


### Rule Order / Precedence

Firewall systems have processing order/precedence.

Important:

    A broad rule can make a later specific rule ineffective.

When troubleshooting:

    Do not only check whether the desired rule exists.

Check:

    Which rule/policy/zone actually processes the traffic?


### Ingress vs Egress

- Ingress:
Traffic entering a host/network.

    Internet → Web Server

- Egress:
Traffic leaving a host/network.

    Application Server → External Service

Security principle:

    Control both inbound and outbound traffic where required.

Egress filtering can reduce unauthorized outbound connections
and limit the impact of compromised systems.


### Common Ports

    TCP/22       → SSH
    TCP/80       → HTTP
    TCP/443      → HTTPS
    TCP/53       → DNS
    UDP/53       → DNS
    TCP/25       → SMTP
    TCP/3306     → MySQL/MariaDB
    TCP/5432     → PostgreSQL

Important:

    An open port does NOT mean the application using it is secure.


### Linux Firewalling

nftables:
- Modern Linux packet-filtering framework.
- Current technology to understand for modern Linux firewalling.

firewalld:
- Higher-level firewall management service.
- Provides:
    Zones
    Services
    Ports
    Policies/rules
    Runtime configuration
    Permanent configuration

iptables:
- Traditional firewall rule-management interface.
- Common in older systems and legacy environments.
- Modern Linux distributions may provide compatibility tooling.

ufw:
- Simplified firewall-management interface.
- Commonly encountered on Ubuntu.

Important:

    Do NOT assume iptables, nftables, firewalld, and ufw
    are interchangeable.

First determine what the host actually uses.

**Common Linux Firewall conceptual model:**

    Management Tool
          ↓
      firewalld
          ↓
       nftables
          ↓
    Linux networking/kernel

This is a conceptual model, not a universal stack for every
Linux distribution/version.


### Firewalld Runtime vs Permanent 

Runtime:
    Currently active firewall configuration.

Permanent:
    Configuration intended to survive reload/restart.

Important:

    Runtime change ≠ automatically persistent configuration.

After important changes:

    1. Verify the active configuration.
    2. Verify the permanent configuration.
    3. Reload/restart when appropriate.
    4. Verify again.


### Firewalld Zones

A zone represents a trust/security context.

Common zones:

    public
    internal
    external
    trusted
    home
    work
    block
    drop

Typical meaning:

public:
    Untrusted/public network.
    Expose only required services.

internal:
    Relatively trusted internal network.

trusted:
    Highly permissive.
    Use carefully.

drop:
    Very restrictive.
    Unwanted traffic is silently dropped.

Important:

    Do not rely only on the zone name.
    Always inspect the actual zone configuration.


### Network Segmentation

Separate systems according to trust and function.

Common zones:

    Users
    Servers
    Applications
    Databases
    Management
    Development
    Production
    DMZ

Example:

    Users → Web/App      → ALLOW
    Users → Database    → DENY
    App   → Database    → ALLOW
    Web   → Database    → DENY

Goal:

    Reduce unnecessary communication
    and limit lateral movement.

**Microsegmentation:**
Applies granular network controls between individual workloads,
applications, services, or systems.

 Note:

    Traditional segmentation:
        Network → Network

    Microsegmentation:
        Workload → Workload
        Service  → Service


### NAT vs Firewall

**Firewall:**
    Controls traffic policy.

    ALLOW
    DENY
    LOG

**NAT:**
    Translates IP addresses and/or ports.

**SNAT:**
    Changes source address.

**DNAT:**
    Changes destination address and/or port.

**Masquerading:**
    Common form of dynamic source NAT.

**Note:**
    
    NAT      = Changes addressing
    Firewall = Controls traffic

### Host vs Network Firewall

Host-based firewall:
- Runs directly on a server/workstation.
- Controls traffic to/from that host.

Network firewall:
- Controls traffic between networks/security zones.

Layered model:

    Network Firewall
          ↓
      Host Firewall
          ↓
      Application

### Firewall vs Other Security Controls

Firewall:
    Controls network traffic.

IDS:
    Detects suspicious activity.

IPS:
    Detects and attempts to prevent suspicious activity.

WAF:
    Provides application-level protection,
    primarily for web traffic.

EDR:
    Protects and monitors endpoints.

Important:

    Firewall = One security layer,
    NOT the entire security architecture.


### Firewall Logging

Log useful traffic where appropriate for:

    Troubleshooting
    Auditing
    Security monitoring

Avoid excessive logging because it can:

    Create noise
    Consume storage
    Make important events harder to identify

Troubleshooting should correlate:

    Firewall logs
    Application logs
    System logs
    DNS logs
    Routing information
    Authentication logs

### Troubleshooting Flow

When a connection fails:

    1. Is the application/service running?
    2. Is it listening on the expected port?
    3. Is it listening on the correct IP/interface?
    4. Is the source IP correct?
    5. Is the destination IP correct?
    6. Is routing correct?
    7. Is DNS resolving correctly?
    8. Is the host firewall allowing traffic?
    9. Is a network/cloud firewall blocking traffic?
    10. Is NAT/port forwarding involved?
    11. Is SELinux/security policy involved?
    12. Is the application itself rejecting the connection?
    13. Check relevant logs.
    14. Test from the actual source network.

### Useful Linux Commands

Listening services:

    ss -lntup

IP addresses:

    ip addr

Routing:

    ip route

firewalld state:

    firewall-cmd --state

Active firewalld zones:

    firewall-cmd --get-active-zones

Current zone:

    firewall-cmd --list-all

All zones:

    firewall-cmd --list-all-zones

nftables ruleset:

    nft list ruleset

Legacy/compatibility iptables inspection:

    iptables -L -n -v


### Common Firewall Problems

Problem:
    Port allowed but application is not listening.

Problem:
    Application listens only on 127.0.0.1.

Problem:
    Host firewall allows traffic but network/cloud firewall blocks it.

Problem:
    Runtime configuration changed but not persisted.

Problem:
    Interface is assigned to the wrong firewalld zone.

Problem:
    Broad rule/policy takes precedence over intended rule.

Problem:
    Return traffic is not handled correctly.

Problem:
    NAT/port forwarding is incorrect.

Problem:
    DNS resolves to the wrong address.

Problem:
    Routing is incorrect.

Problem:
    SELinux/security policy blocks the operation.

Problem:
    Application itself rejects the connection.

Important:
```
    Do not automatically blame the firewall.
```
Check:
```
    Application
    Listening socket
    IP
    Routing
    DNS
    Firewall
    NAT
    SELinux/security policy
    Network/cloud firewall
```
### Firewall Changing Process

     Requirement
        ↓
      Design
        ↓
     Implement
        ↓
      Test
        ↓
     Monitor
        ↓
     Document
        ↓
      Review
        ↓
   Remove when obsolete


### Firewall Testing
Test both Expected allow & Expected deny:
```
    ALLOW → connection succeeds
```
```
    DENY → connection fails
```
And verify:
```
    Rule precedence
    Stateful return traffic
    NAT behavior
    Runtime configuration
    Permanent configuration
    Reload/restart behavior
```
### Rule Documentation
For important firewall rules record:
```
    Source
    Destination
    Protocol/Port
    Direction
    Action
    Purpose
    Owner
    Change Reference
    Review Date
```
Example:
```
    Source:        10.10.20.0/24
    Destination:   10.20.30.50
    Protocol:      TCP
    Port:          443
    Direction:     INGRESS
    Action:        ALLOW
    Purpose:       Application access
    Owner:         Application Team
    Change Ref:    CHG-1234
    Review Date:   2026-12-01
```

### Firewall Rule
Every firewall rule should answer:
```
       WHO?
        ↓
      SOURCE


      WHERE?
        ↓
    DESTINATION


      WHAT?
        ↓
    PROTOCOL / PORT


    WHICH DIRECTION?
        ↓
    INGRESS / EGRESS


    WHAT ACTION?
        ↓
    ALLOW / DENY / LOG


       WHY?
        ↓
     PURPOSE
```
### Quick Note
```

                    FIREWALL
                       |
        +--------------+--------------+
        |              |              |
      WHERE?         LAYER?         POLICY?
        |              |              |
   Host / Network   L3 / L4 / L7   Allow / Deny
        |              |              |
        +--------------+--------------+
                       |
             SOURCE → DESTINATION
                       |
              PROTOCOL / PORT
                       |
                INGRESS / EGRESS
                       |
               STATE / PRECEDENCE
                       |
                LOG / MONITOR
                       |
             TEST / DOCUMENT / REVIEW
```
---
### Terminal Session
    
```
[aadarsha@labserver ~]$ hostname
labserver

[aadarsha@labserver ~]$ hostname -I
192.168.254.2 

[aadarsha@labserver ~]$ rpm -q firewalld
firewalld-2.4.3-2.el10.noarch

[aadarsha@labserver ~]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; preset: ena>
     Active: active (running) since Sun 2026-08-30 16:00:11 +0545; 2min 20s ago
 Invocation: 6593824752514d2e9fb1cf478ff2c22e
       Docs: man:firewalld(1)
   Main PID: 914 (firewalld)
      Tasks: 2 (limit: 10630)
     Memory: 45M (peak: 45.2M)
        CPU: 442ms
     CGroup: /system.slice/firewalld.service
             └─914 /usr/bin/python3 -sP /usr/sbin/firewalld --nofork --nopid

Aug 30 16:00:11 labserver systemd[1]: Started firewalld.service - firewalld - dynami>
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Sun Aug 30 15:59:14 +0545 2026 on tty1

[root@labserver ~]# firewall-cmd --list-all		 # public zone is listed by default
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client http mountd nfs rpc-bind ssh
  ports: 5050/tcp 8080/tcp 1122/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.0/24" service name="ntp" accept
[root@labserver ~]#

[root@labserver ~]# man firewall-cmd 

# Zones

[root@labserver ~]# firewall-cmd --list-all-zones
block
  target: %%REJECT%%
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: 
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

dmz
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

drop
  target: DROP
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: 
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

external
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: yes
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

home
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client mdns samba-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

internal
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client mdns samba-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

nm-shared
  target: ACCEPT
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: dhcp dns ssh
  ports: 
  protocols: icmp ipv6-icmp
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule priority="32767" reject

public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client http mountd nfs rpc-bind ssh
  ports: 5050/tcp 8080/tcp 1122/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.0/24" service name="ntp" accept

trusted
  target: ACCEPT
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: 
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

work
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
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

[root@labserver ~]# firewall-cmd --get-default-zone
public

[root@labserver ~]# firewall-cmd --list-all --zone=public
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client http mountd nfs rpc-bind ssh
  ports: 5050/tcp 8080/tcp 1122/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.0/24" service name="ntp" accept
[root@labserver ~]#

[root@labserver ~]# firewall-cmd --list-all --zone=home
home
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client mdns samba-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@labserver ~]# firewall-cmd --list-all --zone=work
work
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
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
```

  **Add and verify a rich rule in the home zone, switch home to the default zone, then restore public as the default zone.**
```
[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client http mountd nfs rpc-bind ssh
  ports: 5050/tcp 8080/tcp 1122/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.0/24" service name="ntp" accept

[root@labserver ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.254.11" port port="8080" protocol="tcp" accept' --zone=home
success

[root@labserver ~]# firewall-cmd --reload
success

[root@labserver ~]# firewall-cmd --list-all --zone=home
home
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client mdns samba-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.11" port port="8080" protocol="tcp" accept
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --set-default-zone=home
success

[root@labserver ~]# firewall-cmd --get-default-zone
home

[root@labserver ~]# firewall-cmd --list-all
home (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client mdns samba-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.11" port port="8080" protocol="tcp" accept
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --list-all --zone=public
public
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client http mountd nfs rpc-bind ssh
  ports: 5050/tcp 8080/tcp 1122/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.254.0/24" service name="ntp" accept

[root@labserver ~]# firewall-cmd --set-default-zone=public
success

[root@labserver ~]# firewall-cmd --get-default-zone
public

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client http mountd nfs rpc-bind ssh
  ports: 5050/tcp 8080/tcp 1122/tcp
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

**Use cron to automatically switch the firewalld default zone: public at 6:00 AM, work at 9:00 AM, and home at 6:00 PM.**
```
[root@labserver ~]# crontab -e
crontab: installing new crontab
Backup of root's previous crontab saved to /root/.cache/crontab/crontab.bak

[root@labserver ~]# crontab -l
# Min         Hr         M         DoM         DoW         < command/script >

  00          6          *          *           *         firewall-cmd --set-default-zone=public
  
  00          9          *          *           *         firewall-cmd --set-default-zone=work

  00         18          *          *           *         firewall-cmd --set-default-zone=home

[root@labserver ~]# 
```

```
[root@labserver ~]# firewall-cmd --get-services
...

[root@labserver ~]# firewall-cmd --permanent --add-service={dns,ftp,smtp}
success

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client http ssh
  ports: 5050/tcp 8080/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.5.0/24" port port="22" protocol="tcp" accept
[root@labserver ~]# 

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
  services: cockpit dhcpv6-client dns ftp http smtp ssh
  ports: 5050/tcp 8080/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.5.0/24" port port="22" protocol="tcp" accept
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --list-all | grep services
  services: cockpit dhcpv6-client dns ftp http smtp ssh
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --permanent --remove-service={dns,ftp,smtp}
success
[root@labserver ~]#

 # use permanent to persist after rebooting otherwise it will exist temporarily 

[root@labserver ~]# firewall-cmd --list-all | grep services
  services: cockpit dhcpv6-client dns ftp http smtp ssh
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --reload
success
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --list-all | grep services
  services: cockpit dhcpv6-client http ssh
[root@labserver ~]# 

 # multiple ports

[root@labserver ~]# firewall-cmd --permanent --add-port={7766/tcp,1122/tcp,2001-2013/tcp}
success

[root@labserver ~]# firewall-cmd --list-all | grep ports
  ports: 5050/tcp 8080/tcp
  forward-ports: 
  source-ports: 
 
[root@labserver ~]# firewall-cmd --reload
success

[root@labserver ~]# firewall-cmd --list-all | grep ports
  ports: 5050/tcp 8080/tcp 7766/tcp 1122/tcp 2001-2013/tcp
  forward-ports: 
  source-ports: 
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --permanent --remove-port={7766/tcp,1122/tcp,2001-2013/tcp}
success

[root@labserver ~]# firewall-cmd --reload
success

[root@labserver ~]# firewall-cmd --list-all | grep ports
  ports: 5050/tcp 8080/tcp
  forward-ports: 
  source-ports: 
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client http ssh
  ports: 5050/tcp 8080/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.5.0/24" port port="22" protocol="tcp" accept
[root@labserver ~]# 
[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$
```
---



    