---
title: "PL - 005 — Ansible Practice: Variables, Loops, Register, and Debug"
date: 2026-09-10
draft: false
---

### Lab Session

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
[cnode@control-node ansible-lab]$

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  vars-play.yml
[cnode@control-node ansible-lab]$ vim vars-play.yml 
[cnode@control-node ansible-lab]$ cat vars-play.yml 
---
# Use of variables

- name: Deploy web, mail, and database services to test and production hosts
  hosts: testprod
  vars:
    web_pkg: httpd
    mail_pkg: postfix
    db_pkg: mariadb-server
    services:
      - httpd
      - postfix
      - mariadb
    firewall_rules:
      - http
      - https
      - smtp
      - mysql

  tasks:
    - name: Install the latest version of the packages
      ansible.builtin.yum:
        name:
          - "{{ web_pkg }}"
          - "{{ mail_pkg }}"
          - "{{ db_pkg }}"
        state: latest

    - name: Start services, if not started
      ansible.builtin.service:
        name: "{{ item }}"
        state: started
        enabled: yes
      with_items: "{{ services }}"

    - name: Permanently enable http, https, smtp, mysql
      ansible.posix.firewalld:
        service: "{{ item }}"
        state: enabled
        permanent: true
        immediate: true
      loop: "{{ firewall_rules }}"
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check vars-play.yml

playbook: vars-play.yml
[cnode@control-node ansible-lab]$ echo $?
0
[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.ping
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
[cnode@control-node ansible-lab]$ ansible-playbook vars-play.yml

PLAY [play to deploy web, mail and db services on the test and prod services] *******

TASK [Gathering Facts] **************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Install the latest version of the packages] ***********************************
changed: [prodserver]
changed: [testserver]

TASK [Start services, if not started] ***********************************************
changed: [testserver] => (item=httpd)
changed: [prodserver] => (item=httpd)
changed: [testserver] => (item=postfix)
changed: [prodserver] => (item=postfix)
changed: [testserver] => (item=mariadb)
changed: [prodserver] => (item=mariadb)

TASK [Permanently enable http, https, smtp, mysql] **********************************
changed: [testserver] => (item=http)
changed: [prodserver] => (item=http)
changed: [prodserver] => (item=https)
changed: [testserver] => (item=https)
changed: [prodserver] => (item=smtp)
changed: [testserver] => (item=smtp)
changed: [prodserver] => (item=mysql)
changed: [testserver] => (item=mysql)

PLAY RECAP **************************************************************************
prodserver                 : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.shell -a "rpm -q httpd postfix mariadb-server"
testserver | CHANGED | rc=0 >>
httpd-2.4.63-16.el10.x86_64
postfix-3.8.5-11.el10.x86_64
mariadb-server-10.11.18-1.el10.x86_64
prodserver | CHANGED | rc=0 >>
httpd-2.4.63-16.el10.x86_64
postfix-3.8.5-11.el10.x86_64
mariadb-server-10.11.18-1.el10.x86_64
[cnode@control-node ansible-lab]$

[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.shell -a "systemctl is-active httpd postfix mariadb"
prodserver | CHANGED | rc=0 >>
active
active
active
testserver | CHANGED | rc=0 >>
active
active
active
[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.shell -a "systemctl is-enabled httpd postfix mariadb"
testserver | CHANGED | rc=0 >>
enabled
enabled
enabled
prodserver | CHANGED | rc=0 >>
enabled
enabled
enabled
[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.shell -a "firewall-cmd --list-services"
prodserver | CHANGED | rc=0 >>
cockpit dhcpv6-client http https mysql smtp ssh
testserver | CHANGED | rc=0 >>
cockpit dhcpv6-client http https mysql smtp ssh
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-doc debug
...
```
---

```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  vars-play.yml
[cnode@control-node ansible-lab]$ cd ../done/
[cnode@control-node done]$ ls
files  multiplays.yml  undeploy-webserver.yml  webserver.yml
[cnode@control-node done]$ cp ../ansible-lab/ansible.cfg .
[cnode@control-node done]$ ls
ansible.cfg  files  multiplays.yml  undeploy-webserver.yml  webserver.yml
[cnode@control-node done]$ cp ../ansible-lab/inventory .
[cnode@control-node done]$ ls
ansible.cfg  files  inventory  multiplays.yml  undeploy-webserver.yml  webserver.yml
[cnode@control-node done]$ cat ansible.cfg 
[defaults]
inventory = ./inventory
remote_user = cnode

[privilege_escalation]
become = true
become_method = sudo
[cnode@control-node done]$ cat inventory 
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
[cnode@control-node done]$ cat webserver.yml 
---

- name: Configure Apache web server
  hosts: testprod
  
  tasks:

  # Install the Apache HTTP server package.
  - name: Install Apache web server
    ansible.builtin.yum:
       name: httpd
       state: latest

  # Start Apache and configure it to start automatically at boot. 
  - name: Start and enable Apache service
    ansible.builtin.service:
       name: httpd
       state: started
       enabled: yes

  # Open HTTP port 80 firewalld and make the change persistent.
  - name: Allow HTTP traffic through the firewall
    ansible.posix.firewalld:
      service: http
      state: enabled
      permanent: true
      immediate: true

  # Deploy the website's index page to Apache's document root.
  - name: Deploy website index page
    ansible.builtin.copy:
      src: files/index.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: '0644'
[cnode@control-node done]$ vim webserver.yml 
[cnode@control-node done]$ ansible-playbook --syntax-check webserver.yml
ERROR! Invalid variable name in 'register' specified: 'service-out'
[cnode@control-node done]$ vim webserver.yml 
[cnode@control-node done]$ ansible-playbook --syntax-check webserver.yml
ERROR! Invalid variable name in 'register' specified: 'service-out'
[cnode@control-node done]$ vim webserver.yml 
[cnode@control-node done]$ ansible-playbook --syntax-check webserver.yml

playbook: webserver.yml
[cnode@control-node done]$ cat webserver.yml
---

- name: Configure Apache web server
  hosts: testprod
  
  tasks:

  # Install the Apache HTTP server package.
  - name: Install Apache web server
    ansible.builtin.yum:
       name: httpd
       state: latest

  # Start Apache and configure it to start automatically at boot. 
  - name: Start and enable Apache service
    ansible.builtin.service:
       name: httpd
       state: started
       enabled: yes
    register: service_out
  
  # Display the result returned by the Apache service task. 
  - name: Print return information from the previous task
    ansible.builtin.debug:
      var: service_out

  # Open HTTP port 80 firewalld and make the change persistent.
  - name: Allow HTTP traffic through the firewall
    ansible.posix.firewalld:
      service: http
      state: enabled
      permanent: true
      immediate: true

  # Deploy the website's index page to Apache's document root.
  - name: Deploy website index page
    ansible.builtin.copy:
      src: files/index.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: '0644'
[cnode@control-node done]$ ansible-playbook webserver.yml

PLAY [Configure Apache web server] **************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Install Apache web server] ****************************************************************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Start and enable Apache service] **********************************************************************************************************
ok: [testserver]
ok: [prodserver]

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
...
            "WatchdogUSec": "0"
        }
    }
}

TASK [Allow HTTP traffic through the firewall] **************************************************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Deploy website index page] ****************************************************************************************************************
changed: [testserver]
changed: [prodserver]

PLAY RECAP **************************************************************************************************************************************
prodserver                 : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node done]$ 
```
---