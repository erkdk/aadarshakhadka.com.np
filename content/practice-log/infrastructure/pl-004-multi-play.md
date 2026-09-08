---
title: "PL - 004 — Ansible Practice: Multi-Play Playbook for Users and Groups"
date: 2026-09-08
draft: false
---

### Lab Session

```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  files  inventory  undeploy-webserver.yml  webserver.yml
[cnode@control-node ansible-lab]$ vim usercreate.yml

[cnode@control-node ansible-lab]$ ansible-doc user
/EXAMPLES

[cnode@control-node ansible-lab]$ ansible-doc group
/EXAMPLES

[cnode@control-node ansible-lab]$ vim usercreate.yml
[cnode@control-node ansible-lab]$ ls
ansible.cfg  files  inventory  undeploy-webserver.yml  usercreate.yml  webserver.yml
[cnode@control-node ansible-lab]$ mv usercreate.yml multiplays.yml
[cnode@control-node ansible-lab]$ ls
ansible.cfg  files  inventory  multiplays.yml  undeploy-webserver.yml  webserver.yml
[cnode@control-node ansible-lab]$ mkdir done
[cnode@control-node ansible-lab]$ mv undeploy-webserver.yml webserver.yml done/
[cnode@control-node ansible-lab]$ ls
ansible.cfg  done  files  inventory  multiplays.yml
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check multiplays.yml 
[WARNING]: Could not match supplied host pattern, ignoring: dev

playbook: multiplays.yml
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
[cnode@control-node ansible-lab]$ vim multiplays.yml 
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check multiplays.yml

playbook: multiplays.yml
[cnode@control-node ansible-lab]$

[cnode@control-node ansible-lab]$ ls
ansible.cfg  done  files  inventory  multiplays.yml
[cnode@control-node ansible-lab]$ vim multiplays.yml 
[cnode@control-node ansible-lab]$ cat multiplays.yml 
---
# Creating a multi-play playbook

- name: Create groups and users on develop nodes
  hosts: develop

  tasks:
    
    # Create a group "employee"

    - name: Create a group "employee"
      ansible.builtin.group:
        name: employee
        state: present

    # Create a group "devgroup"

    - name: Create a group "devgroup"
      ansible.builtin.group:
        name: devgroup
        state: present
 
    # create a user "user1"      
   
    - name: Add the user 'user1'
      ansible.builtin.user:
        name: user1
        group: employee
        groups: devgroup
        uid: 1005
        state: present

- name: Create groups and users on testprod nodes
  hosts: testprod

  tasks:
    
    # Create a group "staff"

    - name: Create a group "staff"
      ansible.builtin.group:
        name: staff
        state: present

    # Create a group "maintainer"

    - name: Create a group "maintainer"
      ansible.builtin.group:
        name: maintainer
        state: present
 
    # create a user "user2"      
   
    - name: Add the user 'user2'
      ansible.builtin.user:
        name: user2
        group: staff
        groups: maintainer
        uid: 1010
        state: present
[cnode@control-node ansible-lab]$ ansible-playbook --check multiplays.yml

PLAY [Create groups and users on develop nodes] *************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Create a group "employee"] ****************************************************
changed: [dev2]
changed: [dev1]

TASK [Create a group "devgroup"] ****************************************************
changed: [dev2]
changed: [dev1]

TASK [Add the user 'user1'] *********************************************************
changed: [dev2]
changed: [dev1]

PLAY [Create groups and users on testprod nodes] ************************************

TASK [Gathering Facts] **************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Create a group "staff"] *******************************************************
changed: [prodserver]
changed: [testserver]

TASK [Create a group "maintainer"] **************************************************
changed: [testserver]
changed: [prodserver]

TASK [Add the user 'user2'] *********************************************************
changed: [testserver]
changed: [prodserver]

PLAY RECAP **************************************************************************
dev1                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
prodserver                 : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ ansible-playbook multiplays.yml 

PLAY [Create groups and users on develop nodes] *****************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************
ok: [dev1]
ok: [dev2]

TASK [Create a group "employee"] ********************************************************************************************************************************************
changed: [dev1]
changed: [dev2]

TASK [Create a group "devgroup"] ********************************************************************************************************************************************
changed: [dev2]
changed: [dev1]

TASK [Add the user 'user1'] *************************************************************************************************************************************************
changed: [dev1]
changed: [dev2]

PLAY [Create groups and users on testprod nodes] ****************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Create a group "staff"] ***********************************************************************************************************************************************
changed: [prodserver]
changed: [testserver]

TASK [Create a group "maintainer"] ******************************************************************************************************************************************
changed: [testserver]
changed: [prodserver]

TASK [Add the user 'user2'] *************************************************************************************************************************************************
changed: [testserver]
changed: [prodserver]

PLAY RECAP ******************************************************************************************************************************************************************
dev1                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
prodserver                 : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible develop -a 'id user1'
dev2 | CHANGED | rc=0 >>
uid=1005(user1) gid=1001(employee) groups=1001(employee),1002(devgroup)
dev1 | CHANGED | rc=0 >>
uid=1005(user1) gid=1001(employee) groups=1001(employee),1002(devgroup)
[cnode@control-node ansible-lab]$ ansible develop -m ansible.builtin.getent -a "database=passwd"
dev1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3",
        "getent_passwd": {
            ...
            "cnode": [
                "x",
                "1000",
                "1000",
                "Control Node",
                "/home/cnode",
                "/bin/bash"
            ],
            ...
            "user1": [
                "x",
                "1005",
                "1001",
                "",
                "/home/user1",
                "/bin/bash"
            ]
        }
    },
    "changed": false
}
dev2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3",
        "getent_passwd": {
            ...
            "user1": [
                "x",
                "1005",
                "1001",
                "",
                "/home/user1",
                "/bin/bash"
            ]
        }
    },
    "changed": false
}
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible develop -a "getent group employee"
dev2 | CHANGED | rc=0 >>
employee:x:1001:
dev1 | CHANGED | rc=0 >>
employee:x:1001:
[cnode@control-node ansible-lab]$ ansible develop -a "getent group devgroup"
dev1 | CHANGED | rc=0 >>
devgroup:x:1002:user1
dev2 | CHANGED | rc=0 >>
devgroup:x:1002:user1
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible testprod -a "id user2"
testserver | CHANGED | rc=0 >>
uid=1010(user2) gid=1001(staff) groups=1001(staff),1002(maintainer)
prodserver | CHANGED | rc=0 >>
uid=1010(user2) gid=1001(staff) groups=1001(staff),1002(maintainer)
[cnode@control-node ansible-lab]$ ansible testprod -a "getent group staff"
testserver | CHANGED | rc=0 >>
staff:x:1001:
prodserver | CHANGED | rc=0 >>
staff:x:1001:
[cnode@control-node ansible-lab]$ ansible testprod -a "getent group maintainer"
testserver | CHANGED | rc=0 >>
maintainer:x:1002:user2
prodserver | CHANGED | rc=0 >>
maintainer:x:1002:user2
[cnode@control-node ansible-lab]$ echo $?
0
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-doc uri
...
[cnode@control-node ansible-lab]$
```