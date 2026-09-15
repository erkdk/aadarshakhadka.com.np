---
title: "PL - 004 — Ansible Practice: Managing Secrets, Ansible Facts"
date: 2026-09-12
draft: false
---

### Managing Secrets

### Using Ansible Vault

```
             Secret
                |
                v
       +----------------+
       | Ansible Vault  |
       |   Encryption   |
       +----------------+
                |
                v
          Encrypted File
                |
                v
          Git Repository
                |
                v
         ansible-playbook
                |
          Vault password
                |
                v
         Decrypt at runtime
                |
                v
         Use secret securely
                |
                v
          Target System
```

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
ansible.cfg  inventory  vars-play.yml
[cnode@control-node ansible-lab]$ mv vars-play.yml ../done/
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ ansible-vault create secret.yml
New Vault password: 
Confirm New Vault password: 
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  secret.yml
[cnode@control-node ansible-lab]$ ls -l secret.yml 
-rw-------. 1 cnode cnode 484 Sep  9 13:49 secret.yml
[cnode@control-node ansible-lab]$ cat secret.yml 
$ANSIBLE_VAULT;1.1;AES256
31346562356565383666336333613262303966326261336238643963653433613135386563623438
3735373036363637373432313931363465643432636266320a323839633832326436356165663931
63323432393761623661663363613666343666396439666235643138306263323462363737346565
3063663961343333330a313337663763616530306137623630313633353762323561363431663639
36343539666335643037323131653238306664623137306235363764323031306432313131623331
6537376564366539666561376132613662646639326530353539
[cnode@control-node ansible-lab]$ ansible-vault view secret.yml 
Vault password: 
ERROR! Decryption failed (no vault secrets were found that could decrypt) on secret.yml for secret.yml
[cnode@control-node ansible-lab]$ ansible-vault view secret.yml
Vault password: 
name: aadarsha
password: Nepal_123
[cnode@control-node ansible-lab]$ 
```

---

```

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  secret.yml
[cnode@control-node ansible-lab]$ vim secretfile.yml
[cnode@control-node ansible-lab]$ cat secretfile.yml 
name: aadarsha
password: secret_password
[cnode@control-node ansible-lab]$ ansible-vault encrypt secretfile.yml
New Vault password: 
Confirm New Vault password: 
[WARNING]: Error in vault password prompt (default): Passwords do not match
ERROR! Passwords do not match
[cnode@control-node ansible-lab]$ ansible-vault encrypt secretfile.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful
[cnode@control-node ansible-lab]$ cat secretfile.yml 
$ANSIBLE_VAULT;1.1;AES256
38663636393038663237656236396631663465383862386565333362313136653464306435653265
6365356638353263653437393733393638313536306539360a323765643462653961396338643064
36333564663061393637623538336564636666666133616465366331363361313338313333363538
3034656531323562640a303230623437313030623961306438393261646662616337383934336665
30663632393530363637353134633634646233653461356563386230366433393862373463313632
6265316164326134623837323238656434373838363266366532
[cnode@control-node ansible-lab]$ ansible-vault view secretfile.yml
Vault password: 
name: aadarsha
password: secret_password
[cnode@control-node ansible-lab]$ 
```

---

```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  secretfile.yml  secret.yml
[cnode@control-node ansible-lab]$ ansible-vault rekey secret.yml 
Vault password: 
New Vault password: 
Confirm New Vault password: 
Rekey successful
[cnode@control-node ansible-lab]$ 
[cnode@control-node ansible-lab]$ ansible-vault view secret.yml 
Vault password: 
name: aadarsha
password: Nepal_123
[cnode@control-node ansible-lab]$ 
```

---

```
[cnode@control-node ansible-lab]$ ansible-vault --help

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  secretfile.yml  secret.yml
[cnode@control-node ansible-lab]$ ansible-vault decrypt secret.yml 
Vault password: 
Decryption successful
[cnode@control-node ansible-lab]$ cat secret.yml 
name: aadarsha
password: Nepal_123
[cnode@control-node ansible-lab]$ 
```

---

```

[cnode@control-node ansible-lab]$ ansible-inventory --list
{
    "_meta": {
        "hostvars": {}
    },
    "all": {
        "children": [
            "ungrouped",
            "develop",
            "testprod"
        ]
    },
    "develop": {
        "hosts": [
            "dev1",
            "dev2"
        ]
    },
    "production": {
        "hosts": [
            "prodserver"
        ]
    },
    "test": {
        "hosts": [
            "testserver"
        ]
    },
    "testprod": {
        "children": [
            "test",
            "production"
        ]
    }
}
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  secretfile.yml  secret.yml
[cnode@control-node ansible-lab]$ mkdir vars
[cnode@control-node ansible-lab]$ ansible-vault create vars/userinfo.yml
New Vault password: 
Confirm New Vault password: 
[cnode@control-node ansible-lab]$ ls vars/
userinfo.yml
[cnode@control-node ansible-lab]$ vi vault_usercreate.yml
[cnode@control-node ansible-lab]$ vim vault_usercreate.yml 
[cnode@control-node ansible-lab]$ cat vault_usercreate.yml 
---
 # 
 
 - name: Create users using ansible vault
   hosts: dev
   
   vars_files:
     - vars/userinfo.yml
   
   tasks:
   - name: Add a user
     ansible.builtin.user:
       name: "{{ username }}"
       state: present
       password: "{{ password | password_hash('sha512') }}"
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check --ask-vault-pass vault_usercreate.yml
Vault password: 
[WARNING]: Could not match supplied host pattern, ignoring: dev

playbook: vault_usercreate.yml
[cnode@control-node ansible-lab]$ vim vault_usercreate.yml 
[cnode@control-node ansible-lab]$ cat vault_usercreate.yml 
---
 # 
 
 - name: Create users using ansible vault
   hosts: develop
   
   vars_files:
     - vars/userinfo.yml
   
   tasks:
   - name: Add a user
     ansible.builtin.user:
       name: "{{ username }}"
       state: present
       password: "{{ password | password_hash('sha512') }}"
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check --ask-vault-pass vault_usercreate.yml
Vault password: 

playbook: vault_usercreate.yml
[cnode@control-node ansible-lab]$ ls
ansible.cfg  secretfile.yml  vars
inventory    secret.yml      vault_usercreate.yml
[cnode@control-node ansible-lab]$ echo "KhoYo123!" >passfile
[cnode@control-node ansible-lab]$ cat passfile 
KhoYo123!
[cnode@control-node ansible-lab]$ ls -l passfile 
-rw-r--r--. 1 cnode cnode 10 Sep  9 14:18 passfile
[cnode@control-node ansible-lab]$ chmod 400 passfile 
[cnode@control-node ansible-lab]$ ls -l passfile 
-r--------. 1 cnode cnode 10 Sep  9 14:18 passfile
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check --vault-password-file=passfile vault_usercreate.yml

playbook: vault_usercreate.yml
[cnode@control-node ansible-lab]$ ansible-playbook --vault-password-file=passfile vault_usercreate.yml

PLAY [Create users using ansible vault] ****************************************

TASK [Gathering Facts] *********************************************************
ok: [dev2]
ok: [dev1]

TASK [Add a user] **************************************************************
[DEPRECATION WARNING]: Encryption using the Python crypt module is deprecated. 
The Python crypt module is deprecated and will be removed from Python 3.13. 
Install the passlib library for continued encryption functionality. This 
feature will be removed in version 2.17. Deprecation warnings can be disabled 
by setting deprecation_warnings=False in ansible.cfg.
changed: [dev1]
changed: [dev2]

PLAY RECAP *********************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 
```
```
ansible-doc user

:
/EXAMPLES

/password
```
```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  passfile        secret.yml  vault_usercreate.yml
inventory    secretfile.yml  vars
[cnode@control-node ansible-lab]$ ansible develop -m ansible.builtin.shell -a "id ram"
dev1 | CHANGED | rc=0 >>
uid=1006(ram) gid=1006(ram) groups=1006(ram)
dev2 | CHANGED | rc=0 >>
uid=1006(ram) gid=1006(ram) groups=1006(ram)
[cnode@control-node ansible-lab]$

[cnode@dev1 ~]$ hostname
dev1
[cnode@dev1 ~]$ grep ram /etc/passwd
ram:x:1006:1006::/home/ram:/bin/bash
[cnode@dev1 ~]$ 
```

---
---

### Managing Ansible Facts

>Ansible facts are variables that are automatically discovered by ansible on a managed host
 
#### Jinja2 template:
 
A Jinja2 template is a text-based file (typically HTML, XML, or configuration files) that combines static content with placeholder variables and control structures. Jinja2 is a fast, highly extensible template engine for Python inspired by Django's template system. It isolates business logic from presentation by letting you dynamically inject data into your layout before rendering the final output. 

```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ ansible all --list-hosts
  hosts (4):
    dev1
    dev2
    testserver
    prodserver
[cnode@control-node ansible-lab]$ ls ../done/
ansible.cfg     passfile                vars
files           secretfile.yml          vars-play.yml
inventory       secret.yml              vault_usercreate.yml
multiplays.yml  undeploy-webserver.yml  webserver.yml
[cnode@control-node ansible-lab]$ cp ../done/files/* ../done/webserver.yml .
[cnode@control-node ansible-lab]$ ls
ansible.cfg  index.html  inventory  webserver.yml
[cnode@control-node ansible-lab]$ cp -r ../done/files .
[cnode@control-node ansible-lab]$ ls
ansible.cfg  files  index.html  inventory  webserver.yml
[cnode@control-node ansible-lab]$ rm index.html 
[cnode@control-node ansible-lab]$ ls
ansible.cfg  files  inventory  webserver.yml
[cnode@control-node ansible-lab]$ ls files/
index.html
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ vim webserver.yml 
[cnode@control-node ansible-lab]$ ls
ansible.cfg  files  inventory  webserver.yml
[cnode@control-node ansible-lab]$ vim inventory 
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
[cnode@control-node ansible-lab]$ cat ansible.cfg 
[defaults]
inventory = ./inventory
remote_user = cnode

[privilege_escalation]
become = true
become_method = sudo
[cnode@control-node ansible-lab]$ vim files/index.html
[cnode@control-node ansible-lab]$ cp files/index.html files/index.html.j2
[cnode@control-node ansible-lab]$ vim files/index.html.j2
[cnode@control-node ansible-lab]$ cat files/index.html.j2 
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <h2>Machine: {{ ansible_hostname }}
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>
[cnode@control-node ansible-lab]$ vim webserver.yml 
[cnode@control-node ansible-lab]$ vim webserver.yml 
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check webserver.yml 

playbook: webserver.yml
[cnode@control-node ansible-lab]$ ansible-playbook webserver.yml

PLAY [Configure Apache web server] *********************************************

TASK [Gathering Facts] *********************************************************
ok: [testserver]
ok: [prodserver]

TASK [Install Apache web server] ***********************************************
ok: [testserver]
ok: [prodserver]

TASK [Start and enable Apache service] *****************************************
ok: [prodserver]
ok: [testserver]

TASK [Print return information from the previous task] *************************
ok: [testserver] => {
    "service_out": {
        "changed": false,
        "enabled": true,
        "failed": false,
        "name": "httpd",
        "state": "started",
        "status": {
            "AccessSELinuxContext": "system_u:object_r:httpd_unit_file_t:s0",
            "ActiveEnterTimestamp": "Wed 2026-09-09 12:08:33 +0545",
            ...
	    "WantedBy": "multi-user.target",
            "Wants": "-.mount httpd-init.service",
            "WantsMountsFor": "/var/tmp /tmp",
            "WatchdogSignal": "6",
            "WatchdogTimestampMonotonic": "0",
            "WatchdogUSec": "0"
        }
    }
}

TASK [Allow HTTP traffic through the firewall] *********************************
ok: [prodserver]
ok: [testserver]

TASK [Deploy website index page] ***********************************************
changed: [prodserver]
changed: [testserver]

PLAY RECAP *********************************************************************
prodserver                 : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 
```

---

```
[cnode@control-node ansible-lab]$ ansible testprod -m ansible.builtin.shell -a "curl 0"
testserver | CHANGED | rc=0 >>
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <h2>Machine: testserver</h2>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--100   222  100   222    0     0   221k      0 --:--:-- --:--:-- --:--:--  216k
prodserver | CHANGED | rc=0 >>
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <h2>Machine: prodserver</h2>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--100   222  100   222    0     0   156k      0 --:--:-- --:--:-- --:--:--  216k
[cnode@control-node ansible-lab]$
```

---

#### on testserver and prodserver
```
[cnode@testserver ~]$ cat /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <h2>Machine: testserver</h2>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>
[cnode@testserver ~]$ 

[cnode@prodserver ~]$ cat /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <h2>Machine: prodserver</h2>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>
[cnode@prodserver ~]$ 

[cnode@testserver ~]$ curl 0
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <h2>Machine: testserver</h2>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>
[cnode@testserver ~]$ 

[cnode@prodserver ~]$ curl 0
<!DOCTYPE html>
<html>
<head>
    <title>Ansible Lab</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <h2>Machine: prodserver</h2>
    <p>This Apache web server was configured using Ansible.</p>
</body>
</html>
[cnode@prodserver ~]$
```
---