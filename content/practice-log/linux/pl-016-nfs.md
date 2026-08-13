---
title: "PL - 016 — NFS & Network-Based Storage Systems"
date: 2026-08-10
draft: false
---

### File System & Network Storage
```text
   Network Storage
   │
   ├── NAS
   │   └── File-level storage
   │       ├── NFS
   │       └── SMB/CIFS
   │
   └── SAN
       └── Block-level storage
           ├── iSCSI
           └── Fibre Channel
```

#### File-Level and Block-Level Storage
- Network Attached Storage( NAS ) --> NAS provides file-level access, commonly through NFS/SMB.
- Storage Area Network( SAN ) --> SAN provides block-level access, commonly through Fibre Channel or iSCSI.
 
### Network File Systems

A network file system allows files and directories on a server to be accessed by clients over a network.

NFS and SMB/CIFS are network file-sharing protocols. They provide file-level access to remote filesystems and use the underlying network stack, typically TCP/IP, for communication.

A network file system should not be confused with a distributed filesystem platform. NFS provides network access to a filesystem hosted by a server, while platforms such as CephFS and GlusterFS are designed as distributed storage/filesystem systems.

```text
   Network File Sharing
   │
   ├── NFS
   │   └── Commonly used in Linux / Unix environments
   │
   ├── SMB / CIFS
   │   └── Commonly used in Windows / Mixed-OS environments
   │
   └── Distributed Filesystem Platforms
       ├── CephFS
       └── GlusterFS
```
### Microsoft DFS

Microsoft Distributed File System (DFS) provides:
- DFS Namespaces — presents shared folders through a logical namespace.
- DFS Replication — replicates folders between Windows servers.

| Feature                    | **SMB / Samba**                                      | **NFS**                                                   |
| -------------------------- | ---------------------------------------------------- | --------------------------------------------------------- |
| **Primary environment**    | Windows / Mixed OS                                   | Linux / Unix / Mixed OS                                   |
| **Protocol**               | SMB/CIFS                                             | NFS                                                       |
| **Main purpose**           | Network file and printer sharing                     | Network file sharing                                      |
| **Authentication**         | Local accounts, NTLM, Kerberos, Active Directory    | UID/GID-based identity; Kerberos available with NFSv4    |
| **Typical production use** | Enterprise Windows file shares                       | Linux servers, virtualization, shared application storage |
| **Choose when**            | Windows/AD integration is important                  | Linux/Unix shared storage is required                     |


### What is Kerberos?

Kerberos is a ticket-based network authentication protocol. A client authenticates with the Kerberos Key Distribution Center (KDC) and obtains tickets that can be used to authenticate to network services without sending the user's password to those services.

**Kerberos = ticket-based network authentication**

```text
   User / Client
        │
        │ Authentication
        ▼
   Kerberos KDC
   (Key Distribution Center)
        │
        │ Service ticket
        ▼
   User / Client
        │
        │ Service request with Kerberos credentials
        ▼
   NFS Server
        │
        │ Verify Kerberos identity
        ▼
   File / Directory Access
```

**Authentication vs Authorization:**

```text
Kerberos
   ↓
Authentication
"Who are you?"

Permissions / ACLs
   ↓
Authorization
"What can you access?"
```

---

### Lab Topology

```text
NFS Server
-----------
Hostname: labserver
IP:       192.168.254.2

NFS Client
----------
Hostname: client1
IP:       192.168.254.3

Network
-------
192.168.254.0/24
```

### Terminal Session

``` 
 # server

[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ date
Tue Aug 11 05:54:38 AM +0545 2026
 
[aadarsha@labserver ~]$ hostname -I
192.168.254.2
 
[root@labserver ~]# mkdir -p /shared-storage/dev_data

[root@labserver ~]# cat /etc/fstab 
#
# /etc/fstab
# Created by anaconda on Mon Jul 13 08:44:14 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=b36b9ac1-df90-470c-92a9-ac5acaafacea /                       xfs     defaults        0 0
UUID=9f8e3f3e-f509-45f6-9fdf-d60e45d81cf2 /boot                   xfs     defaults        0 0
UUID=dda09a6b-dc33-4229-b517-b49f47416287 /var                    xfs     defaults        0 0
UUID=7d4ec80b-c095-487b-8ef7-d984232ca220 none                    swap    defaults        0 0
[root@labserver ~]# 
[root@labserver ~]# vi /etc/fstab 
[root@labserver ~]# 
[root@labserver ~]# cat /etc/fstab 
#
# /etc/fstab
# Created by anaconda on Mon Jul 13 08:44:14 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=b36b9ac1-df90-470c-92a9-ac5acaafacea /                       xfs     defaults        0 0
UUID=9f8e3f3e-f509-45f6-9fdf-d60e45d81cf2 /boot                   xfs     defaults        0 0
UUID=dda09a6b-dc33-4229-b517-b49f47416287 /var                    xfs     defaults        0 0
UUID=7d4ec80b-c095-487b-8ef7-d984232ca220 none                    swap    defaults        0 0

/dev/sdb                                 /shared-storage/dev_data  xfs    defaults        0 0
[root@labserver ~]# 

[root@labserver ~]# systemctl daemon-reload

[root@labserver ~]# mount -a

[root@labserver ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   20G  1.8G   19G   9% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                342M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            960M  485M  476M  51% /boot
/dev/mapper/cs-var   5.0G  232M  4.8G   5% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
/dev/sdb              10G  228M  9.8G   3% /shared-storage/dev_data
[root@labserver ~]# 

[root@labserver ~]# mkdir -p /shared-storage/{hr_data,research_data,support_data}

[root@labserver ~]# ls /shared-storage/
dev_data  hr_data  research_data  support_data
[root@labserver ~]# 

 # Configuring NFS Server / NAS Storage

  # Attach the disk or prepare the storage
  
  # 1. Install required packages
  # 2. Start NFS server
  # 3. Configure the firewall to allow NFS traffic.
     #    Restrict access to trusted client networks or hosts in production.
     #    NFSv3 uses additional RPC-related services such as rpcbind and mountd.
     #    NFSv4 uses TCP port 2049 and has a simpler network/port model.

  # 4. Share the folders using NFS
  
 # Configuring the NFS server

  # server

[aadarsha@labserver ~]$ whoami
aadarsha

[aadarsha@labserver ~]$ date
Tue Aug 11 05:54:38 AM +0545 2026
 
[aadarsha@labserver ~]$ hostname -I
192.168.254.2

[root@labserver ~]# hostname
labserver

[aadarsha@labserver ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   20G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
sdc           8:32   0    5G  0 disk 
sdd           8:48   0    4G  0 disk 
└─cs-root   253:0    0   20G  0 lvm  /
sr0          11:0    1 1024M  0 rom  

[root@labserver ~]# fdisk -l /dev/sdb 
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes 

[root@labserver ~]# mkfs -t xfs /dev/sdb               #  mkfs.xfs /dev/sdb

meta-data=/dev/sdb               isize=512    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
[root@labserver ~]#

[root@labserver ~]# blkid /dev/sdb
/dev/sdb: UUID="1cc69d24-cfd1-4d5b-a4d4-74783f8db6b3" BLOCK_SIZE="512" TYPE="xfs"
[root@labserver ~]# 
 
[root@labserver ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   20G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk /shared-storage/dev_data
sdc           8:32   0    5G  0 disk 
sdd           8:48   0    4G  0 disk 
└─cs-root   253:0    0   20G  0 lvm  /
sr0          11:0    1 1024M  0 rom  

[root@labserver ~]# rpm -q nfs-utils
package nfs-utils is not installed

[root@labserver ~]# yum -y install nfs-utils
...
Installed:
  gssproxy-0.9.2-10.el10.x86_64                 libev-4.33-15.el10.x86_64                    
  libnfsidmap-1:2.8.3-9.el10.x86_64             libverto-libev-0.3.2-10.el10.x86_64          
  nfs-utils-1:2.8.3-9.el10.x86_64               quota-1:4.09-10.el10.x86_64                  
  quota-nls-1:4.09-10.el10.noarch               rpcbind-1.2.9-0.el10.x86_64                  
  sssd-nfs-idmap-2.13.1-2.el10.x86_64          
Complete!
[root@labserver ~]# 

[root@labserver ~]# rpm -q nfs-utils
nfs-utils-2.8.3-9.el10.x86_64

[root@labserver ~]# systemctl is-active nfs-server
inactive
 
[root@labserver ~]# systemctl start nfs-server

[root@labserver ~]# systemctl is-active nfs-server
active

[root@labserver ~]# systemctl is-enabled nfs-server
disabled

[root@labserver ~]# systemctl enable nfs-server
Created symlink '/etc/systemd/system/multi-user.target.wants/nfs-server.service' → '/usr/lib/systemd/system/nfs-server.service'.

[root@labserver ~]# systemctl is-enabled nfs-server
enabled

[root@labserver ~]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client http ssh
  ports: 5050/tcp 8080/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.5.0/24" port port="22" protocol="tcp" accept

[root@labserver ~]# firewall-cmd --list-all | grep services
  services: cockpit dhcpv6-client http ssh

[root@labserver ~]# firewall-cmd --permanent --add-service={nfs,mountd,rpc-bind}
success
 
[root@labserver ~]# firewall-cmd --reload
success

[root@labserver ~]# firewall-cmd --list-all | grep services
  services: cockpit dhcpv6-client http mountd nfs rpc-bind ssh

  # Add some files in the shared directories
 
[root@labserver ~]# ls /shared-storage/
dev_data  hr_data  research_data  support_data
 
 
[root@labserver ~]# vi /shared-storage/dev_data/dev-file 

[root@labserver ~]# vi /shared-storage/dev_data/server-created

[root@labserver ~]# vi /shared-storage/hr_data/hr-file
 
[root@labserver ~]# vi /shared-storage/research_data/research-file
 
[root@labserver ~]# vi /shared-storage/support_data/support-file

[root@labserver ~]# mkdir -p /public_data

[root@labserver ~]# vi /public_data/server-public-data

 # share the folders using NFS
 
[root@labserver ~]# cat /etc/exports
[root@labserver ~]# 

[root@labserver ~]# vi /etc/exports
 
[root@labserver ~]# cat /etc/exports
# <Path of the dir to be shared>                     <Allowed Hosts>(<Options>)
/shared-storage/dev_data/                           192.168.254.0/24(rw)      
/shared-storage/hr_data/                            192.168.254.3(rw,sync)
/shared-storage/research_data/                      research1.local.domain(ro) 192.168.254.1(rw)
/shared-storage/support_data/                       *.local.domain(ro)
/public_data                                        *(ro)
[root@labserver ~]# 

 # Since, we have not configured DNS, let's use IP Address only

[root@labserver ~]# vi /etc/exports

[root@labserver ~]# cat /etc/exports
# <Path of the dir to be shared>                     <Allowed Hosts>(<Options>)
/shared-storage/dev_data/                           192.168.254.0/24(rw)      
/shared-storage/hr_data/                            192.168.254.3(rw,sync)
/shared-storage/research_data/                      192.168.254.3(ro) 192.168.254.1(rw)
/shared-storage/support_data/                       192.168.254.0/24(ro)
/public_data                                        *(ro)

 
[root@labserver ~]# ls -ld /shared-storage/dev_data
drwxr-xr-x. 2 root root 22 Aug 12 05:14 /shared-storage/dev_data
 
 # The default permissions for others is r-x, so client get this permission only, but in some cases, we need write permission also

[root@labserver ~]# rpm -q nfs-utils
nfs-utils-2.8.3-9.el10.x86_64

[root@labserver ~]# grep nobody /etc/passwd
nobody:x:65534:65534:Kernel Overflow User:/:/usr/sbin/nologin
```
 
### NFS `root_squash` and the `nobody` User

By default, NFS uses `root_squash`.

When a client accesses an NFS export as the `root` user, the NFS server maps the remote `root` identity (UID 0) to an anonymous identity, commonly represented by the local `nobody` user.

```text
Client
  │
  │ root (UID 0)
  ▼
NFS Server
  │
  │ root_squash
  ▼
Anonymous identity
  │
  ▼
nobody
```
This prevents a remote client administrator from automatically obtaining root privileges over files on the NFS server.

root_squash is enabled by default and should normally be retained.

Important: nobody is not a general "permission for all NFS clients." NFS access is still controlled by the exported filesystem's ownership, permissions, ACLs, and NFS export options.

### NFS Access Control Model

NFS access is determined by multiple layers:
```text
                    NFS Client
                        │
                        ▼
              NFS export restrictions
              (/etc/exports)
                        │
                        ▼
               NFS security identity
               (AUTH_SYS / Kerberos)
                        │
                        ▼
             Linux UID/GID ownership
                        │
                        ▼
              Mode bits / POSIX ACLs
                        │
                        ▼
                 Final access
```
```
[root@labserver ~]# ls -ld /shared-storage/dev_data/
drwxr-xr-x. 2 root root 22 Aug 12 05:14 /shared-storage/dev_data/

[root@labserver ~]# chown nobody:root /shared-storage/dev_data/

[root@labserver ~]# chown nobody:root /shared-storage/hr_data/
 
[root@labserver ~]# chown nobody:root /shared-storage/support_data/

[root@labserver ~]# chown nobody:root /shared-storage/research_data/

[root@labserver ~]# ls -ld /shared-storage/dev_data/
drwxr-xr-x. 2 nobody root 22 Aug 12 05:14 /shared-storage/dev_data/
```

### `sync` vs `async`

`sync`
- The NFS server does not reply to a request until changes from the request have been written to stable storage.
- Provides stronger durability semantics.
- May have lower write performance.

`async`
- The NFS server may reply before changes are committed to stable storage.
- Can improve performance.
- Carries greater risk of data loss if the server fails before pending changes are committed.

**Production guidance:** Use `sync` unless there is a specific, understood reason to use `async`.

```
[root@labserver ~]# cat /etc/exports
# <Path of the dir to be shared>                     <Allowed Hosts>(<Options>)
/shared-storage/dev_data/                           192.168.254.0/24(rw)      
/shared-storage/hr_data/                            192.168.254.3(rw,sync)
/shared-storage/research_data/                      192.168.254.3(ro) 192.168.254.1(rw)
/shared-storage/support_data/                       192.168.254.0/24(ro)
/public_data                                        *(ro)

 # Note: To use authentication use : NFS + Kerberos
 
[root@labserver ~]# systemctl is-active nfs-server
active

[root@labserver ~]# systemctl restart nfs-server

[root@labserver ~]# systemctl is-active nfs-server
active

[root@labserver ~]# systemctl is-enabled nfs-server
enabled

[root@labserver ~]# exportfs
/shared-storage/hr_data
		192.168.254.3
/shared-storage/research_data
		192.168.254.3
/shared-storage/research_data
		192.168.254.1
/shared-storage/dev_data
		192.168.254.0/24
/shared-storage/support_data
		192.168.254.0/24
/public_data  	<world>
[root@labserver ~]# 

[root@labserver ~]# exportfs -rav
exporting 192.168.254.3:/shared-storage/research_data
exporting 192.168.254.1:/shared-storage/research_data
exporting 192.168.254.3:/shared-storage/hr_data
exporting 192.168.254.0/24:/shared-storage/support_data
exporting 192.168.254.0/24:/shared-storage/dev_data
exporting *:/public_data
[root@labserver ~]# 

[root@labserver ~]# exportfs -v
/shared-storage/hr_data
		192.168.254.3(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
/shared-storage/research_data
		192.168.254.3(sync,wdelay,hide,no_subtree_check,sec=sys,ro,secure,root_squash,no_all_squash)
/shared-storage/research_data
		192.168.254.1(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
/shared-storage/dev_data
		192.168.254.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
/shared-storage/support_data
		192.168.254.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,ro,secure,root_squash,no_all_squash)
/public_data  	<world>(sync,wdelay,hide,no_subtree_check,sec=sys,ro,secure,root_squash,no_all_squash)
[root@labserver ~]# 

[root@labserver ~]# firewall-cmd --list-all | grep services
  services: cockpit dhcpv6-client http mountd nfs rpc-bind ssh

[root@labserver ~]# hostname -I
192.168.254.2 

 # Configuring the NFS Client

  # client-1
 
[aadarkdk@labserver ~]$ date
Tue Aug 11 05:54:21 AM +0545 2026
 
[aadarkdk@labserver ~]$ whoami
aadarkdk

[aadarkdk@labserver ~]$ hostname -I
192.168.254.3 
```

### NFS Versions

Modern Linux clients support multiple NFS protocol versions.

Common versions include:

- NFSv3
- NFSv4
- NFSv4.0
- NFSv4.1
- NFSv4.2

If the NFS version is not explicitly specified, the Linux NFS client negotiates a supported version. On RHEL 10, the client tries NFSv4.2 first and negotiates down if necessary.

For modern deployments, NFSv4 is generally preferred unless compatibility with NFSv3 is required.

```
 # Accessing NFS-Shared Folders (from Client-Side)
 
  # 1. Install required package
  
[aadarkdk@labserver ~]$ whoami
aadarkdk

[aadarkdk@labserver ~]$ su - root
Password: 
Last login: Tue Aug 11 11:28:41 +0545 2026 on tty1
 
[root@labserver ~]# yum -y install nfs-utils

[root@labserver ~]# rpm -q nfs-utils
nfs-utils-2.8.3-9.el10.x86_64

[root@labserver ~]# grep nobody /etc/passwd
nobody:x:65534:65534:Kernel Overflow User:/:/usr/sbin/nologin

  # 2. Show the list of NFS-shared Folders
    
    # showmount -e <NFS-server>

[aadarkdk@labserver ~]$ showmount -e 192.168.254.2
Export list for 192.168.254.2:
/public_data                  *
/shared-storage/support_data  192.168.254.0/24
/shared-storage/dev_data      192.168.254.0/24
/shared-storage/research_data 192.168.254.1,192.168.254.3
/shared-storage/hr_data       192.168.254.3
[aadarkdk@labserver ~]$ 
 
 # 3. Access (Mount) the NFS-shared Folder
 
[root@labserver ~]# ls /mnt/
[root@labserver ~]# 

[root@labserver ~]# mkdir -p /mnt/dev_data

[root@labserver ~]# ls /mnt/
dev_data

  # I. Temporary Access (Mount)

 # mount -t nfs <NFS-server>:<shared-folders> <mount-point>

[root@labserver ~]# df -h
Filesystem                              Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                      16G  1.8G   15G  11% /
devtmpfs                                831M     0  831M   0% /dev
tmpfs                                   853M     0  853M   0% /dev/shm
tmpfs                                   342M  4.9M  337M   2% /run
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                               2.0G  505M  1.5G  26% /boot
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                   171M  4.0K  171M   1% /run/user/1000
[root@labserver ~]# 

[root@labserver ~]# mount -t nfs 192.168.254.2:/shared-storage/dev_data /mnt/dev_data
[root@labserver ~]# 

[root@labserver ~]# df -h
Filesystem                              Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                      16G  1.8G   15G  11% /
devtmpfs                                831M     0  831M   0% /dev
tmpfs                                   853M     0  853M   0% /dev/shm
tmpfs                                   342M  4.9M  337M   2% /run
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                               2.0G  505M  1.5G  26% /boot
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                   171M  4.0K  171M   1% /run/user/1000
192.168.254.2:/shared-storage/dev_data   10G  227M  9.8G   3% /mnt/dev_data
[root@labserver ~]# 

[root@labserver ~]# cd /mnt/dev_data/

[root@labserver dev_data]# ls
dev-file  server-created

[root@labserver dev_data]# cat server-created 
this is file is created by NFS server

[root@labserver dev_data]# vi client1-created

[root@labserver dev_data]# ls
client1-created  dev-file  server-created

[root@labserver dev_data]# ls -l client1-created 
-rw-r--r--. 1 nobody nobody 39 Aug 12 13:48 client1-created

[root@labserver dev_data]# ls -l
total 12
-rw-r--r--. 1 nobody nobody 39 Aug 12 13:48 client1-created
-rw-r--r--. 1 root   root   25 Aug 12 05:14 dev-file
-rw-r--r--. 1 root   root   38 Aug 12 13:04 server-created

[root@labserver dev_data]# cd -
/root

[root@labserver ~]# ls -l /mnt/dev_data/client1-created 
-rw-r--r--. 1 nobody nobody 39 Aug 12 13:48 /mnt/dev_data/client1-created
[root@labserver ~]# 

 # Test reach of client1-file on the NFS server
   
   # server
   
[root@labserver ~]# hostname -I
192.168.254.2 
 
[root@labserver ~]# ls /shared-storage/dev_data/
client1-created  dev-file  server-created

[root@labserver ~]# cat /shared-storage/dev_data/client1-created 
this file is created by the client1...
[root@labserver ~]#

  # client1
  
[root@labserver ~]# reboot

[root@labserver ~]# Connection to 192.168.254.3 closed by remote host.
Connection to 192.168.254.3 closed.

aadarkdk@pop-os:~$ ssh aadarkdk@192.168.254.3
aadarkdk@192.168.254.3's password: 

[aadarkdk@labserver ~]$ su - root
Password: 

[root@labserver ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   16G  1.8G   15G  11% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                342M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            2.0G  505M  1.5G  26% /boot
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
[root@labserver ~]#

  # I. Permanent Mounting on Client1
  
[root@labserver ~]# cat /etc/fstab 
#
# /etc/fstab
# Created by anaconda on Thu Aug  6 08:51:59 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=0f0d6855-f185-460e-8a01-2388e54c0386 /                       xfs     defaults        0 0
UUID=08459ec7-f93f-45b9-82c1-8722d79c82df /boot                   xfs     defaults        0 0
UUID=21d72bdc-d1b3-4336-bf31-fbcf8239f87d none                    swap    defaults        0 0
[root@labserver ~]# 

[root@labserver ~]# vi /etc/fstab 
[root@labserver ~]# cat /etc/fstab 
#
# /etc/fstab
# Created by anaconda on Thu Aug  6 08:51:59 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=0f0d6855-f185-460e-8a01-2388e54c0386 /                       xfs     defaults        0 0
UUID=08459ec7-f93f-45b9-82c1-8722d79c82df /boot                   xfs     defaults        0 0
UUID=21d72bdc-d1b3-4336-bf31-fbcf8239f87d none                    swap    defaults        0 0
192.168.254.2:/shared-storage/dev_data    /mnt/dev_data/          nfs     defaults        0 0
[root@labserver ~]# 

[root@labserver ~]# systemctl daemon-reload 

[root@labserver ~]# mount -a

[root@labserver ~]# df -h
Filesystem                              Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                      16G  1.8G   15G  11% /
devtmpfs                                831M     0  831M   0% /dev
tmpfs                                   853M     0  853M   0% /dev/shm
tmpfs                                   342M  4.9M  337M   2% /run
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                               2.0G  505M  1.5G  26% /boot
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                   171M  4.0K  171M   1% /run/user/1000
192.168.254.2:/shared-storage/dev_data   10G  227M  9.8G   3% /mnt/dev_data
[root@labserver ~]# reboot

[root@labserver ~]# Connection to 192.168.254.3 closed by remote host.
Connection to 192.168.254.3 closed.
aadarkdk@pop-os:~$
s
aadarkdk@pop-os:~$ ssh aadarkdk@192.168.254.3
aadarkdk@192.168.254.3's password: 
Last login: Wed Aug 12 13:57:16 2026 from 192.168.254.152
[aadarkdk@labserver ~]$ df -h
Filesystem                              Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                      16G  1.8G   15G  11% /
devtmpfs                                831M     0  831M   0% /dev
tmpfs                                   853M     0  853M   0% /dev/shm
tmpfs                                   342M  4.9M  337M   2% /run
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                               2.0G  505M  1.5G  26% /boot
192.168.254.2:/shared-storage/dev_data   10G  227M  9.8G   3% /mnt/dev_data
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                   171M  4.0K  171M   1% /run/user/1000
[aadarkdk@labserver ~]$ 


 # addvantage of permanent vs temporary mounts in NFS, since background process runs in permanent mounting, performance decreases,
 # practical note and docs here
 
 
  # III. On Demand Mounting
  
 So, use autofs --> on demand mounting   ( # importand for RHCSA exam )
 
 [root@labserver ~]# yum -y install autofs
 
 [root@labserver ~]# systemctl enable autofs
Created symlink '/etc/systemd/system/multi-user.target.wants/autofs.service' → '/usr/lib/systemd/system/autofs.service'.

[root@labserver ~]# systemctl status autofs
○ autofs.service - Automounts filesystems on demand
     Loaded: loaded (/usr/lib/systemd/system/autofs.service; enabled; preset: disabled)
     Active: inactive (dead)
[root@labserver ~]# 

[root@labserver ~]# systemctl start autofs

[root@labserver ~]# systemctl status autofs
● autofs.service - Automounts filesystems on demand
     Loaded: loaded (/usr/lib/systemd/system/autofs.service; enabled; preset: disabled)
     Active: active (running) since Wed 2026-08-12 06:00:17 +0545; 3s ago
 Invocation: d894658f7d9340ed994bd89ff6bcb279
   Main PID: 2896 (automount)
      Tasks: 6 (limit: 10630)
     Memory: 1.9M (peak: 2.5M)
        CPU: 22ms
     CGroup: /system.slice/autofs.service
             └─2896 /usr/sbin/automount --systemd-service --dont-check-daemon

Aug 12 06:00:17 labserver systemd[1]: Starting autofs.service - Automounts filesystems on de>
Aug 12 06:00:17 labserver (utomount)[2896]: autofs.service: Referenced but unset environment>
Aug 12 06:00:17 labserver systemd[1]: Started autofs.service - Automounts filesystems on dem>
[root@labserver ~]#

 [root@labserver ~]# systemctl is-active autofs
active

[root@labserver ~]# systemctl is-enabled autofs
enabled

[root@labserver ~]# hostnamectl set-hostname client1

[root@client1 ~]# hostname
client1

[root@client1 ~]# ls /
afs  bin  boot  dev  etc  home  lib  lib64  media  misc  mnt  net  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

 # using autofs, the mount folder will be created automatically

[root@client1 ~]# ls /nfsdata/dev-data
ls: cannot access '/nfsdata/dev-data': No such file or directory

  # Config file of autofs: /etc/auto.master
 
[root@client1 ~]# vi /etc/auto.master
[root@client1 ~]# cat /etc/auto.master
...
/nfsdata                    /etc/client1-auto.nfs		# added part
[root@client1 ~]# 

[root@client1 ~]# vi /etc/client1-auto.nfs

[root@client1 ~]# cat /etc/client1-auto.nfs 
developer-data           -rw,sync        192.168.254.2:/shared-storage/dev_data
[root@client1 ~]#

[root@client1 ~]# ls /nfsdata/developer-data
ls: cannot access '/nfsdata/developer-data': No such file or directory

[root@client1 ~]# systemctl restart autofs

[root@client1 ~]# ls /nfsdata/

[root@client1 ~]# cd /nfsdata/

[root@client1 nfsdata]# pwd
/nfsdata

[root@client1 nfsdata]# ls

[root@client1 nfsdata]# cd developer-data

[root@client1 developer-data]# ls
client1-created  dev-file  server-created

[root@client1 developer-data]# df -h
Filesystem                              Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                      16G  1.8G   15G  11% /
devtmpfs                                831M     0  831M   0% /dev
tmpfs                                   853M     0  853M   0% /dev/shm
tmpfs                                   342M  4.9M  337M   2% /run
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                               2.0G  505M  1.5G  26% /boot
192.168.254.2:/shared-storage/dev_data   10G  227M  9.8G   3% /mnt/dev_data
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                   171M  4.0K  171M   1% /run/user/1000
192.168.254.2:/shared-storage/dev_data   10G  227M  9.8G   3% /nfsdata/developer-data
[root@client1 developer-data]#

[root@client1 developer-data]# mkdir auto-dir1

[root@client1 developer-data]# touch auto-file1

[root@client1 developer-data]# ls
auto-dir1  auto-file1  client1-created  dev-file  server-created
[root@client1 developer-data]#

 # Testing from server
  # server
 
[root@labserver ~]# hostname
labserver

[root@labserver ~]# ls /shared-storage/dev_data/
auto-dir1  auto-file1  client1-created  dev-file  server-created

  # Add time out

[root@client1 developer-data]# vi /etc/auto.master

[root@client1 developer-data]# cat /etc/auto.master
...
/nfsdata                    /etc/client1-auto.nfs   --timeout=60
...
[root@client1 developer-data]#

[root@client1 developer-data]# systemctl restart autofs

[root@client1 developer-data]# df -h
Filesystem                              Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                      16G  1.8G   15G  11% /
devtmpfs                                831M     0  831M   0% /dev
tmpfs                                   853M     0  853M   0% /dev/shm
tmpfs                                   342M  4.9M  337M   2% /run
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                               2.0G  505M  1.5G  26% /boot
192.168.254.2:/shared-storage/dev_data   10G  227M  9.8G   3% /mnt/dev_data
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                   171M  4.0K  171M   1% /run/user/1000
192.168.254.2:/shared-storage/dev_data   10G  227M  9.8G   3% /nfsdata/developer-data

 # 1 minute later...

[root@client1 developer-data]# df -h
Filesystem                              Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                      16G  1.8G   15G  11% /
devtmpfs                                831M     0  831M   0% /dev
tmpfs                                   853M     0  853M   0% /dev/shm
tmpfs                                   342M  4.9M  337M   2% /run
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                               2.0G  505M  1.5G  26% /boot
192.168.254.2:/shared-storage/dev_data   10G  227M  9.8G   3% /mnt/dev_data
tmpfs                                   1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                   171M  4.0K  171M   1% /run/user/1000
[root@client1 developer-data]# 

[root@client1 developer-data]# exit
logout
[aadarkdk@client1 ~]$ exit
logout
Connection to 192.168.254.3 closed.
aadarkdk@pop-os:~$
```