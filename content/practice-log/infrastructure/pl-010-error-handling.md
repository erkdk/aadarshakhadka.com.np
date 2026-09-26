---
title: "PL - 010 — Ansible Practice: Error Handling, File Management & Remote File Operations"
date: 2026-09-23
draft: false
---

### Error Handling in Automation

```
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
[cnode@control-node ansible-lab]$ 

[cnode@testserver ~]$ ls
[cnode@testserver ~]$ rpm -q httpd nginx mariadb-server php
package httpd is not installed
package nginx is not installed
mariadb-server-10.11.18-1.el10.x86_64
package php is not installed
[cnode@testserver ~]$ sudo rpm -e --nodeps mariadb-server
Removed '/etc/systemd/system/multi-user.target.wants/mariadb.service'.
Removed '/etc/systemd/system/mysql.service'.
Removed '/etc/systemd/system/mysqld.service'.
[cnode@testserver ~]$ rpm -q httpd nginx mariadb-server php
package httpd is not installed
package nginx is not installed
package mariadb-server is not installed
package php is not installed
[cnode@testserver ~]$ 

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ vi playbook_handle-error.yml
[cnode@control-node ansible-lab]$ vim playbook_handle-error.yml 

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  playbook_handle-error.yml
[cnode@control-node ansible-lab]$ cat playbook_handle-error.yml 
---
# Installing different packages

- name: Play handling errors
  hosts: test
  become: true

  tasks:

    - name: Install the latest version of Apache
      ansible.builtin.dnf:
        name: htpd
        state: latest

    - name: Install the latest version of MariaDB
      ansible.builtin.dnf:
        name: mariadb-server
        state: latest

    - name: Install the latest version of PHP
      ansible.builtin.dnf:
        name: php
        state: latest

    - name: Install the latest version of Samba
      ansible.builtin.dnf:
        name: samba
        state: latest
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check playbook_handle-error.yml

playbook: playbook_handle-error.yml
[cnode@control-node ansible-lab]$ ansible-playbook playbook_handle-error.yml

PLAY [Play handling errors] *********************************************************

TASK [Gathering Facts] **************************************************************
ok: [testserver]

TASK [Install the latest version of Apache] *****************************************
fatal: [testserver]: FAILED! => {"changed": false, "failures": ["No package htpd available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}

PLAY RECAP **************************************************************************
testserver                 : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


# If first tasks fails, remaining other automatically fails because ansible runs sequentially, thus 
using errors handling concepts, where previous errors don't depend on successive tasks.


# To ignore error and proceed the next module
# The preceeding module should not depend on the previous module
# For example: web deployment depends on web server existance/installation


[cnode@control-node ansible-lab]$ vim playbook_handle-error.yml
[cnode@control-node ansible-lab]$ cat playbook_handle-error.yml 
---
# Installing different packages

- name: Play handling errors
  hosts: test
  become: true

  tasks:

    - name: Install the latest version of Apache
      ansible.builtin.dnf:
        name: htpd
        state: latest
      ignore_errors: yes

    - name: Install the latest version of MariaDB
      ansible.builtin.dnf:
        name: mariadb-server
        state: latest
      ignore_errors: yes

    - name: Install the latest version of PHP
      ansible.builtin.dnf:
        name: php
        state: latest
      ignore_errors: yes

    - name: Install the latest version of Samba
      ansible.builtin.dnf:
        name: samba
        state: latest
      ignore_errors: yes
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check playbook_handle-error.yml

playbook: playbook_handle-error.yml
[cnode@control-node ansible-lab]$ ansible-playbook --check playbook_handle-error.yml

PLAY [Play handling errors] *********************************************************

TASK [Gathering Facts] **************************************************************
ok: [testserver]

TASK [Install the latest version of Apache] *****************************************
fatal: [testserver]: FAILED! => {"changed": false, "failures": ["No package htpd available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
...ignoring

TASK [Install the latest version of MariaDB] ****************************************
changed: [testserver]

TASK [Install the latest version of PHP] ********************************************
changed: [testserver]

TASK [Install the latest version of Samba] ******************************************
changed: [testserver]

PLAY RECAP **************************************************************************
testserver                 : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   

[cnode@control-node ansible-lab]$ 

#  errors are ignored and next tasks are performed successfully.

# But, actually in this case, httpd gets installed as it depends on mariadb-server

# NOTE: don't use the similar packages for same purpose
# For example: if using httpd don't use NGINX, there will be port conflict.

---

[cnode@control-node ansible-lab]$ cat playbook_handle-error-2.yml 
---
# Installing different packages

- name: Play handling errors
  hosts: test
  become: true

  tasks:

    - name: Install the latest version of Apache
      ansible.builtin.dnf:
        name: httpd
        state: latest
      register: httpd_out
      ignore_errors: yes

    - name: Install the latest version of Nginx
      ansible.builtin.dnf:
        name: nginx
        state: latest
      when: "http_out.rc == 1"
      ignore_errors: yes

    - name: Install the latest version of MariaDB
      ansible.builtin.dnf:
        name: mariadb-server
        state: latest
      ignore_errors: yes

    - name: Install the latest version of PHP
      ansible.builtin.dnf:
        name: php
        state: latest
      ignore_errors: yes

    - name: Install the latest version of Samba
      ansible.builtin.dnf:
        name: samba
        state: latest
      ignore_errors: yes
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ cp playbook_handle-error.yml playbook_handle-error-2.yml 
[cnode@control-node ansible-lab]$ vim playbook_handle-error-2.yml 
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check playbook_handle-error-2.yml

playbook: playbook_handle-error-2.yml
[cnode@control-node ansible-lab]$ ansible-playbook --check playbook_handle-error-2.yml

PLAY [Play handling errors] *********************************************************

TASK [Gathering Facts] **************************************************************
ok: [testserver]

TASK [Install the latest version of Apache] *****************************************
changed: [testserver]

TASK [Install the latest version of Nginx] ******************************************
fatal: [testserver]: FAILED! => {"msg": "The conditional check 'http_out.rc == 1' failed. The error was: error while evaluating conditional (http_out.rc == 1): 'http_out' is undefined. 'http_out' is undefined\n\nThe error appears to be in '/home/cnode/ansible-lab/playbook_handle-error-2.yml': line 21, column 13, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n        state: latest\n      when: \"http_out.rc == 1\"\n            ^ here\n"}
...ignoring

TASK [Install the latest version of MariaDB] ****************************************
changed: [testserver]

TASK [Install the latest version of PHP] ********************************************
changed: [testserver]

TASK [Install the latest version of Samba] ******************************************
changed: [testserver]

PLAY RECAP **************************************************************************
testserver                 : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
[cnode@control-node ansible-lab]$ 


# But

[cnode@control-node ansible-lab]$ vim playbook_handle-error-2.yml 
[cnode@control-node ansible-lab]$ cat playbook_handle-error-2.yml 
---
# Installing different packages

- name: Play handling errors
  hosts: test
  become: true

  tasks:

    - name: Install the latest version of Apache
      ansible.builtin.dnf:
        name: htpd
        state: latest
      register: httpd_out
      ignore_errors: yes

    - name: Install the latest version of Nginx
      ansible.builtin.dnf:
        name: nginx
        state: latest
      when: "http_out.rc == 1"
      ignore_errors: yes

    - name: Install the latest version of MariaDB
      ansible.builtin.dnf:
        name: mariadb-server
        state: latest
      ignore_errors: yes

    - name: Install the latest version of PHP
      ansible.builtin.dnf:
        name: php
        state: latest
      ignore_errors: yes

    - name: Install the latest version of Samba
      ansible.builtin.dnf:
        name: samba
        state: latest
      ignore_errors: yes
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check playbook_handle-error-2.yml

playbook: playbook_handle-error-2.yml
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook --check playbook_handle-error-2.yml

PLAY [Play handling errors] *********************************************************

TASK [Gathering Facts] **************************************************************
ok: [testserver]

TASK [Install the latest version of Apache] *****************************************
fatal: [testserver]: FAILED! => {"changed": false, "failures": ["No package htpd available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}
...ignoring

TASK [Install the latest version of Nginx] ******************************************
fatal: [testserver]: FAILED! => {"msg": "The conditional check 'http_out.rc == 1' failed. The error was: error while evaluating conditional (http_out.rc == 1): 'http_out' is undefined. 'http_out' is undefined\n\nThe error appears to be in '/home/cnode/ansible-lab/playbook_handle-error-2.yml': line 21, column 13, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n        state: latest\n      when: \"http_out.rc == 1\"\n            ^ here\n"}
...ignoring

TASK [Install the latest version of MariaDB] ****************************************
changed: [testserver]

TASK [Install the latest version of PHP] ********************************************
changed: [testserver]

TASK [Install the latest version of Samba] ******************************************
changed: [testserver]

PLAY RECAP **************************************************************************
testserver                 : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=2   

[cnode@control-node ansible-lab]$ 
```
---


### Ansible block, rescue and always

Ansible provides block, rescue, and always to organize tasks and handle errors in a controlled way.

#### How It Works
    
- block — Contains the main tasks that Ansible attempts to execute.
- rescue — Runs only when a task inside the block fails.
- always — Runs regardless of whether the block succeeds or the rescue section runs.

```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  playbook_handle-error-2.yml  playbook_handle-error.yml
[cnode@control-node ansible-lab]$ cp playbook_handle-error-2.yml playbook_handle-error-3.yml
[cnode@control-node ansible-lab]$ ls 
ansible.cfg  playbook_handle-error-2.yml  playbook_handle-error.yml
inventory    playbook_handle-error-3.yml

[cnode@control-node ansible-lab]$ vim playbook_handle-error-3.yml 
[cnode@control-node ansible-lab]$ cat playbook_handle-error-3.yml
---
# Handling errors using Block-Rescue-Always

- name: Handle errors using block, rescue, and always
  hosts: test
  become: true

  tasks:
    - name: Install Apache or fall back to Nginx
      block:
        - name: Install the latest version of Apache
          ansible.builtin.dnf:
            name: httpd
            state: latest

      rescue:
        - name: Install the latest version of Nginx if Apache installation fails
          ansible.builtin.dnf:
            name: nginx
            state: latest

      always:
        - name: Install the latest version of MariaDB
          ansible.builtin.dnf:
            name: mariadb-server
            state: latest
[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check playbook_handle-error-3.yml

playbook: playbook_handle-error-3.yml
[cnode@control-node ansible-lab]$ ansible-playbook --check playbook_handle-error-3.yml 

PLAY [Handle errors using block, rescue, and always] ********************************

TASK [Gathering Facts] **************************************************************
ok: [testserver]

TASK [Install the latest version of Apache] *****************************************
changed: [testserver]

TASK [Install the latest version of MariaDB] ****************************************
changed: [testserver]

PLAY RECAP **************************************************************************
testserver                 : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ vim playbook_handle-error-3.yml
[cnode@control-node ansible-lab]$ cat playbook_handle-error-3.yml
---
# Handling errors using Block-Rescue-Always

- name: Handle errors using block, rescue, and always
  hosts: test
  become: true

  tasks:
    - name: Install Apache or fall back to Nginx
      block:
        - name: Install the latest version of Apache
          ansible.builtin.dnf:
            name: htpd
            state: latest

      rescue:
        - name: Install the latest version of Nginx if Apache installation fails
          ansible.builtin.dnf:
            name: nginx
            state: latest

      always:
        - name: Install the latest version of MariaDB
          ansible.builtin.dnf:
            name: mariadb-server
            state: latest
[cnode@control-node ansible-lab]$ ansible-playbook --check playbook_handle-error-3.yml

PLAY [Handle errors using block, rescue, and always] ********************************

TASK [Gathering Facts] **************************************************************
ok: [testserver]

TASK [Install the latest version of Apache] *****************************************
fatal: [testserver]: FAILED! => {"changed": false, "failures": ["No package htpd available."], "msg": "Failed to install some of the specified packages", "rc": 1, "results": []}

TASK [Install the latest version of Nginx if Apache installation fails] *************
changed: [testserver]

TASK [Install the latest version of MariaDB] ****************************************
changed: [testserver]

PLAY RECAP **************************************************************************
testserver                 : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0   

[cnode@control-node ansible-lab]$ 
```
---

```
### File Transfers | File Deployment & Dynamic Configuration Management

# 'file' module

--> any operations: vi, touch, mkdir, rm, ln, chmod, chown, SELinux, ... etc operations can be done using file mode

[cnode@control-node ansible-lab]$ ansible-doc file

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ vi file_operation_playbook.yml
[cnode@control-node ansible-lab]$ vim file_operation_playbook.yml
[cnode@control-node ansible-lab]$ cat file_operation_playbook.yml 
---
# file operations

- name: play to create a playbook
  hosts: develop

  tasks:
    - name: Create an empty file
      ansible.builtin.file:
        path: /home/cnode/devinfo
        state: touch
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check file_operation_playbook.yml

playbook: file_operation_playbook.yml
[cnode@control-node ansible-lab]$ ansible-playbook file_operation_playbook.yml

PLAY [play to create a playbook] ****************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Create an empty file] *********************************************************
changed: [dev2]
changed: [dev1]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@dev1 ~]$ ls
devinfo
[cnode@dev1 ~]$ ls -l /home/cnode/devinfo 
-rw-r--r--. 1 root root 0 Sep 14 06:33 /home/cnode/devinfo
[cnode@dev1 ~]$ 
[cnode@dev1 ~]$ whoami
cnode
[cnode@dev1 ~]$ 

 # But, owner is root and group is root. As, ( become = true )
 
[cnode@control-node ansible-lab]$ vim file_operation_playbook.yml
[cnode@control-node ansible-lab]$ cat file_operation_playbook.yml 
---
# file operations

- name: play to create a playbook
  hosts: develop

  tasks:
    - name: Create an empty file
      ansible.builtin.file:
        path: /home/cnode/devinfo
        state: touch
        owner: ram
        group: devgroup
        mode: '0700'
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check file_operation_playbook.yml

playbook: file_operation_playbook.yml
[cnode@control-node ansible-lab]$ ansible-playbook file_operation_playbook.yml

PLAY [play to create a playbook] ****************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Create an empty file] *********************************************************
changed: [dev1]
changed: [dev2]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@dev1 ~]$ ls -l devinfo 
-rwx------. 1 ram devgroup 0 Sep 14 06:48 devinfo
[cnode@dev1 ~]$ 

[cnode@dev2 ~]$ ls -l devinfo 
-rwx------. 1 ram devgroup 0 Sep 14 06:48 devinfo
[cnode@dev2 ~]$ 
```
---

---


```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ cp ../done/file_operation_playbook.yml file_createdir.yml
[cnode@control-node ansible-lab]$ ls
ansible.cfg  file_createdir.yml  inventory
[cnode@control-node ansible-lab]$ vim file_createdir.yml
[cnode@control-node ansible-lab]$ cat file_createdir.yml
---
# Create a directory

- name: playbook to create a directory
  hosts: develop
  
  tasks:
   - name: Create a directory
     ansible.builtin.file:
       path: /home/cnode/practice/ansible
       state: directory
       recurse: yes
       owner: ram
       group: devgroup
       mode: '3770'
[cnode@control-node ansible-lab]$ 


[cnode@dev1 ~]$ ls
[cnode@dev1 ~]$ 


[cnode@dev2 ~]$ ls
[cnode@dev2 ~]$ 

[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check file_createdir.yml

playbook: file_createdir.yml
[cnode@control-node ansible-lab]$ ansible-playbook --check file_createdir.yml

PLAY [playbook to create a directory] ***********************************************

TASK [Gathering Facts] **************************************************************
ok: [dev2]
ok: [dev1]

TASK [Create a directory] ***********************************************************
changed: [dev1]
changed: [dev2]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ ansible-playbook file_createdir.yml

PLAY [playbook to create a directory] ***********************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Create a directory] ***********************************************************
changed: [dev1]
changed: [dev2]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@dev1 ~]$ ls
practice
[cnode@dev1 ~]$ ls -ld practice
drwxrws--T. 3 ram devgroup 21 Sep 14 08:53 practice 
[cnode@dev1 ~]$ cd practice/
-bash: cd: practice/: Permission denied
[cnode@dev1 ~]$ 


[cnode@dev2 ~]$ ls
practice
[cnode@dev2 ~]$ ls -ld practice/
drwxrws--T. 3 ram devgroup 21 Sep 14 08:54 practice/
[cnode@dev2 ~]$ 

# Creating soft link

[cnode@control-node ansible-lab]$ ls
ansible.cfg  file_createdir.yml  inventory
[cnode@control-node ansible-lab]$ cp file_createdir.yml file_softlink.yml
[cnode@control-node ansible-lab]$ ansible-doc file
/EXAMPLES


[cnode@control-node ansible-lab]$ ls
ansible.cfg  file_createdir.yml  file_softlink.yml  inventory
[cnode@control-node ansible-lab]$ vim file_softlink.yml 
[cnode@control-node ansible-lab]$ cat file_softlink.yml 
---
# Create a softlink

- name: playbook to create a softlink
  hosts: develop
  
  tasks:
   - name: Create a softlink (symlink)
     ansible.builtin.file:
       src: /home/cnode/practice/ansible
       dest: /tmp/softps
       state: link
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check file_softlink.yml

playbook: file_softlink.yml
[cnode@control-node ansible-lab]$ ansible-playbook --check file_softlink.yml

PLAY [playbook to create a softlink] ************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev2]
ok: [dev1]

TASK [Create a softlink (symlink)] **************************************************
changed: [dev1]
changed: [dev2]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@dev1 ~]$ ls
practice
[cnode@dev1 ~]$ ls -ld
drwx------. 5 cnode cnode 127 Sep 15 10:46 .
[cnode@dev1 ~]$ sudo tree
.
└── practice
    └── ansible

3 directories, 0 files
[cnode@dev1 ~]$ 


[cnode@dev2 ~]$ ls 
practice
[cnode@dev2 ~]$ 

[cnode@control-node ansible-lab]$ ls
ansible.cfg  file_createdir.yml  file_softlink.yml  inventory
[cnode@control-node ansible-lab]$ ansible-playbook file_softlink.yml

PLAY [playbook to create a softlink] ************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev2]
ok: [dev1]

TASK [Create a softlink (symlink)] **************************************************
changed: [dev1]
changed: [dev2]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@dev1 ~]$ ls
practice
[cnode@dev1 ~]$ sudo ls /tmp/
softps
[cnode@dev1 ~]$ sudo ls -l /tmp/softps 
lrwxrwxrwx. 1 root root 28 Sep 15 10:50 /tmp/softps -> /home/cnode/practice/ansible
[cnode@dev1 ~]$ 

[cnode@dev2 ~]$ ls 
practice
[cnode@dev2 ~]$ ls /tmp/
softps

[cnode@dev2 ~]$ ls -l /tmp/softps 
lrwxrwxrwx. 1 root root 28 Sep 15 10:50 /tmp/softps -> /home/cnode/practice/ansible
[cnode@dev2 ~]$ 

[cnode@dev1 ~]$ ls -Z devinfo 
unconfined_u:object_r:user_home_t:s0 devinfo
[cnode@dev1 ~]$ 

[cnode@dev2 ~]$ ls -Z devinfo 
unconfined_u:object_r:user_home_t:s0 devinfo
[cnode@dev2 ~]$ 

# Changing SELinux Security Context Type to: samba_share_t
 
[cnode@control-node ansible-lab]$ pwd
/home/cnode/ansible-lab
[cnode@control-node ansible-lab]$ cp ../done/file_operation_playbook.yml file_selinux.yml
[cnode@control-node ansible-lab]$ vim file_selinux.yml
[cnode@control-node ansible-lab]$ cat file_selinux.yml 
---
# file operations

- name: play to create a playbook
  hosts: develop

  tasks:
    - name: Create an empty file
      ansible.builtin.file:
        path: /home/cnode/devinfo
        owner: ram
        group: devgroup
        mode: '0600'
        setype: samba_share_t
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check file_selinux.yml

playbook: file_selinux.yml
[cnode@control-node ansible-lab]$ ansible-playbook file_selinux.yml

PLAY [play to create a playbook] ****************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Create an empty file] *********************************************************
changed: [dev2]
changed: [dev1]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@dev1 ~]$ ls -Z devinfo 
unconfined_u:object_r:samba_share_t:s0 devinfo
[cnode@dev1 ~]$ 


[cnode@dev2 ~]$ ls -Z devinfo
unconfined_u:object_r:samba_share_t:s0 devinfo
[cnode@dev2 ~]$ 

# fetching log files from different VMs on the control node:

# On hosts machines:


[cnode@dev1 ~]$ ls /var/log
anaconda         dnf.log              maillog            secure
audit            dnf.rpm.log          maillog-20260906   secure-20260906
btmp             firewalld            maillog-20260913   secure-20260913
chrony           hawkey.log           messages           spooler
cron             hawkey.log-20260906  messages-20260906  spooler-20260906
cron-20260906    hawkey.log-20260913  messages-20260913  spooler-20260913
cron-20260913    httpd                private            sssd
dnf.librepo.log  lastlog              samba              wtmp
[cnode@dev1 ~]$ 

[cnode@dev2 ~]$ ls /var/log
anaconda         dnf.log              maillog            secure
audit            dnf.rpm.log          maillog-20260906   secure-20260906
btmp             firewalld            maillog-20260913   secure-20260913
chrony           hawkey.log           messages           spooler
cron             hawkey.log-20260906  messages-20260906  spooler-20260906
cron-20260906    hawkey.log-20260913  messages-20260913  spooler-20260913
cron-20260913    httpd                private            sssd
dnf.librepo.log  lastlog              samba              wtmp
[cnode@dev2 ~]$ 


[cnode@control-node ansible-lab]$ vim fetch_logfiles.yml
[cnode@control-node ansible-lab]$ cat fetch_logfiles.yml 
---
# Fetch the files

- name: play to fetch remote files to the local system
  hosts: all
  tasks:

    - name: fetch files from remote hosts to the local
      ansible.builtin.fetch:
        src: /var/log/secure
        dest: /tmp/log/secure
        flat: no
[cnode@control-node ansible-lab]$ ls /tmp/log/secure
ls: cannot access '/tmp/log/secure': No such file or directory
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check fetch_logfiles.yml 
 
playbook: fetch_logfiles.yml
[cnode@control-node ansible-lab]$ ansible-playbook fetch_logfiles.yml 

PLAY [play to fetch remote files to the local system] *******************************

TASK [Gathering Facts] **************************************************************
ok: [prodserver]
ok: [dev2]
ok: [testserver]
ok: [dev1]

TASK [fetch files from remote hosts to the local] ***********************************
changed: [testserver]
changed: [dev1]
changed: [prodserver]
changed: [dev2]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
prodserver                 : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ ls /tmp/log/secure/
dev1  dev2  prodserver  testserver

[cnode@control-node ansible-lab]$ ls /tmp/log/secure/testserver/var/log/secure 
/tmp/log/secure/testserver/var/log/secure
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ls /tmp/log/secure/dev1/var/log/secure 
/tmp/log/secure/dev1/var/log/secure
[cnode@control-node ansible-lab]$ 

# adding lines on files

[cnode@control-node ansible-lab]$ vim lineinfile_addline.yml
[cnode@control-node ansible-lab]$ cat lineinfile_addline.yml 
---

- name: play to add data in a file
  hosts: develop

  tasks:
    - name: Add a line to a file if the file does not exist, without passing regexp
      ansible.builtin.lineinfile:
        path: /home/cnode/file1
        line: this is the first line on 192.168.254.16 foo.lab.net foo
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check lineinfile_addline.yml

playbook: lineinfile_addline.yml
[cnode@control-node ansible-lab]$ ansible-playbook --check lineinfile_addline.yml 

PLAY [play to add data in a file] ***************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Add a line to a file if the file does not exist, without passing regexp] ******
fatal: [dev2]: FAILED! => {"changed": false, "msg": "Destination /home/cnode/file1 does not exist !", "rc": 257}
fatal: [dev1]: FAILED! => {"changed": false, "msg": "Destination /home/cnode/file1 does not exist !", "rc": 257}

PLAY RECAP **************************************************************************
dev1                       : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
dev2                       : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ vim lineinfile_addline.yml 
[cnode@control-node ansible-lab]$ cat lineinfile_addline.yml 
---

- name: play to add data in a file
  hosts: develop

  tasks:
    - name: Add a line to a file if the file does not exist, without passing regexp
      ansible.builtin.lineinfile:
        path: /home/cnode/file1
        line: this is the first line on 192.168.254.16 foo.lab.net foo
        create: yes
[cnode@control-node ansible-lab]$ ansible-playbook --check lineinfile_addline.yml 

PLAY [play to add data in a file] ***************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Add a line to a file if the file does not exist, without passing regexp] ******
changed: [dev2]
changed: [dev1]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ vim lineinfile_addline.yml 
[cnode@control-node ansible-lab]$ cat lineinfile_addline.yml
---

- name: play to add data in a file
  hosts: develop

  tasks:
    - name: Add a line to a file if the file does not exist, without passing regexp
      ansible.builtin.lineinfile:
        path: /home/cnode/devinfo
        line: this is the first line on 192.168.254.16 foo.lab.net foo
        create: yes
[cnode@control-node ansible-lab]$ ansible-playbook lineinfile_addline.yml

PLAY [play to add data in a file] ***************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Add a line to a file if the file does not exist, without passing regexp] ******
changed: [dev2]
changed: [dev1]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@dev1 ~]$ sudo cat devinfo 
this is the first line on 192.168.254.16 foo.lab.net foo
[cnode@dev1 ~]$ 

[cnode@control-node ansible-lab]$ vim lineinfile_addline.yml 
[cnode@control-node ansible-lab]$ cat lineinfile_addline.yml 
---

- name: play to add data in a file
  hosts: develop

  tasks:
    - name: Add a line to a file if the file does not exist, without passing regexp
      ansible.builtin.lineinfile:
        path: /home/cnode/devinfo
        line: this is the second line added in devinfo
        state: present
[cnode@control-node ansible-lab]$ ansible-playbook lineinfile_addline.yml 

[cnode@dev1 ~]$ sudo cat devinfo
this is the first line on 192.168.254.16 foo.lab.net foo
this is the second line added in devinfo
[cnode@dev1 ~]$ 

  
  [cnode@control-node ansible-lab]$ vim lineinfile_addline.yml 
[cnode@control-node ansible-lab]$ cat lineinfile_addline.yml 
---

- name: play to add data in a file
  hosts: develop

  tasks:
    - name: Add a line to a file if the file does not exist, without passing regexp
      ansible.builtin.lineinfile:
        path: /home/cnode/devinfo
        line: this is the second line added in devinfo
        state: absent
[cnode@control-node ansible-lab]$ ansible-playbook lineinfile_addline.yml 

PLAY [play to add data in a file] ***************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Add a line to a file if the file does not exist, without passing regexp] ******
changed: [dev1]
changed: [dev2]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 
  
[cnode@dev1 ~]$ sudo cat devinfo
this is the first line on 192.168.254.16 foo.lab.net foo
[cnode@dev1 ~]$ 


# Changing the SELinux 

[cnode@dev1 ~]$ getenforce 
Enforcing
[cnode@dev1 ~]$ cat /etc/sysconfig/selinux 
...
SELINUX=enforcing
...
[cnode@dev1 ~]$ cat /etc/selinux/config
...
SELINUX=enforcing
...

[cnode@control-node ansible-lab]$ vim lineinfile_addline.yml 
[cnode@control-node ansible-lab]$ cat lineinfile_addline.yml 
---
# Search and replace 

- name: play to add data in a file
  hosts: develop

  tasks:

    - name: Change SELinux mode
      ansible.builtin.lineinfile:
        path: /etc/selinux/config
        regexp: '^SELINUX='
        line: SELINUX=permissive
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check lineinfile_addline.yml

playbook: lineinfile_addline.yml
[cnode@control-node ansible-lab]$ ansible-playbook lineinfile_addline.yml

PLAY [play to add data in a file] ***************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev2]
ok: [dev1]

TASK [Change SELinux mode] **********************************************************
changed: [dev1]
changed: [dev2]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@dev1 ~]$ cat /etc/selinux/config
...
SELINUX=permissive
...


# blockinfile multiple lines

[cnode@dev1 ~]$ ls
devinfo  practice
[cnode@dev1 ~]$ 

[cnode@dev2 ~]$ ls
devinfo  practice
[cnode@dev2 ~]$ 


[cnode@control-node ansible-lab]$ vim blockinfile_addblock.yml 
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check blockinfile_addblock.yml

playbook: blockinfile_addblock.yml
[cnode@control-node ansible-lab]$ cat blockinfile_addblock.yml
---
# Add a block of lines in a file

- name: To add a block in file
  hosts: develop

  tasks:
  - name: Insert/Update multiple lines
    ansible.builtin.blockinfile:
      path: /home/cnode/devinfo
      append_newline: true
      prepend_newline: true
      block: |
        Match User ansible-agent -- addedline1
        PasswordAuthentication no -- addedline2
        <add multiple lines>
[cnode@control-node ansible-lab]$ ansible-playbook blockinfile_addblock.yml

PLAY [To add a block in file] *******************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev2]
ok: [dev1]

TASK [Insert/Update multiple lines] *************************************************
changed: [dev2]
changed: [dev1]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@dev1 ~]$ sudo cat devinfo 
this is the first line on 192.168.254.16 foo.lab.net foo

# BEGIN ANSIBLE MANAGED BLOCK
Match User ansible-agent -- addedline1
PasswordAuthentication no -- addedline2
<add multiple lines>
# END ANSIBLE MANAGED BLOCK
[cnode@dev1 ~]$ 
```

### Delete a file
```
[cnode@dev1 ~]$ ls
devinfo  practice
[cnode@dev1 ~]$ 

[cnode@dev2 ~]$ ls
devinfo  practice
[cnode@dev2 ~]$ 

[cnode@control-node ansible-lab]$ vim file_deletefile.yml

[cnode@control-node ansible-lab]$ cat file_deletefile.yml 
---
# file operation: delete a file

- name: play to delete a file
  hosts: develop

  tasks:
    - name: Delete the existing files
      ansible.builtin.file:
        path: /home/cnode/devinfo
        state: absent
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook file_deletefile.yml 

PLAY [play to delete a file] ********************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Delete the existing files] ****************************************************
changed: [dev2]
changed: [dev1]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$

[cnode@dev1 ~]$ ls
practice
[cnode@dev1 ~]$

[cnode@dev2 ~]$ ls
devinfo  practice
[cnode@dev2 ~]$ ls
practice
[cnode@dev2 ~]$ 
```
---