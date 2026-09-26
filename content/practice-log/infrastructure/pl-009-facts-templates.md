---
title: "PL - 006 — Ansible Lab: Facts, Jinja2 Templates, Loops, Variables, Conditionals, and User Management"
date: 2026-09-20
draft: false
---

### Lab Session:

```
[cnode@control-node ~]$ ls
ansible-lab  done  validate_nodes.sh
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

[cnode@control-node ~]$ cd ansible-lab/
[cnode@control-node ansible-lab]$ ls
ansible.cfg  files  inventory  webserver.yml

[cnode@control-node ansible-lab]$ ansible testprod -m setup -a 'filter=*ipv4*'
testserver | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.254.18"
        ],
        "ansible_default_ipv4": {
            "address": "192.168.254.18",
            "alias": "enp0s3",
            "broadcast": "192.168.254.255",
            "gateway": "192.168.254.254",
            "interface": "enp0s3",
            "macaddress": "08:00:47:e2:6f:f1",
            "mtu": 1500,
            "netmask": "255.255.255.0",
            "network": "192.168.254.0",
            "prefix": "24",
            "type": "ether"
        },
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
prodserver | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.254.19"
        ],
        "ansible_default_ipv4": {
            "address": "192.168.254.19",
            "alias": "enp0s3",
            "broadcast": "192.168.254.255",
            "gateway": "192.168.254.254",
            "interface": "enp0s3",
            "macaddress": "08:00:37:8f:95:5e",
            "mtu": 1500,
            "netmask": "255.255.255.0",
            "network": "192.168.254.0",
            "prefix": "24",
            "type": "ether"
        },
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible prodserver -m setup -a 'filter=*ipv4*'
prodserver | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.254.19"
        ],
        "ansible_default_ipv4": {
            "address": "192.168.254.19",
            "alias": "enp0s3",
            "broadcast": "192.168.254.255",
            "gateway": "192.168.254.254",
            "interface": "enp0s3",
            "macaddress": "08:10:27:9d:95:5e",
            "mtu": 1500,
            "netmask": "255.255.255.0",
            "network": "192.168.254.0",
            "prefix": "24",
            "type": "ether"
        },
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}

[cnode@control-node ansible-lab]$ ansible prodserver -m setup -a 'filter=*cpu*'
prodserver | SUCCESS => {
    "ansible_facts": {
        "ansible_processor_vcpus": 2,
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible prodserver -m setup -a 'filter=*bios*'
prodserver | SUCCESS => {
    "ansible_facts": {
        "ansible_bios_date": "12/01/2006",
        "ansible_bios_vendor": "innotek GmbH",
        "ansible_bios_version": "VirtualBox",
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ vim files/index.html.j2 
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check webserver.yml 

playbook: webserver.yml
[cnode@control-node ansible-lab]$ ansible-playbook webserver.yml

PLAY [Configure Apache web server] **************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Install Apache web server] ****************************************************************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Start and enable Apache service] **********************************************************************************************************
ok: [prodserver]
ok: [testserver]

TASK [Print return information from the previous task] ******************************************************************************************
ok: [testserver] => {
    "service_out": {
        "changed": false,
        "enabled": true,
        "failed": false,
        "name": "httpd",
        "state": "started",
        "status": {
...
        }
    }
}

TASK [Allow HTTP traffic through the firewall] **************************************************************************************************
ok: [prodserver]
ok: [testserver]

TASK [Deploy website index page] ****************************************************************************************************************
changed: [prodserver]
changed: [testserver]

PLAY RECAP **************************************************************************************************************************************
prodserver                 : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.shell -a "curl 0"
testserver | CHANGED | rc=0 >>
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <h2>Machine's Hostname(old-style): testserver</h2>
    <h2>Machine's Hostname(new-style): testserver</h2>
    <h2>No.of CPUs in this system (old-style): 2 </h2>
    <h2>No.of CPUs in this system (new-style): 2 </h2>
    <h2>IPv4 Address(old-style): 192.168.254.18</h2>
    <h2>IPv4 Address(new-style): 192.168.254.18</h2>
    <h2>BIOS date of this system (old-style): 12/01/2006 </h2>
<h2>BIOS date of this system (old-style): 12/01/2006 </h2>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   637  100   637    0     0   639k      0 --:--:-- --:--:-- --:--:--  622k
prodserver | CHANGED | rc=0 >>
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <h2>Machine's Hostname(old-style): prodserver</h2>
    <h2>Machine's Hostname(new-style): prodserver</h2>
    <h2>No.of CPUs in this system (old-style): 2 </h2>
    <h2>No.of CPUs in this system (new-style): 2 </h2>
    <h2>IPv4 Address(old-style): 192.168.254.19</h2>
    <h2>IPv4 Address(new-style): 192.168.254.19</h2>
    <h2>BIOS date of this system (old-style): 12/01/2006 </h2>
<h2>BIOS date of this system (old-style): 12/01/2006 </h2>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   637  100   637    0     0   662k      0 --:--:-- --:--:-- --:--:--  622k
[cnode@control-node ansible-lab]$ 



-----------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------

time 27 minutes (sep 9)

# Creating playbooks with loop and conditional tasks


[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ vim loop-usercreate.yml
[cnode@control-node ansible-lab]$ cat loop-usercreate.yml 
---
# Loops

- name: Create users
  hosts: develop

  vars:
    devusers:
      - devuser1
      - devuser2
      - devuser3
      - devuser4
      - devuser5

  tasks:
    - name: Add developer users
      ansible.builtin.user:
        name: "{{ item }}"
      with_items: "{{ devusers }}"
[cnode@control-node ansible-lab]$ ansible develop -m ansible.builtin.shell -a "tail -5 /etc/passwd"
dev1 | CHANGED | rc=0 >>
chrony:x:997:996:chrony system user:/var/lib/chrony:/sbin/nologin
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin
cnode:x:1000:1000:Control Node:/home/cnode:/bin/bash
user1:x:1005:1001::/home/user1:/bin/bash
ram:x:1006:1006::/home/ram:/bin/bash
dev2 | CHANGED | rc=0 >>
chrony:x:997:996:chrony system user:/var/lib/chrony:/sbin/nologin
systemd-coredump:x:995:995:systemd Core Dumper:/:/usr/sbin/nologin
cnode:x:1000:1000:Control Node:/home/cnode:/bin/bash
user1:x:1005:1001::/home/user1:/bin/bash
ram:x:1006:1006::/home/ram:/bin/bash
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check loop-usercreate.yml

playbook: loop-usercreate.yml
[cnode@control-node ansible-lab]$ ansible-playbook loop-usercreate.yml

PLAY [Create users] ************************************************************

TASK [Gathering Facts] *********************************************************
ok: [dev1]
ok: [dev2]

TASK [Add developer users] *****************************************************
changed: [dev1] => (item=devuser1)
changed: [dev2] => (item=devuser1)
changed: [dev1] => (item=devuser2)
changed: [dev2] => (item=devuser2)
changed: [dev1] => (item=devuser3)
changed: [dev2] => (item=devuser3)
changed: [dev1] => (item=devuser4)
changed: [dev2] => (item=devuser4)
changed: [dev1] => (item=devuser5)
changed: [dev2] => (item=devuser5)

PLAY RECAP *********************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ ansible develop -m ansible.builtin.shell -a "tail -5 /etc/passwd"
dev2 | CHANGED | rc=0 >>
devuser1:x:1007:1007::/home/devuser1:/bin/bash
devuser2:x:1008:1008::/home/devuser2:/bin/bash
devuser3:x:1009:1009::/home/devuser3:/bin/bash
devuser4:x:1010:1010::/home/devuser4:/bin/bash
devuser5:x:1011:1011::/home/devuser5:/bin/bash
dev1 | CHANGED | rc=0 >>
devuser1:x:1007:1007::/home/devuser1:/bin/bash
devuser2:x:1008:1008::/home/devuser2:/bin/bash
devuser3:x:1009:1009::/home/devuser3:/bin/bash
devuser4:x:1010:1010::/home/devuser4:/bin/bash
devuser5:x:1011:1011::/home/devuser5:/bin/bash
[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ vim loop-usercreate.yml 
[cnode@control-node ansible-lab]$ cat loop-usercreate.yml 
---
# Loops

- name: Create users
  hosts: develop

  vars:
    devusers:
      - devuser2
      - devuser3
      - devuser4
      - devuser5

  tasks:
    - name: Add developer users
      ansible.builtin.user:
        name: "{{ item }}"
        state: absent
        remove: yes
      with_items: "{{ devusers }}"
[cnode@control-node ansible-lab]$ ansible-playbook loop-usercreate.yml

PLAY [Create users] *****************************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Add developer users] **********************************************************
changed: [dev2] => (item=devuser2)
changed: [dev2] => (item=devuser3)
changed: [dev1] => (item=devuser2)
changed: [dev2] => (item=devuser4)
changed: [dev1] => (item=devuser3)
changed: [dev2] => (item=devuser5)
changed: [dev1] => (item=devuser4)
changed: [dev1] => (item=devuser5)

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ ansible develop -m ansible.builtin.shell -a "id deveuser5"
dev2 | FAILED | rc=1 >>
id: ‘deveuser5’: no such usernon-zero return code
dev1 | FAILED | rc=1 >>
id: ‘deveuser5’: no such usernon-zero return code
[cnode@control-node ansible-lab]$ 




[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  loop-usercreate.yml
[cnode@control-node ansible-lab]$ mkdir vars
[cnode@control-node ansible-lab]$ vim vars/newusers.yml
[cnode@control-node ansible-lab]$ cat vars/newusers.yml 
newusers:
  - name: newuser1
    pw: newuser1pass
  - name: newuser2
    pw: newuser2pass
  - name: newuser3
    pw: newuser3pass
  - name: newuser4pass
    pw: newuser4pass
[cnode@control-node ansible-lab]$ cp loop-usercreate.yml loop-newusercreate.yml 

[cnode@control-node ansible-lab]$ vim loop-newusercreate.yml

[cnode@control-node ansible-lab]$ cat loop-newusercreate.yml 
---
# Loops

- name: Create new users from file
  hosts: develop

  vars_files:
    - vars/newusers.yml

  tasks:
    - name: Add some new users
      ansible.builtin.user:
        name: "{{ item.name }}"
        state: present
        password: "{{ item.pw | password_hash('sha512') }}"
      with_items: "{{ newusers }}"
[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  loop-newusercreate.yml  loop-usercreate.yml  vars
[cnode@control-node ansible-lab]$ vim loop-newusercreate.yml 
[cnode@control-node ansible-lab]$ cat loop-newusercreate.yml 
---
# Provision users and their required groups on development hosts.

- name: Provision development users
  hosts: develop
  become: true

  vars_files:
    - vars/newusers.yml

  vars:
    # Groups required by the development environment.
    newgroups:
      - developers
      - managers
      - admins
      - employees

    # Supplementary groups assigned to newly created users.
    user_groups:
      - developers
      - managers
      - admins

  tasks:

    - name: Create required user groups
      ansible.builtin.group:
        name: "{{ item }}"
        state: present
      loop: "{{ newgroups }}"
      loop_control:
        label: "{{ item }}"

    - name: Create development users
      ansible.builtin.user:
        name: "{{ item.name }}"
        state: present
        password: "{{ item.pw | password_hash('sha512') }}"
        group: employees
        groups: "{{ user_groups | join(',') }}"
      loop: "{{ newusers }}"
      loop_control:
        label: "{{ item.name }}"
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check loop-newusercreate.yml

playbook: loop-newusercreate.yml
[cnode@control-node ansible-lab]$ ansible-playbook --check loop-newusercreate.yml

PLAY [Provision development users] **************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev2]
ok: [dev1]

TASK [Create required user groups] **************************************************
changed: [dev1] => (item=developers)
changed: [dev2] => (item=developers)
changed: [dev1] => (item=managers)
changed: [dev2] => (item=managers)
changed: [dev1] => (item=admins)
changed: [dev2] => (item=admins)
changed: [dev1] => (item=employees)
changed: [dev2] => (item=employees)

TASK [Create development users] *****************************************************
[DEPRECATION WARNING]: Encryption using the Python crypt module is deprecated. The 
Python crypt module is deprecated and will be removed from Python 3.13. Install the 
passlib library for continued encryption functionality. This feature will be removed
 in version 2.17. Deprecation warnings can be disabled by setting 
deprecation_warnings=False in ansible.cfg.
changed: [dev1] => (item=newuser1)
changed: [dev2] => (item=newuser1)
changed: [dev1] => (item=newuser2)
changed: [dev2] => (item=newuser2)
changed: [dev1] => (item=newuser3)
changed: [dev2] => (item=newuser3)
changed: [dev1] => (item=newuser4pass)
changed: [dev2] => (item=newuser4pass)

PLAY RECAP **************************************************************************
dev1                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ ansible-playbook loop-newusercreate.yml

PLAY [Provision development users] **************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev2]
ok: [dev1]

TASK [Create required user groups] **************************************************
changed: [dev2] => (item=developers)
changed: [dev1] => (item=developers)
changed: [dev2] => (item=managers)
changed: [dev1] => (item=managers)
changed: [dev1] => (item=admins)
changed: [dev2] => (item=admins)
changed: [dev1] => (item=employees)
changed: [dev2] => (item=employees)

TASK [Create development users] *****************************************************
[DEPRECATION WARNING]: Encryption using the Python crypt module is deprecated. The 
Python crypt module is deprecated and will be removed from Python 3.13. Install the 
passlib library for continued encryption functionality. This feature will be removed
 in version 2.17. Deprecation warnings can be disabled by setting 
deprecation_warnings=False in ansible.cfg.
changed: [dev2] => (item=newuser1)
changed: [dev1] => (item=newuser1)
changed: [dev1] => (item=newuser2)
changed: [dev2] => (item=newuser2)
changed: [dev1] => (item=newuser3)
changed: [dev2] => (item=newuser3)
changed: [dev1] => (item=newuser4pass)
changed: [dev2] => (item=newuser4pass)

PLAY RECAP **************************************************************************
dev1                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ ansible-playbook loop-newusercreate.yml

PLAY [Provision development users] ******************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************
ok: [dev2]
ok: [dev1]

TASK [Create required user groups] ******************************************************************************************************************************************
ok: [dev1] => (item=developers)
ok: [dev2] => (item=developers)
ok: [dev1] => (item=managers)
ok: [dev2] => (item=managers)
ok: [dev2] => (item=admins)
ok: [dev1] => (item=admins)
ok: [dev1] => (item=employees)
ok: [dev2] => (item=employees)

TASK [Create development users] *********************************************************************************************************************************************
[DEPRECATION WARNING]: Encryption using the Python crypt module is deprecated. The Python crypt module is deprecated and will be removed from Python 3.13. Install the 
passlib library for continued encryption functionality. This feature will be removed in version 2.17. Deprecation warnings can be disabled by setting 
deprecation_warnings=False in ansible.cfg.
changed: [dev1] => (item=newuser1)
changed: [dev2] => (item=newuser1)
changed: [dev1] => (item=newuser2)
changed: [dev2] => (item=newuser2)
changed: [dev1] => (item=newuser3)
changed: [dev2] => (item=newuser3)
changed: [dev1] => (item=newuser4pass)
changed: [dev2] => (item=newuser4pass)

PLAY RECAP ******************************************************************************************************************************************************************
dev1                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ ansible develop -m ansible.builtin.shell -a 'getent group developers managers admins employees'
dev1 | CHANGED | rc=0 >>
developers:x:1008:newuser1,newuser2,newuser3,newuser4pass
managers:x:1009:newuser1,newuser2,newuser3,newuser4pass
admins:x:1010:newuser1,newuser2,newuser3,newuser4pass
employees:x:1011:
dev2 | CHANGED | rc=0 >>
developers:x:1008:newuser1,newuser2,newuser3,newuser4pass
managers:x:1009:newuser1,newuser2,newuser3,newuser4pass
admins:x:1010:newuser1,newuser2,newuser3,newuser4pass
employees:x:1011:

[cnode@control-node ansible-lab]$ ansible develop -m ansible.builtin.shell -a 'getent passwd newuser1 newuser2 newuser3 newuser4pass'
dev1 | CHANGED | rc=0 >>
newuser1:x:1008:1011::/home/newuser1:/bin/bash
newuser2:x:1009:1011::/home/newuser2:/bin/bash
newuser3:x:1010:1011::/home/newuser3:/bin/bash
newuser4pass:x:1011:1011::/home/newuser4pass:/bin/bash
dev2 | CHANGED | rc=0 >>
newuser1:x:1008:1011::/home/newuser1:/bin/bash
newuser2:x:1009:1011::/home/newuser2:/bin/bash
newuser3:x:1010:1011::/home/newuser3:/bin/bash
newuser4pass:x:1011:1011::/home/newuser4pass:/bin/bash
[cnode@control-node ansible-lab]$

[cnode@control-node ansible-lab]$ ansible develop -m ansible.builtin.shell -a 'id newuser1'
dev2 | CHANGED | rc=0 >>
uid=1008(newuser1) gid=1011(employees) groups=1011(employees),1008(developers),1009(managers),1010(admins)
dev1 | CHANGED | rc=0 >>
uid=1008(newuser1) gid=1011(employees) groups=1011(employees),1008(developers),1009(managers),1010(admins)
[cnode@control-node ansible-lab]$ 





-----


[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  loop-newusercreate.yml  loop-usercreate.yml  vars
[cnode@control-node ansible-lab]$ cp loop-usercreate.yml createusers.yml
[cnode@control-node ansible-lab]$ vim createusers.yml 



[cnode@control-node ansible-lab]$ vim createusers.yml
[cnode@control-node ansible-lab]$ cat createusers.yml 
---
# Create users 

- name: Create users on different machines
  hosts: all

  vars:
    devusers:
      - devuser2
      - devuser3
      - devuser4
      - devuser5

    testusers:
      - testuser1
      - testuser2
      - testuser3
      - testuser4
    
    produsers:
      - produser1
      - produser2
      - produser3
      - produser4

  tasks:
    - name: Add developer users
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      with_items: "{{ devusers }}"
      when: inventory_hostname in groups['develop']

    - name: Add test users
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      with_items: "{{ testusers }}"
      when: inventory_hostname in groups['test']

    - name: Add production users
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      with_items: "{{ produsers }}"
      when: inventory_hostname in groups['production']
[cnode@control-node ansible-lab]$ cat inventory 
[myself]
control-node

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
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check createusers.yml 

playbook: createusers.yml

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
[cnode@control-node ansible-lab]$ ansible-playbook --check createusers.yml 

PLAY [Create users on different machines] **********************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [prodserver]
ok: [dev2]
ok: [dev1]
ok: [testserver]

TASK [Add developer users] *************************************************************************************************************************************************
skipping: [testserver] => (item=devuser2) 
skipping: [testserver] => (item=devuser3) 
skipping: [testserver] => (item=devuser4) 
skipping: [prodserver] => (item=devuser2) 
skipping: [testserver] => (item=devuser5) 
skipping: [prodserver] => (item=devuser3) 
skipping: [testserver]
skipping: [prodserver] => (item=devuser4) 
skipping: [prodserver] => (item=devuser5) 
skipping: [prodserver]
changed: [dev2] => (item=devuser2)
changed: [dev1] => (item=devuser2)
changed: [dev1] => (item=devuser3)
changed: [dev2] => (item=devuser3)
changed: [dev2] => (item=devuser4)
changed: [dev1] => (item=devuser4)
changed: [dev2] => (item=devuser5)
changed: [dev1] => (item=devuser5)

TASK [Add test users] ******************************************************************************************************************************************************
skipping: [dev1] => (item=testuser1) 
skipping: [dev1] => (item=testuser2) 
skipping: [dev1] => (item=testuser3) 
skipping: [dev1] => (item=testuser4) 
skipping: [dev1]
skipping: [dev2] => (item=testuser1) 
skipping: [dev2] => (item=testuser2) 
skipping: [dev2] => (item=testuser3) 
skipping: [dev2] => (item=testuser4) 
skipping: [dev2]
skipping: [prodserver] => (item=testuser1) 
skipping: [prodserver] => (item=testuser2) 
skipping: [prodserver] => (item=testuser3) 
skipping: [prodserver] => (item=testuser4) 
skipping: [prodserver]
changed: [testserver] => (item=testuser1)
changed: [testserver] => (item=testuser2)
changed: [testserver] => (item=testuser3)
changed: [testserver] => (item=testuser4)

TASK [Add production users] ************************************************************************************************************************************************
skipping: [dev1] => (item=produser1) 
skipping: [dev1] => (item=produser2) 
skipping: [dev1] => (item=produser3) 
skipping: [dev1] => (item=produser4) 
skipping: [dev1]
skipping: [testserver] => (item=produser1) 
skipping: [dev2] => (item=produser1) 
skipping: [dev2] => (item=produser2) 
skipping: [dev2] => (item=produser3) 
skipping: [testserver] => (item=produser2) 
skipping: [testserver] => (item=produser3) 
skipping: [dev2] => (item=produser4) 
skipping: [dev2]
skipping: [testserver] => (item=produser4) 
skipping: [testserver]
changed: [prodserver] => (item=produser1)
changed: [prodserver] => (item=produser2)
changed: [prodserver] => (item=produser3)
changed: [prodserver] => (item=produser4)

PLAY RECAP *****************************************************************************************************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
prodserver                 : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
testserver                 : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ ansible-playbook createusers.yml

PLAY [Create users on different machines] **********************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [prodserver]
ok: [testserver]
ok: [dev1]
ok: [dev2]

TASK [Add developer users] *************************************************************************************************************************************************
skipping: [testserver] => (item=devuser2) 
skipping: [testserver] => (item=devuser3) 
skipping: [testserver] => (item=devuser4) 
skipping: [testserver] => (item=devuser5) 
skipping: [testserver]
skipping: [prodserver] => (item=devuser2) 
skipping: [prodserver] => (item=devuser3) 
skipping: [prodserver] => (item=devuser4) 
skipping: [prodserver] => (item=devuser5) 
skipping: [prodserver]
changed: [dev1] => (item=devuser2)
changed: [dev2] => (item=devuser2)
changed: [dev2] => (item=devuser3)
changed: [dev1] => (item=devuser3)
changed: [dev2] => (item=devuser4)
changed: [dev1] => (item=devuser4)
changed: [dev2] => (item=devuser5)
changed: [dev1] => (item=devuser5)

TASK [Add test users] ******************************************************************************************************************************************************
skipping: [dev1] => (item=testuser1) 
skipping: [dev1] => (item=testuser2) 
skipping: [dev1] => (item=testuser3) 
skipping: [dev1] => (item=testuser4) 
skipping: [dev2] => (item=testuser1) 
skipping: [dev1]
skipping: [dev2] => (item=testuser2) 
skipping: [dev2] => (item=testuser3) 
skipping: [dev2] => (item=testuser4) 
skipping: [prodserver] => (item=testuser1) 
skipping: [dev2]
skipping: [prodserver] => (item=testuser2) 
skipping: [prodserver] => (item=testuser3) 
skipping: [prodserver] => (item=testuser4) 
skipping: [prodserver]
changed: [testserver] => (item=testuser1)
changed: [testserver] => (item=testuser2)
changed: [testserver] => (item=testuser3)
changed: [testserver] => (item=testuser4)

TASK [Add production users] ************************************************************************************************************************************************
skipping: [dev1] => (item=produser1) 
skipping: [dev1] => (item=produser2) 
skipping: [dev1] => (item=produser3) 
skipping: [dev1] => (item=produser4) 
skipping: [dev2] => (item=produser1) 
skipping: [dev1]
skipping: [dev2] => (item=produser2) 
skipping: [dev2] => (item=produser3) 
skipping: [dev2] => (item=produser4) 
skipping: [testserver] => (item=produser1) 
skipping: [dev2]
skipping: [testserver] => (item=produser2) 
skipping: [testserver] => (item=produser3) 
skipping: [testserver] => (item=produser4) 
skipping: [testserver]
changed: [prodserver] => (item=produser1)
changed: [prodserver] => (item=produser2)
changed: [prodserver] => (item=produser3)
changed: [prodserver] => (item=produser4)

PLAY RECAP *****************************************************************************************************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
prodserver                 : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
testserver                 : ok=2    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ ls
ansible.cfg  createusers.yml  inventory  loop-newusercreate.yml  loop-usercreate.yml  vars
[cnode@control-node ansible-lab]$ vim deleteusers.yml
[cnode@control-node ansible-lab]$ cat deleteusers.yml 
---
- name: Delete all created users from lab machines
  hosts: all
  become true
  gather_facts: false

  vars:
    users_to_delete:
      - devuser1
      - devuser2
      - devuser3
      - devuser4
      - devuser5
      - testuser1
      - testuser2
      - testuser3
      - testuser4
      - produser1
      - produser2
      - produser3
      - produser4
   
   tasks:
     - name: Delete created users
       ansible.builtin.user:
         name: "{{ item }}"
         state: absent
         remove: true
       loop: "{{ users_to_delete }}"

[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check deleteusers.yml 

[cnode@control-node ansible-lab]$ vim deleteusers.yml 
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check deleteusers.yml 

playbook: deleteusers.yml
[cnode@control-node ansible-lab]$ cat deleteusers.yml 
---
- name: Delete all created users from lab machines
  hosts: all
  become: true
  gather_facts: false

  vars:
    users_to_delete:
      - devuser1
      - devuser2
      - devuser3
      - devuser4
      - devuser5
      - testuser1
      - testuser2
      - testuser3
      - testuser4
      - produser1
      - produser2
      - produser3
      - produser4
   
  tasks:
    - name: Delete created users
      ansible.builtin.user:
         name: "{{ item }}"
         state: absent
         remove: true
      loop: "{{ users_to_delete }}"

[cnode@control-node ansible-lab]$ ansible-playbook --check deleteusers.yml 

PLAY [Delete all created users from lab machines] **************************************************************************************************************************

TASK [Delete created users] ************************************************************************************************************************************************
changed: [dev2] => (item=devuser1)
ok: [testserver] => (item=devuser1)
ok: [prodserver] => (item=devuser1)
changed: [dev2] => (item=devuser2)
ok: [testserver] => (item=devuser2)
ok: [prodserver] => (item=devuser2)
changed: [dev1] => (item=devuser1)
ok: [prodserver] => (item=devuser3)
changed: [dev1] => (item=devuser2)
changed: [dev2] => (item=devuser3)
ok: [testserver] => (item=devuser3)
ok: [prodserver] => (item=devuser4)
changed: [dev1] => (item=devuser3)
changed: [dev2] => (item=devuser4)
ok: [testserver] => (item=devuser4)
ok: [prodserver] => (item=devuser5)
changed: [dev1] => (item=devuser4)
changed: [dev2] => (item=devuser5)
ok: [testserver] => (item=devuser5)
ok: [prodserver] => (item=testuser1)
changed: [dev1] => (item=devuser5)
ok: [dev2] => (item=testuser1)
changed: [testserver] => (item=testuser1)
ok: [prodserver] => (item=testuser2)
ok: [dev1] => (item=testuser1)
changed: [testserver] => (item=testuser2)
ok: [dev2] => (item=testuser2)
ok: [prodserver] => (item=testuser3)
ok: [dev1] => (item=testuser2)
ok: [dev2] => (item=testuser3)
changed: [testserver] => (item=testuser3)
ok: [prodserver] => (item=testuser4)
ok: [dev1] => (item=testuser3)
changed: [testserver] => (item=testuser4)
ok: [dev2] => (item=testuser4)
ok: [dev1] => (item=testuser4)
changed: [prodserver] => (item=produser1)
ok: [testserver] => (item=produser1)
ok: [dev2] => (item=produser1)
changed: [prodserver] => (item=produser2)
ok: [dev1] => (item=produser1)
ok: [testserver] => (item=produser2)
ok: [dev2] => (item=produser2)
changed: [prodserver] => (item=produser3)
ok: [dev1] => (item=produser2)
ok: [testserver] => (item=produser3)
ok: [dev2] => (item=produser3)
ok: [dev1] => (item=produser3)
changed: [prodserver] => (item=produser4)
ok: [testserver] => (item=produser4)
ok: [dev2] => (item=produser4)
ok: [dev1] => (item=produser4)

PLAY RECAP *****************************************************************************************************************************************************************
dev1                       : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
prodserver                 : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook deleteusers.yml 

PLAY [Delete all created users from lab machines] **************************************************************************************************************************

TASK [Delete created users] ************************************************************************************************************************************************
ok: [prodserver] => (item=devuser1)
ok: [testserver] => (item=devuser1)
ok: [prodserver] => (item=devuser2)
ok: [testserver] => (item=devuser2)
changed: [dev1] => (item=devuser1)
changed: [dev2] => (item=devuser1)
ok: [prodserver] => (item=devuser3)
ok: [testserver] => (item=devuser3)
ok: [prodserver] => (item=devuser4)
ok: [testserver] => (item=devuser4)
changed: [dev1] => (item=devuser2)
changed: [dev2] => (item=devuser2)
ok: [prodserver] => (item=devuser5)
ok: [testserver] => (item=devuser5)
ok: [prodserver] => (item=testuser1)
ok: [prodserver] => (item=testuser2)
changed: [dev1] => (item=devuser3)
changed: [dev2] => (item=devuser3)
ok: [prodserver] => (item=testuser3)
ok: [prodserver] => (item=testuser4)
changed: [testserver] => (item=testuser1)
changed: [dev1] => (item=devuser4)
changed: [dev2] => (item=devuser4)
changed: [prodserver] => (item=produser1)
changed: [testserver] => (item=testuser2)
changed: [dev1] => (item=devuser5)
changed: [dev2] => (item=devuser5)
ok: [dev1] => (item=testuser1)
ok: [dev2] => (item=testuser1)
ok: [dev1] => (item=testuser2)
ok: [dev2] => (item=testuser2)
changed: [prodserver] => (item=produser2)
changed: [testserver] => (item=testuser3)
ok: [dev1] => (item=testuser3)
ok: [dev2] => (item=testuser3)
ok: [dev1] => (item=testuser4)
ok: [dev2] => (item=testuser4)
ok: [dev1] => (item=produser1)
ok: [dev2] => (item=produser1)
changed: [prodserver] => (item=produser3)
changed: [testserver] => (item=testuser4)
ok: [dev1] => (item=produser2)
ok: [dev2] => (item=produser2)
ok: [testserver] => (item=produser1)
ok: [dev2] => (item=produser3)
ok: [dev1] => (item=produser3)
changed: [prodserver] => (item=produser4)
ok: [testserver] => (item=produser2)
ok: [dev2] => (item=produser4)
ok: [dev1] => (item=produser4)
ok: [testserver] => (item=produser3)
ok: [testserver] => (item=produser4)

PLAY RECAP *****************************************************************************************************************************************************************
dev1                       : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
prodserver                 : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 
```