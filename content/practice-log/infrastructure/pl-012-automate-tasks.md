---
title: "PL - 012 — Automating Linux system Administration Tasks with Ansible"
date: 2026-09-29
draft: false
---

### Automating Linux system Adminstration Tasks

i. Software and Subscription Management
ii. Scheduling Tasks

---

### Lab Session:

```
aadarkdk@pop-os:~$ ssh cnode@192.168.254.15
cnode@192.168.254.15's password: 
Last login: Wed Sep 16 06:10:27 2026
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

[cnode@control-node ~]$
```
---

### Configure yum repository, manually and using Ansible:

Working mechanism of installing the package manually:

```
 # when a package is installed:
[cnode@control-node ~]$ # yum/dnf -y install package_name
[cnode@control-node ~]$ 
[cnode@control-node ~]$ cd /etc/yum.repos.d/
[cnode@control-node yum.repos.d]$ pwd
/etc/yum.repos.d
[cnode@control-node yum.repos.d]$ ls
centos-addons.repo  centos.repo

# This files contain the meta link from where the packages are copied

[cnode@control-node yum.repos.d]$ cat centos-addons.repo 
[highavailability]
name=CentOS Stream $releasever - HighAvailability
metalink=https://mirrors.centos.org/metalink?repo=centos-highavailability-$stream&arch=$basearch&protocol=https,http
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-SHA256
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=0

[highavailability-debuginfo]
name=CentOS Stream $releasever - HighAvailability - Debug
metalink=https://mirrors.centos.org/metalink?repo=centos-highavailability-debug-$stream&arch=$basearch&protocol=https,http
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-SHA256
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

...

[cnode@control-node yum.repos.d]$ cat centos.repo 
[baseos]
name=CentOS Stream $releasever - BaseOS
metalink=https://mirrors.centos.org/metalink?repo=centos-baseos-$stream&arch=$basearch&protocol=https,http
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-SHA256
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
countme=1
enabled=1

[baseos-debuginfo]
name=CentOS Stream $releasever - BaseOS - Debug
metalink=https://mirrors.centos.org/metalink?repo=centos-baseos-debug-$stream&arch=$basearch&protocol=https,http
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-SHA256
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

...

[crb-debuginfo]
name=CentOS Stream $releasever - CRB - Debug
metalink=https://mirrors.centos.org/metalink?repo=centos-crb-debug-$stream&arch=$basearch&protocol=https,http
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-SHA256
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0

[crb-source]
name=CentOS Stream $releasever - CRB - Source
metalink=https://mirrors.centos.org/metalink?repo=centos-crb-source-$stream&arch=source&protocol=https,http
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-SHA256
gpgcheck=1
repo_gpgcheck=0
metadata_expire=6h
enabled=0
[cnode@control-node yum.repos.d]$ 
```
---


```
[cnode@control-node yum.repos.d]$ ls
centos-addons.repo  centos.repo
[cnode@control-node yum.repos.d]$ sudo mkdir backup
[sudo] password for cnode: 
[cnode@control-node yum.repos.d]$ ls
backup  centos-addons.repo  centos.repo
[cnode@control-node yum.repos.d]$ sudo mv *.repo backup/
[cnode@control-node yum.repos.d]$ ls
backup
[cnode@control-node yum.repos.d]$ ls backup/
centos-addons.repo  centos.repo
[cnode@control-node yum.repos.d]$ 
```

```
# In two develop machines: manually removing defaults repo files

aadarkdk@pop-os:~$ ssh cnode@192.168.254.16
cnode@192.168.254.16's password: 
Last login: Wed Sep 16 05:11:19 2026 from 192.168.254.152
[cnode@dev1 ~]$ cd /etc/yum.repos.d/
[cnode@dev1 yum.repos.d]$ ls
centos-addons.repo  centos.repo
[cnode@dev1 yum.repos.d]$ mkdir backup
mkdir: cannot create directory ‘backup’: Permission denied
[cnode@dev1 yum.repos.d]$ sudo mkdir backup
[cnode@dev1 yum.repos.d]$ ls
backup  centos-addons.repo  centos.repo
[cnode@dev1 yum.repos.d]$ sudo mv *.repo backup/
[cnode@dev1 yum.repos.d]$ ls
backup
[cnode@dev1 yum.repos.d]$ ls backup/
centos-addons.repo  centos.repo
[cnode@dev1 yum.repos.d]$ 


aadarkdk@pop-os:~$ ssh cnode@192.168.254.17
cnode@192.168.254.17's password: 
Last login: Wed Sep 16 06:09:45 2026
[cnode@dev2 ~]$ cd /etc/yum.repos.d/
[cnode@dev2 yum.repos.d]$ ls
centos-addons.repo  centos.repo
[cnode@dev2 yum.repos.d]$ mkdir backup
mkdir: cannot create directory ‘backup’: Permission denied
[cnode@dev2 yum.repos.d]$ sudo mkdir backup
[cnode@dev2 yum.repos.d]$ sudo mv *.repo backup/
[cnode@dev2 yum.repos.d]$ ls
backup
[cnode@dev2 yum.repos.d]$ ls backup/
centos-addons.repo  centos.repo
[cnode@dev2 yum.repos.d]$ 
```

---

# Manually creating repo configuration file

```
[cnode@control-node yum.repos.d]$ ls
backup
[cnode@control-node yum.repos.d]$ sudo vim local-custom.repo
[sudo] password for cnode: 
[cnode@control-node yum.repos.d]$ ls
backup  local-custom.repo
[cnode@control-node yum.repos.d]$ cat local-custom.repo 
[baseos]
name=baseos repo
baseurl=http://mylocalserver.local.domain/BaseOS
enabled=1
gpgcheck=0

[appstream]
name=appstream repo
baseurl=http://mylocalserver.local.domain/AppStream
enabled=1
gpgcheck=0
[cnode@control-node yum.repos.d]$ ansible-doc yum-repository
[WARNING]: yum-repository was not found


[cnode@control-node yum.repos.d]$ ansible-doc yum_repository

[cnode@control-node yum.repos.d]$ ansible-doc rpm_key

[cnode@control-node yum.repos.d]$ pwd
/etc/yum.repos.d
[cnode@control-node yum.repos.d]$ ls /etc/pki/rpm-gpg/
RPM-GPG-KEY-centosofficial-PQC  RPM-GPG-KEY-centosofficial-SHA256  RPM-GPG-KEY-CentOS-SIG-Extras-SHA512
[cnode@control-node yum.repos.d]$ ls /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-PQC 
/etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-PQC
[cnode@control-node yum.repos.d]$ 
```

---


```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ hostname
control-node
[cnode@control-node ansible-lab]$ pwd
/home/cnode/ansible-lab
[cnode@control-node ansible-lab]$ vim yum_repo-pkgserver.yml
[cnode@control-node ansible-lab]$ cat yum_repo-pkgserver.yml 
---
- name: Play to configure yum repo on the remote hosts
  hosts: develop 

  tasks:
  - name: Create a baseos yum repo
    ansible.builtin.yum_repository:
      name: mybaseos
      description: My BaseOS YUM repo
      file: mylocal
      baseurl: http://mylocalserver.local.domain/BaseOS
      enabled: yes
      gpgcheck: yes

  - name: Create a appstream yum repo
    ansible.builtin.yum_repository:
      name: myappstream
      description: My AppStream YUM repo
      file: mylocal
      baseurl: http://mylocalserver.local.domain/AppStream
      enabled: yes
      gpgcheck: yes

  - name: Import a key from a file
    ansible.builtin.rpm_key:
      state: present
      key: /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial-PQC
[cnode@control-node ansible-lab]$

[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check yum_repo-pkgserver.yml

playbook: yum_repo-pkgserver.yml
[cnode@control-node ansible-lab]$ ansible-playbook yum_repo-pkgserver.yml

PLAY [Play to configure yum repo on the remote hosts] *******************************

TASK [Gathering Facts] **************************************************************
ok: [dev2]
ok: [dev1]

TASK [Create a baseos yum repo] *****************************************************
changed: [dev2]
changed: [dev1]

TASK [Create a appstream yum repo] **************************************************
changed: [dev1]
changed: [dev2]

TASK [Import a key from a file] *****************************************************
changed: [dev2]
changed: [dev1]

PLAY RECAP **************************************************************************
dev1                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 
```


### In develop machines:

```
[cnode@dev2 yum.repos.d]$ ls
backup  mylocal.repo
[cnode@dev2 yum.repos.d]$ cat mylocal.repo 
[mybaseos]
baseurl = http://mylocalserver.local.domain/BaseOS
enabled = 1
gpgcheck = 1
name = My BaseOS YUM repo

[myappstream]
baseurl = http://mylocalserver.local.domain/AppStream
enabled = 1
gpgcheck = 1
name = My AppStream YUM repo

[cnode@dev2 yum.repos.d]$ 
[cnode@dev2 yum.repos.d]$ sudo dnf repolist
repo id                               repo name
myappstream                           My AppStream YUM repo
mybaseos                              My BaseOS YUM repo
[cnode@dev2 yum.repos.d]$ 


[cnode@dev1 yum.repos.d]$ ls
backup  mylocal.repo
[cnode@dev1 yum.repos.d]$ cat mylocal.repo 
[mybaseos]
baseurl = http://mylocalserver.local.domain/BaseOS
enabled = 1
gpgcheck = 1
name = My BaseOS YUM repo

[myappstream]
baseurl = http://mylocalserver.local.domain/AppStream
enabled = 1
gpgcheck = 1
name = My AppStream YUM repo

[cnode@dev1 yum.repos.d]$ 

[cnode@dev1 yum.repos.d]$ sudo yum repolist
repo id                               repo name
myappstream                           My AppStream YUM repo
mybaseos                              My BaseOS YUM repo
[cnode@dev1 yum.repos.d]$ 
[cnode@dev1 yum.repos.d]$ pwd
/etc/yum.repos.d
[cnode@dev1 yum.repos.d]$ 
```

---

```
[cnode@control-node ansible-lab]$ ansible develop -m ansible.builtin.file -a 'path=/etc/yum.repos.d/mylocal.repo state=absent'
dev2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "path": "/etc/yum.repos.d/mylocal.repo",
    "state": "absent"
}
dev1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "path": "/etc/yum.repos.d/mylocal.repo",
    "state": "absent"
}
[cnode@control-node ansible-lab]$ 

```

----
----
----
----

### Cronjob

```
[cnode@dev1 yum.repos.d]$ crontab -l
no crontab for cnode
[cnode@dev1 yum.repos.d]$ sudo cat /var/spool/cron/cnode
cat: /var/spool/cron/cnode: No such file or directory
[cnode@dev1 yum.repos.d]$ 

[cnode@dev2 ~]$ ls /etc/cron.d
0hourly
[cnode@dev2 ~]$ 

[cnode@dev2 yum.repos.d]$ crontab -l
no crontab for cnode
[cnode@dev2 yum.repos.d]$ sudo cat /var/spool/cron/cnode
cat: /var/spool/cron/cnode: No such file or directory
[cnode@dev2 yum.repos.d]$ 

[cnode@dev1 yum.repos.d]$ ls /etc/cron.d
0hourly
[cnode@dev1 yum.repos.d]$ 
```
---

```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  yum_repo-pkgserver.yml
[cnode@control-node ansible-lab]$ mv yum_repo-pkgserver.yml ../done/
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ vim cronjobs.yml
[cnode@control-node ansible-lab]$ cat cronjobs.yml
---
- name: Play to install cron job on develop machines
  hosts: develop

  tasks:
    - name: Schedule cron job to run at 5 AM everyday
      ansible.builtin.cron:
        name: "Check logged in users"
        minute: "00"
        hour: "9"
        job: "who >> /tmp/who-output"
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check cronjobs.yml

playbook: cronjobs.yml
[cnode@control-node ansible-lab]$ ansible-playbook cronjobs.yml

PLAY [Play to install cron job on develop machines] *********************************

TASK [Gathering Facts] **************************************************************
ok: [dev2]
ok: [dev1]

TASK [Schedule cron job to run at 5 AM everyday] ************************************
changed: [dev1]
changed: [dev2]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 
```

---

```
[cnode@control-node ansible-lab]$ ansible develop -m command -a 'whoami'
dev2 | CHANGED | rc=0 >>
root
dev1 | CHANGED | rc=0 >>
root
[cnode@control-node ansible-lab]$ 

[cnode@dev2 ~]$ sudo crontab -l
#Ansible: Check logged in users
00 9 * * * who >> /tmp/who-output
[cnode@dev2 ~]$ whoami
cnode
[cnode@dev2 ~]$ sudo ls /var/spool/cron/
root
[cnode@dev2 ~]$ sudo cat /var/spool/cron/root
#Ansible: Check logged in users
00 9 * * * who >> /tmp/who-output
[cnode@dev2 ~]$ 

[cnode@dev1 ~]$ sudo crontab -l
#Ansible: Check logged in users
00 9 * * * who >> /tmp/who-output
[cnode@dev1 ~]$ 
```

---

```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  cronjobs.yml  inventory
[cnode@control-node ansible-lab]$ cp cronjobs.yml cronjobs2.yml 
[cnode@control-node ansible-lab]$ vim cronjobs2.yml 
[cnode@control-node ansible-lab]$ cat cronjobs2.yml 
---
- name: Play to install cron job on develop machines
  hosts: develop

  tasks:
    - name: Schedule cron job to run at 5:30 PM on the first day every month
      ansible.builtin.cron:
        name: "Check disk info"
        minute: "30"
        hour: "17"
        day: "1"
        month: "*"
        weekday: "*"
        job: "df -h >> /tmp/who-output"
        user: cnode
[cnode@control-node ansible-lab]$ ls
ansible.cfg  cronjobs2.yml  cronjobs.yml  inventory
[cnode@control-node ansible-lab]$ ansible-playbook cronjobs2.yml 

PLAY [Play to install cron job on develop machines] *********************************

TASK [Gathering Facts] **************************************************************
ok: [dev1]
ok: [dev2]

TASK [Schedule cron job to run at 5:30 PM on the first day every month] *************
changed: [dev1]
changed: [dev2]

PLAY RECAP **************************************************************************
dev1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
dev2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 

---

[cnode@dev2 ~]$ sudo ls /var/spool/cron/
cnode  root
[cnode@dev2 ~]$ sudo cat /var/spool/cron/cnode
#Ansible: Check disk info
30 17 1 * * df -h >> /tmp/who-output
[cnode@dev2 ~]$ crontab -l
#Ansible: Check disk info
30 17 1 * * df -h >> /tmp/who-output
[cnode@dev2 ~]$ sudo crontab -l
#Ansible: Check logged in users
00 9 * * * who >> /tmp/who-output
[cnode@dev2 ~]$ 
```

#### Note:
- All modules have idempotent property. But, ansible.builtin.command or ansible.builtin.shell  module does not have idempotent property

---