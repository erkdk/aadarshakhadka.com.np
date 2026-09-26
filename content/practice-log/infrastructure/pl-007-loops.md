---
title: "PL - 007 — Ansible Loops, Conditionals, Variables and User Management"
date: 2026-09-15
draft: false
---

### Concepts Covered in This Practical

### 1. Loops

Ansible loops allow the same task to be executed multiple times for different items.

The modern syntax is `loop`:

```yaml
- name: Create developer users
  ansible.builtin.user:
    name: "{{ item }}"
    state: present
  loop: "{{ devusers }}"
```

Here, `item` represents the current value from the `devusers` list.

For example:

```yaml
devusers:
  - devuser1
  - devuser2
  - devuser3
  - devuser4
  - devuser5
```

The task runs once for each user.

#### `with_items`

`with_items` is an older loop syntax that performs the same basic operation:

```yaml
- name: Create developer users
  ansible.builtin.user:
    name: "{{ item }}"
  with_items: "{{ devusers }}"
```

Although `with_items` still works, `loop` is generally preferred for new playbooks.

---

### 2. Looping Over Dictionaries

Loops can also be used with lists containing multiple key-value pairs.

For example:

```yaml
newusers:
  - name: newuser1
    pw: newuser1pass

  - name: newuser2
    pw: newuser2pass

  - name: newuser3
    pw: newuser3pass
```

The values can be accessed using `item.name` and `item.pw`:

```yaml
- name: Create users
  ansible.builtin.user:
    name: "{{ item.name }}"
    password: "{{ item.pw | password_hash('sha512') }}"
  loop: "{{ newusers }}"
```

Here:

* `item.name` → username
* `item.pw` → password

This is useful when each item contains multiple pieces of information.

---

### 3. Conditionals

Ansible conditionals allow a task to run only when a particular condition is true.

The `when` keyword is used for conditions.

Example:

```yaml
- name: Add developer users
  ansible.builtin.user:
    name: "{{ item }}"
    state: present
  loop: "{{ devusers }}"
  when: inventory_hostname in groups['develop']
```

This means:

> Run this task only if the current host belongs to the `develop` inventory group.

For example, if the inventory contains:

```ini
[develop]
dev1
dev2

[test]
testserver

[production]
prodserver
```

The developer-user task runs on:

```text
dev1
dev2
```

and is skipped on:

```text
testserver
prodserver
```

---

### 4. Variables

Variables allow values to be stored separately from tasks and reused throughout a playbook.

Example:

```yaml
vars:
  devusers:
    - devuser1
    - devuser2
    - devuser3
```

The variable can then be used with:

```yaml
loop: "{{ devusers }}"
```

Variables make playbooks easier to maintain because values do not have to be repeated throughout the playbook.

---

### 5. Variables Using `vars_files`

Variables can also be stored in a separate YAML file.

For example:

```text
vars/
└── newusers.yml
```

`vars/newusers.yml`:

```yaml
newusers:
  - name: newuser1
    pw: newuser1pass

  - name: newuser2
    pw: newuser2pass

  - name: newuser3
    pw: newuser3pass

  - name: newuser4
    pw: newuser4pass
```

The playbook can load this file using:

```yaml
vars_files:
  - vars/newusers.yml
```

The variable can then be used:

```yaml
loop: "{{ newusers }}"
```

This is useful when a playbook contains a large amount of data that should be kept separate from the task definitions.

---

### 6. User Management

The `ansible.builtin.user` module is used to create, modify, and remove Linux users.

#### Create a user

```yaml
- name: Create user
  ansible.builtin.user:
    name: devuser1
    state: present
```

`state: present` means that the user should exist.

If the user does not exist, Ansible creates it.

If the user already exists, Ansible normally reports:

```text
ok
```

rather than:

```text
changed
```

#### Remove a user

```yaml
- name: Remove user
  ansible.builtin.user:
    name: devuser1
    state: absent
```

`state: absent` means that the user should not exist.

---

### 7. Removing a User's Home Directory

The `remove` parameter can be used when deleting a user:

```yaml
- name: Delete user and home directory
  ansible.builtin.user:
    name: devuser1
    state: absent
    remove: true
```

This requests removal of the user's home directory and associated files.

In the practical, this was used in the cleanup playbook:

```yaml
users_to_delete:
  - devuser1
  - devuser2
  - devuser3
  - devuser4
  - devuser5
```

and then:

```yaml
loop: "{{ users_to_delete }}"
```

---

### 8. Group Management

The `ansible.builtin.group` module is used to create or remove Linux groups.

Example:

```yaml
- name: Create required groups
  ansible.builtin.group:
    name: "{{ item }}"
    state: present
  loop:
    - developers
    - managers
    - admins
    - employees
```

This creates the following groups:

```text
developers
managers
admins
employees
```

Using a loop avoids writing four separate tasks.

---

### 9. Primary Group vs Supplementary Groups

A Linux user can have:

* one **primary group**
* one or more **supplementary groups**

In your playbook:

```yaml
ansible.builtin.user:
  name: "{{ item.name }}"
  group: employees
  groups: "{{ user_groups | join(',') }}"
```

The `group` parameter specifies the **primary group**:

```yaml
group: employees
```

The `groups` parameter specifies **supplementary groups**:

```yaml
groups: "{{ user_groups | join(',') }}"
```

For example:

```yaml
user_groups:
  - developers
  - managers
  - admins
```

A resulting user could have:

```text
Primary group:
employees

Supplementary groups:
developers
managers
admins
```

This can be verified with:

```bash
id newuser1
```

Example output:

```text
uid=1008(newuser1) gid=1011(employees) \
groups=1011(employees),1008(developers),1009(managers),1010(admins)
```

---

### 10. Password Hashing

Linux does not normally store plain-text passwords in `/etc/passwd`.

When creating a user with a password, the password should be hashed.

In your playbook:

```yaml
password: "{{ item.pw | password_hash('sha512') }}"
```

The Jinja2 filter:

```text
password_hash('sha512')
```

converts the password into a SHA-512 password hash.

For example:

```yaml
password: "{{ item.pw | password_hash('sha512') }}"
```

The password value:

```text
newuser1pass
```

is converted into a password hash before being passed to the user module.

> **Note:** Avoid storing real production passwords directly in a playbook or variables file. For real environments, use Ansible Vault or another secrets-management solution.

---

### 11. Privilege Escalation with `become`

Some operations require administrator/root privileges.

For example, creating users and groups normally requires root privileges.

Ansible can use privilege escalation with:

```yaml
become: true
```

Example:

```yaml
- name: Provision development users
  hosts: develop
  become: true
```

This tells Ansible to execute the relevant tasks with elevated privileges.

This is why the playbook can create users and groups on the managed hosts.

---

### 12. Idempotency

Idempotency is one of the important concepts in Ansible.

It means that running the same playbook repeatedly should produce the same desired state without making unnecessary changes.

For example:

```yaml
- name: Create user
  ansible.builtin.user:
    name: devuser1
    state: present
```

On the first run, Ansible may report:

```text
changed
```

because the user had to be created.

On a subsequent run, if the user already exists and matches the desired configuration, Ansible should report:

```text
ok
```

This is one of the major differences between configuration management with Ansible and simply running shell commands repeatedly.

In your practical, you observed this behavior when running:

```bash
ansible-playbook loop-newusercreate.yml
```

multiple times.

---

### 13. `--syntax-check`

Before running a playbook, Ansible can check its YAML/playbook syntax:

```bash
ansible-playbook --syntax-check playbook.yml
```

For example:

```bash
ansible-playbook --syntax-check createusers.yml
```

A successful result looks like:

```text
playbook: createusers.yml
```

This checks whether the playbook structure and syntax are valid.

It does **not** actually execute the tasks.

---

### 14. Check Mode with `--check`

Ansible also provides check mode:

```bash
ansible-playbook --check playbook.yml
```

Example:

```bash
ansible-playbook --check createusers.yml
```

Check mode attempts to show what Ansible would change without actually applying the changes.

It is useful for reviewing a playbook before making changes to managed hosts.

However, check mode is not a perfect simulation for every Ansible module, so its output should be interpreted accordingly.

---

### 15. Inventory Groups

The inventory defines which hosts belong to which groups.

Your inventory contains:

```ini
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
```

This creates the following logical structure:

```text
develop
├── dev1
└── dev2

test
└── testserver

production
└── prodserver

testprod
├── test
└── production
```

The special `:children` syntax allows one inventory group to contain other groups.

Therefore:

```bash
ansible testprod -m ...
```

targets both:

```text
testserver
prodserver
```

---

### 16. Fact Gathering

Ansible automatically gathers system information called **facts** at the beginning of a playbook by default.

You can also explicitly inspect facts using:

```bash
ansible prodserver -m setup
```

You used filters to retrieve specific information.

#### IPv4 information

```bash
ansible prodserver -m setup -a 'filter=*ipv4*'
```

This provided information such as:

```text
192.168.254.19
```

#### CPU information

```bash
ansible prodserver -m setup -a 'filter=*cpu*'
```

This returned:

```text
ansible_processor_vcpus: 2
```

#### BIOS information

```bash
ansible prodserver -m setup -a 'filter=*bios*'
```

This returned information such as:

```text
ansible_bios_date
ansible_bios_vendor
ansible_bios_version
```

Facts can be referenced inside playbooks.

For example:

```jinja2
{{ ansible_hostname }}
```

and:

```jinja2
{{ ansible_processor_vcpus }}
```

---

### 17. Jinja2 Templates

Ansible can use Jinja2 templates to generate dynamic configuration or HTML files.

Your practical used:

```text
index.html.j2
```

A template can contain variables such as:

```jinja2
<h2>Machine's Hostname: {{ ansible_hostname }}</h2>

<h2>No. of CPUs: {{ ansible_processor_vcpus }}</h2>

<h2>IPv4 Address: {{ ansible_default_ipv4.address }}</h2>

<h2>BIOS date: {{ ansible_bios_date }}</h2>
```

When Ansible processes the template, the variables are replaced with the actual facts from each managed host.

Therefore, the same template produced different results for:

```text
testserver
```

and:

```text
prodserver
```

For example:

```text
testserver → 192.168.254.18
prodserver → 192.168.254.19
```

This demonstrates how templates can create host-specific configuration from a single template file.

---

### 18. Cleanup with `state: absent`

The `ansible.builtin.user` module can also be used for cleanup.

Example:

```yaml
- name: Delete created users
  ansible.builtin.user:
    name: "{{ item }}"
    state: absent
    remove: true
  loop: "{{ users_to_delete }}"
```

The list contains all users that should be removed:

```yaml
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
```

Using one cleanup task with a loop is much easier to maintain than writing a separate task for every user.

---

## Key Ansible Ideas Demonstrated

This practical combines several important Ansible concepts:

```text
Inventory
   ↓
Groups
   ↓
Variables / vars_files
   ↓
Loops
   ↓
Conditionals
   ↓
Modules
   ↓
Facts
   ↓
Jinja2 templates
   ↓
Idempotent configuration
   ↓
Validation and cleanup
```

The main pattern to remember is:

```yaml
- name: Task description
  ansible.builtin.module:
    parameter: "{{ variable }}"
  loop: "{{ list }}"
  when: condition
```
This pattern is the foundation for writing more flexible Ansible playbooks.

---
---
---
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
```
---

```
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