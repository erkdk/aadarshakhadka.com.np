---
title: "PL - 008 — Ansible Conditionals, Handlers and Tags"
date: 2026-09-17
draft: false
---

### Ansible Conditionals with Loops
Ansible conditionals allow tasks to be executed only when specified conditions are true.
- with_items --> looping
- when --> conditional execution

>Note: The passwords shown here are lab-only examples. In a real environment, credentials should not be stored in plaintext in a playbook or variables file. Use Ansible Vault or another secrets-management solution.
```
[cnode@control-node ansible-lab]$ cat ansible.cfg 
[defaults]
inventory = ./inventory
remote_user = cnode

[privilege_escalation]
become = true
become_method = sudo
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
[cnode@control-node ansible-lab]$ vim vars/newemployee.yml
[cnode@control-node ansible-lab]$ cat vars/newemployee.yml 
---
newemp:
  - name: suman
    pass: suman123
    team: developers
  - name: raman
    pass: raman123
    team: developers
  - name: ravi
    pass: ravi123
    team: production
  - name: tom
    pass: tom123
    team: testers 
[cnode@control-node ansible-lab]$ vim create_newemp.yml
[cnode@control-node ansible-lab]$ cat create_newemp.yml
---
# Create new employees

- name: play to create new employee
  hosts: all
  vars_files:
    - vars/newemployee.yml

  tasks:
    - name: Create a group
      ansible.builtin.group:
        name: newemp
        state: present

    - name: Create developer accounts
      ansible.builtin.user:
        name: "{{ item.name }}"
        password: "{{ item.pass | password_hash('sha512') }}"
        groups: newemp
      with_items: "{{ newemp }}"
      when: 
        - item.team == "developers"
        - inventory_hostname in groups['develop']

    - name: Create production accounts
      ansible.builtin.user:
        name: "{{ item.name }}"
        password: "{{ item.pass | password_hash('sha512') }}"
        groups: newemp
      with_items: "{{ newemp }}"
      when: 
        - item.team == "production" 
        - inventory_hostname in groups['production']

    - name: Create test accounts
      ansible.builtin.user:
        name: "{{ item.name }}"
        password: "{{ item.pass | password_hash('sha512') }}"
        groups: newemp
      with_items: "{{ newemp }}"
      when: 
        - item.team == "testers" 
        - inventory_hostname in groups['test']
[cnode@control-node ansible-lab]$ 

PLAY [play to create new employee] **************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [testserver]
ok: [prodserver]
ok: [dev2]

TASK [Create a group] ***************************************************************
changed: [prodserver]
changed: [dev1]
changed: [testserver]
changed: [dev2]

TASK [Create developer accounts] ****************************************************
[DEPRECATION WARNING]: Encryption using the Python crypt module is deprecated. The 
Python crypt module is deprecated and will be removed from Python 3.13. Install the 
passlib library for continued encryption functionality. This feature will be removed
 in version 2.17. Deprecation warnings can be disabled by setting 
deprecation_warnings=False in ansible.cfg.
skipping: [prodserver] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'}) 
skipping: [testserver] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'}) 
skipping: [testserver] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'}) 
skipping: [testserver] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
skipping: [prodserver] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'}) 
skipping: [prodserver] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
skipping: [testserver] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 
skipping: [testserver]
skipping: [prodserver] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 
skipping: [prodserver]
changed: [dev1] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'})
changed: [dev2] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'})
changed: [dev2] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'})
skipping: [dev2] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
changed: [dev1] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'})
skipping: [dev2] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 
skipping: [dev1] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
skipping: [dev1] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 

TASK [Create production accounts] ***************************************************
skipping: [dev1] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'}) 
skipping: [dev1] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'}) 
skipping: [dev1] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
skipping: [dev1] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 
skipping: [dev2] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'}) 
skipping: [dev2] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'}) 
skipping: [dev2] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
skipping: [dev1]
skipping: [dev2] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 
skipping: [dev2]
skipping: [testserver] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'}) 
skipping: [testserver] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'}) 
skipping: [testserver] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
skipping: [prodserver] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'}) 
skipping: [testserver] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 
skipping: [prodserver] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'}) 
skipping: [testserver]
changed: [prodserver] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'})
skipping: [prodserver] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 

TASK [Create test accounts] *********************************************************
skipping: [dev1] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'}) 
skipping: [dev1] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'}) 
skipping: [dev1] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
skipping: [dev1] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 
skipping: [dev2] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'}) 
skipping: [dev1]
skipping: [dev2] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'}) 
skipping: [dev2] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
skipping: [dev2] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 
skipping: [testserver] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'}) 
skipping: [dev2]
skipping: [testserver] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'}) 
skipping: [testserver] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
skipping: [prodserver] => (item={'name': 'suman', 'pass': 'suman123', 'team': 'developers'}) 
skipping: [prodserver] => (item={'name': 'raman', 'pass': 'raman123', 'team': 'developers'}) 
skipping: [prodserver] => (item={'name': 'ravi', 'pass': 'ravi123', 'team': 'production'}) 
skipping: [prodserver] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'}) 
skipping: [prodserver]
changed: [testserver] => (item={'name': 'tom', 'pass': 'tom123', 'team': 'testers'})

PLAY RECAP **************************************************************************
dev1                       : ok=3    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
dev2                       : ok=3    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
prodserver                 : ok=3    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
testserver                 : ok=3    changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ ansible develop -m shell -a 'id suman; id raman; getent group newemp'
dev2 | CHANGED | rc=0 >>
uid=1012(suman) gid=1013(suman) groups=1013(suman),1012(newemp)
uid=1013(raman) gid=1014(raman) groups=1014(raman),1012(newemp)
newemp:x:1012:suman,raman
dev1 | CHANGED | rc=0 >>
uid=1012(suman) gid=1013(suman) groups=1013(suman),1012(newemp)
uid=1013(raman) gid=1014(raman) groups=1014(raman),1012(newemp)
newemp:x:1012:suman,raman
[cnode@control-node ansible-lab]$ ansible production -m shell -a 'id ravi; getent group newemp'
prodserver | CHANGED | rc=0 >>
uid=1011(ravi) gid=1011(ravi) groups=1011(ravi),1003(newemp)
newemp:x:1003:ravi
[cnode@control-node ansible-lab]$ ansible test -m shell -a 'id tom; getent group newemp'
testserver | CHANGED | rc=0 >>
uid=1011(tom) gid=1011(tom) groups=1011(tom),1003(newemp)
newemp:x:1003:tom
[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ ssh cnode@dev1
Last login: Sun Sep 13 07:30:16 2026 from 192.168.254.15
[cnode@dev1 ~]$ id suman
uid=1012(suman) gid=1013(suman) groups=1013(suman),1012(newemp)
[cnode@dev1 ~]$ id raman
uid=1013(raman) gid=1014(raman) groups=1014(raman),1012(newemp)
[cnode@dev1 ~]$ getent group newemp
newemp:x:1012:suman,raman
[cnode@dev1 ~]$
cnode@dev1 ~]$ exit
logout
Connection to dev1 closed.
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ssh cnode@prodserver
Last login: Sun Sep 13 07:30:56 2026 from 192.168.254.15
[cnode@prodserver ~]$ id ravi
uid=1011(ravi) gid=1011(ravi) groups=1011(ravi),1003(newemp)
[cnode@prodserver ~]$ getent group newemp
newemp:x:1003:ravi
[cnode@prodserver ~]$ exit
logout
Connection to prodserver closed.
[cnode@control-node ansible-lab]$ 
```

---
 
### Handlers

Handlers are special tasks in Ansible that run only when they are notified by another task.

A handler runs when it is notified by a task that reports a change. By default, notified handlers run after the regular tasks in the play have completed.

``` 
 
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ mkdir files
[cnode@control-node ansible-lab]$ vim files/index.html
[cnode@control-node ansible-lab]$ cat files/index.html
<html>
<head>
    <title>Ansible Handler Practice</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <p>This page was deployed using Ansible.</p>
</body>
</html>

[cnode@control-node ansible-lab]$ vim handlers.yml
[cnode@control-node ansible-lab]$ cat handlers.yml 
---
- name: Practice Ansible handlers
  hosts: develop
  become: true

  tasks:
    - name: Install Apache
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Copy website
      ansible.builtin.copy:
        src: files/index.html
        dest: /var/www/html/index.html
      notify:
        - Restart apache

  handlers:
    - name: Restart apache
      ansible: builtin.service:
        name: httpd
        state: restarted
[cnode@control-node ansible-lab]$
 
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check handlers.yml 
ERROR! We were unable to read either as JSON nor YAML, these are the errors we got from each:
JSON: Expecting value: line 1 column 1 (char 0)

Syntax Error while loading YAML.
  mapping values are not allowed in this context

The error appears to be in '/home/cnode/ansible-lab/handlers.yml': line 21, column 31, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

    - name: Restart apache
      ansible: builtin.service:
                              ^ here
[cnode@control-node ansible-lab]$ vim +21 handlers.yml 
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check handlers.yml 

playbook: handlers.yml
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook handlers.yml 

PLAY [Practice Ansible handlers] ****************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev2]
ok: [dev1]

TASK [Install Apache] ***************************************************************
changed: [dev1]
changed: [dev2]

TASK [Copy website] *****************************************************************
changed: [dev1]
changed: [dev2]

RUNNING HANDLER [Restart apache] ****************************************************
changed: [dev2]
changed: [dev1]

PLAY RECAP **************************************************************************
dev1                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook handlers.yml

PLAY [Practice Ansible handlers] ********************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************
ok: [dev1]
ok: [dev2]

TASK [Install Apache] *******************************************************************************************************************************************************
ok: [dev1]
ok: [dev2]

TASK [Copy website] *********************************************************************************************************************************************************
ok: [dev1]
ok: [dev2]

PLAY RECAP ******************************************************************************************************************************************************************
dev1                       : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ cat files/index.html 
<html>
<head>
    <title>Ansible Handler Practice</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <p>This page was deployed using Ansible.</p>
</body>
</html>
[cnode@control-node ansible-lab]$ vim files/index.html 
[cnode@control-node ansible-lab]$ cat files/index.html 
<html>
<head>
    <title>Ansible Handler Practice</title>
</head>
<body>
    <h1>Hello from Ansible Handler!</h1>
    <p>This page was deployed using Ansible.</p>
</body>
</html>
[cnode@control-node ansible-lab]$ ansible-playbook handlers.yml

PLAY [Practice Ansible handlers] ****************************************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Install Apache] ***************************************************************
ok: [dev1]
ok: [dev2]

TASK [Copy website] *****************************************************************
changed: [dev1]
changed: [dev2]

RUNNING HANDLER [Restart apache] ****************************************************
changed: [dev2]
changed: [dev1]

PLAY RECAP **************************************************************************
dev1                       : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$  
```

#### Here:
- If index.html is changed or copied for the first time, the Copy website task reports changed and the Restart apache handler runs.
- If index.html is already up to date, the task reports ok and the handler does not run.
- If index.html is modified later, the task reports changed again and the handler runs again.

#### Conclusion:
Handlers are event-driven tasks that are triggered by notifications from tasks that report a change.

---

### Ansible Tags
A tag is a label assigned to an Ansible task that allows specific tasks or groups of tasks to be selectively executed from a playbook.

Tags are useful when a playbook contains multiple independent tasks and you want to execute only a particular set of tasks without running the entire playbook.

Ansible tags are labels assigned to tasks that allow selected tasks to be executed without running the entire playbook.

```
[cnode@control-node ansible-lab]$ vim tags_maildeploy.yml 
[cnode@control-node ansible-lab]$ cat tags_maildeploy.yml 
---
# Ansible Tags Practice

- name: Deploy SMTP and POP/IMAP services
  hosts: testprod
  become: true

  tasks:

    - name: Install SMTP server package
      ansible.builtin.yum:
        name: postfix
        state: present
      tags:
        - smtp_deploy

    - name: Start SMTP service
      ansible.builtin.service:
        name: postfix
        state: started
        enabled: true
      tags:
        - smtp_deploy

    - name: Allow SMTP packets through firewall
      ansible.posix.firewalld:
        service: smtp
        state: enabled
        permanent: true
        immediate: true
      tags:
        - smtp_deploy

    - name: Install POP and IMAP server package
      ansible.builtin.yum:
        name: dovecot
        state: present
      tags:
        - dovecot_deploy

    - name: Start POP and IMAP service
      ansible.builtin.service:
        name: dovecot
        state: started
        enabled: true
      tags:
        - dovecot_deploy

    - name: Allow POP3 packets through firewall
      ansible.posix.firewalld:
        service: pop3
        state: enabled
        permanent: true
        immediate: true
      tags:
        - dovecot_deploy
[cnode@control-node ansible-lab]$ ansible-playbook tags_maildeploy.yml --list-tags

playbook: tags_maildeploy.yml

  play #1 (testprod): Deploy SMTP and POP/IMAP services	TAGS: []
      TASK TAGS: [dovecot_deploy, smtp_deploy]
      

[cnode@control-node ansible-lab]$ ansible-playbook tags_maildeploy.yml --list-tags --tags smtp_deploy

playbook: tags_maildeploy.yml

  play #1 (testprod): Deploy SMTP and POP/IMAP services	TAGS: []
      TASK TAGS: [smtp_deploy]
[cnode@control-node ansible-lab]$ ansible-playbook tags_maildeploy.yml --list-tasks --tags dovecot_deploy

playbook: tags_maildeploy.yml

  play #1 (testprod): Deploy SMTP and POP/IMAP services	TAGS: []
    tasks:
      Install POP and IMAP server package	TAGS: [dovecot_deploy]
      Start POP and IMAP service	TAGS: [dovecot_deploy]
      Allow POP3 packets through firewall	TAGS: [dovecot_deploy]
[cnode@control-node ansible-lab]$ 


[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check tags_maildeploy.yml 

playbook: tags_maildeploy.yml
[cnode@control-node ansible-lab]$ ansible-playbook tags_maildeploy.yml --tags smtp_deploy

PLAY [Deploy SMTP and POP/IMAP services] ********************************************

TASK [Gathering Facts] **************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Install SMTP server package] **************************************************
ok: [prodserver]
ok: [testserver]

TASK [Start SMTP service] ***********************************************************
ok: [testserver]
ok: [prodserver]

TASK [Allow SMTP packets through firewall] ******************************************
ok: [prodserver]
ok: [testserver]

PLAY RECAP **************************************************************************
prodserver                 : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook tags_maildeploy.yml --tags dovecot_deploy

PLAY [Deploy SMTP and POP/IMAP services] ********************************************

TASK [Gathering Facts] **************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Install POP and IMAP server package] ******************************************
changed: [prodserver]
changed: [testserver]

TASK [Start POP and IMAP service] ***************************************************
changed: [prodserver]
changed: [testserver]

TASK [Allow POP3 packets through firewall] ******************************************
changed: [testserver]
changed: [prodserver]

PLAY RECAP **************************************************************************
prodserver                 : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 
```
---