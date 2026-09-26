---
title: "PL - 011 — Ansible Practice: Ansible Inclusion with Variables, Tasks, and Playbooks"
date: 2026-09-25
draft: false
---

### Ansible Inclusion

>Ansible inclusion means incorporating variables, tasks, or entire playbooks into an Ansible execution, allowing reusable components to be organized separately.

### Lab Session
```
[cnode@control-node ~]$ ls
ansible-lab  done  validate_nodes.sh
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
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ cat ansible.cfg 
[defaults]
inventory = ./inventory
remote_user = cnode

[privilege_escalation]
become = true
become_method = sudo
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
[cnode@control-node ansible-lab]$
```
---

```
[cnode@control-node ansible-lab]$ mkdir vars
[cnode@control-node ansible-lab]$ vim vars/variables.yml
[cnode@control-node ansible-lab]$ cat vars/variables.yml 
---
fwll_pkg: firewalld
web_pkg: httpd
[cnode@control-node ansible-lab]$ mkdir tasks
[cnode@control-node ansible-lab]$ vim tasks/tasks.yml
[cnode@control-node ansible-lab]$ cat tasks/tasks.yml 
---
- name: Install {{ web_pkg }} package
  ansible.builtin.dnf:
    name: "{{ web_pkg }}"
    state: latest

- name: Install {{ fwll_pkg }} package
  ansible.builtin.dnf:
    name: "{{ fwll_pkg }}"
    state: latest

- name: Start the {{ web_svc }} service
  ansible.builtin.service:
    name: "{{ web_svc }}"
    state: "{{ svc_state }}"
    enabled: yes

- name: Start the {{ fwll_svc }} service
  ansible.builtin.service:
    name: "{{ fwll_svc }}"
    state: "{{ svc_state }}"
    enabled: yes

- name: Allow http packets through the firewall
  ansible.posix.firewalld:
    service: "{{ fwll_rule }}"
    state: "{{ fwll_rule_state }}"
    permanent: true
    immediate: true
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ vim index.html
[cnode@control-node ansible-lab]$ cat index.html 
<h1>Hello from prodserver</h1>
[cnode@control-node ansible-lab]$ 

┌─[cnode@control-node]─[~/ansible-lab]
└──╼ $ ls
ansible.cfg  inventory  plays  tasks  vars

┌─[cnode@control-node]─[~/ansible-lab]
└──╼ $ ansible-doc ansible.builtin.import_playbook

[cnode@control-node ansible-lab]$ vim webdeploy_inclusion.yml
[cnode@control-node ansible-lab]$ vim webdeploy_inclusion.yml
[cnode@control-node ansible-lab]$ vim tasks/tasks.yml 
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  plays  tasks  vars  webdeploy_inclusion.yml  webdeploy.yml
[cnode@control-node ansible-lab]$ vim plays/test.yml 
[cnode@control-node ansible-lab]$ cat plays/test.yml
---
- name: Test web site
  hosts: production
  become: false

  tasks:
  - name: test website
    ansible.builtin.uri:
      url: "{{ url }}"
      status_code: 200
[cnode@control-node ansible-lab]$ vim webdeploy_inclusion.yml 
[cnode@control-node ansible-lab]$ 
```
---

```
[cnode@prodserver ~]$ hostname
prodserver
[cnode@prodserver ~]$ hostname -I
192.168.254.19 
[cnode@prodserver ~]$ whoami
cnode
[cnode@prodserver ~]$ rpm -q httpd
package httpd is not installed
[cnode@prodserver ~]$ 

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  plays  tasks  vars  webdeploy_inclusion.yml  webdeploy.yml

[cnode@control-node ansible-lab]$ cat webdeploy_inclusion.yml 
---
- name: Play using inclusion
  hosts: production
  vars:
    index_page: index.html
    index_dest: /var/www/html/index.html

  tasks:
    - name: Include variables
      ansible.builtin.include_vars:
        file: vars/variables.yml

    - name: Include/load tasks
      ansible.builtin.include_tasks:
        file: tasks/tasks.yml
      vars:
        web_svc: httpd
        svc_state: started
        fwll_svc: firewalld
        fwll_rule: http
        fwll_rule_state: enabled

    - name: Copy index.html file
      ansible.builtin.copy:
        src: "{{ index_page }}"
        dest: "{{ index_dest }}"

- name: Play-2 testing website
  ansible.builtin.import_playbook: plays/test.yml
  vars:
    url: http://192.168.254.19
[cnode@control-node ansible-lab]$ 

┌─[cnode@control-node]─[~/ansible-lab]
└──╼ $ ansible-playbook --syntax-check webdeploy_inclusion.yml 
playbook: webdeploy_inclusion.yml
```
---

### Final run and output:

```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  index.html  inventory  plays  tasks  vars  webdeploy_inclusion.yml
[cnode@control-node ansible-lab]$ cat index.html 
<h1>Hello from prodserver</h1>
[cnode@control-node ansible-lab]$ cat vars/variables.yml 
---
fwll_pkg: firewalld
web_pkg: httpd
[cnode@control-node ansible-lab]$ cat tasks/tasks.yml 
---
- name: Install {{ web_pkg }} package
  ansible.builtin.dnf:
    name: "{{ web_pkg }}"
    state: latest

- name: Install {{ fwll_pkg }} package
  ansible.builtin.dnf:
    name: "{{ fwll_pkg }}"
    state: latest

- name: Start the {{ web_svc }} service
  ansible.builtin.service:
    name: "{{ web_svc }}"
    state: "{{ svc_state }}"
    enabled: yes

- name: Start the {{ fwll_svc }} service
  ansible.builtin.service:
    name: "{{ fwll_svc }}"
    state: "{{ svc_state }}"
    enabled: yes

- name: Allow http packets through the firewall
  ansible.posix.firewalld:
    service: "{{ fwll_rule }}"
    state: "{{ fwll_rule_state }}"
    permanent: true
    immediate: true
[cnode@control-node ansible-lab]$ cat plays/test.yml 
---
- name: Test web site
  hosts: production
  become: false

  tasks:
  - name: test website
    ansible.builtin.uri:
      url: "{{ url }}"
      status_code: 200
[cnode@control-node ansible-lab]$ cat webdeploy_inclusion.yml 
---
- name: Play using inclusion
  hosts: production
  vars:
    index_page: index.html
    index_dest: /var/www/html/index.html

  tasks:
    - name: Include variables
      ansible.builtin.include_vars:
        file: vars/variables.yml

    - name: Include/load tasks
      ansible.builtin.include_tasks:
        file: tasks/tasks.yml
      vars:
        web_svc: httpd
        svc_state: started
        fwll_svc: firewalld
        fwll_rule: http
        fwll_rule_state: enabled

    - name: Copy index.html file
      ansible.builtin.copy:
        src: "{{ index_page }}"
        dest: "{{ index_dest }}"

- name: Play-2 testing website
  ansible.builtin.import_playbook: plays/test.yml
  vars:
    url: http://192.168.254.19
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check webdeploy_inclusion.yml 

playbook: webdeploy_inclusion.yml
[cnode@control-node ansible-lab]$ ansible-playbook webdeploy_inclusion.yml

PLAY [Play using inclusion] *********************************************************

TASK [Gathering Facts] **************************************************************
ok: [prodserver]

TASK [Include variables] ************************************************************
ok: [prodserver]

TASK [Include/load tasks] ***********************************************************
included: /home/cnode/ansible-lab/tasks/tasks.yml for prodserver

TASK [Install httpd package] ********************************************************
ok: [prodserver]

TASK [Install firewalld package] ****************************************************
ok: [prodserver]

TASK [Start the httpd service] ******************************************************
ok: [prodserver]

TASK [Start the firewalld service] **************************************************
ok: [prodserver]

TASK [Allow http packets through the firewall] **************************************
ok: [prodserver]

TASK [Copy index.html file] *********************************************************
ok: [prodserver]

PLAY [Test web site] ****************************************************************

TASK [Gathering Facts] **************************************************************
ok: [prodserver]

TASK [test website] *****************************************************************
ok: [prodserver]

PLAY RECAP **************************************************************************
prodserver                 : ok=11   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 
```
---