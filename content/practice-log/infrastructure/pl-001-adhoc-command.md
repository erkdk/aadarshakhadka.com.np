---
title: "PL - 001 — Installation ansible-core and Ad-hoc commands"
date: 2026-09-03
draft: false
---

# Ansible Ad-Hoc Commands

Ansible **ad-hoc commands** are quick, one-line commands executed from the
control node using the `ansible` command-line tool.

They are useful for performing a **single task** across one or more managed
hosts without writing a reusable Ansible playbook.

Ad-hoc commands are particularly useful for:

- Testing connectivity to managed hosts
- Gathering system information
- Managing users and groups
- Managing files and directories
- Installing or removing packages
- Starting, stopping, or restarting services
- Executing commands on multiple hosts
- Performing quick administrative tasks

---

## 1. Basic Syntax

The general syntax of an Ansible ad-hoc command is:

```bash
   ansible <host-pattern> -m <module-name> -a "<module-arguments>" [options]
```
```
<pattern>	     --->  Specifies which managed hosts should be targeted
-m	             --->  Specifies the Ansible module to use
<module>	     --->  The module that performs the task
-a	             --->  Provides arguments to the module
<module arguments>   --->  Parameters required by that module
```
---
```
 # Install required packages on the Control Node
 
[cnode@control-node ~]$ hostname
control-node

[cnode@control-node ~]$ hostname -I
192.168.254.15 

[cnode@control-node ~]$ rpm -q ansible-core
package ansible-core is not installed

[cnode@control-node ~]$ ls /etc/yum.repos.d/
centos-addons.repo  centos.repo

[cnode@control-node ~]$ su - root
...

[root@control-node ~]# dnf install -y ansible-core
...
Installed:
  ansible-core-1:2.16.19-2.el10.noarch     git-core-2.52.0-1.el10.x86_64             python3-cffi-1.16.0-7.el10.x86_64       python3-cryptography-49.0.0-1.el10.x86_64   
  python3-jinja2-3.1.6-1.el10.noarch       python3-markupsafe-2.1.3-6.el10.x86_64    python3-packaging-24.2-2.el10.noarch    python3-ply-3.11-25.el10.noarch             
  python3-pycparser-2.20-16.el10.noarch    python3-resolvelib-1.0.1-6.el10.noarch   

Complete!
[root@control-node ~]# 

[root@control-node ~]# rpm -q ansible-core
ansible-core-2.16.19-2.el10.noarch

[root@control-node ~]# which ansible-core
/usr/bin/which: no ansible-core in (/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin)
[root@control-node ~]# ansible --version
ansible [core 2.16.19]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.12/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.12.13 (main, Apr 16 2026, 00:00:00) [GCC 14.3.1 20251022 (Red Hat 14.3.1-4)] (/usr/bin/python3)
  jinja version = 3.1.6
  libyaml = True
[root@control-node ~]# 

[root@control-node ~]# exit
logout
[cnode@control-node ~]$ whoami
cnode
[cnode@control-node ~]$ hostname
control-node
[cnode@control-node ~]$ 

[cnode@control-node ~]$ ls
validate_nodes.sh
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

```
# Create ansible project

[cnode@control-node ~]$ mkdir ansible-lab

[cnode@control-node ~]$ cd ansible-lab/

[cnode@control-node ansible-lab]$ vi inventory

[cnode@control-node ansible-lab]$ cat inventory
192.168.254.16
192.168.254.17
192.168.254.18
192.168.254.19
[cnode@control-node ansible-lab]$
[cnode@control-node ansible-lab]$ cat /etc/hosts
# Loopback entries; do not change.
# For historical reasons, localhost precedes localhost.localdomain:
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
# See hosts(5) for proper format and other examples:
# 192.168.1.10 foo.example.org foo
# 192.168.1.13 bar.example.org bar

# add IPs for 5 nodes
192.168.254.15 control-node
192.168.254.16 dev1
192.168.254.17 dev2
192.168.254.18 testserver
192.168.254.19 prodserver
[cnode@control-node ansible-lab]$

[cnode@control-node ansible-lab]$ vim inventory 

[cnode@control-node ansible-lab]$ cat inventory 
[develop]
dev1
dev2

[test]
testserver

[production]
prodserver

[all:vars]
ansible_user=cnode
ansible_become=true
ansible_become_method=sudo
[cnode@control-node ansible-lab]$ vim inventory 

[cnode@control-node ansible-lab]$ cat inventory 
[develop]
dev1 ansible_host=192.168.254.16
dev2 ansible_host=192.168.254.17

[test]
testserver

[production]
prodserver

[all:vars]
ansible_user=cnode
ansible_become=true
ansible_become_method=sudo
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ vim inventory 

[cnode@control-node ansible-lab]$ cat inventory 
[develop]
dev1
dev2

[test]
testserver

[production]
prodserver

[all:vars]
ansible_user=cnode
ansible_become=true
ansible_become_method=sudo
[cnode@control-node ansible-lab]$ 
```

---

```
[cnode@control-node ansible-lab]$ ls
inventory
[cnode@control-node ansible-lab]$ ansible-inventory --graph
@all:
  |--@ungrouped:
[cnode@control-node ansible-lab]$ ansible-inventory -i inventory --graph
@all:
  |--@ungrouped:
  |--@develop:
  |  |--dev1
  |  |--dev2
  |--@test:
  |  |--testserver
  |--@production:
  |  |--prodserver
  
[cnode@control-node ansible-lab]$ ansible all -i inventory -m ping
testserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
dev2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
prodserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
dev1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible develop -i inventory -m ping
dev1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
dev2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
[cnode@control-node ansible-lab]$ ansible test -i inventory -m ping
testserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
[cnode@control-node ansible-lab]$

[cnode@control-node ansible-lab]$ pwd
/home/cnode/ansible-lab
[cnode@control-node ansible-lab]$ ansible develop -i inventory --list-hosts
  hosts (2):
    dev1
    dev2
[cnode@control-node ansible-lab]$ ansible production -i inventory --list-hosts
  hosts (1):
    prodserver
[cnode@control-node ansible-lab]$ ansible all -i inventory --list-hosts
  hosts (4):
    dev1
    dev2
    testserver
    prodserver
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible test,production -i inventory --list-hosts
  hosts (2):
    testserver
    prodserver
[cnode@control-node ansible-lab]$
```

---

```
 # Creating Ansible config file
 
[cnode@control-node ansible-lab]$ ls
inventory
[cnode@control-node ansible-lab]$ pwd
/home/cnode/ansible-lab
[cnode@control-node ansible-lab]$ vim ansible.cfg
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ cat ansible.cfg 
[defaults]
inventory=/home/cnode/ansible-lab/inventory
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible develop --list-hosts
  hosts (2):
    dev1
    dev2
[cnode@control-node ansible-lab]$ ansible all --list-hosts
  hosts (4):
    dev1
    dev2
    testserver
    prodserver
    
[cnode@control-node ansible-lab]$ ansible all -m ping
dev2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
prodserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
testserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
dev1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
[cnode@control-node ansible-lab]$   

```

---

```
[cnode@control-node ansible-lab]$ ansible --version
ansible [core 2.16.19]
  config file = /home/cnode/ansible-lab/ansible.cfg
  configured module search path = ['/home/cnode/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.12/site-packages/ansible
  ansible collection location = /home/cnode/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.12.13 (main, Apr 16 2026, 00:00:00) [GCC 14.3.1 20251022 (Red Hat 14.3.1-4)] (/usr/bin/python3)
  jinja version = 3.1.6
  libyaml = True
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ls /etc/ansible/
ansible.cfg  hosts  roles
[cnode@control-node ansible-lab]$

[cnode@control-node ansible-lab]$ cat /etc/ansible/ansible.cfg 
# Since Ansible 2.12 (core):
# To generate an example config file (a "disabled" one with all default settings, commented out):
#               $ ansible-config init --disabled > ansible.cfg
#
# Also you can now have a more complete file by including existing plugins:
# ansible-config init --disabled -t all > ansible.cfg

# For previous versions of Ansible you can check for examples in the 'stable' branches of each version
# Note that this file was always incomplete  and lagging changes to configuration settings

# for example, for 2.9: https://github.com/ansible/ansible/blob/stable-2.9/examples/ansible.cfg
[cnode@control-node ansible-lab]$

```

---

Getting help of Ansible modules
```
[cnode@control-node ansible-lab]$ ansible-doc user

[cnode@control-node ansible-lab]$ ansible-doc ansible.builtin.user
...
/EXAMPLES
...
:q

```

---

```
[cnode@control-node ansible-lab]$ vi ansible.cfg 
[cnode@control-node ansible-lab]$ cat ansible.cfg 
[defaults]
inventory=/home/cnode/ansible-lab/inventory
remote_user=cnode
[cnode@control-node ansible-lab]$ vi ansible.cfg 
[cnode@control-node ansible-lab]$ vim ansible.cfg 
[cnode@control-node ansible-lab]$ cat ansible.cfg 
[defaults]
inventory=/home/cnode/ansible-lab/inventory
remote_user=cnode

[privilege_escalation]
become=true
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible test -m ansible.builtin.user -a 'name=testuser'
testserver | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1001,
    "home": "/home/testuser",
    "name": "testuser",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1001
}
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible test -m ansible.builtin.user -a 'name=testuser'
testserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "append": false,
    "changed": false,
    "comment": "",
    "group": 1001,
    "home": "/home/testuser",
    "move_home": false,
    "name": "testuser",
    "shell": "/bin/bash",
    "state": "present",
    "uid": 1001
}
[cnode@control-node ansible-lab]$ 

# In managed node: testserver

[cnode@testserver ~]$ grep testuser /etc/passwd
testuser:x:1001:1001::/home/testuser:/bin/bash
[cnode@testserver ~]$ 

```
---