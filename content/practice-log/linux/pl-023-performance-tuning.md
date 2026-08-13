---
title: "PL - 023 — System Performance Tuning with Tuned Daemon"
date: 2026-08-13
draft: false
---

# System Performance Tuning
System performance tuning involves optimizing hardware and kernel resource utilization—such as CPU governors, virtual memory, disk I/O schedulers, and network buffers—to maximize throughput, minimize latency, and stabilize workloads.

- Resource Bottlenecks: Default OS configurations are generic; tuning aligns kernel parameters directly with specific workload requirements (e.g., databases, virtualization guests, high-throughput network nodes).

- Dynamic Tuning Daemon: tuned applies predefined kernel and hardware configurations through profiles. It can make limited dynamic adjustments based on system state (e.g., AC vs. battery power, CPU load), but it is primarily a profile-driven framework rather than a continuous real-time adaptive system.

- Management Utility: tuned-adm is used for querying, recommending, and switching operational profiles.

- Profile Verification: Real-world operations require verifying profile compliance to ensure system configuration drift has not altered applied kernel parameters (tuned-adm verify).

> Note: This guide covers profile selection and activation. True performance tuning requires workload-specific benchmarking before and after applying changes.

 ### Production Considerations

The following points must be addressed before applying any tuned profile to production servers:

- Risk Assessment: Changing kernel parameters can cause instability, OOM kills, or boot failures. Schedule changes during a maintenance window and ensure backups are current.

- Staging Validation: Never apply a profile directly to production. Test the identical profile under realistic workload in a staging or pre-production environment first.

- Rollback Plan: Document the pre-change state. Use tuned-adm off to revert to system defaults, or switch back to the previous profile if performance degrades.

- Configuration Conflicts: tuned overrides values set in /etc/sysctl.conf and other static configuration files. Audit your configuration management (Ansible, Puppet, Chef) to prevent silent conflicts and configuration drift.

- Custom Profiles: Production workloads rarely fit generic profiles. Derive a custom profile from throughput-performance or latency-performance and tune it to your specific application requirements.

- Baseline Metrics: Capture baseline metrics (CPU steal, I/O wait, context switches, network retransmits, application latency/throughput) before and after activation. Without measurable data, tuning is speculative.

- Persistence & Upgrades: tuned profiles persist across reboots, but OS upgrades can reset or deprecate profiles. Enforce desired profiles through configuration management and verify after patching.

- Change Control: Every kernel parameter change in production should be ticketed, peer-reviewed, and signed off according to your organization's change management policy.

 ### Terminal Session
```
 # Tuning Daemon Installation & Service Management

aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Wed Aug 12 20:02:30 2026 from 192.168.254.152
[aadarsha@labserver ~]$ 

[aadarsha@labserver ~]$ date
Thu Aug 13 05:06:57 AM +0545 2026

[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ hostname
labserver 

[aadarsha@labserver ~]$ rpm -q tuned
package tuned is not installed

# Switching to root (use sudo for production environments)

[aadarsha@labserver ~]$ su - root
Password: 
Last login: Wed Aug 12 20:50:51 +0545 2026 on tty1

[root@labserver ~]# dnf -y install tuned

[root@labserver ~]# systemctl status tuned
○ tuned.service - Dynamic System Tuning Daemon
     Loaded: loaded (/usr/lib/systemd/system/tuned.service; enabled; preset: enabled)
     Active: inactive (dead)
       Docs: man:tuned(8)
             man:tuned.conf(5)
             man:tuned-adm(8)
[root@labserver ~]# 

[root@labserver ~]# systemctl start tuned

[root@labserver ~]# systemctl enable --now tuned

[root@labserver ~]# systemctl status tuned
● tuned.service - Dynamic System Tuning Daemon
     Loaded: loaded (/usr/lib/systemd/system/tuned.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-08-13 05:16:53 +0545; 4s ago
   Invocation: 451915505551408a912ee063a0e0bf76
       Docs: man:tuned(8)
             man:tuned.conf(5)
             man:tuned-adm(8)
   Main PID: 2641 (tuned)
      Tasks: 4 (limit: 10630)
     Memory: 14.6M (peak: 16.1M)
        CPU: 237ms
     CGroup: /system.slice/tuned.service
             └─2641 /usr/bin/python3 -Es /usr/sbin/tuned -l -P
...
[root@labserver ~]# 

 # Listing available tuned profiles

[root@labserver ~]# tuned-adm list
Available profiles:
- accelerator-performance     - Throughput performance based tuning with disabled higher latency STOP states
- aws                         - Optimize for aws ec2 instances
- balanced                    - General non-specialized tuned profile
- balanced-battery            - Balanced profile biased towards power savings changes for battery
- desktop                     - Optimize for the desktop use-case
- hpc-compute                 - Optimize for HPC compute workloads
- intel-sst                   - Configure for Intel Speed Select Base Frequency
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
- optimize-serial-console     - Optimize for serial console use.
- powersave                   - Optimize for low power consumption
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
- virtual-guest               - Optimize for running inside a virtual guest
- virtual-host                - Optimize for running KVM guests
Current active profile: virtual-guest
[root@labserver ~]# 

 # Inspecting what a profile changes before applying it

[root@labserver ~]# tuned-adm profile_info virtual-guest
Profile name:
virtual-guest

Profile summary:
Optimize for running inside a virtual guest

Profile description:

[root@labserver ~]# 

[root@labserver ~]# tuned-adm profile_info network-throughput
Profile name:
network-throughput

Profile summary:
Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks

Profile description:

[root@labserver ~]#

 # Finding the most suitable profile (basic auto-detection)

[root@labserver ~]# tuned-adm recommend
virtual-guest  

[root@labserver ~]# tuned-adm active
Current active profile: virtual-guest

# The recommendation matches the active profile.
# This is a reasonable starting point, not a guarantee of optimal performance.
# Workload-specific benchmarking is required for true optimization.

[root@labserver ~]# tuned-adm profile virtual-guest               # activate the required profile

[root@labserver ~]# systemctl restart tuned

[root@labserver ~]# tuned-adm active
Current active profile: virtual-guest 

[root@labserver ~]# tuned-adm recommend
virtual-guest

 # Verifying that the system settings match the active profile

[root@labserver ~]# tuned-adm verify

[root@labserver ~]# tuned-adm verify --ignore-missing
Verification succeeded, current system settings match the preset profile.
See TuneD log file ('/var/log/tuned/tuned.log') for details.
[root@labserver ~]# 

 # Disabling tuned and reverting to system defaults

[root@labserver ~]# tuned-adm off
[root@labserver ~]# tuned-adm active
No current active profile.
[root@labserver ~]#

 # Re-activating the profile after verification

[root@labserver ~]# tuned-adm profile virtual-guest
[root@labserver ~]# tuned-adm active
Current active profile: virtual-guest
[root@labserver ~]# 

 # Checking applied sysctl values for confirmation

[root@labserver ~]# sysctl vm.swappiness vm.dirty_ratio
vm.swappiness = 30
vm.dirty_ratio = 30

[root@labserver ~]# exit
logout
[aadarsha@labserver ~]$ exit
logout
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$
```