---
title: "PL - 002 — Ansible Fundamentals: Inventory, Ad-Hoc Commands, and Collections"
date: 2026-09-06
draft: false
---

```
[cnode@control-node ~]$ pwd
/home/cnode
[cnode@control-node ~]$ ls
ansible-lab  validate_nodes.sh
[cnode@control-node ~]$ cd ansible-lab/
[cnode@control-node ansible-lab]$ pwd
/home/cnode/ansible-lab
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ cat ansible.cfg 
[defaults]
inventory=/home/cnode/ansible-lab/inventory
remote_user=cnode

[privilege_escalation]
become=true
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
[cnode@control-node ansible-lab]$ vim ansible.cfg
[cnode@control-node ansible-lab]$ cat inventory 
[develop]
dev1
dev2

[test]
testserver

[production]
prodserver

[testprod:children]
test
prodserver
[cnode@control-node ansible-lab]$ cat ansible.cfg
[defaults]
inventory = ./inventory
remote_user = cnode

[privilege_escalation]
become = true
become_method = sudo

[cnode@control-node ansible-lab]$ vim inventory 
[cnode@control-node ansible-lab]$ cat inventory 
[develop]
dev1
dev2

[test]
testserver

[production]
prodserver

[testprod:children]
test
production
[cnode@control-node ansible-lab]$ ansible-inventory --graph
@all:
  |--@ungrouped:
  |--@develop:
  |  |--dev1
  |  |--dev2
  |--@testprod:
  |  |--@test:
  |  |  |--testserver
  |  |--@production:
  |  |  |--prodserver
[cnode@control-node ansible-lab]$ ansible all -m ping
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
prodserver | SUCCESS => {
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
[cnode@control-node ansible-lab]$ 
```

---

```
[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.user -a 'name=testuser state=absent'
prodserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "name": "testuser",
    "state": "absent"
}
testserver | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "force": false,
    "name": "testuser",
    "remove": false,
    "state": "absent"
}
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.command -a 'sudo useradd testuser'
prodserver | FAILED | rc=9 >>
useradd: user 'testuser' already existsnon-zero return code
testserver | FAILED | rc=9 >>
useradd: user 'testuser' already existsnon-zero return code
[cnode@control-node ansible-lab]$ echo $?
0
[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.command -a 'id testuser'
testserver | CHANGED | rc=0 >>
uid=1001(testuser) gid=1001(testuser) groups=1001(testuser)
prodserver | CHANGED | rc=0 >>
uid=1001(testuser) gid=1001(testuser) groups=1001(testuser)
[cnode@control-node ansible-lab]$ 

# before deletion

[cnode@testserver ~]$ grep testuser /etc/passwd
testuser:x:1001:1001::/home/testuser:/bin/bash

[cnode@prodserver ~]$ grep testuser /etc/passwd
testuser:x:1001:1001::/home/testuser:/bin/bash
 
# after deletion

[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.user -a 'name=testuser state=absent remove=yes'
prodserver | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "force": false,
    "name": "testuser",
    "remove": true,
    "state": "absent"
}
testserver | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "force": false,
    "name": "testuser",
    "remove": true,
    "state": "absent"
}
[cnode@control-node ansible-lab]$ 

# and

[cnode@prodserver ~]$ grep testuser /etc/passwd
[cnode@prodserver ~]$ 

[cnode@testserver ~]$ grep testuser /etc/passwd
[cnode@testserver ~]$ 

[cnode@control-node ansible-lab]$ ansible testprod -m command -a 'id testuser'
testserver | FAILED | rc=1 >>
id: ‘testuser’: no such usernon-zero return code
prodserver | FAILED | rc=1 >>
id: ‘testuser’: no such usernon-zero return code
[cnode@control-node ansible-lab]$ echo $?
2
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
[cnode@control-node ansible-lab]$ ansible-galaxy --version
ansible-galaxy [core 2.16.19]
  config file = /home/cnode/ansible-lab/ansible.cfg
  configured module search path = ['/home/cnode/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.12/site-packages/ansible
  ansible collection location = /home/cnode/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible-galaxy
  python version = 3.12.13 (main, Apr 16 2026, 00:00:00) [GCC 14.3.1 20251022 (Red Hat 14.3.1-4)] (/usr/bin/python3)
  jinja version = 3.1.6
  libyaml = True
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-galaxy collection list

# /usr/share/ansible/collections/ansible_collections
Collection               Version
------------------------ -------
microsoft.sql            2.6.6  
redhat.leapp             1.7.6  
redhat.rhel_system_roles 2.4.2  
[cnode@control-node ansible-lab]$

[cnode@control-node ansible-lab]$ sudo dnf list *roles*
Last metadata expiration check: 0:17:15 ago on Mon 07 Sep 2026 01:22:42 PM +0545.
Installed Packages
rhel-system-roles.noarch                                                              2.4.2-0.1.el10                                                              @appstream
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ sudo dnf -y install rhel-system-roles
Last metadata expiration check: 0:33:48 ago on Mon 07 Sep 2026 01:22:42 PM +0545.
Package rhel-system-roles-2.4.2-0.1.el10.noarch is already installed.
Dependencies resolved.
Nothing to do.
Complete!
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ dnf list ansible-core ansible
Last metadata expiration check: 0:01:41 ago on Mon 07 Sep 2026 01:58:36 PM +0545.
Installed Packages
ansible-core.noarch                                                               1:2.16.19-2.el10                                                                @appstream
[cnode@control-node ansible-lab]$ 

# To install collection, general syntax:

```
ansible-galaxy collection install <namespace>.<collection>
```
[cnode@control-node ansible-lab]$ ansible-galaxy collection install community.general
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/community-general-13.3.0.tar.gz to /home/cnode/.ansible/tmp/ansible-local-2435cqh60vx4/tmpp15y7osm/community-general-13.3.0-dnc2bc61
Installing 'community.general:13.3.0' to '/home/cnode/.ansible/collections/ansible_collections/community/general'
community.general:13.3.0 was installed successfully
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/community-library_inventory_filtering_v1-1.1.5.tar.gz to /home/cnode/.ansible/tmp/ansible-local-2435cqh60vx4/tmpp15y7osm/community-library_inventory_filtering_v1-1.1.5-h8b16zcm
Installing 'community.library_inventory_filtering_v1:1.1.5' to '/home/cnode/.ansible/collections/ansible_collections/community/library_inventory_filtering_v1'
community.library_inventory_filtering_v1:1.1.5 was installed successfully
[cnode@control-node ansible-lab]$
```

The general pattern is:
```
<vendor/organization>.<collection>
```
example:
```
ansible.posix
community.general
community.crypto
amazon.aws
cisco.ios
cisco.nxos
containers.podman
redhat.rhel_system_roles
```

```
[cnode@control-node ansible-lab]$ ansible-doc -l
ansible.builtin.add_host               Add a host (and alternatively a group) to...
ansible.builtin.apt                    Manages apt-packages                     
ansible.builtin.apt_key                Add or remove an apt key                 
ansible.builtin.apt_repository         Add and remove APT repositories          
... 
ansible.builtin.ping                   Try to connect to host, verify a usable p...
ansible.builtin.pip                    Manages Python library dependencies      
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ rpm -q ansible-core
ansible-core-2.16.19-2.el10.noarch
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ rpm -q ansible-core
ansible-core-2.16.19-2.el10.noarch
[cnode@control-node ansible-lab]$ sudo dnf list ansible*
[sudo] password for cnode: 
Last metadata expiration check: 1:06:40 ago on Sun 06 Sep 2026 06:51:36 PM +0545.
Error: No matching Packages to list
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ sudo dnf -y install ansible-collection*
Last metadata expiration check: 1:09:15 ago on Sun 06 Sep 2026 06:51:36 PM +0545.
Dependencies resolved.
=====================================================================================
 Package                             Arch      Version            Repository    Size
=====================================================================================
Installing:
 ansible-collection-microsoft-sql    noarch    2.6.6-1.el10       appstream    211 k
 ansible-collection-redhat-leapp     noarch    1.7.6-1.el10       appstream    174 k
Installing dependencies:
 rhel-system-roles                   noarch    2.4.2-0.1.el10     appstream    5.2 M

Transaction Summary
=====================================================================================
Install  3 Packages

Total download size: 5.6 M
Installed size: 28 M
Downloading Packages:
(1/3): ansible-collection-microsoft-sql-2.6.6-1.el10  57 kB/s | 211 kB     00:03    
(2/3): ansible-collection-redhat-leapp-1.7.6-1.el10.  29 kB/s | 174 kB     00:05    
[MIRROR] rhel-system-roles-2.4.2-0.1.el10.noarch.rpm: Curl error (56): Failure when receiving data from the peer for http://mirror.nevacloud.com/centos-stream/10-stream/AppStream/x86_64/os/Packages/rhel-system-roles-2.4.2-0.1.el10.noarch.rpm [Recv failure: Connection reset by peer]
[MIRROR] rhel-system-roles-2.4.2-0.1.el10.noarch.rpm: Curl error (56): Failure when receiving data from the peer for https://mirror.nevacloud.com/centos-stream/10-stream/AppStream/x86_64/os/Packages/rhel-system-roles-2.4.2-0.1.el10.noarch.rpm [OpenSSL SSL_read: SSL_ERROR_SYSCALL, errno 0]
(3/3): rhel-system-roles-2.4.2-0.1.el10.noarch.rpm    11 kB/s | 5.2 MB     07:44    
-------------------------------------------------------------------------------------
Total                                                 12 kB/s | 5.6 MB     07:46     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Running scriptlet: rhel-system-roles-2.4.2-0.1.el10.noarch                     1/1 
  Running scriptlet: ansible-collection-microsoft-sql-2.6.6-1.el10.noarch        1/1 
  Preparing        :                                                             1/1 
  Installing       : rhel-system-roles-2.4.2-0.1.el10.noarch                     1/3 
  Installing       : ansible-collection-microsoft-sql-2.6.6-1.el10.noarch        2/3 
  Installing       : ansible-collection-redhat-leapp-1.7.6-1.el10.noarch         3/3 

Installed:
  ansible-collection-microsoft-sql-2.6.6-1.el10.noarch                               
  ansible-collection-redhat-leapp-1.7.6-1.el10.noarch                                
  rhel-system-roles-2.4.2-0.1.el10.noarch                                            

Complete!
[cnode@control-node ansible-lab]$ 
```