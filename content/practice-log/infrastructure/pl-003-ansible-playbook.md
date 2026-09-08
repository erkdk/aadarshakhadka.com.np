---
title: "PL - 003 — Ansible Playbook: Automating Apache Web Server Deployment"
date: 2026-09-07
draft: false
---
### Manual Steps to Configure the Apache Web Server
The complete manual process is:
```bash
	                    Install Apache
	                        sudo dnf -y install httpd
	                    	  ↓
	                    Start and enable Apache
	                        sudo systemctl enable --now httpd
	                    	  ↓
	                    Allow HTTP through firewall
	                        sudo firewall-cmd --permanent --add-service=http
	                    	  ↓
	                    Reload firewall
	                        sudo firewall-cmd --reload
	                    	  ↓
	                    Create index.html
	                        sudo vi /var/www/html/index.html
	                    	  ↓
	                    Set ownership
	                        sudo chown root:root /var/www/html/index.html
	                    	  ↓
	                    Set permissions
	                        sudo chmod 644 /var/www/html/index.html
	                    	  ↓
	                    Test Apache
	                        sudo systemctl is-active httpd
	                    	  ↓
	                    Test website
	                        curl http://localhost
```
```
[cnode@control-node ~]$ ls
ansible-lab  validate_nodes.sh
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
[cnode@control-node ansible-lab]$ cat ansible.cfg 
[defaults]
inventory = ./inventory
remote_user = cnode

[privilege_escalation]
become = true
become_method = sudo
[cnode@control-node ansible-lab]$ 
```   
---
### Automation steps to configure web servers on two managed nodes
``` 
[cnode@control-node ansible-lab]$ vi webserver.yml

[cnode@control-node ansible-lab]$ ansible-doc copy

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  webserver.yml
[cnode@control-node ansible-lab]$ mkdir files
[cnode@control-node ansible-lab]$ ls
ansible.cfg  files  inventory  webserver.yml
[cnode@control-node ansible-lab]$ vim files/index.html
[cnode@control-node ansible-lab]$ cat files/index.html 
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>

[cnode@control-node ansible-lab]$ vim webserver.yml
[cnode@control-node ansible-lab]$ pwd
/home/cnode/ansible-lab
[cnode@control-node ansible-lab]$ vim webserver.yml
[cnode@control-node ansible-lab]$ cat webserver.yml 
- name: Configure Apache web server
  hosts: testprod
  
  tasks:

### Install the Apache HTTP server package.
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
      src: /home/cnode/ansible-lab/files/index.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: '0644'
[cnode@control-node ansible-lab]$ vim webserver.yml
[cnode@control-node ansible-lab]$ cat webserver.yml
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
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ls
ansible.cfg  files  inventory  webserver.yml
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-galaxy collection list

# /home/cnode/.ansible/collections/ansible_collections
Collection                               Version
---------------------------------------- -------
community.crypto                         3.4.0  
community.general                        13.3.0 
community.library_inventory_filtering_v1 1.1.5  

# /usr/share/ansible/collections/ansible_collections
Collection                               Version
---------------------------------------- -------
microsoft.sql                            2.6.6  
redhat.leapp                             1.7.6  
redhat.rhel_system_roles                 2.4.2  
[cnode@control-node ansible-lab]$ ansible-galaxy collection install ansible.posix
Starting galaxy collection install process
[WARNING]: Collection community.general does not support Ansible version 2.16.19
[WARNING]: Collection community.crypto does not support Ansible version 2.16.19
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/ansible-posix-2.2.2.tar.gz to /home/cnode/.ansible/tmp/ansible-local-4840p_p0zrme/tmpm4pk34fg/ansible-posix-2.2.2-iqb9w1az
Installing 'ansible.posix:2.2.2' to '/home/cnode/.ansible/collections/ansible_collections/ansible/posix'
ansible.posix:2.2.2 was installed successfully
[cnode@control-node ansible-lab]$ ansible-galaxy collection list | grep ansible.posix 
ansible.posix                            2.2.2  
[cnode@control-node ansible-lab]$ ansible-playbook webserver.yml --syntax-check

playbook: webserver.yml
[cnode@control-node ansible-lab]$ ansible testprod --list-hosts
  hosts (2):
    testserver
    prodserver
[cnode@control-node ansible-lab]$ ansible testprod -m ping
testserver | SUCCESS => {
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
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook webserver.yml --syntax-check

playbook: webserver.yml
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook webserver.yml

PLAY [Configure Apache web server] **************************************************

TASK [Gathering Facts] **************************************************************
ok: [prodserver]
ok: [testserver]

TASK [Install Apache web server] ****************************************************
changed: [testserver]
changed: [prodserver]

TASK [Start and enable Apache service] **********************************************
changed: [testserver]
changed: [prodserver]

TASK [Allow HTTP traffic through the firewall] **************************************
changed: [testserver]
changed: [prodserver]

TASK [Deploy website index page] ****************************************************
changed: [testserver]
changed: [prodserver]

PLAY RECAP **************************************************************************
prodserver                 : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ ansible testprod -m command -a "systemctl is-active httpd"
testserver | CHANGED | rc=0 >>
active
prodserver | CHANGED | rc=0 >>
active
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible testprod -m command -a "curl -s http://localhost"
prodserver | CHANGED | rc=0 >>
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>
testserver | CHANGED | rc=0 >>
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ curl http://testserver
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>

[cnode@control-node ansible-lab]$ curl http://prodserver
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>

[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ansible-playbook webserver.yml --check --diff
...

# DRY RUN

[cnode@control-node ansible-lab]$ ansible-playbook webserver.yml --check

PLAY [Configure Apache web server] *****************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Install Apache web server] *******************************************************************************************************************************************
ok: [prodserver]
ok: [testserver]

TASK [Start and enable Apache service] *************************************************************************************************************************************
ok: [prodserver]
ok: [testserver]

TASK [Allow HTTP traffic through the firewall] *****************************************************************************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Deploy website index page] *******************************************************************************************************************************************
ok: [prodserver]
ok: [testserver]

PLAY RECAP *****************************************************************************************************************************************************************
prodserver                 : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$
```