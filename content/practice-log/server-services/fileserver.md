---
title: "Samba File Server: Configuring SMB Shares, Users, Permissions, ACLs, and SELinux"
date: 2026-10-03
draft: false
---

### File Server

A file server is a server that stores files and makes them available to other computer over a network.

A file server commonly provides:
- Centralized file storage:
  Stores files in one location.

- File Sharing:
  Allows multiple users to access shared files.
  
- Access Control:
  Controls who can read, modify, or delete files.
  
- Data organization:
  Keeps files and folders organized.
  
- Backup:
  Makes centralized backup easier.

- Security:
  Protects files using permissions and authentication.
  

#### Common File-Sharing Protocols
- SMB/CIFS:
  SMB (Server Message Block) is commonly used for file and printer sharing, particularly in Windows environments. CIFS is an older dialect of SMB, while modern Samba deployments normally use SMB2/SMB3 rather than SMB1.
  
- NFS:
  NFS (Network File System) is commonly used for file sharing between Linux/Unix systems.
  
- FTP/SFTP:
  FTP and SFTP are commonly used for transferring files between systems. SFTP operates over SSH and provides encrypted file transfers.
  
  
#### Common File Servers

- NAS devices
- NFS servers
- Windows Server file servers
- SAMBA

#### SAMBA
Samba is an open-source implementation of the SMB protocol. It allows Linux and Unix systems to provide SMB-based file and printer sharing services and to communicate with Windows systems.

One of the useful characteristics of Samba is that it is cross-platform.
- **Configuring SAMBA Server**

### LAB Session:

- Check the File Server
```
[aadarkhadka@fileserver ~]$ whoami
aadarkhadka
[aadarkhadka@fileserver ~]$ hostname -I
192.168.254.24 
[aadarkhadka@fileserver ~]$ hostname
fileserver.lab.local
[aadarkhadka@fileserver ~]$ whoami
aadarkhadka
```

- Install samba packages
```
[aadarkhadka@fileserver ~]$ rpm -q samba samba-client samba-common
package samba is not installed
package samba-client is not installed
samba-common-4.23.5-107.el10.noarch
[aadarkhadka@fileserver ~]$ sudo dnf install samba-client -y
...

[aadarkhadka@fileserver ~]$ rpm -q samba samba-client
samba-4.24.5-101.el10.x86_64
samba-client-4.24.5-101.el10.x86_64
[aadarkhadka@fileserver ~]$ 
```

- Start and enable the samba service

```
[aadarkhadka@fileserver ~]$ sudo systemctl start smb
[aadarkhadka@fileserver ~]$ sudo systemctl enable smb
Created symlink '/etc/systemd/system/multi-user.target.wants/smb.service' → '/usr/lib/systemd/system/smb.service'.
[aadarkhadka@fileserver ~]$ sudo systemctl is-active smb
active
[aadarkhadka@fileserver ~]$ sudo systemctl is-enabled smb
enabled
[aadarkhadka@fileserver ~]$
```

- Configure the firewall
```
[aadarkhadka@fileserver ~]$ sudo firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[aadarkhadka@fileserver ~]$ sudo firewall-cmd --list-services
cockpit dhcpv6-client ssh
[aadarkhadka@fileserver ~]$ 
[aadarkhadka@fileserver ~]$ man firewalld.richlanguage
[aadarkhadka@fileserver ~]$ sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.254.0/24" service name="samba" accept'
success
[aadarkhadka@fileserver ~]$ sudo firewall-cmd --reload
success
[aadarkhadka@fileserver ~]$ sudo firewall-cmd --list-rich-rules
rule family="ipv4" source address="192.168.254.0/24" service name="samba" accept
[aadarkhadka@fileserver ~]$
```

- Create Directories to Share

```
[aadarkhadka@fileserver ~]$ sudo mkdir -p /data/{sales,marketing,hr}
[aadarkhadka@fileserver ~]$ sudo ls /data/
hr  marketing  sales
[aadarkhadka@fileserver ~]$ sudo vi /data/hr/hrdata
[aadarkhadka@fileserver ~]$ sudo vi /data/sales/salesdata
[aadarkhadka@fileserver ~]$ sudo vi /data/marketing/marketingdata

[aadarkhadka@fileserver ~]$ sudo mkdir /data/common
[aadarkhadka@fileserver ~]$ sudo vi /data/common/public
[aadarkhadka@fileserver ~]$ tree /data/
/data/
├── common
│   └── public
├── hr
│   └── hrdata
├── marketing
│   └── marketingdata
└── sales
    └── salesdata

5 directories, 4 files
[aadarkhadka@fileserver ~]$ 

[aadarkhadka@fileserver ~]$ ls /data/
common  hr  marketing  sales

[aadarkhadka@fileserver ~]$ sudo mv /data/common /data/research
[aadarkhadka@fileserver ~]$ ls /data/
hr  marketing  research  sales
[aadarkhadka@fileserver ~]$ ls /data/research/
public

[aadarkhadka@fileserver ~]$ sudo mv /data/research/public /data/research/researchdata
[aadarkhadka@fileserver ~]$ sudo vi /data/research/researchdata
[aadarkhadka@fileserver ~]$ tree /data/
/data/
├── hr
│   └── hrdata
├── marketing
│   └── marketingdata
├── research
│   └── researchdata
└── sales
    └── salesdata

5 directories, 4 files
[aadarkhadka@fileserver ~]$ ls -Z /data/
unconfined_u:object_r:samba_share_t:s0 hr
unconfined_u:object_r:samba_share_t:s0 marketing
    unconfined_u:object_r:default_t:s0 research
unconfined_u:object_r:samba_share_t:s0 sales
[aadarkhadka@fileserver ~]$ 

[aadarkhadka@fileserver ~]$ sudo chcon -t samba_share_t -R /data/research/

[aadarkhadka@fileserver ~]$ ls -Z /data/
unconfined_u:object_r:samba_share_t:s0 hr
unconfined_u:object_r:samba_share_t:s0 marketing
unconfined_u:object_r:samba_share_t:s0 research
unconfined_u:object_r:samba_share_t:s0 sales
[aadarkhadka@fileserver ~]$
```

- Configure SELinux for Samba

```
[aadarkhadka@fileserver ~]$ getenforce
Enforcing
[aadarkhadka@fileserver ~]$ sudo ls -Z /data
unconfined_u:object_r:default_t:s0 hr
unconfined_u:object_r:default_t:s0 marketing
unconfined_u:object_r:default_t:s0 sales
[aadarkhadka@fileserver ~]$

[aadarkhadka@fileserver ~]$ sudo chcon -t samba_share_t /data/*

[aadarkhadka@fileserver ~]$ sudo ls -Z /data
unconfined_u:object_r:samba_share_t:s0 hr
unconfined_u:object_r:samba_share_t:s0 marketing
unconfined_u:object_r:samba_share_t:s0 sales
[aadarkhadka@fileserver ~]$ 
```

- Create Users and Groups

```
 [aadarkhadka@fileserver ~]$ sudo useradd ram
[aadarkhadka@fileserver ~]$ sudo useradd sita
[aadarkhadka@fileserver ~]$ sudo useradd hari
[aadarkhadka@fileserver ~]$ sudo useradd gita
[aadarkhadka@fileserver ~]$ sudo useradd ravi

[aadarkhadka@fileserver ~]$ sudo groupadd sales
[aadarkhadka@fileserver ~]$ sudo groupadd marketing
[aadarkhadka@fileserver ~]$ sudo groupadd hr
 
[aadarkhadka@fileserver ~]$ sudo usermod -aG sales ram
[aadarkhadka@fileserver ~]$ sudo usermod -aG sales hari
[aadarkhadka@fileserver ~]$ sudo usermod -aG marketing sita
[aadarkhadka@fileserver ~]$ sudo usermod -aG marketing gita

[aadarkhadka@fileserver ~]$ sudo usermod -aG hr ravi
[aadarkhadka@fileserver ~]$ sudo usermod -aG hr hari

[aadarkhadka@fileserver ~]$ tail /etc/group
aadarkhadka:x:1000:
named:x:25:
ram:x:1001:
sita:x:1002:
hari:x:1003:
gita:x:1004:
ravi:x:1005:
sales:x:1006:ram,hari
marketing:x:1007:sita,gita
hr:x:1008:ravi,hari
[aadarkhadka@fileserver ~]$ 
```
-  Add Users to the Samba Password Database
```
[aadarkhadka@fileserver ~]$ sudo smbpasswd -a ram
New SMB password:
Retype new SMB password:
Added user ram.
[aadarkhadka@fileserver ~]$ sudo smbpasswd -a hari
New SMB password:
Retype new SMB password:
Added user hari.
[aadarkhadka@fileserver ~]$ sudo smbpasswd -a sita
New SMB password:
Retype new SMB password:
Added user sita.
[aadarkhadka@fileserver ~]$ sudo smbpasswd -a gita
New SMB password:
Retype new SMB password:
Added user gita.
[aadarkhadka@fileserver ~]$ sudo smbpasswd -a ravi
New SMB password:
Retype new SMB password:
Added user ravi.
[aadarkhadka@fileserver ~]$
```
- Configure Samba Shares

```
[aadarkhadka@fileserver ~]$ sudo vi /etc/samba/smb.conf
[aadarkhadka@fileserver ~]$ sudo cat /etc/samba/smb.conf
[sudo] password for aadarkhadka: 
# See smb.conf.example for a more detailed config file or
# read the smb.conf manpage.
# Run 'testparm' to verify the config is correct after
# you modified it.
#
# Note:
# SMB1 is disabled by default. This means clients without support for SMB2 or
# SMB3 are no longer able to connect to smbd (by default).

[global]
	workgroup = SAMBA
	security = user

	passdb backend = tdbsam

	printing = cups
	printcap name = cups
	load printers = yes
	cups options = raw

	# Install samba-usershares package for support
	include = /etc/samba/usershares.conf

[homes]
	comment = Home Directories
	valid users = %S, %D%w%S
	browseable = No
	read only = No
	inherit acls = Yes

[printers]
	comment = All Printers
	path = /var/tmp
	printable = Yes
	create mask = 0600
	browseable = No

[print$]
	comment = Printer Drivers
	path = /var/lib/samba/drivers
	# printadmin is a local group
	write list = printadmin root
	force group = printadmin
	create mask = 0664
	directory mask = 0775

[sales]
comment = Sales data
path = /data/sales
write list = @sales
browseable = yes

[marketing]
comment = Contains marketing data
path = /data/marketing
writeable = yes
browseable = yes
hosts deny = 192.168.254.11 192.168.254.12

[hrdata]
comment = Contains HR data
path = /data/hr
write list = @hr
browseable = yes

[research]
comment = Research data
path = /data/research
writeable = yes
browseable = no
hosts allow = 192.168.254.21
[aadarkhadka@fileserver ~]$ 
```

- Configure Linux Filesystem Permissions
```
[aadarkhadka@fileserver ~]$ ls -ld /data/sales/
drwxr-xr-x. 2 root root 23 Oct  3 12:38 /data/sales/
[aadarkhadka@fileserver ~]$ sudo chgrp sales /data/sales
[sudo] password for aadarkhadka: 
[aadarkhadka@fileserver ~]$ ls -ld /data/sales
drwxr-xr-x. 2 root sales 23 Oct  3 12:38 /data/sales
[aadarkhadka@fileserver ~]$ 
[aadarkhadka@fileserver ~]$ sudo chmod 2770 /data/sales
[aadarkhadka@fileserver ~]$ 
[aadarkhadka@fileserver ~]$ ls -ld /data/sales/
drwxrws---. 2 root sales 23 Oct  3 12:38 /data/sales/ 
[aadarkhadka@fileserver ~]$ ls -dZ /data/sales
unconfined_u:object_r:samba_share_t:s0 /data/sales
[aadarkhadka@fileserver ~]$ 

[aadarkhadka@fileserver ~]$ ls -ld /data/hr
drwxr-xr-x. 2 root root 20 Oct  3 12:38 /data/hr
[aadarkhadka@fileserver ~]$ sudo chown root:hr /data/hr
[aadarkhadka@fileserver ~]$ ls -ld /data/hr/
drwxr-xr-x. 2 root hr 20 Oct  3 12:38 /data/hr/
[aadarkhadka@fileserver ~]$ sudo chmod 2770 /data/hr/
[aadarkhadka@fileserver ~]$ ls -ld /data/hr/
drwxrws---. 2 root hr 20 Oct  3 12:38 /data/hr/
[aadarkhadka@fileserver ~]$ 
[aadarkhadka@fileserver ~]$ ls -ld /data/research
drwxr-xr-x. 2 root root 26 Oct  3 13:51 /data/research
[aadarkhadka@fileserver ~]$ 
```
- Configure ACLs for the Research Share

```
[aadarkhadka@fileserver ~]$ sudo setfacl -m u:ravi:rwx /data/research
[aadarkhadka@fileserver ~]$ sudo ls -ld /data/research/
drwxrwxr-x+ 2 root root 26 Oct  3 13:51 /data/research/
[aadarkhadka@fileserver ~]$ sudo setfacl -m u:ram:rwx /data/research
[aadarkhadka@fileserver ~]$ sudo ls -ld /data/research/
drwxrwxr-x+ 2 root root 26 Oct  3 13:51 /data/research/
[aadarkhadka@fileserver ~]$ getfacl /data/research
getfacl: Removing leading '/' from absolute path names
# file: data/research
# owner: root
# group: root
user::rwx
user:ram:rwx
user:ravi:rwx
group::r-x
mask::rwx
other::r-x

[aadarkhadka@fileserver ~]$ 
```

```
# Restart the service

[aadarkhadka@fileserver ~]$ sudo systemctl restart smb
[aadarkhadka@fileserver ~]$ sudo systemctl is-active smb
active
[aadarkhadka@fileserver ~]$ sudo systemctl is-enabled smb
enabled
[aadarkhadka@fileserver ~]$
```
- Install Samba Client Tools on the Client and Discover Samba Shares from the Client
```
# Accessing the files from different machines

[aadarkhadka@client ~]$ sudo dnf install -y samba-client

[aadarkhadka@client ~]$ sudo dnf install -y cifs-utils		# common internet file system

[aadarkhadka@client ~]$ smbclient -L 192.168.254.24 -U ram
Password for [SAMBA\ram]:

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	sales           Disk      Sales data
	marketing       Disk      Contains marketing data
	hrdata          Disk      Contains HR data
	IPC$            IPC       IPC Service (Samba 4.24.5)
	ram             Disk      Home Directories
SMB1 disabled -- no workgroup available
[aadarkhadka@client ~]$ 
```
- Mount the HR Share Using CIFS
```
[aadarkhadka@client ~]$ sudo mkdir /mnt/hr
[aadarkhadka@client ~]$ ls /mnt/hr
[aadarkhadka@client ~]$ ls /mnt
hr

[aadarkhadka@client ~]$ sudo mount -t cifs //192.168.254.24/hrdata /mnt/hr -o username=ravi
Password for ravi@//192.168.254.24/hrdata: 
[aadarkhadka@client ~]$ df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/cs-root       16G  1.5G   15G  10% /
devtmpfs                 830M     0  830M   0% /dev
tmpfs                    853M     0  853M   0% /dev/shm
tmpfs                    341M  4.9M  337M   2% /run
tmpfs                    1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                2.0G  405M  1.6G  21% /boot
tmpfs                    1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                    171M  4.0K  171M   1% /run/user/1000
//192.168.254.24/hrdata   16G  1.5G   15G  10% /mnt/hr
[aadarkhadka@client ~]$ 
[aadarkhadka@client ~]$ ls -l /mnt/hr
total 0
[aadarkhadka@client ~]$ sudo touch /mnt/hr/test.txt
[aadarkhadka@client ~]$ ls -l /mnt/hr
total 0
-rwxr-xr-x. 1 root root 0 Oct  3 18:20 test.txt
[aadarkhadka@client ~]$ 
```

- Test File Access 

```
[aadarkhadka@fileserver ~]$ sudo ls -l /data/hr/
total 4
-rw-r--r--. 1 root root 30 Oct  3 12:38 hrdata
-rwxr--r--. 1 ravi hr    0 Oct  3 18:20 test.txt
[aadarkhadka@fileserver ~]$ 
```
---

#### Access-Control Layers
One of the main lessons from this lab is that Samba file access involves multiple layers.
A simplified view is:
```
	Client
	  │
	  │ SMB/CIFS
	  ▼
	Samba
	  │
	  ├── Samba authentication
	  │
	  ├── smb.conf share configuration
	  │
	  ├── Host restrictions
	  │
	  ▼
 Linux filesystem permissions
	  │
	  ├── Owner
	  ├── Group
	  └── ACLs
	  │
	  ▼
	SELinux
	  │
	  ▼
     File / Directory
```
Access can therefore be affected by several independent controls.

**For example:**

- The user may need a valid Samba account.

- The user may need to be allowed by the share configuration.

- The client may need to satisfy host restrictions.

- Linux filesystem permissions must permit the operation.

- An applicable ACL must permit the operation.

- SELinux must allow Samba to access the resource.

- The network firewall must permit the required traffic.

- This is an important concept when troubleshooting Samba.

---

#### NOTE:

- Samba provides the network file-sharing layer
Samba allows Linux to provide SMB shares that clients can access over the network.

- Linux permissions still matter
Adding a user to a Samba share does not automatically bypass Linux filesystem permissions.

- Groups simplify departmental access
Instead of assigning permissions to every user individually, users can be placed into groups such as:
```
sales
marketing
hr
```
The groups can then be used to manage access to directories.

- Setgid is useful for shared directories
Using:
```
chmod 2770 directory
```
on a collaborative directory sets the setgid bit and helps maintain group ownership for newly created content.

- ACLs provide more granular control
ACLs allow individual users such as ram and ravi to receive permissions without changing the primary owner/group arrangement.

- SELinux adds another security layer
On an SELinux-enabled system, the Unix permissions may look correct while access can still be denied because of an inappropriate SELinux context or policy.

- Firewalls control network reachability
Even a correctly configured Samba server cannot be accessed if the required network traffic is blocked by the firewall.

- testparm should be part of the workflow
Before restarting or relying on a modified Samba configuration, validating smb.conf with:
```
sudo testparm
```
helps catch configuration problems early.

---