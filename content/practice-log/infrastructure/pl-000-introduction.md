---
title: "PL - 001 — Infrastructure Automation & Ansible | Local Linux Lab Environment for Ansible"
date: 2026-08-31
draft: false
---
### Infrastructure Automation

Infrastructure Automation is the programmatic lifecycle management of computing environments using code. It replaces manual, error-prone terminal commands with repeatable workflows to provision, configure, and secure servers, networks, and services.

>Instead of manually logging into 20 servers to update firewall rules, tweak sshd_config, or deploy patches, an infrastructure automation tool applies these changes concurrently across the entire fleet in seconds—guaranteeing 100% consistency, eliminating configuration drift, and ensuring full idempotency (where running the automation 10 times yields the exact same safe, stable state).

### Automation Concepts:

* **Idempotency:** 

  The core property of robust automation ensuring that applying a playbook or configuration multiple times results in the exact same system state, executing modifications *only* when the current state diverges from the desired state.

* **Declarative vs. Imperative:** 
  The paradigm of defining how infrastructure states are expressed and achieved.
  
  * **Imperative:** 
  Writing a procedural sequence of explicit commands (*how* to do it), such as checking if a package exists, running `apt-get install`, and manually executing error checks. If a step fails halfway, recovery is complex.
  
  * **Declarative:** Defining the final target state (*what* it should look like), such as stating `package: nginx` with `state: present`. The automation engine analyzes the current environment, calculates the delta, and executes the exact necessary steps automatically.

* **Configuration Drift:** 
  The silent operational degradation where servers diverge from their baseline security and compliance configurations due to unversioned, ad-hoc, manual hotfixes applied directly in production.
  * *Example case:* A sysadmin logs into a production web node to troubleshoot an issue and temporarily opens port 22 or modifies a global SSL cipher. This creates an unrecorded vulnerability. Infrastructure automation prevents drift by running scheduled compliance checks (e.g., nightly ansible-pull runs) that automatically overwrite and revert any unauthorized manual changes back to the golden baseline.

* **Agentless vs. Agent-based Architecture:** 
  The underlying transport and management topology used to communicate with and enforce states on managed nodes.
  * **Agentless:**
   Manages remote targets over native, secure administrative protocols (like SSH or WinRM) without requiring a locally running background daemon or client software permanently installed on the target nodes (e.g., Ansible). This reduces resource overhead and minimizes the attack surface.
   
  * **Agent-based:** Relies on a dedicated local service (daemon) running persistently on every managed node that periodically checks in with a central master server to fetch configuration policies and report compliance metrics (e.g., Puppet, Chef).

---

### Infrastructure Automation Tools Ecosystem

In production environments, tools are divided based on their specific layer in the pipeline:

#### 1. Provisioning & Infrastructure as Code (IaC)

*Focuses on building and managing the underlying cloud or physical resources (servers, networking, VPCs, subnets).*
* **Terraform / OpenTofu:** Industry standard, declarative IaC for multi-cloud provisioning.
* **AWS CloudFormation:** Native, declarative provisioning specifically for AWS infrastructure.
* **Pulumi:** Modern IaC that allows provisioning infrastructure using general-purpose programming languages (Python, TypeScript, Go).

#### 2. Configuration Management & Orchestration
*Focuses on managing what happens inside the server (installing packages, configuring services, managing users and security baselines).*
* **Ansible:** Agentless, human-readable (YAML-based), and widely used for configuration management and application deployment over SSH.
* **Puppet:** Agent-based, declarative configuration management tool utilizing a master-agent topology.
* **Chef:** Configuration management using a Ruby-DSL-based approach for defining system states.

#### 3. Server Image Building
*Focuses on creating immutable, pre-configured server images before deployment.*
* **HashiCorp Packer:** Automates the creation of identical machine images (AMI, Docker images, ISOs) for multiple platforms from a single source configuration.

---

### Ansible

* An open-source enterprise automation engine used for configuration management, application deployment, cloud provisioning, and intra-service orchestration.

* Automates the configuration and enforcement of system states across local or remote infrastructure through human-readable YAML playbooks.

* **Core Philosophy / Slogan:** *"Simple, Yet Powerful"*—designed with a low learning curve while remaining powerful enough to orchestrate complex multi-tier enterprise deployments.

* **Origin & Lifecycle:** Initially created by Michael DeHaan in 2012 and acquired by Red Hat in 2015. It serves as the core automation framework for Red Hat Enterprise Linux (RHEL) systems automation.

* Ansible modules and core code are written entirely in **Python**, utilizing JSON for standard module output communication over secure shells.

### Ansible Architecture & Components

```
                          Control Node
              ┌─────────────────────────────────────┐
              │                                     │
              │        Ansible                      │
              │           ├── Inventory             │
              │           ├── Playbooks             │
              │           ├── Roles / Collections   │
              │           ├── ansible.cfg           │
              │           └── Plugins / Modules     │
              │                                     │
              └──────────────────┬──────────────────┘
                                 │
                         SSH / WinRM / Other
                                 │
                   ┌─────────────┴─────────────┐
                   │                           │
             Managed Node                 Managed Node
             (Linux/Unix)                  (Windows)
            ┌──────────────┐             ┌──────────────┐
            │ No permanent │             │ No permanent │
            │ Ansible agent│             │ Ansible agent│
            │              │             │              │
            │ Python*      │             │ PowerShell / │
            │ Modules      │             │ WinRM/PSRP*  │
            │ Local facts  │             │              │
            └──────────────┘             └──────────────┘
* Requirements depend on the module/connection being used.
```
#### 1. Control Node
 * The single machine where Ansible is installed and from which all automation jobs, playbooks, and ad-hoc commands are initiated.
 
* Must run a Unix-like operating system (Linux, macOS). Windows cannot act as an Ansible Control Node. It requires Python installed locally.
* Houses the project inventory, playbooks, custom modules, variable files, and the `ansible.cfg` configuration file.

#### 2. Managed Nodes (Targets)
* The remote servers, virtual machines, network devices, or cloud instances being managed, configured, and controlled by the Control Node.
* **Requirements:** 
  * Must have **Python (3.x)** installed on the remote host so Ansible can execute pushed modules.
  * Must allow inbound administrative access from the Control Node (typically via **SSH** on Linux/Unix or **WinRM** on Windows).
* Managed nodes require **no persistent background daemons, agents, or client software** running continuously. Ansible connects temporarily, executes the requested modules, cleans up the temporary files, and closes the connection.

#### 3. Inventory
* A file (or dynamic script) containing a structured list of managed nodes grouped logically so Ansible knows *where* to run automation tasks.
* Can be static files written in **INI** or **YAML** format, or generated dynamically via inventory plugins querying cloud APIs (AWS, Azure, VMware) or CMDBs.
* **Example Structure (INI):**
```ini
        [webservers]
        web1.example.com ansible_host=192.168.1.10
        web2.example.com ansible_host=192.168.1.11

        [dbservers]
        db1.example.com ansible_host=192.168.1.20
```  
#### 4. Modules
* Reusable, standalone units of code (often referred to as task plugins) that Ansible executes on remote managed nodes.
* Each module targets a specific desired state (e.g., managing a package with `dnf`, starting a service with `systemd`, or modifying a file with `lineinfile`). They are idempotent by design.
* When a playbook runs, Ansible temporarily copies the required module script over SSH to the managed node, executes it via the remote Python interpreter, parses the JSON output, and deletes the temporary script.

#### 5. Playbooks
* Ordered, declarative configuration blueprints written in **YAML** that define a series of plays and tasks to be executed against specific inventory groups.
* Used for complex orchestration, multi-node application deployment, and continuous server hardening configuration baselines.

#### 6. Plugins & Collections
* **Plugins:** Pieces of code that augment Ansible's core functionality (e.g., connection plugins, action plugins, callback plugins, and filter plugins).

* **Collections:** The modern distribution format for Ansible content, packaging multiple roles, playbooks, modules, and plugins into a single downloadable bundle via Ansible Galaxy (e.g., `ansible.posix`, `ansible.builtin`).

---
## Lab Session
```
	================================================================================
		            ANSIBLE MULTI-SERVER PRACTICE LAB
	================================================================================

		                      ADMINISTRATOR
		                            |
		                            | SSH / Console
		                            v
	+-------------------------------------------------------------------------------+
	|                             CONTROL NODE                                      |
	|                                                                               |
	|                         Hostname : control-node                               |
	|                         IP       : 192.168.254.15                             |
	|                                                                               |
	|  +-------------------------------------------------------------------------+  |
	|  | 			 Ansible Control Software                            |  |
	|  | 			                                                     |  |
	|  | 			ansible-core                                         |  |
	|  | 			     |                                               |  |
	|  | 			     +---- Playbook: playbook.yml                    |  |
	|  | 			     |       |                                       |  |
	|  | 			     |       +---- Task 1 ----> Ansible Module       |  |
	|  | 			     |       +---- Task 2 ----> Ansible Module       |  |
	|  | 			     |       +---- Task 3 ----> Ansible Module       |  |
	|  | 			     |                                               |  |
	|  | 			     +---- Inventory                                 |  |
	|  | 			     |       |                                       |  |
	|  | 			     |       +---- [dev]                             |  |
	|  | 			     |       +---- [testserver]                      |  |
	|  | 			     |       +---- [productionserver]                |  |
	|  | 			     |                                               |  |
	|  | 			     +---- ansible.cfg                               |  |
	|  |                                                                         |  |
	|  +-------------------------------------------------------------------------+  |
	|                                                                               |
	+-----------------------------------+-------------------------------------------+
		                            |
		                            | Ansible connection
		                            | SSH (Linux/Unix)
		                            |
		                            | Authentication:
		                            | SSH public key 
		                            |
		                            v
		                 +-----------------------+
		                 |  NETWORK / LAB LAN    |
		                 |  192.168.254.0/24     |
		                 +-----------+-----------+
		                             |
	     +-----------------------+-----------------------+-------------------------------+
	     |                       |                       |                               |
	     |                       |                       |                               |
	     v                       v                       v                               v
+----------------------+  +----------------------+  +----------------------+   +-----------------------+
|  DEV1 MANAGED NODE   |  |  DEV2 MANAGED NODE   |  |  TEST MANAGED NODE   |   |  PROD MANAGED NODE    |
|----------------------|  |----------------------|  |----------------------|   |-----------------------|
|   dev1.local         |  |   dev2.local         |  |   testserver.local   |   |   prodserver.local    |
|   192.168.254.16     |  |   192.168.254.17     |  |   192.168.254.18     |   |   192.168.254.19      |
|                      |  |                      |  |                      |   |                       |
|   No Ansible         |  |   No Ansible         |  |   No Ansible         |   |   No Ansible          |
|   agent required     |  |   agent required     |  |   agent required     |   |   agent required      |
|                      |  |                      |  |                      |   |                       |
|   Receives temporary |  |   Receives temporary |  |   Receives temporary |   |   Receives temporary  |
|   Ansible modules    |  |   Ansible modules    |  |   Ansible modules    |   |   Ansible modules     |
|   Executes tasks     |  |   Executes tasks     |  |   Executes tasks     |   |   Executes tasks      |
|   Returns results    |  |   Returns results    |  |   Returns results    |   |   Returns results     |
+----------------------+  +----------------------+  +----------------------+   +-----------------------+
		                             
	    
	================================================================================
		                      EXECUTION FLOW
	================================================================================
                        
                        	  1. Administrator
                        		  |
                        		  v
                        	  2. Writes / modifies playbook.yml
                        		  |
                        		  v
                        	  3. Defines target hosts using Inventory
                        		  |
                        		  v
                        	  4. Runs Ansible
                        		  |
                        		  |  ansible-playbook -i inventory playbook.yml
                        		  v
                        	  5. Control Node reads:
                        		  |
                        		  +---- ansible.cfg
                        		  |
                        		  +---- Inventory
                        		  |
                        		  +---- Playbook
                        		  |
                        		  +---- Variables / Roles / Collections
                        		  |
                        		  v
                        	  6. Ansible determines target hosts
                        		  |
                        		  v
                        	  7. Ansible connects to managed nodes
                        		  |
                        		  |  SSH
                        		  v
                        	  8. Ansible executes required modules/tasks
                                                  |
                             +----------------+----------------+----------------+
                             |                |                |                |
                             v                v                v                v
                           dev1             dev2          testserver       prodserver
                             |                |                |                |
                             +----------------+----------------+----------------+
                                                      |
                                                      v
                        	  9. Managed nodes execute the requested operations
                        		  |
                        		  v
                        	 10. Results are returned to Control Node
                        		  |
                        		  v
                        	 11. Ansible reports:
                        		  |
                        		  +---- changed
                        		  +---- ok
                        		  +---- failed
                        		  +---- skipped
                        		  +---- unreachable
                        		  |
                        		  v
                        	 12. Administrator reviews the result
```

---

### Preparing a Local Linux Environment for Ansible

#### Overview

This section documents the preparation and validation of five CentOS Stream 10 virtual machines before installing `ansible-core` and beginning Ansible configuration.

The virtual machines were cloned from a common source VM, so the initial environment required host-specific identity, networking, and access configuration before the systems could be treated as independent hosts in an Ansible-managed environment.

#### Environment

| Host | Role | Final IPv4 Address |
|---|---|---|
| `control-node` | Ansible control node | `192.168.254.15/24` |
| `dev1` | Managed host | `192.168.254.16/24` |
| `dev2` | Managed host | `192.168.254.17/24` |
| `testserver` | Managed host | `192.168.254.18/24` |
| `prodserver` | Managed host | `192.168.254.19/24` |

#### Common Network Configuration

- **Default gateway:** `192.168.254.254`
- **DNS server:** `192.168.254.254`
- **Operating system:** CentOS Stream 10 (Coughlan)
- **Python:** 3.12.13
- **SSH service:** enabled and active
- **SELinux:** Enforcing

#### Preparation and Validation Performed

The following tasks were completed before installing `ansible-core`:

1. Configured unique hostnames and static IPv4 addresses.
2. Verified the resulting network routes and host addressing.
3. Detected that the cloned VMs inherited identical SSH host keys.
4. Regenerated the SSH host keys so each VM has a unique SSH host identity and fingerprint.
5. Verified that the cloned VMs had identical machine IDs.
6. Regenerated the machine ID on the cloned systems to establish unique system identities.
7. Verified that the virtual machines have unique network-interface MAC addresses.
8. Verified the operating system, Python runtime, SSH service, and SELinux status.
9. Configured static hostname resolution for the Ansible environment.
10. Established initial SSH connectivity and verified the hosts' SSH fingerprints.
11. Generated an ED25519 SSH key pair for the `cnode` account on the control node.
12. Deployed the public key to the `cnode` account on each managed host.
13. Verified SSH access using public-key authentication without requiring an interactive password.
14. Configured passwordless `sudo` for the `cnode` account on the managed hosts.
15. Verified that `sudo -n` can obtain administrative privileges without prompting for a password.

At the end of this preparation phase, the five VMs have distinct network and system identities, verified SSH connectivity, non-interactive privilege escalation, and the baseline software and security configuration required to proceed with Ansible.

---

```
# Static Network and Hostname Configuration:
# The VMs were cloned from a common source VM and initially received
# temporary DHCP addresses and the source VM's hostname.
#
# Configure each VM with a unique hostname and static IPv4 address.
# Configure the default gateway and DNS server, then reload and
# reactivate the NetworkManager connection to apply the changes.
#
# Final network topology:
#   control-node  192.168.254.15/24
#   dev1          192.168.254.16/24
#   dev2          192.168.254.17/24
#   testserver    192.168.254.18/24
#   prodserver    192.168.254.19/24
#
# Common network settings:
#   Gateway: 192.168.254.254
#   DNS:     192.168.254.254
#
# Verify the hostname, IP address, and routing configuration
# after applying the changes to each VM.
```

#### control-node
```
aadarkdk@pop-os:~$ ssh cnode@192.168.254.250
cnode@192.168.254.250's password: 
Last login: Mon Aug 31 22:06:10 2026

[cnode@control-node ~]$ su - root
Password: 
Last login: Mon Aug 31 21:49:18 +0545 2026 on tty1

[root@control-node ~]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
[connection]
id=enp0s3
uuid=3c8d5849-b024-3eb4-a127-11501dc8e39b
type=ethernet
autoconnect-priority=-999
interface-name=enp0s3
timestamp=1788180904

[ethernet]

[ipv4]
method=auto

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
[root@control-node ~]# vi /etc/NetworkManager/system-connections/enp0s3.nmconnection 

[root@control-node ~]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
[connection]
id=enp0s3
uuid=3c8d5849-b024-3eb4-a127-11501dc8e39b
type=ethernet
autoconnect-priority=-999
interface-name=enp0s3
timestamp=1788180904

[ethernet]

[ipv4]
method=manual
address1=192.168.254.15/24
gateway=192.168.254.1
dns=192.168.254.1;

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]

[root@control-node ~]# ip route
default via 192.168.254.254 dev enp0s3 proto dhcp src 192.168.254.250 metric 100 
192.168.254.0/24 dev enp0s3 proto kernel scope link src 192.168.254.250 metric 100 

[root@control-node ~]# vi /etc/NetworkManager/system-connections/enp0s3.nmconnection 

[root@control-node ~]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
[connection]
id=enp0s3
uuid=3c8d5849-b024-3eb4-a127-11501dc8e39b
type=ethernet
autoconnect-priority=-999
interface-name=enp0s3
timestamp=1788180904

[ethernet]

[ipv4]
method=manual
address1=192.168.254.15/24
gateway=192.168.254.254
dns=192.168.254.254;

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
[root@control-node ~]# nmcli connection reload

[root@control-node ~]# nmcli connection up enp0s3

-----

aadarkdk@pop-os:~$ ssh cnode@192.168.254.15
The authenticity of host '192.168.254.15 (192.168.254.15)' can't be established.
ED25519 key fingerprint is SHA256:956Jc2yJk4RNMZDdU2rbRUixJZKHs3BvDT+S/iSfq3E.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:101: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.254.15' (ED25519) to the list of known hosts.
cnode@192.168.254.15's password: 
Last login: Mon Aug 31 22:07:18 2026 from 192.168.254.152
[cnode@control-node ~]$
 
[cnode@control-node ~]$ ip route
default via 192.168.254.254 dev enp0s3 proto static metric 100 
192.168.254.0/24 dev enp0s3 proto kernel scope link src 192.168.254.15 metric 100 

[cnode@control-node ~]$ hostname -I
192.168.254.15 

[cnode@control-node ~]$ hostname
control-node
[cnode@control-node ~]$
```

#### dev1  
```
aadarkdk@pop-os:~$ ssh cnode@192.168.254.252

[cnode@dev1 ~]$ hostname
dev1

[cnode@dev1 ~]$ su - root

[root@dev1 ~]# hostname -I
192.168.254.252 

[root@dev1 ~]# vi /etc/NetworkManager/system-connections/enp0s3.nmconnection 

[root@dev1 ~]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
...
[ipv4]
method=manual
address1=192.168.254.16/24
gateway=192.168.254.254
dns=192.168.254.254;
...

[root@dev1 ~]# nmcli connection reload

[root@dev1 ~]# nmcli connection up enp0s3

-----

aadarkdk@pop-os:~$ ssh cnode@192.168.254.16

[cnode@dev1 ~]$ hostname -I
192.168.254.16
 
[cnode@dev2 ~]$ hostname
dev1

[cnode@dev1 ~]$ su - root

[root@dev1 ~]# poweroff

[root@dev1 ~]# Connection to 192.168.254.16 closed by remote host.
Connection to 192.168.254.16 closed.
aadarkdk@pop-os:~$ 
```
               
#### dev2
```
aadarkdk@pop-os:~$ ssh cnode@192.168.254.251

[cnode@control-node ~]$ su - root

[root@control-node ~]# hostnamectl set-hostname dev2

[root@control-node ~]# hostname
dev2

[root@control-node ~]# hostname -I
192.168.254.251 

[root@control-node ~]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
...
[ipv4]
method=auto
...

[root@control-node ~]# vi /etc/NetworkManager/system-connections/enp0s3.nmconnection 

[root@control-node ~]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
...
[ipv4]
method=manual
address1=192.168.254.17/24
gateway=192.168.254.254
dns=192.168.254.254;
...

[root@control-node ~]# nmcli connection reload

[root@control-node ~]# nmcli connection up enp0s3

-----

aadarkdk@pop-os:~$ ssh cnode@192.168.254.17

[cnode@dev2 ~]$ whoami
cnode

[cnode@dev2 ~]$ hostname
dev2

[cnode@dev2 ~]$ hostname -I
192.168.254.17
[cnode@dev2 ~]$ 
```

#### testserver
```
aadarkdk@pop-os:~$ ssh cnode@192.168.254.253

[cnode@control-node ~]$ su - root

[root@control-node ~]# hostname
control-node

[root@control-node ~]# hostnamectl set-hostname testserver

[root@control-node ~]# hostname
testserver

[root@control-node ~]# hostname -I
192.168.254.253 

[root@control-node ~]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
...
[ipv4]
method=auto
...

[root@control-node ~]# vi /etc/NetworkManager/system-connections/enp0s3.nmconnection 

[root@control-node ~]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
...
[ipv4]
method=manual
address1=192.168.254.18/24
gateway=192.168.254.254
dns=192.168.254.254;
...

[root@control-node ~]# nmcli connection reload

[root@control-node ~]# nmcli connection up enp0s3

-----

aadarkdk@pop-os:~$ ssh cnode@192.168.254.18

[cnode@testserver ~]$ hostname
testserver

[cnode@testserver ~]$ hostname -I
192.168.254.18 

[cnode@testserver ~]$ su - root

[root@testserver ~]# poweroff

[root@testserver ~]# Connection to 192.168.254.18 closed by remote host.
Connection to 192.168.254.18 closed.
aadarkdk@pop-os:~$ 
```

#### prodserver
```
aadarkdk@pop-os:~$ ssh cnode@192.168.254.1
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:956Jc2yJk4RNMZDdU2rbRUixJZKHs3BvDT+S/iSfq3E.
Please contact your system administrator.
Add correct host key in /home/aadarkdk/.ssh/known_hosts to get rid of this message.
Offending ED25519 key in /home/aadarkdk/.ssh/known_hosts:91
  remove with:
  ssh-keygen -f "/home/aadarkdk/.ssh/known_hosts" -R "192.168.254.1"
Host key for 192.168.254.1 has changed and you have requested strict checking.
Host key verification failed.

aadarkdk@pop-os:~$ ssh-keygen -f "/home/aadarkdk/.ssh/known_hosts" -R "192.168.254.1" 
# Host 192.168.254.1 found: line 91
/home/aadarkdk/.ssh/known_hosts updated.
Original contents retained as /home/aadarkdk/.ssh/known_hosts.old

aadarkdk@pop-os:~$ ssh cnode@192.168.254.1

[cnode@control-node ~]$ hostname
control-node

[cnode@control-node ~]$ su - root

[root@control-node ~]# hostnamectl set-hostname prodserver

[root@control-node ~]# hostname
prodserver

[root@control-node ~]# hostname -I
192.168.254.1 

[root@control-node ~]# cat /etc/redhat-release 
CentOS Stream release 10 (Coughlan)

[root@control-node ~]# vi /etc/NetworkManager/system-connections/enp0s3.nmconnection 

[root@control-node ~]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
...
[ipv4]
method=manual
address1=192.168.254.19/24
gateway=192.168.254.254
dns=192.168.254.254;
...

[root@control-node ~]# nmcli connection reload

[root@control-node ~]# nmcli connection up enp0s3

-----

aadarkdk@pop-os:~$ ssh cnode@192.168.254.19
...

[cnode@prodserver ~]$ hostname
prodserver

[cnode@prodserver ~]$ hostname -I
192.168.254.19 

[cnode@prodserver ~]$ 
Broadcast message from root@control-node on tty1 (Tue 2026-09-01 05:33:52 +0545):

The system will power off now!

Connection to 192.168.254.19 closed by remote host.
Connection to 192.168.254.19 closed.
aadarkdk@pop-os:~$ 
```

---

```
# Verify Initial SSH Host-Key Identity:
  # The VMs were cloned from a common source VM and initially inherited
  # the same SSH host ED25519 key.
  # Verify the ED25519 host-key fingerprint on each VM. Identical
  # fingerprints confirm that the cloned systems share the same SSH
  # host identity and therefore require host-key regeneration before
  # being treated as independent hosts.

[cnode@control-node ~]$ hostname
control-node
[cnode@control-node ~]$ hostname -I
192.168.254.15
[cnode@control-node ~]$ sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
[sudo] password for cnode: 
256 SHA256:956Jc2yJk4RNMZDdU2rbRUixJZKHs3BvDT+S/iSfq3E no comment (ED25519)
[cnode@control-node ~]$ 

[cnode@dev1 ~]$ hostname
dev1
[cnode@dev1 ~]$ hostname -I
192.168.254.16 
[cnode@dev1 ~]$ sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub 
[sudo] password for cnode: 
256 SHA256:956Jc2yJk4RNMZDdU2rbRUixJZKHs3BvDT+S/iSfq3E no comment (ED25519)
[cnode@dev1 ~]$

[cnode@dev2 ~]$ hostname
dev2
[cnode@dev2 ~]$ hostname -I
192.168.254.17 
[cnode@dev2 ~]$ sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub 
[sudo] password for cnode: 
256 SHA256:956Jc2yJk4RNMZDdU2rbRUixJZKHs3BvDT+S/iSfq3E no comment (ED25519)
[cnode@dev2 ~]$ 

[cnode@testserver ~]$ hostname
testserver
[cnode@testserver ~]$ hostname -I
192.168.254.18 
[cnode@testserver ~]$ sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub 
[sudo] password for cnode: 
256 SHA256:956Jc2yJk4RNMZDdU2rbRUixJZKHs3BvDT+S/iSfq3E no comment (ED25519)
[cnode@testserver ~]$ 

[cnode@prodserver ~]$ hostname
prodserver
[cnode@prodserver ~]$ hostname -I
192.168.254.19 
[cnode@prodserver ~]$ sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
[sudo] password for cnode: 
256 SHA256:956Jc2yJk4RNMZDdU2rbRUixJZKHs3BvDT+S/iSfq3E no comment (ED25519)
[cnode@prodserver ~]$ 
```

---

```
# SSH Host Key Regeneration:
  # The VMs were cloned from a common source image, resulting in identical
  # SSH host keys across the cloned systems.
  # Regenerate the SSH host keys on each VM so that every host has a unique
  # SSH identity and fingerprint.
  # Verify that the regenerated ED25519 host key has a unique fingerprint.

[root@control-node ~]# rm -f /etc/ssh/ssh_host_*
[root@control-node ~]# ssh-keygen -A
ssh-keygen: generating new host keys: RSA ECDSA ED25519 
[root@control-node ~]# systemctl restart sshd
[root@control-node ~]# ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub 
256 SHA256:Ma/dLVjXXyx0ylf3aL+0YS+tSG0kUDg1w0Vw92j7l9M root@control-node (ED25519)
[root@control-node ~]# 

[root@dev1 ~]# rm -f /etc/ssh/ssh_host_*
[root@dev1 ~]# ssh-keygen -A
ssh-keygen: generating new host keys: RSA ECDSA ED25519 
[root@dev1 ~]# systemctl restart sshd
[root@dev1 ~]# ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub 
256 SHA256:azfUA0s7EbsCFZDBGkNh63Nu1LYJz6wNWfExzUul8ac root@dev1 (ED25519)
[root@dev1 ~]# 

[root@dev2 ~]# rm -f /etc/ssh/ssh_host_*
[root@dev2 ~]# ssh-keygen -A
ssh-keygen: generating new host keys: RSA ECDSA ED25519 
[root@dev2 ~]# systemctl restart sshd
[root@dev2 ~]# ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub 
256 SHA256:6mV3gOc4p76Pkubhz6MHKgOoSCL2f74WTDb7N4qSiaE root@dev2 (ED25519)
[root@dev2 ~]# 

[root@testserver ~]# rm -f /etc/ssh/ssh_host_*
[root@testserver ~]# ssh-keygen -A
ssh-keygen: generating new host keys: RSA ECDSA ED25519 
[root@testserver ~]# systemctl restart sshd
[root@testserver ~]# ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub 
256 SHA256:ZH30Fj4KAiCsOpaFqBCOvDgseLAdqDuCMIBST0aSxjw root@testserver (ED25519)
[root@testserver ~]# 

[root@prodserver ~]# rm -f /etc/ssh/ssh_host_*
[root@prodserver ~]# ssh-keygen -A
ssh-keygen: generating new host keys: RSA ECDSA ED25519
[root@prodserver ~]# systemctl restart sshd
[root@prodserver ~]# ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
256 SHA256:tPhjZ9BtNDjeq3dSdRHkh5yAf4H5YtScxGR11/C/9uU root@prodserver (ED25519)
[root@prodserver ~]#
```

---

```
# System Machine-ID Regeneration / MAC Verification:
  # The VMs were cloned from a common source image, resulting in identical system machine IDs.
  # Regenerate the system machine ID on the cloned VMs so that each system
  # has a unique machine identity, consistent with independently provisioned hosts.

  # Also
  # Verify that each cloned VM has a unique network interface MAC address. ( ip link show enp0s3 )

[root@control-node ~]# ip link show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:4a:48:12 brd ff:ff:ff:ff:ff:ff
    altname enx0800274a4812
[root@control-node ~]# cat /etc/machine-id
ad75d97adc314f43901d793e15c0b95b
[root@control-node ~]#

[root@dev1 ~]# ip link show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:e2:51:dd brd ff:ff:ff:ff:ff:ff
    altname enx080027e251dd
[root@dev1 ~]# cat /etc/machine-id 
ad75d97adc314f43901d793e15c0b95b
[root@dev1 ~]# rm -f /etc/machine-id
[root@dev1 ~]# systemd-machine-id-setup
Initializing machine ID from random generator.
[root@dev1 ~]# cat /etc/machine-id 
262365ca06544d0a8cddf54f90ade952
[root@dev1 ~]# 

[root@dev2 ~]# ip link show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:5d:5c:6c brd ff:ff:ff:ff:ff:ff
    altname enx0800275d5c6c
[root@dev2 ~]# cat /etc/machine-id 
ad75d97adc314f43901d793e15c0b95b
[root@dev2 ~]# cat /etc/machine-id 
ad75d97adc314f43901d793e15c0b95b
[root@dev2 ~]# rm -f /etc/machine-id
[root@dev2 ~]# systemd-machine-id-setup 
Initializing machine ID from random generator.
[root@dev2 ~]# cat /etc/machine-id 
c733130f00a9469ba5efcb0f9ff233dd
[root@dev2 ~]# 

[root@testserver ~]# ip link show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:e3:6f:f1 brd ff:ff:ff:ff:ff:ff
    altname enx080027e36ff1
[root@testserver ~]# cat /etc/machine-id 
ad75d97adc314f43901d793e15c0b95b
[root@testserver ~]# cat /etc/machine-id 
ad75d97adc314f43901d793e15c0b95b
[root@testserver ~]# rm -f /etc/machine-id 
[root@testserver ~]# systemd-machine-id-setup
Initializing machine ID from random generator.
[root@testserver ~]# cat /etc/machine-id 
ddb9f838c83a4ec9836a59a4d29ede2b
[root@testserver ~]# 

[root@prodserver ~]# ip link show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:8d:95:5e brd ff:ff:ff:ff:ff:ff
    altname enx0800278d955e
[root@prodserver ~]# cat /etc/machine-id 
ad75d97adc314f43901d793e15c0b95b
[root@prodserver ~]# rm -f /etc/machine-id 
[root@prodserver ~]# cat /etc/machine-id
cat: /etc/machine-id: No such file or directory
[root@prodserver ~]# systemd-machine-id-setup 
Initializing machine ID from random generator.
[root@prodserver ~]# cat /etc/machine-id 
2dcb7a8265d246e2943745c1dca20f14
[root@prodserver ~]#
```
---

```
# Managed Host Baseline Verification:
  # Verify the OS version, Python runtime, SSH service, and SELinux status
  # on all managed hosts before proceeding with Ansible configuration.
  #
  # Expected:
  #   OS:      CentOS Stream 10 (Coughlan)
  #   Python:  3.12.x
  #   SSHD:    enabled and active
  #   SELinux: Enforcing

[root@prodserver ~]# cat /etc/redhat-release 
CentOS Stream release 10 (Coughlan)
[root@prodserver ~]# python3 --version
Python 3.12.13
[root@prodserver ~]# systemctl is-enabled sshd
enabled
[root@prodserver ~]# systemctl is-active sshd
active
[root@prodserver ~]# getenforce
Enforcing
[root@prodserver ~]#

[root@testserver ~]# cat /etc/redhat-release 
CentOS Stream release 10 (Coughlan)
[root@testserver ~]# python3 --version
Python 3.12.13
[root@testserver ~]# systemctl is-enabled sshd
enabled
[root@testserver ~]# systemctl is-active sshd
active
[root@testserver ~]# getenforce
Enforcing
[root@testserver ~]# 

[root@dev1 ~]# cat /etc/redhat-release 
CentOS Stream release 10 (Coughlan)
[root@dev1 ~]# python3 --version
Python 3.12.13
[root@dev1 ~]# systemctl is-enabled sshd
enabled
[root@dev1 ~]# systemctl is-active sshd
active
[root@dev1 ~]# getenforce
Enforcing
[root@dev1 ~]# 

[root@dev2 ~]# cat /etc/redhat-release 
CentOS Stream release 10 (Coughlan)
[root@dev2 ~]# python3 --version
Python 3.12.13
[root@dev2 ~]# systemctl is-enabled sshd
enabled
[root@dev2 ~]# systemctl is-active sshd
active
[root@dev2 ~]# getenforce
Enforcing
[root@dev2 ~]# 
```

---

```
# Ansible Control Node Preparation:
  # Verify the control-node operating system, Python runtime,
  # SSH service, SELinux status, and user environment.

[root@control-node ~]# cat /etc/redhat-release 
CentOS Stream release 10 (Coughlan)
[root@control-node ~]# python3 --version
Python 3.12.13
[root@control-node ~]# systemctl is-enabled sshd
enabled
[root@control-node ~]# systemctl is-active sshd
active
[root@control-node ~]# getenforce
Enforcing 
[root@control-node ~]# ls -la ~/.ssh/
total 0
drwx------. 2 root root   6 Aug 31 18:45 .
dr-xr-x---. 3 root root 147 Aug 31 19:06 ..
[root@control-node ~]# exit
logout
[cnode@control-node ~]$ echo $HOME
/home/cnode 
[cnode@control-node ~]$ whoami
cnode
[cnode@control-node ~]$
```

---

```
# Hostname Resolution:
  # Configure static hostname resolution for the Ansible control node and
  # all managed hosts.
  # Verify that each hostname resolves to the expected IP address.
 
[cnode@control-node ~]$ sudo vi /etc/hosts
[sudo] password for cnode: 
[cnode@control-node ~]$ sudo cat /etc/hosts
...
# Static hostname-to-IP mappings for the Ansible environment
192.168.254.15 control-node
192.168.254.16 dev1
192.168.254.17 dev2
192.168.254.18 testserver
192.168.254.19 prodserver

[cnode@control-node ~]$ getent hosts dev1
192.168.254.16  dev1
[cnode@control-node ~]$ getent hosts dev2
192.168.254.17  dev2
[cnode@control-node ~]$ getent hosts testserver
192.168.254.18  testserver
[cnode@control-node ~]$ getent hosts prodserver
192.168.254.19  prodserver
```

---

```
# Initial SSH Host Verification:
  # Establish initial SSH connectivity to each managed host.
  # Verify each host-key fingerprint before accepting it into known_hosts.
  # Password authentication is used temporarily for initial access.

[cnode@control-node ~]$ ssh cnode@dev1
The authenticity of host 'dev1 (192.168.254.16)' can't be established.
ED25519 key fingerprint is SHA256:azfUA0s7EbsCFZDBGkNh63Nu1LYJz6wNWfExzUul8ac.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'dev1' (ED25519) to the list of known hosts.
cnode@dev1's password: 
Last login: Tue Sep  1 12:40:10 2026 from 192.168.254.152
[cnode@dev1 ~]$ hostname
dev1
[cnode@dev1 ~]$ exit
logout
Connection to dev1 closed.
 
  # similarly,
  
[cnode@control-node ~]$ ssh cnode@dev2
...
[cnode@dev2 ~]$ hostname
dev2
[cnode@dev2 ~]$ exit
logout
Connection to dev2 closed.

[cnode@control-node ~]$ ssh cnode@testserver
...
[cnode@testserver ~]$ hostname
testserver
[cnode@testserver ~]$ exit
logout
Connection to testserver closed.

[cnode@control-node ~]$ ssh cnode@prodserver
...
[cnode@prodserver ~]$ hostname
prodserver
[cnode@prodserver ~]$ exit
logout
Connection to prodserver closed.
```

---

```
# SSH Key-Based Authentication:
  # Generate an ED25519 user key pair for Ansible SSH authentication.
  # The private key remains securely on the control node.
  # The public key is deployed to the cnode account on each managed host.

[cnode@control-node ~]$ ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
Generating public/private ed25519 key pair.
Created directory '/home/cnode/.ssh'.
Enter passphrase for "/home/cnode/.ssh/id_ed25519" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/cnode/.ssh/id_ed25519
Your public key has been saved in /home/cnode/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:JID9AkYQTHuKJ/pOF6jBQubgAc3/1i+g+dC6gK6pc/s cnode@control-node
The key's randomart image is:
+--[ED25519 256]--+
|*B.o.            |
|..B ..           |
|.* + .. .        |
|O +.o .o         |
|==o .o .S        |
|o*   o+ .        |
|+ o o+.. .       |
|o+.ooo  . .      |
|*=+oEo.  .       |
+----[SHA256]-----+
[cnode@control-node ~]$ ls -la ~/.ssh/
total 8
drwx------. 2 cnode cnode  46 Sep  1 15:05 .
drwx------. 3 cnode cnode  95 Sep  1 15:05 ..
-rw-------. 1 cnode cnode 411 Sep  1 15:05 id_ed25519
-rw-r--r--. 1 cnode cnode 100 Sep  1 15:05 id_ed25519.pub
[cnode@control-node ~]$ ssh-keygen -lf ~/.ssh/id_ed25519.pub 
256 SHA256:JID9AkYQTHuKJ/pOF6jBQubgAc3/1i+g+dC6gK6pc/s cnode@control-node (ED25519)
[cnode@control-node ~]$ 
```

---

```
# Deploy SSH Public Key:
  # Install the control-node cnode user's public key in the
  # ~/.ssh/authorized_keys file of each managed host.
  # This enables subsequent SSH public-key authentication without an interactive password prompt.

[cnode@control-node ~]$ ssh-copy-id -i ~/.ssh/id_ed25519.pub cnode@dev1
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/cnode/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
cnode@dev1's password: 

Number of key(s) added: 1

Now try logging into the machine, with: "ssh -i /home/cnode/.ssh/id_ed25519 'cnode@dev1'"
and check to make sure that only the key(s) you wanted were added.

[cnode@control-node ~]$

# similarly for other machines also

[cnode@control-node ~]$ ssh-copy-id -i ~/.ssh/id_ed25519.pub cnode@dev2

[cnode@control-node ~]$ ssh-copy-id -i ~/.ssh/id_ed25519.pub cnode@testserver

[cnode@control-node ~]$ ssh-copy-id -i ~/.ssh/id_ed25519.pub cnode@prodserver
```

---

```
# Verify Passwordless SSH / Public-Key Authentication :
  # Verify SSH connectivity from the Ansible control node to managed hosts
  # using public-key authentication without an interactive password prompt.

[cnode@control-node ~]$ ssh cnode@dev1
Last login: Tue Sep  1 15:30:35 2026 from 192.168.254.15
[cnode@dev1 ~]$ hostname
dev1
[cnode@dev1 ~]$ exit
logout
Connection to dev1 closed.

[cnode@control-node ~]$ ssh cnode@dev2
Last login: Tue Sep  1 15:30:34 2026 from 192.168.254.15
[cnode@dev2 ~]$ hostname
dev2
[cnode@dev2 ~]$ exit
logout
Connection to dev2 closed.

[cnode@control-node ~]$ ssh cnode@testserver
Last login: Tue Sep  1 15:31:04 2026 from 192.168.254.15
[cnode@testserver ~]$ hostname
testserver
[cnode@testserver ~]$ exit
logout
Connection to testserver closed.

[cnode@control-node ~]$ ssh cnode@prodserver
Last login: Tue Sep  1 15:31:46 2026 from 192.168.254.15
[cnode@prodserver ~]$ hostname
prodserver
[cnode@prodserver ~]$ exit
logout
Connection to prodserver closed.
[cnode@control-node ~]$ 
```

---

```
# Configure Passwordless Sudo for Ansible Privilege Escalation:
  # Allow the cnode user to execute administrative commands via sudo
  # without requiring an interactive password

[cnode@control-node ~]$ ssh cnode@dev1
Last login: Tue Sep  1 15:36:07 2026 from 192.168.254.15
[cnode@dev1 ~]$ sudo -i
[sudo] password for cnode: 
[root@dev1 ~]# visudo -f /etc/sudoers.d/cnode
[root@dev1 ~]# chmod 0440 /etc/sudoers.d/cnode 
[root@dev1 ~]# ls -l /etc/sudoers.d/cnode
-r--r-----. 1 root root 30 Sep  1 16:09 /etc/sudoers.d/cnode
[root@dev1 ~]# visudo -c
/etc/sudoers: parsed OK
/etc/sudoers.d/cnode: parsed OK
[root@dev1 ~]# su - cnode
Last login: Tue Sep  1 16:08:16 +0545 2026 from 192.168.254.15 on pts/2
[cnode@dev1 ~]$ sudo -n whoami
root
[cnode@dev1 ~]$ exit
logout
[root@dev1 ~]# exit
logout
[cnode@dev1 ~]$ exit
logout
Connection to dev1 closed.

[cnode@control-node ~]$ ssh cnode@dev2
Last login: Tue Sep  1 15:35:51 2026 from 192.168.254.15
[cnode@dev2 ~]$ sudo -i
[sudo] password for cnode: 
[root@dev2 ~]# visudo -f /etc/sudoers.d/cnode
[root@dev2 ~]# chmod 0440 /etc/sudoers.d/cnode
[root@dev2 ~]# visudo -c
/etc/sudoers: parsed OK
/etc/sudoers.d/cnode: parsed OK
[root@dev2 ~]# su - cnode
Last login: Tue Sep  1 16:47:28 +0545 2026 from 192.168.254.15 on pts/2
[cnode@dev2 ~]$ sudo -n whoami
root
[cnode@dev2 ~]$ exit
logout
[root@dev2 ~]# exit
logout
[cnode@dev2 ~]$ exit
logout
Connection to dev2 closed.

[cnode@control-node ~]$ ssh cnode@testserver
Last login: Tue Sep  1 15:36:12 2026 from 192.168.254.15
[cnode@testserver ~]$ sudo -i
[sudo] password for cnode: 
[root@testserver ~]# visudo -f /etc/sudoers.d/cnode
[root@testserver ~]# chmod 0440 /etc/sudoers.d/cnode
[root@testserver ~]# visudo -c
/etc/sudoers: parsed OK
/etc/sudoers.d/cnode: parsed OK
[root@testserver ~]# su - cnode
Last login: Tue Sep  1 16:48:44 +0545 2026 from 192.168.254.15 on pts/2
[cnode@testserver ~]$ sudo -n whoami
root
[cnode@testserver ~]$ exit
logout
[root@testserver ~]# exit
logout
[cnode@testserver ~]$ exit
logout
Connection to testserver closed.

[cnode@control-node ~]$ ssh cnode@prodserver
Last login: Tue Sep  1 16:03:41 2026 from 192.168.254.15
[cnode@prodserver ~]$ sudo -i
[sudo] password for cnode: 
[root@prodserver ~]# visudo -f /etc/sudoers.d/cnode
[root@prodserver ~]# chmod 0440 /etc/sudoers.d/cnode
[root@prodserver ~]# visudo -c
/etc/sudoers: parsed OK
/etc/sudoers.d/cnode: parsed OK
[root@prodserver ~]# su - cnode
Last login: Tue Sep  1 16:50:08 +0545 2026 from 192.168.254.15 on pts/1
[cnode@prodserver ~]$ sudo -n whoami
root
[cnode@prodserver ~]$ exit
logout
[root@prodserver ~]# exit
logout
[cnode@prodserver ~]$ exit
logout
Connection to prodserver closed.
[cnode@control-node ~]$ 
```
---

```
# Validation:
  # Verify SSH public-key authentication from the control node to all managed
  # hosts and confirm that the cnode account can obtain root privileges
  # non-interactively via sudo.
  #
  # Password authentication is disabled and BatchMode is enabled to ensure
  # the test succeeds only through non-interactive SSH authentication.

[cnode@control-node ~]$ ls
[cnode@control-node ~]$ vi validate_nodes.sh
[cnode@control-node ~]$ cat validate_nodes.sh 
#!/bin/bash

nodes=(dev1 dev2 testserver prodserver)

for host in "${nodes[@]}";
do
   echo "----- $host -----"

   ssh -o BatchMode=yes \
       -o PasswordAuthentication=no \
       "cnode@$host" 'hostname; sudo -n whoami'
    
   echo
done
[cnode@control-node ~]$ chmod +x validate_nodes.sh
[cnode@control-node ~]$ ./validate_nodes.sh
----- dev1 -----
dev1
root

----- dev2 -----
dev2
root

----- testserver -----
testserver
root

----- prodserver -----
prodserver
root

[cnode@control-node ~]$ 
```
---