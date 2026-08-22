---
title: "Linux Virtualization Internals: From Intel VT-x to KVM, QEMU, and libvirt"
date: 2026-08-22
draft: false
---

### Virtualization
Virtualization is the abstraction of physical computing resources to create isolated virtual resources that can be independently managed and used.
In the context of server virtualization, a physical machine's CPU, memory, storage, and I/O resources are abstracted and presented to one or more **virtual machines (VMs)**.
Each VM is provided with virtual hardware and can run its own operating system.

> **A VM is not physical hardware. It is an isolated execution environment whose hardware interface is virtualized.**

---

### How is Virtualization Achieved?

Virtualization is achieved by introducing a software layer that abstracts and manages access to physical resources.

The virtualization layer:

* provides virtual hardware to the guest
* controls access to physical resources
* isolates workloads
* maps/schedules virtual resources onto physical resources

Conceptually:

```text
									Guest OS
									   │
								Virtual Hardware
									   │
								Virtualization Layer
							           │
								Physical Hardware
```
The exact implementation differs by resource and virtualization technology.
Virtualization is fundamentally about **resource abstraction, controlled access, and isolation**.

---

### Hypervisor

A **hypervisor** is the software layer that creates and manages virtual machines and controls their access to physical computing resources on physical hardware.

It is responsible for providing the virtualization environment in which VMs execute.

```text
								┌───────────────────────────┐
								│   VM 1  │  VM 2  │  VM 3  │
								├───────────────────────────┤
								│         Hypervisor        │
								├───────────────────────────┤
								│     Physical Hardware     │
								└───────────────────────────┘
```

The hypervisor manages or coordinates:
* CPU execution
* memory isolation and mapping
* virtual devices
* I/O
* VM lifecycle
* resource allocation

Modern CPUs provide hardware-assisted virtualization extensions such as:
- Intel VT-x
- AMD-V

These processor features allow the CPU to support virtualization-specific execution modes and privileged operations.
```text
					    Guest OS1 | Guest OS2  | Guest OS3
					 	                 │
						           		 ▼
						     	  Virtual Hardware
								         │
						           		 ▼
						    	     Hypervisor
						           		 │
						           		 ▼
						    	 Physical Hardware
```
The guest OS operates against virtual CPU, memory, storage, and network interfaces presented by the virtualization stack.

The hypervisor provides that virtual hardware interface and manages how those virtual resources are implemented using the underlying physical infrastructure.

---

### Type I vs Type II Hypervisors

#### 1. Type I Hypervisors (Bare-Metal)

* Run directly on physical hardware.
* Do not depend on a conventional general-purpose host OS underneath.
* Provide efficient access to physical resources.
* Commonly used for servers, data centers, and virtualization infrastructure.
* Examples: **VMware ESXi, Microsoft Hyper-V, Xen**.

```text
							        TYPE I — BARE-METAL

							+---------------------------------+
							|   Guest VM A   |  Guest VM B    |
							|   (Guest OS)   |  (Guest OS)    |
							+---------------------------------+
							|            HYPERVISOR           |
							|  (Hardware Resource Management) |
							+---------------------------------+
							|        PHYSICAL HARDWARE        |
							|  CPU  |  RAM  |  Disk  |  NIC   |
							+---------------------------------+
```

---

#### 2. Type II Hypervisors (Hosted)

* Run as an application on a conventional host operating system.
* Use the host OS for access to the underlying hardware.
* Add an additional software layer compared with the traditional Type I model.
* Commonly used for desktop virtualization, testing, learning, and development.
* Examples: **Oracle VirtualBox, VMware Workstation**.

```text
							        TYPE II — HOSTED

							+-------------------------------+
							|  Guest VM A   |  Guest VM B   |
							|  (Guest OS)   |  (Guest OS)   |
							+-------------------------------+
							|       TYPE II HYPERVISOR      |
							|    (Runs as normal software)  |
							+-------------------------------+
							|       HOST OPERATING SYSTEM   |
							|      Windows / Linux / macOS  |
							+-------------------------------+
							|       PHYSICAL HARDWARE       |
							|    CPU | RAM | Disk | NIC     |
							+-------------------------------+
```
---

### What exactly gets virtualized?

Virtualization does not create additional physical resources. It abstracts existing resources and presents them to VMs as virtual hardware.

- **CPU:** The virtualization layer maps vCPUs to physical CPU resources.
- **Memory:** The guest manages its memory; the virtualization stack maps it to physical memory.
- **Storage:** A virtual disk is backed by a disk image, logical volume, physical device, or other storage backend.
- **Network:** A VM uses a virtual NIC; the virtualization stack connects it to the underlying network.
- **I/O:** Other devices can also be virtualized or passed through, depending on the technology and configuration.

The same principle applies to other devices such as storage controllers, USB devices, GPUs, and other I/O devices, depending on the virtualization technology and configuration.

---

### Hardware-Assisted Virtualization

Hardware-assisted virtualization uses CPU features specifically designed to support virtualization, 
allowing the hypervisor to run guest operating systems efficiently while retaining control over privileged operations.

The two common CPU virtualization extensions are:

- Intel VT-x
- AMD-V

These extensions provide processor-level support for running virtual machines and handling transitions between guest and hypervisor execution

---
### KVM + QEMU

KVM (Kernel-based Virtual Machine) is a Linux kernel virtualization facility that uses CPU hardware virtualization extensions such as Intel VT-x and AMD-V to enable the Linux kernel to run virtual machines. KVM
- Turns a Linux operating system into a hypervisor
- Lets one physical computer run many virtual machines
- Built directly into the core Linux system


QEMU is a machine emulator and virtualizer that provides virtual hardware and emulates devices for VMs.
QEMU can perform CPU emulation with TCG or use an accelerator such as KVM for hardware-assisted virtualization

```text
					                    Guest VM
				    	                   │
				        	               ▼
				            	  ┌─────────────────┐
				            	  │      QEMU       │
				            	  │                 │
				            	  │ VM process      │
				            	  │ virtual devices │
				            	  │ machine model   │
				            	  └────────┬────────┘
				            	           │
				            	     /dev/kvm ioctls
				            	           │
				            	           ▼
				            	  ┌─────────────────┐
				            	  │      KVM        │
				            	  │ Linux kernel    │
				            	  │ VM/vCPU API     │
				            	  └────────┬────────┘
				                	       │
				                    	   ▼
				            	  CPU virtualization
				            		  + host kernel
				            		  + physical hardware
				
```
QEMU provides the user-space virtual machine process and device model, while KVM provides the kernel interface for hardware-assisted CPU virtualization and related VM/vCPU operations. QEMU interacts with KVM through the /dev/kvm ioctl API.

### Virtualization Software Stack
```
- KVM          --> Linux kernel virtualization facility; provides hardware-assisted CPU virtualization.
- QEMU         --> User-space VM software; provides virtual machine process and virtual hardware/device emulation.
- libvirt
    │
    ├── API / client libraries
    │
    ├── QEMU driver
    │       │
    │       └── virtqemud
    │
    ├── network driver
    │   	    │
    │   	    └── virtnetworkd
    │
    └── storage driver
           	 │
           	 └── virtstoraged
- virt-manager --> GUI client for managing libvirt-based VMs.
```
>Historically, libvirt used the monolithic libvirtd daemon. Modern libvirt also provides modular daemons such as virtqemud, virtnetworkd, and virtstoraged; the exact deployment depends on the distribution and configuration.
```text
			    	                Virtual Machines
			            	               │
			            	               ▼
			            	          virt-manager
			            	        (GUI management)
			            	               │
			            	               ▼
			            	            libvirt
			            	    (management API/service)
			            	               │
			            	               ▼
			            	          QEMU / KVM
			            	  ┌────────────┴────────────┐
			            	  │                         │
			            	QEMU                       KVM
			    	   	User-space VM             Linux kernel
			    	   	/ device model            virtualization
			    	          │                         │
			    	          └────────────┬────────────┘
			    	                       ▼
			    	                Physical Hardware
```
---
### Lab Session
```
# =================================================================================
# 1. HOST OS & KERNEL IDENTIFICATION
#    Verify host Linux distribution, kernel release version, and CPU architecture.
#    Ensures target system compatibility for KVM hypervisor modules and tooling.
# =================================================================================

# OS distribution, metadata and version codename

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

# active Linux kernel version (KVM driver interface depends on kernel release)

aadarkdk@pop-os:~$ uname -r
7.0.11-76070011-generic

# confirm host CPU hardware architecture

aadarkdk@pop-os:~$ uname -m
x86_64

# ===================================================================================
# 2. HARDWARE VIRTUALIZATION EXTENSIONS CHECK
#    Interrogate host CPU capabilities via lscpu to confirm hardware-assisted
#    virtualization support (Intel VT-x or AMD-V extensions must be enabled in BIOS).
# ===================================================================================

# host CPU architecture definition

aadarkdk@pop-os:~$ lscpu | grep 'Architecture'
Architecture:                            x86_64

# CPU vendor string and model name

aadarkdk@pop-os:~$ lscpu | grep -E 'Vendor ID|Model name'
Vendor ID:                               GenuineIntel
Model name:                              11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz

# hardware virtualization flag exposure  ( VT-x for Intel / AMD-V for AMD )

aadarkdk@pop-os:~$ lscpu | grep 'Virtualization'
Virtualization:                          VT-x

# ==============================================================================
# 3. KERNEL HYPERVISOR MODULES & DEVICE INTERFACE
#    Validate that the KVM kernel-space drivers are loaded into host memory
#    and verify character device node ownership and access permissions.
# ==============================================================================

# confirm kvm core kernel module (kvm.ko) and vendor module (kvm_intel.ko) are loaded

aadarkdk@pop-os:~$ lsmod | grep kvm
kvm_intel             532480  1
kvm                  1503232  3 kvm_intel
irqbypass              16384  1 kvm
aadarkdk@pop-os:~$

# check system group assignments for the active user account
# On this host, the user belongs to the kvm and libvirt groups, 
  which allows access to the relevant device/socket interfaces according to the host's configured permissions.
  Exact group requirements are distribution- and configuration-dependent.

aadarkdk@pop-os:~$ groups
aadarkdk adm dialout sudo kvm lpadmin libvirt

# inspect the KVM character device node (/dev/kvm) permissions
# Programs like QEMU issue ioctl() calls to this node for hardware-accelerated guest execution

aadarkdk@pop-os:~$ ls -l /dev/kvm 
crw-rw----+ 1 root kvm 10, 232 Aug 21 09:08 /dev/kvm

```
 - /dev/kvm is the device interface exposed by the Linux KVM subsystem to userspace.
Programs such as QEMU can use this interface to ask KVM to perform hardware-assisted virtualization.

```
# ==============================================================================
# 4. USERSPACE EMULATION & ORCHESTRATION DAEMON
# Verify QEMU binary availability, libvirtd management daemon state,
# and CLI API connectivity.
# ==============================================================================

# check installed version of the QEMU x86_64 machine emulator

aadarkdk@pop-os:~$ qemu-system-x86_64 --version
QEMU emulator version 6.2.0 (Debian 1:6.2+dfsg-2ubuntu6.30)
Copyright (c) 2003-2021 Fabrice Bellard and the QEMU Project developers

# list existing virtual machine domains (active and inactive) via libvirt client CLI

aadarkdk@pop-os:~$ virsh list --all
 Id   Name   State
--------------------

# inspect current libvirt connection URI 
# (qemu:///system connects to the system-level libvirt instance. 
   Access to its read-write management socket can carry privileges equivalent to root, depending on the configured authentication and authorization.)

aadarkdk@pop-os:~$ virsh uri
qemu:///system

# query systemd unit status for the libvirt virtualization daemon (libvirtd)

aadarkdk@pop-os:~$ systemctl status libvirtd --no-pager
● libvirtd.service - Virtualization daemon
     Loaded: loaded (/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2026-08-21 09:08:50 +0545; 10h ago
TriggeredBy: ● libvirtd-admin.socket
             ● libvirtd.socket
             ● libvirtd-ro.socket
       Docs: man:libvirtd(8)
             https://libvirt.org
   Main PID: 1005 (libvirtd)
      Tasks: 21 (limit: 32768)
     Memory: 42.3M
        CPU: 1.133s
     CGroup: /system.slice/libvirtd.service
             ├─1005 /usr/sbin/libvirtd
             ├─1620 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
             └─1622 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper

Aug 21 09:08:50 pop-os systemd[1]: Started Virtualization daemon.
Aug 21 09:08:51 pop-os dnsmasq[1620]: started, version 2.90 cachesize 150
Aug 21 09:08:51 pop-os dnsmasq[1620]: compile time options: IPv6 GNU-getopt DBus no-UBus i18n IDN2 DHCP DHCPv6 no-Lua TFTP conntrack ipset no-nftset auth cryptohash DNSSEC l…notify dumpfile
Aug 21 09:08:51 pop-os dnsmasq-dhcp[1620]: DHCP, IP range 192.168.122.2 -- 192.168.122.254, lease time 1h
Aug 21 09:08:51 pop-os dnsmasq-dhcp[1620]: DHCP, sockets bound exclusively to interface virbr0
Aug 21 09:08:51 pop-os dnsmasq[1620]: reading /etc/resolv.conf
Aug 21 09:08:51 pop-os dnsmasq[1620]: using nameserver 127.0.0.53#53
Aug 21 09:08:51 pop-os dnsmasq[1620]: read /etc/hosts - 9 names
Aug 21 09:08:51 pop-os dnsmasq[1620]: read /var/lib/libvirt/dnsmasq/default.addnhosts - 0 names
Aug 21 09:08:51 pop-os dnsmasq-dhcp[1620]: read /var/lib/libvirt/dnsmasq/default.hostsfile
Hint: Some lines were ellipsized, use -l to show in full.
aadarkdk@pop-os:~$

# check installed version of the virt-install CLI provisioning utility 

aadarkdk@pop-os:~$ virt-install --version
4.0.0
```
