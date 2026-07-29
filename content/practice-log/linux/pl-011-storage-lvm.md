---
title: "PL - 011 — Storage Management & Logical Volume Administration"
date: 2026-07-29
draft: false
---

#### Concepts:
  > Partition, 
  > Mounting, 
  > File System Type,
  > LVM, 
  > Troubleshooting
  
### Terminal Session
```
[aadarsha@localhost ~]$ whoami
aadarsha

[aadarsha@localhost ~]$ date
Mon Jul 13 06:30:08 PM +0545 2026

[aadarsha@localhost ~]$ su - root
Password: 
Last login: Mon Jul 13 14:49:30 +0545 2026 on pts/0
[root@localhost ~]# 

[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sr0          11:0    1 1024M  0 rom  
[root@localhost ~]# 

 # Here:
 
 #------------------------------------------------------------------------------------------------------------------------
 #   Column    |  		Description	                     |	             Example Values
 #------------------------------------------------------------------------------------------------------------------------
 #   MAJ       | Major number - identifies the device driver type    | 8 (SCSI/SATA), 11 (CD-ROM), 253 (device-mapper/LVM)
 #------------------------------------------------------------------------------------------------------------------------
 #   MIN       | Minor number - identifies specific device instance  | 0, 1, 2, 3 (sequential per device)
 #------------------------------------------------------------------------------------------------------------------------
 #   RM        | Removable media flag                                | 0 (fixed), 1 (removable)
 #------------------------------------------------------------------------------------------------------------------------
 #   SIZE      | Device capacity in human-readable format            | 25.1G, 2M, 1024M
 #------------------------------------------------------------------------------------------------------------------------
 #   RO        | Read-only flag                                      | 0 (read-write), 1 (read-only)
 #------------------------------------------------------------------------------------------------------------------------
 #   TYPE      | Device classification                               | disk, part, lvm, rom, loop, crypt
 #------------------------------------------------------------------------------------------------------------------------
 # MOUNTPOINTS | Mount point path(s)                                 | /, /boot, /var, [SWAP]
 #------------------------------------------------------------------------------------------------------------------------
 
[root@localhost ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.5G   16G   9% /
devtmpfs             830M     0  830M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            960M  386M  575M  41% /boot
/dev/mapper/cs-var   5.0G  157M  4.8G   4% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
[root@localhost ~]# 

 # Disk Partitioning Tools:

  #-----------------------------------------------------------------------------------------------------------------------------------------
  # Tool        | Partition Table Support | Key Feature                             | When to Use
  #-----------------------------------------------------------------------------------------------------------------------------------------
  #1. fdisk     | MBR (msdos) only        | Traditional, interactive CLI            | Legacy systems, <2TB disks, simple MBR setups
  #-----------------------------------------------------------------------------------------------------------------------------------------
  #2. gdisk     | GPT only                | GPT-native, supports >2TB disks         | Modern systems with UEFI, disks >2TB
  #-----------------------------------------------------------------------------------------------------------------------------------------
  #3. parted    | Both MBR & GPT          | Advanced scripting, resizing, alignment | Versatile tool, automated scripts, mixed environments
  #-----------------------------------------------------------------------------------------------------------------------------------------

[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sr0          11:0    1 1024M  0 rom  
[root@localhost ~]# 

[root@localhost ~]# fdisk -l /dev/sda
Disk /dev/sda: 25.08 GiB, 26926350336 bytes, 52590528 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9214A82B-D2B3-419C-B8D1-27CF23F9B767

Device       Start      End  Sectors Size Type
/dev/sda1     2048     6143     4096   2M BIOS boot
/dev/sda2     6144  2103295  2097152   1G Linux extended boot
/dev/sda3  2103296 52439039 50335744  24G Linux LVM
[root@localhost ~]# 
```
### Creating Partitioning: 

This lab demonstrates how to create an additional virtual disk in Oracle VirtualBox and configure an **MBR (Master Boot Record)** partition table using the Linux `fdisk` utility. The exercise covers the creation of primary, extended, and logical partitions.

#### Environment
| Component | Details |
|-----------|---------|
| Host OS | Pop!_OS |
| Hypervisor | Oracle VirtualBox |
| Guest OS | CentOS Stream 10 |
| Existing Disk | `/dev/sda` |
| Additional Disk | `/dev/sdb` (10 GB) |
| Partition Scheme | MBR (DOS) |
---
```
 # Initially:

[root@localhost ~]# whoami
root
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sr0          11:0    1 1024M  0 rom  
[root@localhost ~]# 

[root@localhost ~]# poweroff
[root@localhost ~]# Connection to 192.168.254.8 closed by remote host.
Connection to 192.168.254.8 closed.
aadarkdk@pop-os:~$ 
aadarkdk@pop-os:~$ VBoxManage list vms
"server" {45206028-d5a9-4bde-a5ca-a7526d7a6ece}
"centos10-server1" {2b2ea4f3-1293-4626-be6c-d323569e0b84}
aadarkdk@pop-os:~$ 
aadarkdk@pop-os:~$ VBoxManage createmedium disk \
  --filename "$HOME/VirtualBox VMs/centos10-server1/disk2.vdi" \
  --size 10240 \
  --format VDI
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Medium created. UUID: ca420a16-9b9e-4741-ab50-1527de3edc2e
aadarkdk@pop-os:~$ 
aadarkdk@pop-os:~$ VBoxManage showvminfo "centos10-server1"
Name:                        centos10-server1
Encryption:                  disabled
Groups:                      /
Platform Architecture:       x86
Guest OS:                    Red Hat (64-bit)
...
...
VM process priority:         default
Storage Controllers:
#0: 'IDE', Type: PIIX4, Instance: 0, Ports: 2 (max 2), Bootable
  Port 0, Unit 0: Empty
#1: 'SATA', Type: IntelAhci, Instance: 0, Ports: 1 (max 30), Bootable
  Port 0, Unit 0: UUID: 51d7a839-95a3-41b4-bc34-ceb8f5aa0353
    Location: "/home/aadarkdk/VirtualBox VMs/centos10-server1/centos10-server1.vdi"
NIC 1:                       MAC: 08002763E705, Attachment: Bridged Interface 'wlp4s0', Cable connected: on, Trace: off (file: none), Type: 82540EM, Reported speed: 0 Mbps, Boot priority: 0, Promisc Policy: deny, Bandwidth group: none
NIC 2:                       disabled
...
...
aadarkdk@pop-os:~$

 # The newly created disk is attached to the existing SATA controller on **Port 1**.

aadarkdk@pop-os:~$ VBoxManage storageattach "centos10-server1" \
  --storagectl "SATA" \
  --port 1 \
  --device 0 \
  --type hdd \
  --medium "$HOME/VirtualBox VMs/centos10-server1/disk2.vdi"
aadarkdk@pop-os:~$ 
aadarkdk@pop-os:~$ VBoxManage showvminfo "centos10-server1"
Name:                        centos10-server1
Encryption:                  disabled
Groups:                      /
Platform Architecture:       x86
Guest OS:                    Red Hat (64-bit)
...
...
Storage Controllers:
#0: 'IDE', Type: PIIX4, Instance: 0, Ports: 2 (max 2), Bootable
  Port 0, Unit 0: Empty
#1: 'SATA', Type: IntelAhci, Instance: 0, Ports: 2 (max 30), Bootable
  Port 0, Unit 0: UUID: 51d7a839-95a3-41b4-bc34-ceb8f5aa0353
    Location: "/home/aadarkdk/VirtualBox VMs/centos10-server1/centos10-server1.vdi"
  Port 1, Unit 0: UUID: ca420a16-9b9e-4741-ab50-1527de3edc2e
    Location: "/home/aadarkdk/VirtualBox VMs/centos10-server1/disk2.vdi"
NIC 1:                       MAC: 08002763E705, Attachment: Bridged Interface 'wlp4s0', Cable connected: on, Trace: off (file: none), Type: 82540EM, Reported speed: 0 Mbps, Boot priority: 0, Promisc Policy: deny, Bandwidth group: none
NIC 2:                       disabled
...
...
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ VBoxManage startvm "centos10-server1"
Waiting for VM "centos10-server1" to power on...
VM "centos10-server1" has been successfully started.
aadarkdk@pop-os:~$ 
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.8
aadarsha@192.168.254.8's password: 
Last login: Mon Jul 13 19:26:31 2026
[aadarsha@localhost ~]$ 
[aadarsha@localhost ~]$ su - root
Password: 
Last login: Mon Jul 13 18:30:21 +0545 2026 on pts/0
[root@localhost ~]# 
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk                   <----- this is newly attached disk
sr0          11:0    1 1024M  0 rom  
[root@localhost ~]# 

 # Creating MBR-based Disk Partitions 

[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
sr0          11:0    1 1024M  0 rom  
[root@localhost ~]# 

 # sdb --> has no partitioning, we will use MBR-based partitioning on it.

[root@localhost ~]# fdisk -l /dev/sdb 
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
[root@localhost ~]# 

 # Since the disk is uninitialized, `fdisk` automatically creates a new **DOS (MBR)** partition table before partition creation begins.

[root@localhost ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0xd02aa787.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-20971519, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519): +1G     <--- Primary Partition 1 (1 GB)

Created a new partition 1 of type 'Linux' and of size 1 GiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot Start     End Sectors Size Id Type
/dev/sdb1        2048 2099199 2097152   1G 83 Linux

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (2099200-20971519, default 2099200): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2099200-20971519, default 20971519): +2G  <--- Primary Partition 2 (2 GB)

Created a new partition 2 of type 'Linux' and of size 2 GiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start     End Sectors Size Id Type
/dev/sdb1          2048 2099199 2097152   1G 83 Linux
/dev/sdb2       2099200 6293503 4194304   2G 83 Linux

Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (3,4, default 3): 
First sector (6293504-20971519, default 6293504): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (6293504-20971519, default 20971519): +512M  <--- Primary Partition 3 (512 MB)

Created a new partition 3 of type 'Linux' and of size 512 MiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 2099199 2097152    1G 83 Linux
/dev/sdb2       2099200 6293503 4194304    2G 83 Linux
/dev/sdb3       6293504 7342079 1048576  512M 83 Linux

 # Creating the Extended Partition

  --> MBR supports a maximum of four partition entries. To create additional partitions beyond the primary partitions, an Extended Partition is required.

Command (m for help): n
Partition type
   p   primary (3 primary, 0 extended, 1 free)
   e   extended (container for logical partitions)
Select (default e): e

Selected partition 4
First sector (7342080-20971519, default 7342080): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (7342080-20971519, default 20971519): 

Created a new partition 4 of type 'Extended' and of size 6.5 GiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended

Command (m for help): n
All primary partitions are in use.
Adding logical partition 5
First sector (7344128-20971519, default 7344128): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (7344128-20971519, default 20971519): +200M   <--- Logical Partition 5 (200 MB)

Created a new partition 5 of type 'Linux' and of size 200 MiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 83 Linux

Command (m for help): n
All primary partitions are in use.
Adding logical partition 6
First sector (7755776-20971519, default 7755776): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (7755776-20971519, default 20971519): +800M  <--- Logical Partition 6 (800 MB)

Created a new partition 6 of type 'Linux' and of size 800 MiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 83 Linux
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@localhost ~]# 
[root@localhost ~]# 
[root@localhost ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    1G  0 part 
├─sdb2        8:18   0    2G  0 part 
├─sdb3        8:19   0  512M  0 part 
├─sdb4        8:20   0    1K  0 part 
├─sdb5        8:21   0  200M  0 part 
└─sdb6        8:22   0  800M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@localhost ~]# 

  # w ---> Save & Exit
  # q / CTRL + C --> Quit

[root@localhost ~]# partprobe /dev/sdb		#  Reload the Partition Table
[root@localhost ~]# 
[root@localhost ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 83 Linux
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
[root@localhost ~]# 
[root@localhost ~]# 

 # Summary

  # The following MBR partition layout was successfully created on `/dev/sdb`:

| Partition   | Type     | Size            |
|-------------|----------|-----------------|
| `/dev/sdb1` | Primary  | 1 GB            |
| `/dev/sdb2` | Primary  | 2 GB            |
| `/dev/sdb3` | Primary  | 512 MB          |
| `/dev/sdb4` | Extended | Remaining Space |
| `/dev/sdb5` | Logical  | 200 MB          |
| `/dev/sdb6` | Logical  | 800 MB          |
```
  #### Key Takeaways

   - Successfully added a secondary virtual disk using Oracle VirtualBox.
   - Initialized the disk with an **MBR (DOS)** partition table.
   - Created **three primary partitions**.
   - Created **one extended partition**.
   - Created **two logical partitions** within the extended partition.
   - Verified the final partition layout using both `lsblk` and `fdisk -l`.

```
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.5
aadarsha@192.168.254.5's password: 
Last login: Wed Jul 15 10:22:16 2026

[aadarsha@aadar ~]$ su - root
Password: 
Last login: Wed Jul 15 05:51:48 +0545 2026 on tty1

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    1G  0 part 
├─sdb2        8:18   0    2G  0 part 
├─sdb3        8:19   0  512M  0 part 
├─sdb4        8:20   0    1K  0 part 
├─sdb5        8:21   0  200M  0 part 
└─sdb6        8:22   0  800M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

  # lsblk Columns:
  
    # NAME        --> Block device name
    # MAJ         --> Major Number (Kernel device driver ID)
    #                 Same driver = Same major number
    #                 Example:
    #                   sda, sdb -> 8 (SCSI/SATA driver)

    # MIN         --> Minor Number (Specific device/partition ID)
    #                 Unique within the same major number.
    #                 Example:
    #                   sda  -> 8:0
    #                   sda1 -> 8:1
    #                   sda2 -> 8:2
    #                   sdb  -> 8:16

    # RM          --> Removable Device
    #                 0 = Fixed disk
    #                 1 = Removable (USB, CD/DVD)

    # SIZE        --> Device capacity

    # RO          --> Read Only
    #                 0 = Writable
    #                 1 = Read-only

    # TYPE        --> Device type
    #                 disk  = Physical disk
    #                 part  = Partition
    #                 lvm   = Logical Volume
    #                 rom   = CD/DVD device

    # MOUNTPOINTS --> Mounted filesystem location(s)

    # BIOS: Basic Input/Output System
      #   Legacy BIOS systems : MBR ( Master Boot Record )
      #   Modern UEFI (Unified Extensible Firmware Interface) systems (current standard) : GPT ( GUID Partition Table )

  # Partition Table Standards

   # MBR (Master Boot Record)

      #   Firmware: Legacy BIOS

          #   Features:
 	  #     - Supports disks up to 2 TB
 	  #     - Maximum 4 Primary partitions
          #     - Can create:
 	  #          4 Primary partitions
          #          OR
          #          3 Primary + 1 Extended partition
          #     - Extended partition can contain Logical partitions

          #   Limitations:
          #     - 2 TB disk size limit
          #     - Limited partition entries
          #     - No backup partition table

   # GPT (GUID Partition Table)

      #   Firmware: UEFI
 
 	  #   Features:
 	  #     - Modern partition table standard
 	  #     - Supports disks larger than 2 TB
 	  #     - Typically supports 128 partitions
 	  #     - Stores backup partition table
 	  #     - Uses unique partition identifiers (GUID)
 	  #     - More reliable than MBR
 
 	  #   Advantages:
 	  #     - Required for UEFI boot
 	  #     - Better data integrity
 	  #     - Suitable for modern systems

   # Other Partition Table Standards:
      # APM ( Apple Partition Map )
        #   Used by:  Older Apple PowerPC Macintosh systems
        #   Replaced by GPT on modern Macs

      # BSD Disklabel
        #   Used by: BSD operating systems
        #   Provides partition information inside a disk

     # Sun Disklabel (VTOC)
        #   Used by: Solaris systems
        #   Traditional partitioning scheme

  # Loop / No Partition Table
    #   Used for:
    #     - Disk images
    #     - Virtual block devices
    #   Example:    /dev/loop0

  # Linux Device Naming:
    # SATA / SAS / SCSI / USB
      #   /dev/sda
      #   /dev/sdb
      #   /dev/sdc

    # VirtIO Virtual Disk
      #   /dev/vda
      #   /dev/vdb

    # NVMe SSD
      #   /dev/nvme0n1
      #   /dev/nvme0n1p1

      #   nvme0  --> Controller
      #   n1     --> Namespace
      #   p1     --> Partition

    # MMC / SD Card
      #   /dev/mmcblk0
      #   /dev/mmcblk0p1
      
   # Useful Commands:
     # lsblk                 --> List block devices
     # lsblk -f              --> Show filesystems
     # fdisk -l              --> Show partition table
     # parted -l             --> Show partition information
     # blkid                 --> Show UUID & filesystem
     # cat /proc/partitions  --> Kernel partition info

[root@aadar ~]# ls /dev/
autofs              hpet          rtc0      tty    tty28  tty48  ttyS1        vcsa4
block               hugepages     sda       tty0   tty29  tty49  ttyS2        vcsa5
bsg                 hwrng         sda1      tty1   tty3   tty5   ttyS3        vcsa6
bus                 initctl       sda2      tty10  tty30  tty50  udmabuf      vcsu
cdrom               input         sda3      tty11  tty31  tty51  uhid         vcsu1
char                kmsg          sdb       tty12  tty32  tty52  uinput       vcsu2
console             log           sdb1      tty13  tty33  tty53  urandom      vcsu3
core                loop-control  sdb2      tty14  tty34  tty54  usbmon0      vcsu4
cpu                 mapper        sdb3      tty15  tty35  tty55  usbmon1      vcsu5
cpu_dma_latency     mcelog        sdb4      tty16  tty36  tty56  usbmon2      vcsu6
cpu_wakeup_latency  mem           sdb5      tty17  tty37  tty57  userfaultfd  vfio
cs                  mqueue        sdb6      tty18  tty38  tty58  vcs          vga_arbiter
cuse                net           sg0       tty19  tty39  tty59  vcs1         vhci
disk                null          sg1       tty2   tty4   tty6   vcs2         vhost-net
dm-0                nvram         sg2       tty20  tty40  tty60  vcs3         vhost-vsock
dm-1                port          shm       tty21  tty41  tty61  vcs4         vmci
dm-2                ppp           snapshot  tty22  tty42  tty62  vcs5         vsock
dma_heap            ptmx          snd       tty23  tty43  tty63  vcs6         zero
dri                 pts           sr0       tty24  tty44  tty7   vcsa
fd                  random        stderr    tty25  tty45  tty8   vcsa1
full                rfkill        stdin     tty26  tty46  tty9   vcsa2
fuse                rtc           stdout    tty27  tty47  ttyS0  vcsa3
[root@aadar ~]# 
 
[root@aadar ~]# fdisk -l /dev/sda
Disk /dev/sda: 25.08 GiB, 26926350336 bytes, 52590528 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9214A82B-D2B3-419C-B8D1-27CF23F9B767

Device       Start      End  Sectors Size Type
/dev/sda1     2048     6143     4096   2M BIOS boot
/dev/sda2     6144  2103295  2097152   1G Linux extended boot
/dev/sda3  2103296 52439039 50335744  24G Linux LVM
[root@aadar ~]# 

[root@aadar ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 83 Linux
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
[root@aadar ~]# 

 # --> Some space gap is left there to store metadata in Logaical partition but not in primary partition.
 # --> Be very careful while partitioning the disk. It's like slicing the Apple, if not cut in rightly will create the problem.

[root@aadar ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 83 Linux
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
[root@aadar ~]# 

 # Making more Logical Partitioning

[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 83 Linux
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux

Command (m for help): n

All primary partitions are in use.
Adding logical partition 7
First sector (9396224-20971519, default 9396224):  
Last sector, +/-sectors or +/-size{K,M,G,T,P} (9396224-20971519, default 20971519): +2G

Created a new partition 7 of type 'Linux' and of size 2 GiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 83 Linux
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
/dev/sdb7       9396224 13590527  4194304    2G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@aadar ~]# 

 # Changing filesystem ID

[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 83 Linux
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
/dev/sdb7       9396224 13590527  4194304    2G 83 Linux

Command (m for help): t

Partition number (1-7, default 7): 5
Hex code or alias (type L to list all): L

00 Empty            27 Hidden NTFS Win  82 Linux swap / So  c1 DRDOS/sec (FAT-
01 FAT12            39 Plan 9           83 Linux            c4 DRDOS/sec (FAT-
02 XENIX root       3c PartitionMagic   84 OS/2 hidden or   c6 DRDOS/sec (FAT-
03 XENIX usr        40 Venix 80286      85 Linux extended   c7 Syrinx         
04 FAT16 <32M       41 PPC PReP Boot    86 NTFS volume set  da Non-FS data    
05 Extended         42 SFS              87 NTFS volume set  db CP/M / CTOS / .
06 FAT16            4d QNX4.x           88 Linux plaintext  de Dell Utility   
07 HPFS/NTFS/exFAT  4e QNX4.x 2nd part  8e Linux LVM        df BootIt         
08 AIX              4f QNX4.x 3rd part  93 Amoeba           e1 DOS access     
09 AIX bootable     50 OnTrack DM       94 Amoeba BBT       e3 DOS R/O        
0a OS/2 Boot Manag  51 OnTrack DM6 Aux  9f BSD/OS           e4 SpeedStor      
0b W95 FAT32        52 CP/M             a0 IBM Thinkpad hi  ea Linux extended 
0c W95 FAT32 (LBA)  53 OnTrack DM6 Aux  a5 FreeBSD          eb BeOS fs        
0e W95 FAT16 (LBA)  54 OnTrackDM6       a6 OpenBSD          ee GPT            
0f W95 Ext'd (LBA)  55 EZ-Drive         a7 NeXTSTEP         ef EFI (FAT-12/16/
10 OPUS             56 Golden Bow       a8 Darwin UFS       f0 Linux/PA-RISC b
11 Hidden FAT12     5c Priam Edisk      a9 NetBSD           f1 SpeedStor      
12 Compaq diagnost  61 SpeedStor        ab Darwin boot      f4 SpeedStor      
14 Hidden FAT16 <3  63 GNU HURD or Sys  af HFS / HFS+       f2 DOS secondary  
16 Hidden FAT16     64 Novell Netware   b7 BSDI fs          f8 EBBR protective
17 Hidden HPFS/NTF  65 Novell Netware   b8 BSDI swap        fb VMware VMFS    
18 AST SmartSleep   70 DiskSecure Mult  bb Boot Wizard hid  fc VMware VMKCORE 
1b Hidden W95 FAT3  75 PC/IX            bc Acronis FAT32 L  fd Linux raid auto
1c Hidden W95 FAT3  80 Old Minix        be Solaris boot     fe LANstep        
1e Hidden W95 FAT1  81 Minix / old Lin  bf Solaris          ff BBT            
24 NEC DOS        

Aliases:
   linux          - 83
   swap           - 82
   extended       - 05
   uefi           - EF
   raid           - FD
   lvm            - 8E
   linuxex        - 85
Hex code or alias (type L to list all): 8e

Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 8e Linux LVM
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
/dev/sdb7       9396224 13590527  4194304    2G 83 Linux

Command (m for help): p

Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 83 Linux
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 8e Linux LVM
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
/dev/sdb7       9396224 13590527  4194304    2G 83 Linux

Command (m for help): t
Partition number (1-7, default 7): 3
Hex code or alias (type L to list all): 82

Changed type of partition 'Linux' to 'Linux swap / Solaris'.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 82 Linux swap / Solaris
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 8e Linux LVM
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
/dev/sdb7       9396224 13590527  4194304    2G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@aadar ~]# 

	# Filesystem ID (Partition Type ID)

	 # Identifies the intended use of a partition.
	 # Stored in the partition table.

	 # MBR : Uses Partition Type IDs (83, 82, 8e, ...)
	 # GPT : Uses Partition Type GUIDs (gdisk displays shorthand codes like 8300)

	 # Common MBR Partition Type IDs

		# 00 ---> Empty
		# 07 ---> NTFS / exFAT / HPFS
		# 0b ---> FAT32
		# 0f ---> Extended Partition

		# 82 ---> Linux Swap
		# 83 ---> Linux Filesystem (EXT2/3/4, XFS, Btrfs, ...)
		# 8e ---> Linux LVM
		# fd ---> Linux Software RAID
		# ef ---> EFI System Partition
		# ee ---> GPT Protective MBR

	# Common GPT Type Codes (gdisk)

		# 8300 ---> Linux Filesystem
		# 8200 ---> Linux Swap
		# 8E00 ---> Linux LVM
		# FD00 ---> Linux Software RAID
		# EF00 ---> EFI System Partition (ESP)
		# EF02 ---> BIOS Boot Partition (GRUB on GPT + BIOS)

	# Show Partition Types
		# fdisk -l      # MBR
		# gdisk -l      # GPT
		# parted -l     # MBR & GPT
	
	# Show Actual Filesystem
		# lsblk -f
		# blkid

[root@aadar ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 82 Linux swap / Solaris
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 8e Linux LVM
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
/dev/sdb7       9396224 13590527  4194304    2G 83 Linux
[root@aadar ~]# 

 # Deleting a Disk Partion
  
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    1G  0 part 
├─sdb2        8:18   0    2G  0 part 
├─sdb3        8:19   0  512M  0 part 
├─sdb4        8:20   0    1K  0 part 
├─sdb5        8:21   0  200M  0 part 
├─sdb6        8:22   0  800M  0 part 
└─sdb7        8:23   0    2G  0 part 
sr0          11:0    1 1024M  0 rom  
 
[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 82 Linux swap / Solaris
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 8e Linux LVM
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
/dev/sdb7       9396224 13590527  4194304    2G 83 Linux

Command (m for help): d
Partition number (1-7, default 7): 2

Partition 2 has been deleted.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 82 Linux swap / Solaris
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 8e Linux LVM
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
/dev/sdb7       9396224 13590527  4194304    2G 83 Linux

Command (m for help): d
Partition number (1,3-7, default 7): 6

Partition 6 has been deleted.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 82 Linux swap / Solaris
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 8e Linux LVM
/dev/sdb6       9396224 13590527  4194304    2G 83 Linux

Command (m for help): d
Partition number (1,3-6, default 6): 6

Partition 6 has been deleted.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 82 Linux swap / Solaris
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 8e Linux LVM

Command (m for help): d
Partition number (1,3-5, default 5): 4

Partition 4 has been deleted.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 2099199 2097152    1G 83 Linux
/dev/sdb3       6293504 7342079 1048576  512M 82 Linux swap / Solaris

Command (m for help): t
Partition number (1,3, default 3): 3
Hex code or alias (type L to list all): b

Changed type of partition 'Linux swap / Solaris' to 'W95 FAT32'.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 2099199 2097152    1G 83 Linux
/dev/sdb3       6293504 7342079 1048576  512M  b W95 FAT32

Command (m for help): q

[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sdb1          2048  2099199  2097152    1G 83 Linux
/dev/sdb2       2099200  6293503  4194304    2G 83 Linux
/dev/sdb3       6293504  7342079  1048576  512M 82 Linux swap / Solaris
/dev/sdb4       7342080 20971519 13629440  6.5G  5 Extended
/dev/sdb5       7344128  7753727   409600  200M 8e Linux LVM
/dev/sdb6       7755776  9394175  1638400  800M 83 Linux
/dev/sdb7       9396224 13590527  4194304    2G 83 Linux

Command (m for help): d
Partition number (1-7, default 7): 4

Partition 4 has been deleted.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 2099199 2097152    1G 83 Linux
/dev/sdb2       2099200 6293503 4194304    2G 83 Linux
/dev/sdb3       6293504 7342079 1048576  512M 82 Linux swap / Solaris

Command (m for help): t
Partition number (1-3, default 3): 3
Hex code or alias (type L to list all): b

Changed type of partition 'Linux swap / Solaris' to 'W95 FAT32'.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 2099199 2097152    1G 83 Linux
/dev/sdb2       2099200 6293503 4194304    2G 83 Linux
/dev/sdb3       6293504 7342079 1048576  512M  b W95 FAT32

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
 
[root@aadar ~]# 

[root@aadar ~]# partprobe /dev/sdb
[root@aadar ~]# 

 # Cleaning the partition completely ( data must not exit )

[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 2099199 2097152    1G 83 Linux
/dev/sdb2       2099200 6293503 4194304    2G 83 Linux
/dev/sdb3       6293504 7342079 1048576  512M  b W95 FAT32

Command (m for help): d
Partition number (1-3, default 3): 1

Partition 1 has been deleted.

Command (m for help): d
Partition number (2,3, default 3): 3

Partition 3 has been deleted.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Device     Boot   Start     End Sectors Size Id Type
/dev/sdb2       2099200 6293503 4194304   2G 83 Linux

Command (m for help): d
Selected partition 2
Partition 2 has been deleted.

Command (m for help): p

Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@aadar ~]# 

[root@aadar ~]# partprobe /dev/sdb
[root@aadar ~]# 
[root@aadar ~]# fdisk -l /dev/sdb 
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd02aa787
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 
[root@aadar ~]# 
[root@aadar ~]# wipefs --all /dev/sdb
/dev/sdb: 2 bytes were erased at offset 0x000001fe (dos): 55 aa
/dev/sdb: calling ioctl to re-read partition table: Success
[root@aadar ~]# 
[root@aadar ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
[root@aadar ~]# 

[root@aadar ~]# gdisk -l /dev/sdb
-bash: gdisk: command not found

[root@aadar ~]# ls /etc/yum.repos.d/
centos-addons.repo  centos.repo  epel.repo  epel-testing.repo

[root@aadar ~]# yum -y install epel-release
...

[root@aadar ~]# yum -y install gdisk
...

[root@aadar ~]# which gdisk
/usr/sbin/gdisk
[root@aadar ~]# gdisk -l /dev/sdb
GPT fdisk (gdisk) version 1.0.10

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.
Disk /dev/sdb: 20971520 sectors, 10.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): E0D01BE3-976B-4663-AC5C-F48F38174D57
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 20971453 sectors (10.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
[root@aadar ~]# 

[root@aadar ~]# gdisk /dev/sdb
GPT fdisk (gdisk) version 1.0.10

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help): p
Disk /dev/sdb: 20971520 sectors, 10.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2FE270FF-D4FC-4B46-A896-2424A80E0620
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 20971453 sectors (10.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-20971486, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-20971486, default = 20969471) or {+-}size{KMGTP}: +2G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): L
Type search string, or <Enter> to show all codes: 
0700 Microsoft basic data                0701 Microsoft Storage Replica         
0702 ArcaOS Type 1                       0c01 Microsoft reserved                
2700 Windows RE                          3000 ONIE boot                         
3001 ONIE config                         3900 Plan 9                            
4100 PowerPC PReP boot                   4200 Windows LDM data                  
4201 Windows LDM metadata                4202 Windows Storage Spaces            
7501 IBM GPFS                            7f00 ChromeOS kernel                   
7f01 ChromeOS root                       7f02 ChromeOS reserved                 
7f03 ChromeOS firmware                   7f04 ChromeOS mini-OS                  
7f05 ChromeOS hibernate                  8200 Linux swap                        
8300 Linux filesystem                    8301 Linux reserved                    
8302 Linux /home                         8303 Linux x86 root (/)                
8304 Linux x86-64 root (/)               8305 Linux ARM64 root (/)              
8306 Linux /srv                          8307 Linux ARM32 root (/)              
8308 Linux dm-crypt                      8309 Linux LUKS                        
830a Linux IA-64 root (/)                830b Linux x86 root verity             
830c Linux x86-64 root verity            830d Linux ARM32 root verity           
830e Linux ARM64 root verity             830f Linux IA-64 root verity           
8310 Linux /var                          8311 Linux /var/tmp                    
8312 Linux user's home                   8313 Linux x86 /usr                    
8314 Linux x86-64 /usr                   8315 Linux ARM32 /usr                  
Press the <Enter> key to see more codes, q to quit: q

Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/sdb: 20971520 sectors, 10.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2FE270FF-D4FC-4B46-A896-2424A80E0620
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 16777149 sectors (8.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     8300  Linux filesystem

Command (? for help): n
Partition number (2-128, default 2): 
First sector (34-20971486, default = 4196352) or {+-}size{KMGTP}: 
Last sector (4196352-20971486, default = 20969471) or {+-}size{KMGTP}: +1G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/sdb: 20971520 sectors, 10.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2FE270FF-D4FC-4B46-A896-2424A80E0620
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 14679997 sectors (7.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     8300  Linux filesystem
   2         4196352         6293503   1024.0 MiB  8300  Linux filesystem

Command (? for help): n
Partition number (3-128, default 3): 
First sector (34-20971486, default = 6293504) or {+-}size{KMGTP}: 
Last sector (6293504-20971486, default = 20969471) or {+-}size{KMGTP}: +512M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): b00
Changed type of partition to 'Microsoft basic data'

Command (? for help): p
Disk /dev/sdb: 20971520 sectors, 10.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2FE270FF-D4FC-4B46-A896-2424A80E0620
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 13631421 sectors (6.5 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     8300  Linux filesystem
   2         4196352         6293503   1024.0 MiB  8300  Linux filesystem
   3         6293504         7342079   512.0 MiB   0700  Microsoft basic data

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
[root@aadar ~]# 
[root@aadar ~]# 
[root@aadar ~]# partprobe /dev/sdb
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part 
├─sdb2        8:18   0    1G  0 part 
└─sdb3        8:19   0  512M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 
[root@aadar ~]# 
[root@aadar ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
[root@aadar ~]# 
[root@aadar ~]# 
[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data

Command (m for help): n
Partition number (4-128, default 4):  
First sector (7342080-20971486, default 7342080): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (7342080-20971486, default 20969471): +1G

Created a new partition 4 of type 'Linux filesystem' and of size 1 GiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem

Command (m for help): n
Partition number (5-128, default 5): 
First sector (9439232-20971486, default 9439232): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (9439232-20971486, default 20969471): +100M

Created a new partition 5 of type 'Linux filesystem' and of size 100 MiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux filesystem

Command (m for help): t
Partition number (1-5, default 5): 4
Partition type or alias (type L to list all): 8200

Type of partition 4 is unchanged: Linux filesystem.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux filesystem

Command (m for help): t
Partition number (1-5, default 5): 4
Partition type or alias (type L to list all): 82

Changed type of partition 'Linux filesystem' to 'Linux root verity (S390)'.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux root verity (S390)
/dev/sdb5  9439232 9644031  204800  100M Linux filesystem

Command (m for help): t
Partition number (1-5, default 5): 5
Partition type or alias (type L to list all): 

[root@aadar ~]# gdisk /dev/sdb
GPT fdisk (gdisk) version 1.0.10

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help): p
Disk /dev/sdb: 20971520 sectors, 10.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2FE270FF-D4FC-4B46-A896-2424A80E0620
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 20971453 sectors (10.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-20971486, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-20971486, default = 20969471) or {+-}size{KMGTP}: +2G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): L
Type search string, or <Enter> to show all codes: 
0700 Microsoft basic data                0701 Microsoft Storage Replica         
0702 ArcaOS Type 1                       0c01 Microsoft reserved                
2700 Windows RE                          3000 ONIE boot                         
3001 ONIE config                         3900 Plan 9                            
4100 PowerPC PReP boot                   4200 Windows LDM data                  
4201 Windows LDM metadata                4202 Windows Storage Spaces            
7501 IBM GPFS                            7f00 ChromeOS kernel                   
7f01 ChromeOS root                       7f02 ChromeOS reserved                 
7f03 ChromeOS firmware                   7f04 ChromeOS mini-OS                  
7f05 ChromeOS hibernate                  8200 Linux swap                        
8300 Linux filesystem                    8301 Linux reserved                    
8302 Linux /home                         8303 Linux x86 root (/)                
8304 Linux x86-64 root (/)               8305 Linux ARM64 root (/)              
8306 Linux /srv                          8307 Linux ARM32 root (/)              
8308 Linux dm-crypt                      8309 Linux LUKS                        
830a Linux IA-64 root (/)                830b Linux x86 root verity             
830c Linux x86-64 root verity            830d Linux ARM32 root verity           
830e Linux ARM64 root verity             830f Linux IA-64 root verity           
8310 Linux /var                          8311 Linux /var/tmp                    
8312 Linux user's home                   8313 Linux x86 /usr                    
8314 Linux x86-64 /usr                   8315 Linux ARM32 /usr                  
Press the <Enter> key to see more codes, q to quit: q

Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/sdb: 20971520 sectors, 10.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2FE270FF-D4FC-4B46-A896-2424A80E0620
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 16777149 sectors (8.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     8300  Linux filesystem

Command (? for help): n
Partition number (2-128, default 2): 
First sector (34-20971486, default = 4196352) or {+-}size{KMGTP}: 
Last sector (4196352-20971486, default = 20969471) or {+-}size{KMGTP}: +1G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/sdb: 20971520 sectors, 10.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2FE270FF-D4FC-4B46-A896-2424A80E0620
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 14679997 sectors (7.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     8300  Linux filesystem
   2         4196352         6293503   1024.0 MiB  8300  Linux filesystem

Command (? for help): n
Partition number (3-128, default 3): 
First sector (34-20971486, default = 6293504) or {+-}size{KMGTP}: 
Last sector (6293504-20971486, default = 20969471) or {+-}size{KMGTP}: +512M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): b00
Changed type of partition to 'Microsoft basic data'

Command (? for help): p
Disk /dev/sdb: 20971520 sectors, 10.0 GiB
Model: VBOX HARDDISK   
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): 2FE270FF-D4FC-4B46-A896-2424A80E0620
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 13631421 sectors (6.5 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         4196351   2.0 GiB     8300  Linux filesystem
   2         4196352         6293503   1024.0 MiB  8300  Linux filesystem
   3         6293504         7342079   512.0 MiB   0700  Microsoft basic data

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
[root@aadar ~]# 

[root@aadar ~]# partprobe /dev/sdb
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part 
├─sdb2        8:18   0    1G  0 part 
└─sdb3        8:19   0  512M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 
[root@aadar ~]# 
[root@aadar ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
[root@aadar ~]# 
[root@aadar ~]# 
[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data

Command (m for help): n
Partition number (4-128, default 4):  
First sector (7342080-20971486, default 7342080): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (7342080-20971486, default 20969471): +1G

Created a new partition 4 of type 'Linux filesystem' and of size 1 GiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem

Command (m for help): n
Partition number (5-128, default 5): 
First sector (9439232-20971486, default 9439232): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (9439232-20971486, default 20969471): +100M

Created a new partition 5 of type 'Linux filesystem' and of size 100 MiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux filesystem

Command (m for help): t
Partition number (1-5, default 5): 4
Partition type or alias (type L to list all): 8200

Type of partition 4 is unchanged: Linux filesystem.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux filesystem

Command (m for help): t
Partition number (1-5, default 5): 4
Partition type or alias (type L to list all): 82

Changed type of partition 'Linux filesystem' to 'Linux root verity (S390)'.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux root verity (S390)
/dev/sdb5  9439232 9644031  204800  100M Linux filesystem

Command (m for help): t
Partition number (1-5, default 5): 5
Partition type or alias (type L to list all): L
  1 EFI System                     C12A7328-F81F-11D2-BA4B-00A0C93EC93B
  2 MBR partition scheme           024DEE41-33E7-11D3-9D69-0008C781F39F
  3 Intel Fast Flash               D3BFE2DE-3DAF-11DF-BA40-E3A556D89593
  4 BIOS boot                      21686148-6449-6E6F-744E-656564454649
  5 Sony boot partition            F4019732-066E-4E12-8273-346C5641494F
  6 Lenovo boot partition          BFBFAFE7-A34F-448A-9A5B-6213EB736C22
  7 PowerPC PReP boot              9E1A2D38-C612-4316-AA26-8B49521E5A8B
  8 ONIE boot                      7412F7D5-A156-4B13-81DC-867174929325
  9 ONIE config                    D4E6E2CD-4469-46F3-B5CB-1BFF57AFC149
 10 Microsoft reserved             E3C9E316-0B5C-4DB8-817D-F92DF00215AE
 11 Microsoft basic data           EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
 12 Microsoft LDM metadata         5808C8AA-7E8F-42E0-85D2-E1E90434CFB3
 13 Microsoft LDM data             AF9B60A0-1431-4F62-BC68-3311714A69AD
 14 Windows recovery environment   DE94BBA4-06D1-4D40-A16A-BFD50179D6AC
 15 IBM General Parallel Fs        37AFFC90-EF7D-4E96-91C3-2D7AE055B174
 16 Microsoft Storage Spaces       E75CAF8F-F680-4CEE-AFA3-B001E56EFC2D
 17 HP-UX data                     75894C1E-3AEB-11D3-B7C1-7B03A0000000
 18 HP-UX service                  E2A1E728-32E3-11D6-A682-7B03A0000000
 19 Linux swap                     0657FD6D-A4AB-43C4-84E5-0933C84B4F4F
 20 Linux filesystem               0FC63DAF-8483-4772-8E79-3D69D8477DE4
 21 Linux server data              3B8F8425-20E0-4F3B-907F-1A25A76F98E8
 22 Linux root (x86)               44479540-F297-41B2-9AF7-D131D5F0458A
 23 Linux root (x86-64)            4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709
 24 Linux root (Alpha)             6523F8AE-3EB1-4E2A-A05A-18B695AE656F
 25 Linux root (ARC)               D27F46ED-2919-4CB8-BD25-9531F3C16534
 26 Linux root (ARM)               69DAD710-2CE4-4E3C-B16C-21A1D49ABED3
 27 Linux root (ARM-64)            B921B045-1DF0-41C3-AF44-4C6F280D3FAE
 28 Linux root (IA-64)             993D8D3D-F80E-4225-855A-9DAF8ED7EA97
 29 Linux root (LoongArch-64)      77055800-792C-4F94-B39A-98C91B762BB6
 30 Linux root (MIPS-32 LE)        37C58C8A-D913-4156-A25F-48B1B64E07F0
 31 Linux root (MIPS-64 LE)        700BDA43-7A34-4507-B179-EEB93D7A7CA3
 32 Linux root (HPPA/PARISC)       1AACDB3B-5444-4138-BD9E-E5C2239B2346
 33 Linux root (PPC)               1DE3F1EF-FA98-47B5-8DCD-4A860A654D78
 34 Linux root (PPC64)             912ADE1D-A839-4913-8964-A10EEE08FBD2
 35 Linux root (PPC64LE)           C31C45E6-3F39-412E-80FB-4809C4980599
 36 Linux root (RISC-V-32)         60D5A7FE-8E7D-435C-B714-3DD8162144E1
 37 Linux root (RISC-V-64)         72EC70A6-CF74-40E6-BD49-4BDA08E8F224
 38 Linux root (S390)              08A7ACEA-624C-4A20-91E8-6E0FA67D23F9
 39 Linux root (S390X)             5EEAD9A9-FE09-4A1E-A1D7-520D00531306
 40 Linux root (TILE-Gx)           C50CDD70-3862-4CC3-90E1-809A8C93EE2C
 41 Linux reserved                 8DA63339-0007-60C0-C436-083AC8230908
 42 Linux home                     933AC7E1-2EB4-4F13-B844-0E14E2AEF915
 43 Linux RAID                     A19D880F-05FC-4D3B-A006-743F0F84911E
 44 Linux LVM                      E6D6D379-F507-44C2-A23C-238F2A3DF928
 45 Linux variable data            4D21B016-B534-45C2-A9FB-5C16E091FD2D
 46 Linux temporary data           7EC6F557-3BC5-4ACA-B293-16EF5DF639D1
 47 Linux /usr (x86)               75250D76-8CC6-458E-BD66-BD47CC81A812
 48 Linux /usr (x86-64)            8484680C-9521-48C6-9C11-B0720656F69E
Partition type or alias (type L to list all): 
Partition type or alias (type L to list all): 19

Changed type of partition 'Linux filesystem' to 'Linux swap'.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux root verity (S390)
/dev/sdb5  9439232 9644031  204800  100M Linux swap

Command (m for help): t
Partition number (1-5, default 5): 4
Partition type or alias (type L to list all): 20

Changed type of partition 'Linux root verity (S390)' to 'Linux filesystem'.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux swap

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
[root@aadar ~]# 
[root@aadar ~]# partprobe /dev/sdb
[root@aadar ~]#
[root@aadar ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux swap
[root@aadar ~]# 

[aadarsha@aadar ~]$ date
Thu Jul 16 06:43:46 PM +0545 2026
[aadarsha@aadar ~]$ who
aadarsha tty1         2026-07-16 18:42
aadarsha pts/0        2026-07-16 18:42 (192.168.254.32)
[aadarsha@aadar ~]$ whoami
aadarsha

[aadarsha@aadar ~]$ su - root
Password: 
Last login: Thu Jul 16 18:43:34 +0545 2026 on pts/0

 	# 1. Create a Disk Partition
 	# 2. Format the Disk Partition ( i.e. Create Filesystem on the Disk Partition )
 	# Filesystem type: XFS/EXT4/EXT3/EXT2/VFAT
 	
		 # mkfs -t <filesystem type> <disk partition>

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part 
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part 
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

[root@aadar ~]# blkid /dev/sdb
/dev/sdb: PTUUID="2fe270ff-d4fc-4b46-a896-2424a80e0620" PTTYPE="gpt"
[root@aadar ~]# 
[root@aadar ~]# blkid /dev/sda
/dev/sda: PTUUID="9214a82b-d2b3-419c-b8d1-27cf23f9b767" PTTYPE="gpt"
[root@aadar ~]# 
[root@aadar ~]# blkid /dev/sdb1
/dev/sdb1: PARTLABEL="Linux filesystem" PARTUUID="bfd008be-fc0f-4bc0-bd61-a497dc46ed5e"
[root@aadar ~]# 
[root@aadar ~]# blkid /dev/sda1
/dev/sda1: PARTUUID="fa7ebc34-2e7f-48d5-aa3f-8d5d0825cb72"
[root@aadar ~]# 

 # No any formatting above , ---> TYPE = ?

[root@aadar ~]# mkfs -t xfs /dev/sdb1 
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
[root@aadar ~]# 

[root@aadar ~]# blkid /dev/sdb1
/dev/sdb1: UUID="977afb9b-0c71-4240-a672-fc5ee204f457" BLOCK_SIZE="512" TYPE="xfs" PARTLABEL="Linux filesystem" PARTUUID="bfd008be-fc0f-4bc0-bd61-a497dc46ed5e"
[root@aadar ~]# 

[root@aadar ~]# blkid /dev/sdb2
/dev/sdb2: PARTLABEL="Linux filesystem" PARTUUID="f98c8d11-a947-4283-87c0-5f739546e089"
[root@aadar ~]# 
[root@aadar ~]# mkfs -t xfs /dev/sdb2
meta-data=/dev/sdb2              isize=512    agcount=4, agsize=65536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=262144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
[root@aadar ~]# 
[root@aadar ~]# blkid /dev/sdb2
/dev/sdb2: UUID="7cb5523f-0002-4f27-909b-b9d49e6cc983" BLOCK_SIZE="512" TYPE="xfs" PARTLABEL="Linux filesystem" PARTUUID="f98c8d11-a947-4283-87c0-5f739546e089"
[root@aadar ~]# 
[root@aadar ~]# mkfs -t ext3 /dev/sdb2
mke2fs 1.47.1 (20-May-2024)
/dev/sdb2 contains a xfs file system
Proceed anyway? (y,N) y
Creating filesystem with 262144 4k blocks and 65536 inodes
Filesystem UUID: e1795149-365b-4b72-ad2b-01a1229e5c29
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[root@aadar ~]# 
[root@aadar ~]# blkid /dev/sdb2
/dev/sdb2: UUID="e1795149-365b-4b72-ad2b-01a1229e5c29" SEC_TYPE="ext2" BLOCK_SIZE="4096" TYPE="ext3" PARTLABEL="Linux filesystem" PARTUUID="f98c8d11-a947-4283-87c0-5f739546e089"
[root@aadar ~]# 
[root@aadar ~]# 

[root@aadar ~]# dnf install dosfstools

[root@aadar ~]# mkfs -t vfat /dev/sdb3
mkfs.fat 4.2 (2021-01-31)
[root@aadar ~]# blkid /dev/sdb3
/dev/sdb3: UUID="4B4E-63E3" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="Microsoft basic data" PARTUUID="d3b3ac8d-f5c2-47d2-adee-9abfff0bc35b"
[root@aadar ~]# 

[root@aadar ~]# 
[root@aadar ~]# cd /dev/sdb1
-bash: cd: /dev/sdb1: Not a directory
[root@aadar ~]# 

 # 3. Mount the Disk Partition on a Directory

[root@aadar ~]# ls
anaconda-ks.cfg

[root@aadar ~]# mkdir /app1data
[root@aadar ~]# mkdir /app2data
[root@aadar ~]# mkdir /windata
[root@aadar ~]# ls
anaconda-ks.cfg
[root@aadar ~]# ls /home/
aadarsha
[root@aadar ~]# ls
anaconda-ks.cfg
[root@aadar ~]# ls /
afs       app2data  boot  etc   lib    media  opt   root  sbin  sys  usr  windata
app1data  bin       dev   home  lib64  mnt    proc  run   srv   tmp  var
[root@aadar ~]# 

[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            960M  486M  475M  51% /boot
/dev/mapper/cs-var   5.0G  269M  4.7G   6% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part 
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part 
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

 # 3. Mount the Disk Partition on a Directory

	 # I) Case-I: Temporary Mount

         # mount <partition name/UUID> <mount point - directory>

[root@aadar ~]# mount /dev/sdb1 /app1data

[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            960M  486M  475M  51% /boot
/dev/mapper/cs-var   5.0G  269M  4.7G   6% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
/dev/sdb1            2.0G   71M  1.9G   4% /app1data
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part 
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 
 
[root@aadar ~]# mount /dev/sdb2 /app2data
[root@aadar ~]# 
[root@aadar ~]# mount /dev/sdb3 /windata
[root@aadar ~]# 

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb2        8:18   0    1G  0 part /app2data
├─sdb3        8:19   0  512M  0 part /windata
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

[root@aadar ~]# cd /app1data
[root@aadar app1data]# 
[root@aadar app1data]# pwd
/app1data
 
[root@aadar app1data]# ls
[root@aadar app1data]# vi appfile1
[root@aadar app1data]# 
[root@aadar app1data]# ls
appfile1

[root@aadar app1data]# cd /app2data/
[root@aadar app2data]# pwd
/app2data
[root@aadar app2data]# ls
lost+found

 # lost+found ---> for ext1, ext2, ext3, ext4 to restored recovered files

[root@aadar app2data]# vi app2file
[root@aadar app2data]# 
[root@aadar app2data]# ls
app2file  lost+found
[root@aadar app2data]# 
[root@aadar app2data]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            960M  486M  475M  51% /boot
/dev/mapper/cs-var   5.0G  269M  4.7G   6% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
/dev/sdb1            2.0G   71M  1.9G   4% /app1data
/dev/sdb2            975M   64K  924M   1% /app2data
/dev/sdb3            511M  4.0K  511M   1% /windata
[root@aadar app2data]# 

[root@aadar app2data]# cp /etc/*.conf .
[root@aadar app2data]# ls
app2file     kdump.conf     locale.conf     mke2fs.conf       rsyslog.conf    sysctl.conf
chrony.conf  krb5.conf      logrotate.conf  nsswitch.conf     sestatus.conf   vconsole.conf
dracut.conf  ld.so.conf     lost+found      request-key.conf  sudo.conf       xattr.conf
host.conf    libaudit.conf  man_db.conf     resolv.conf       sudo-ldap.conf  yum.conf
[root@aadar app2data]# 

  # All above is temporary mounting

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part 
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part 
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

 	# All temporary partitions are unmounted after reboot.

 	# Mount point is the excess point.

 # Case-II: Permanent Mounting

[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/mapper/cs-var   5.0G  269M  4.7G   6% /var
/dev/sda2            960M  486M  475M  51% /boot
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
[root@aadar ~]# 

[root@aadar ~]# vi /etc/fstab 
[root@aadar ~]# cat /etc/fstab 
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


  # Format:
	# <partition name/UUID>            # <mount point - directory>  <filesystem-type>  <mount option>   0 0
[root@aadar ~]# 

[root@aadar ~]# mount /dev/sdb1 /app1data/
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
[root@aadar ~]# 
[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/mapper/cs-var   5.0G  269M  4.7G   6% /var
/dev/sda2            960M  486M  475M  51% /boot
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
/dev/sdb1            2.0G   71M  1.9G   4% /app1data
[root@aadar ~]# 

 # Unmount a Disk Partition

  # umount <mount point or partition name>

[root@aadar ~]# pwd
/root
[root@aadar ~]# 
[root@aadar ~]# cd /app1data/
[root@aadar app1data]# pwd
/app1data

  # must be in same folder to unmount

[root@aadar app1data]# cd
[root@aadar ~]# 
	
	 # umount /app1data     OR    umount /dev/sdb1

[root@aadar ~]# umount /app1data
[root@aadar ~]# 
[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/mapper/cs-var   5.0G  269M  4.7G   6% /var
/dev/sda2            960M  486M  475M  51% /boot
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
[root@aadar ~]# 
 
[root@aadar ~]# ls /app1data
[root@aadar ~]# 

[root@aadar ~]# vi /etc/fstab 
[root@aadar ~]# cat /etc/fstab 
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

# Format

# <partition name/UUID>          # <mount point - directory>   <filesystem-type>    <mount option>   0 0
  /dev/sdb1                        /app1data                        xfs               defaults       0 0

[root@aadar ~]# 

[root@aadar ~]# blkid /dev/sdb2
/dev/sdb2: UUID="e1795149-365b-4b72-ad2b-01a1229e5c29" SEC_TYPE="ext2" BLOCK_SIZE="4096" TYPE="ext3" PARTLABEL="Linux filesystem" PARTUUID="f98c8d11-a947-4283-87c0-5f739546e089"
[root@aadar ~]# 
[root@aadar ~]# vi /etc/fstab 

[root@aadar ~]# mkdir /backup

[root@aadar ~]# ls /
afs       app2data  bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
app1data  backup    boot  etc  lib   media  opt  root  sbin  sys  usr  windata

[root@aadar ~]# vi /etc/fstab 

[root@aadar ~]# blkid /dev/sdb2
/dev/sdb2: UUID="e1795149-365b-4b72-ad2b-01a1229e5c29" SEC_TYPE="ext2" BLOCK_SIZE="4096" TYPE="ext3" PARTLABEL="Linux filesystem" PARTUUID="f98c8d11-a947-4283-87c0-5f739546e089"
[root@aadar ~]# 
[root@aadar ~]# vi /etc/fstab 

[root@aadar ~]# blkid /dev/sdb3
/dev/sdb3: UUID="4B4E-63E3" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="Microsoft basic data" PARTUUID="d3b3ac8d-f5c2-47d2-adee-9abfff0bc35b"
[root@aadar ~]# 
              
[root@aadar ~]# vi /etc/fstab 
[root@aadar ~]# vi /etc/fstab 

[root@aadar ~]# cat /etc/fstab 
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

# Format

# <partition name/UUID>                        # <mount point - directory>   <filesystem-type>    <mount option>   0 0
  /dev/sdb1                                      /app1data                        xfs               defaults       0 0
UUID="e1795149-365b-4b72-ad2b-01a1229e5c29"      /backup                          ext3              defaults       0 0
UUID="4B4E-63E3"                                 /windata                         vfat              defaults       0 0 
[root@aadar ~]# 

[root@aadar app1data]# # 0 --> file system check order (fsck)
[root@aadar app1data]# # 0 --> order

 # defaults --> (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)

[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/mapper/cs-var   5.0G  269M  4.7G   6% /var
/dev/sda2            960M  486M  475M  51% /boot
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
[root@aadar ~]# 

[root@aadar ~]# mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
[root@aadar ~]# 
[root@aadar ~]# systemctl daemon-reload
 
[root@aadar ~]# mount -a
[root@aadar ~]# 
[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/mapper/cs-var   5.0G  269M  4.7G   6% /var
/dev/sda2            960M  486M  475M  51% /boot
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
/dev/sdb1            2.0G   71M  1.9G   4% /app1data
/dev/sdb2            975M  168K  924M   1% /backup
/dev/sdb3            511M  4.0K  511M   1% /windata
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb2        8:18   0    1G  0 part /backup
├─sdb3        8:19   0  512M  0 part /windata
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

[root@aadar ~]# ls /backup/
app2file     kdump.conf     locale.conf     mke2fs.conf       rsyslog.conf    sysctl.conf
chrony.conf  krb5.conf      logrotate.conf  nsswitch.conf     sestatus.conf   vconsole.conf
dracut.conf  ld.so.conf     lost+found      request-key.conf  sudo.conf       xattr.conf
host.conf    libaudit.conf  man_db.conf     resolv.conf       sudo-ldap.conf  yum.conf
[root@aadar ~]# 
[root@aadar ~]# umount /backup
[
[root@aadar ~]# ls /backup/
[root@aadar ~]# 
[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/mapper/cs-var   5.0G  269M  4.7G   6% /var
/dev/sda2            960M  486M  475M  51% /boot
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
/dev/sdb1            2.0G   71M  1.9G   4% /app1data
/dev/sdb3            511M  4.0K  511M   1% /windata
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part /windata
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

 # umount to ---> prevent from to hide data, or prevent the accidental delete
 
[root@aadar ~]# mount
/dev/mapper/cs-root on / type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=850444k,nr_inodes=212611,mode=755,inode64)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,seclabel,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,seclabel,nsdelegate,memory_recursiveprot)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime,seclabel)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
configfs on /sys/kernel/config type configfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /run type tmpfs (rw,nosuid,nodev,seclabel,size=349156k,nr_inodes=819200,mode=755,inode64)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,nosuid,noexec,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=35,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=5181)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime,seclabel)
debugfs on /sys/kernel/debug type debugfs (rw,nosuid,nodev,noexec,relatime,seclabel)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,nosuid,nodev,relatime,seclabel,pagesize=2M)
tracefs on /sys/kernel/tracing type tracefs (rw,nosuid,nodev,noexec,relatime,seclabel)
tmpfs on /run/credentials/systemd-journald.service type tmpfs (ro,nosuid,nodev,noexec,relatime,nosymfollow,seclabel,size=1024k,nr_inodes=1024,mode=700,inode64,noswap)
fusectl on /sys/fs/fuse/connections type fusectl (rw,nosuid,nodev,noexec,relatime)
/dev/mapper/cs-var on /var type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
/dev/sda2 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
tmpfs on /run/credentials/getty@tty1.service type tmpfs (ro,nosuid,nodev,noexec,relatime,nosymfollow,seclabel,size=1024k,nr_inodes=1024,mode=700,inode64,noswap)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,seclabel,size=174576k,nr_inodes=43644,mode=700,uid=1000,gid=1000,inode64)
/dev/sdb1 on /app1data type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
/dev/sdb3 on /windata type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,errors=remount-ro)
[root@aadar ~]# 

[root@aadar ~]# mount | grep app1data
/dev/sdb1 on /app1data type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
[root@aadar ~]# 

[root@aadar ~]# df -h | grep /app1data
/dev/sdb1            2.0G   71M  1.9G   4% /app1data
[root@aadar ~]# 
[root@aadar ~]# df -h /app1data
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       2.0G   71M  1.9G   4% /app1data
[root@aadar ~]# 

 # defaults --> (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)

[root@aadar ~]# mount | grep app1data
/dev/sdb1 on /app1data type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
[root@aadar ~]# 

 # rw --> read write, to not to allow delete, let's change to ro --> read only

[root@aadar ~]# mount -o remount,ro,noexec /app1data
[root@aadar ~]# 
[root@aadar ~]# mount | grep app1data
/dev/sdb1 on /app1data type xfs (ro,noexec,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
[root@aadar ~]# 
[root@aadar ~]# cd /app1data/
[root@aadar app1data]# pwd
/app1data
[root@aadar app1data]# ls
appfile1
[root@aadar app1data]# rm -f appfile1 
rm: cannot remove 'appfile1': Read-only file system
[root@aadar app1data]# 

 # Because above set: ro

[root@aadar app1data]# cp /etc/passwd .
cp: cannot create regular file './passwd': Read-only file system
[root@aadar app1data]# 

 # Because above set: ro

 # Because above set: ro (read only)

[root@aadar app1data]# mount -o remount,rw /app1data/
[root@aadar app1data]# 
[root@aadar app1data]# cp /etc/passwd .
[root@aadar app1data]# ls
appfile1  passwd
[root@aadar app1data]# 
[root@aadar app1data]# mount | grep app1data
/dev/sdb1 on /app1data type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
[root@aadar app1data]# 
[root@aadar app1data]# mount -o remount,ro /app1data
[root@aadar app1data]# 
[root@aadar app1data]# mount | grep app1data
/dev/sdb1 on /app1data type xfs (ro,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
[root@aadar app1data]# 

  # Troubleshooting Related to Disk Partition

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part /windata
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

[root@aadar ~]# fdisk -l /dev/sdb2
Disk /dev/sdb2: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
[root@aadar ~]# 

[root@aadar ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux swap
[root@aadar ~]# 
[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.

Command (m for help): p

Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux swap

Command (m for help): d
Partition number (1-5, default 5): 2

Partition 2 has been deleted.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux swap

Command (m for help): w
The partition table has been altered.
Syncing disks.

[root@aadar ~]# partprobe /dev/sdb
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb3        8:19   0  512M  0 part /windata
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

  # Since, partition sdb2 is deleted, it will not be seen now, But, since this is saved in /etc/fstab 
 
[root@aadar ~]# cat /etc/fstab 
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

# Format

# <partition name/UUID>                        # <mount point - directory>   <filesystem-type>    <mount option>   0 0
  /dev/sdb1                                      /app1data                        xfs               defaults       0 0
UUID="e1795149-365b-4b72-ad2b-01a1229e5c29"      /backup                          ext3              defaults       0 0
UUID="4B4E-63E3"                                 /windata                         vfat              defaults       0 0 
[root@aadar ~]# 
 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb3        8:19   0  512M  0 part /windata
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 
 
  # But according to file: /etc/fstab there exists mount
    # Thus, in 90% cases, it creates leading to maintenance mode while rebooting the system.

[root@aadar ~]# reboot
[root@aadar ~]# Connection to 192.168.254.5 closed by remote host.
Connection to 192.168.254.5 closed.
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.5
ssh: connect to host 192.168.254.5 port 22: No route to host
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.5
ssh: connect to host 192.168.254.5 port 22: No route to host
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.5
aadarsha@192.168.254.5's password: 
Last login: Thu Jul 16 20:17:06 2026 from 192.168.254.32
[aadarsha@aadar ~]$ su - root
Password: 
Last login: Thu Jul 16 20:17:19 +0545 2026 on pts/0
[root@aadar ~]# 

aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.5
ssh: connect to host 192.168.254.5 port 22: No route to host
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.5
aadarsha@192.168.254.5's password: 
Last login: Thu Jul 16 20:17:06 2026 from 192.168.254.32
[aadarsha@aadar ~]$ su - root
Password: 
Last login: Thu Jul 16 20:17:19 +0545 2026 on pts/0
[root@aadar ~]# 

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb3        8:19   0  512M  0 part /windata
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 
[root@aadar ~]# cat /etc/fstab 
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

# Format

# <partition name/UUID>                        # <mount point - directory>   <filesystem-type>    <mount option>   0 0
  /dev/sdb1                                      /app1data                        xfs               defaults       0 0
# UUID="e1795149-365b-4b72-ad2b-01a1229e5c29"      /backup                          ext3              defaults       0 0
UUID="4B4E-63E3"                                 /windata                         vfat              defaults       0 0 
[root@aadar ~]# 

 [
     this is the one of the most important troubleshooting. So, record a practice demo video.
 ]
 
[root@aadar ~]# cat /etc/fstab 
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

# Format

# <partition name/UUID>                        # <mount point - directory>   <filesystem-type>    <mount option>   0 0
  /dev/sdb1                                      /app1data                        xfs               defaults       0 0
UUID="e1795149-365b-4b72-ad2b-01a1229e5c29"      /backup                          ext3              defaults       0 0
UUID="4B4E-63E3"                                 /windata                         vfat              defaults       0 0 
[root@aadar ~]# 
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb3        8:19   0  512M  0 part /windata
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

 # But according to file: /etc/fstab there exists mount
  # Thus, in 90% cases, it creates leading to maintenance mode while rebooting the system.
 
[root@aadar ~]# reboot
[root@aadar ~]# Connection to 192.168.254.5 closed by remote host.
Connection to 192.168.254.5 closed.
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.5
ssh: connect to host 192.168.254.5 port 22: No route to host
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.5
ssh: connect to host 192.168.254.5 port 22: No route to host
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.5
aadarsha@192.168.254.5's password: 
Last login: Thu Jul 16 20:17:06 2026 from 192.168.254.32
[aadarsha@aadar ~]$ su - root
Password: 
Last login: Thu Jul 16 20:17:19 +0545 2026 on pts/0
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb3        8:19   0  512M  0 part /windata
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 
[root@aadar ~]# cat /etc/fstab 
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

# Format

# <partition name/UUID>                        # <mount point - directory>   <filesystem-type>    <mount option>   0 0
  /dev/sdb1                                      /app1data                        xfs               defaults       0 0
# UUID="e1795149-365b-4b72-ad2b-01a1229e5c29"      /backup                          ext3              defaults       0 0
UUID="4B4E-63E3"                                 /windata                         vfat              defaults       0 0 
[root@aadar ~]# 

[root@aadar ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       389Mi       1.2Gi       4.9Mi       189Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[root@aadar ~]# 

 # Increasing Swap Space

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb3        8:19   0  512M  0 part /windata
├─sdb4        8:20   0    1G  0 part 
└─sdb5        8:21   0  100M  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

 # Steps to increase Swap Space
 
  # 1. Create a Disk partition

[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.

Command (m for help): p

Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux swap

Command (m for help): q

[root@aadar ~]# 
[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sdb1            2.0G   71M  1.9G   4% /app1data
/dev/sdb3            511M  4.0K  511M   1% /windata
/dev/sda2            960M  486M  475M  51% /boot
/dev/mapper/cs-var   5.0G  270M  4.7G   6% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
[root@aadar ~]# 
[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.

Command (m for help): p

Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem
/dev/sdb5  9439232 9644031  204800  100M Linux swap

Command (m for help): d
Partition number (1,3-5, default 5): 5

Partition 5 has been deleted.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 9439231 2097152    1G Linux filesystem

Command (m for help): d
Partition number (1,3,4, default 4): 4

Partition 4 has been deleted.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data

Command (m for help): n
Partition number (2,4-128, default 2): 
First sector (4196352-20971486, default 7342080): 4196352
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-6293503, default 6293503): 6293503

Created a new partition 2 of type 'Linux filesystem' and of size 1 GiB.
Partition #2 contains a ext3 signature.

Do you want to remove the signature? [Y]es/[N]o: Yes

The signature will be removed by a write command.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start     End Sectors  Size Type
/dev/sdb1     2048 4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352 6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504 7342079 1048576  512M Microsoft basic data

Filesystem/RAID signature on partition 2 will be wiped.

Command (m for help): n
Partition number (4-128, default 4): 
First sector (7342080-20971486, default 7342080): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (7342080-20971486, default 20969471): +2G

Created a new partition 4 of type 'Linux filesystem' and of size 2 GiB.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start      End Sectors  Size Type
/dev/sdb1     2048  4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352  6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504  7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 11536383 4194304    2G Linux filesystem

Filesystem/RAID signature on partition 2 will be wiped.

Command (m for help): t
Partition number (1-4, default 4): 4
Partition type or alias (type L to list all): l
  1 EFI System                     C12A7328-F81F-11D2-BA4B-00A0C93EC93B
  2 MBR partition scheme           024DEE41-33E7-11D3-9D69-0008C781F39F
  3 Intel Fast Flash               D3BFE2DE-3DAF-11DF-BA40-E3A556D89593
  4 BIOS boot                      21686148-6449-6E6F-744E-656564454649
  5 Sony boot partition            F4019732-066E-4E12-8273-346C5641494F
  6 Lenovo boot partition          BFBFAFE7-A34F-448A-9A5B-6213EB736C22
  7 PowerPC PReP boot              9E1A2D38-C612-4316-AA26-8B49521E5A8B
  8 ONIE boot                      7412F7D5-A156-4B13-81DC-867174929325
  9 ONIE config                    D4E6E2CD-4469-46F3-B5CB-1BFF57AFC149
 10 Microsoft reserved             E3C9E316-0B5C-4DB8-817D-F92DF00215AE
 11 Microsoft basic data           EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
 12 Microsoft LDM metadata         5808C8AA-7E8F-42E0-85D2-E1E90434CFB3
 13 Microsoft LDM data             AF9B60A0-1431-4F62-BC68-3311714A69AD
 14 Windows recovery environment   DE94BBA4-06D1-4D40-A16A-BFD50179D6AC
 15 IBM General Parallel Fs        37AFFC90-EF7D-4E96-91C3-2D7AE055B174
 16 Microsoft Storage Spaces       E75CAF8F-F680-4CEE-AFA3-B001E56EFC2D
 17 HP-UX data                     75894C1E-3AEB-11D3-B7C1-7B03A0000000
 18 HP-UX service                  E2A1E728-32E3-11D6-A682-7B03A0000000
 19 Linux swap                     0657FD6D-A4AB-43C4-84E5-0933C84B4F4F
 20 Linux filesystem               0FC63DAF-8483-4772-8E79-3D69D8477DE4
 21 Linux server data              3B8F8425-20E0-4F3B-907F-1A25A76F98E8
 22 Linux root (x86)               44479540-F297-41B2-9AF7-D131D5F0458A
 23 Linux root (x86-64)            4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709
 24 Linux root (Alpha)             6523F8AE-3EB1-4E2A-A05A-18B695AE656F
 25 Linux root (ARC)               D27F46ED-2919-4CB8-BD25-9531F3C16534
 26 Linux root (ARM)               69DAD710-2CE4-4E3C-B16C-21A1D49ABED3
 27 Linux root (ARM-64)            B921B045-1DF0-41C3-AF44-4C6F280D3FAE
 28 Linux root (IA-64)             993D8D3D-F80E-4225-855A-9DAF8ED7EA97
 29 Linux root (LoongArch-64)      77055800-792C-4F94-B39A-98C91B762BB6
 30 Linux root (MIPS-32 LE)        37C58C8A-D913-4156-A25F-48B1B64E07F0
 31 Linux root (MIPS-64 LE)        700BDA43-7A34-4507-B179-EEB93D7A7CA3
 32 Linux root (HPPA/PARISC)       1AACDB3B-5444-4138-BD9E-E5C2239B2346
 33 Linux root (PPC)               1DE3F1EF-FA98-47B5-8DCD-4A860A654D78
 34 Linux root (PPC64)             912ADE1D-A839-4913-8964-A10EEE08FBD2
 35 Linux root (PPC64LE)           C31C45E6-3F39-412E-80FB-4809C4980599
 36 Linux root (RISC-V-32)         60D5A7FE-8E7D-435C-B714-3DD8162144E1
 37 Linux root (RISC-V-64)         72EC70A6-CF74-40E6-BD49-4BDA08E8F224
 38 Linux root (S390)              08A7ACEA-624C-4A20-91E8-6E0FA67D23F9
 39 Linux root (S390X)             5EEAD9A9-FE09-4A1E-A1D7-520D00531306
 40 Linux root (TILE-Gx)           C50CDD70-3862-4CC3-90E1-809A8C93EE2C
 41 Linux reserved                 8DA63339-0007-60C0-C436-083AC8230908
 42 Linux home                     933AC7E1-2EB4-4F13-B844-0E14E2AEF915
 43 Linux RAID                     A19D880F-05FC-4D3B-A006-743F0F84911E
 44 Linux LVM                      E6D6D379-F507-44C2-A23C-238F2A3DF928
 45 Linux variable data            4D21B016-B534-45C2-A9FB-5C16E091FD2D
 46 Linux temporary data           7EC6F557-3BC5-4ACA-B293-16EF5DF639D1
 47 Linux /usr (x86)               75250D76-8CC6-458E-BD66-BD47CC81A812
 48 Linux /usr (x86-64)            8484680C-9521-48C6-9C11-B0720656F69E
Partition type or alias (type L to list all): 19

Changed type of partition 'Linux filesystem' to 'Linux swap'.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start      End Sectors  Size Type
/dev/sdb1     2048  4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352  6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504  7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 11536383 4194304    2G Linux swap

Filesystem/RAID signature on partition 2 will be wiped.

Command (m for help): w
The partition table has been altered.
Syncing disks.

[root@aadar ~]# 
[root@aadar ~]# partprobe /dev/sdb
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part /windata
└─sdb4        8:20   0    2G  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

  # 2. Create swap signature (i.e. swap filesystem)

[root@aadar ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       357Mi       1.3Gi       4.9Mi       198Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[root@aadar ~]# 

[root@aadar ~]# mkswap /dev/sdb4
mkswap: /dev/sdb4: warning: don't erase bootbits sectors
        (dos partition table detected). Use -f to force.
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=2ad59f2f-1a7a-4383-9cd3-06f7b21721c3
[root@aadar ~]# 
[root@aadar ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       356Mi       1.3Gi       4.9Mi       198Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[root@aadar ~]# 

 # 3. Activate swap
 
[root@aadar ~]# blkid /dev/sdb4
/dev/sdb4: UUID="2ad59f2f-1a7a-4383-9cd3-06f7b21721c3" TYPE="swap" PTTYPE="dos" PARTUUID="09852c8e-a29d-4cbb-9b62-98c0c55721a1"
[root@aadar ~]# 
[root@aadar ~]# 
[root@aadar ~]# swapon /dev/sdb4            # Temporary Activation
[root@aadar ~]# 
[root@aadar ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       354Mi       1.3Gi       4.9Mi       198Mi       1.3Gi
Swap:          4.0Gi          0B       4.0Gi
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part /windata
└─sdb4        8:20   0    2G  0 part [SWAP]
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

[root@aadar ~]# swapon /dev/sdb4            # Temporary Activation
[root@aadar ~]# 
[root@aadar ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       354Mi       1.3Gi       4.9Mi       198Mi       1.3Gi
Swap:          4.0Gi          0B       4.0Gi
[root@aadar ~]# 

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part /windata
└─sdb4        8:20   0    2G  0 part [SWAP]
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

[root@aadar ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       353Mi       1.3Gi       4.9Mi       198Mi       1.3Gi
Swap:          4.0Gi          0B       4.0Gi
[root@aadar ~]# 

  # To permanently activate SWAP

[root@aadar ~]# blkid /dev/sdb4
/dev/sdb4: UUID="2ad59f2f-1a7a-4383-9cd3-06f7b21721c3" TYPE="swap" PTTYPE="dos" PARTUUID="09852c8e-a29d-4cbb-9b62-98c0c55721a1"
[root@aadar ~]# 
[root@aadar ~]# vi /etc/fstab         # Permanent activation ( boot-time activation )
[root@aadar ~]# cat /etc/fstab 
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

# Format

# <partition name/UUID>                        # <mount point - directory>   <filesystem-type>    <mount option>   0 0
  /dev/sdb1                                      /app1data                        xfs               defaults       0 0
UUID="4B4E-63E3"                                 /windata                         vfat              defaults       0 0 
UUID="2ad59f2f-1a7a-4383-9cd3-06f7b21721c3"      none                             swap              defaults       0 0
[root@aadar ~]# 

[root@aadar ~]# swapoff /dev/sdb4
[root@aadar ~]# 
[root@aadar ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       353Mi       1.3Gi       4.9Mi       200Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[root@aadar ~]# 

[root@aadar ~]# systemctl daemon-reload
[root@aadar ~]# 
[root@aadar ~]# swapon -a                     # for normal partition:  mount -a
[root@aadar ~]# 
[root@aadar ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       355Mi       1.2Gi       4.9Mi       225Mi       1.3Gi
Swap:          4.0Gi          0B       4.0Gi
[root@aadar ~]# 

  # Logical Volume (LV)
    # --> Dynamically Scalable Volume
    # --> Dynamically resizable disk partition

[aadarsha@aadar ~]$ whoami
aadarsha
[aadarsha@aadar ~]$ date
Tue Jul 21 05:14:56 AM +0545 2026
[aadarsha@aadar ~]$ 

[aadarsha@aadar ~]$ su - root
Password: 
[root@aadar ~]# 

  # Creating a Logical Volume (LV)
  # LV is a partition that is extensible unlike flat partition

  # LVM Layout

   # Disk
   #   └── Partition (PV)
   #         └── VG (Volume Group)
   #               ├── LV (root)
   #               ├── LV (swap)
   #               └── LV (var)

[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sdb3            511M  4.0K  511M   1% /windata
/dev/sda2            960M  486M  475M  51% /boot
/dev/sdb1            2.0G   71M  1.9G   4% /app1data
/dev/mapper/cs-var   5.0G  270M  4.7G   6% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
[root@aadar ~]# 

  # Logical Volume --->  /dev/mapper ... 

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part /windata
└─sdb4        8:20   0    2G  0 part [SWAP]
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

  # TYPE  --->  lvm 
 
  # Freeing partitioned disk:

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part /app1data
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part /windata
└─sdb4        8:20   0    2G  0 part [SWAP]
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

[root@aadar ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   17G  1.6G   16G  10% /
devtmpfs             831M     0  831M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sdb3            511M  4.0K  511M   1% /windata
/dev/sda2            960M  486M  475M  51% /boot
/dev/sdb1            2.0G   71M  1.9G   4% /app1data
/dev/mapper/cs-var   5.0G  270M  4.7G   6% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
[root@aadar ~]# 

[root@aadar ~]# umount /app1data
[root@aadar ~]# 
[root@aadar ~]# umount /windata
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part 
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part 
└─sdb4        8:20   0    2G  0 part [SWAP]
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 
[root@aadar ~]# 

  # swapoff /dev/sdb4

[root@aadar ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       362Mi       1.3Gi       4.9Mi       189Mi       1.3Gi
Swap:          4.0Gi          0B       4.0Gi
[root@aadar ~]# 
[root@aadar ~]# swapoff /dev/sdb4
[root@aadar ~]# 
[root@aadar ~]# free -h
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       362Mi       1.3Gi       4.9Mi       189Mi       1.3Gi
Swap:          2.0Gi          0B       2.0Gi
[root@aadar ~]# 
[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
├─sdb1        8:17   0    2G  0 part 
├─sdb2        8:18   0    1G  0 part 
├─sdb3        8:19   0  512M  0 part 
└─sdb4        8:20   0    2G  0 part 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 
 
[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Device       Start      End Sectors  Size Type
/dev/sdb1     2048  4196351 4194304    2G Linux filesystem
/dev/sdb2  4196352  6293503 2097152    1G Linux filesystem
/dev/sdb3  6293504  7342079 1048576  512M Microsoft basic data
/dev/sdb4  7342080 11536383 4194304    2G Linux swap

Command (m for help): d
Partition number (1-4, default 4): 1

Partition 1 has been deleted.

Command (m for help): d
Partition number (2-4, default 4): 2

Partition 2 has been deleted.

Command (m for help): d
Partition number (3,4, default 4): 3

Partition 3 has been deleted.

Command (m for help): d
Selected partition 4
Partition 4 has been deleted.

Command (m for help): 4
4: unknown command

Command (m for help): d
No partition is defined yet!

Command (m for help): p

Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@aadar ~]# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620

Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@aadar ~]# 
[root@aadar ~]# fdisk -l /dev/sdb
Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FE270FF-D4FC-4B46-A896-2424A80E0620
[root@aadar ~]#

[root@aadar ~]# wipefs --all /dev/sdb                       #  RISKY: completely wipe out or clean the disk ---> Never do in production server
/dev/sdb: 8 bytes were erased at offset 0x00000200 (gpt): 45 46 49 20 50 41 52 54
/dev/sdb: 8 bytes were erased at offset 0x27ffffe00 (gpt): 45 46 49 20 50 41 52 54
/dev/sdb: 2 bytes were erased at offset 0x000001fe (PMBR): 55 aa
/dev/sdb: calling ioctl to re-read partition table: Success
[root@aadar ~]# 

[root@aadar ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
sr0          11:0    1 1024M  0 rom  
[root@aadar ~]# 

  # Adding a 10G disk in Oracle VirtualBox from terminal

aadarkdk@pop-os:~$ VBoxManage --version
7.2.8r173730
aadarkdk@pop-os:~$ VBoxManage list vms
"server" {45306028-d5a9-4bde-a5ca-a7526d7a6ece}
"labserver" {ab109ee1-0d1f-4265-b038-2d60bcc14ce4}
aadarkdk@pop-os:~$ 
aadarkdk@pop-os:~$ VBoxManage list runningvms
aadarkdk@pop-os:~$ 
aadarkdk@pop-os:~$ VBoxManage showvminfo "labserver"
Name:                        labserver
...    
VM process priority:         default
Storage Controllers:
#0: 'IDE', Type: PIIX4, Instance: 0, Ports: 2 (max 2), Bootable
  Port 0, Unit 0: Empty
#1: 'SATA', Type: IntelAhci, Instance: 0, Ports: 1 (max 30), Bootable
  Port 0, Unit 0: UUID: 34ff0271-e9fd-464e-9e84-5f715827d250
    Location: "/home/aadarkdk/VirtualBox VMs/labserver/labserver.vdi"
NIC 1:                       MAC: 080027C76CCB, Attachment: Bridged Interface 'wlp4s0', Cable connected: on, Trace: off (file: none), Type: 82540EM, Reported speed: 0 Mbps, Boot priority: 0, Promisc Policy: deny, Bandwidth group: none
NIC 2:                       disabled
NIC 3:                       disabled
...
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ mkdir -p "$HOME/VirtualBox VMs/labserver"
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ VBoxManage storagectl "labserver" \
  --name "SATA" \
  --portcount 2
aadarkdk@pop-os:~$ 
aadarkdk@pop-os:~$ VBoxManage storageattach "labserver" \
  --storagectl "SATA" \
  --port 1 \
  --device 0 \
  --type hdd \
  --medium "$HOME/VirtualBox VMs/labserver/extra10g.vdi"
aadarkdk@pop-os:~$ 
aadarkdk@pop-os:~$ VBoxManage showvminfo "labserver"
Name:                        labserver
...
Storage Controllers:
#0: 'IDE', Type: PIIX4, Instance: 0, Ports: 2 (max 2), Bootable
  Port 0, Unit 0: Empty
#1: 'SATA', Type: IntelAhci, Instance: 0, Ports: 2 (max 30), Bootable
  Port 0, Unit 0: UUID: 34ff0371-e9fd-494e-9e84-5f715846d250
    Location: "/home/aadarkdk/VirtualBox VMs/labserver/labserver.vdi"
  Port 1, Unit 0: UUID: eb54ffe5-1b33-4f34-b117-3db7422925b1
    Location: "/home/aadarkdk/VirtualBox VMs/labserver/extra10g.vdi"
NIC 1:                       MAC: 080027C86CCB, Attachment: Bridged Interface 'wlp4s0', Cable connected: on, Trace: off (file: none), Type: 82540EM, Reported speed: 0 Mbps, Boot priority: 0, Promisc Policy: deny, Bandwidth group: none
NIC 2:                       disabled
NIC 3:                       disabled
...
aadarkdk@pop-os:~$ 

  # Login and Verify

[root@labserver ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
sr0          11:0    1 1024M  0 rom  
[root@labserver ~]# 
[root@labserver ~]# fdisk -l
Disk /dev/sda: 25.08 GiB, 26926350336 bytes, 52590528 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9214A82B-D2B3-419C-B8D1-27CF23F9B767

Device       Start      End  Sectors Size Type
/dev/sda1     2048     6143     4096   2M BIOS boot
/dev/sda2     6144  2103295  2097152   1G Linux extended boot
/dev/sda3  2103296 52439039 50335744  24G Linux LVM

Disk /dev/sdb: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/cs-root: 17 GiB, 18253611008 bytes, 35651584 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/cs-swap: 2 GiB, 2147483648 bytes, 4194304 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/mapper/cs-var: 5 GiB, 5368709120 bytes, 10485760 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
[root@labserver ~]# 

	# Creating a Logical Volume (LV)

	# Steps:
		# 1. Add disk (entire disk ) or Create a disk partition

[root@labserver ~]# lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   17G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:2    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 		       # --->  created this 10G partitioned disk
sr0          11:0    1 1024M  0 rom  
[root@labserver ~]# 

		# 2. Create Physical Volume (PV) using the newly added disk (partitioned disk)

[root@labserver ~]# pvcreate /dev/sdb                  # ---> using entire sdb disk
  Physical volume "/dev/sdb" successfully created.
[root@labserver ~]# 

		# 3. Create Volume Group (VG) using the newly added Physical Volume (PV)

[root@labserver ~]# vgcreate loan_vg /dev/sdb   # loan_vg --> <VG_NAME> (can be any other appropriate names)
  Volume group "loan_vg" successfully created
[root@labserver ~]# 
[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize   VFree  
  cs        1   3   0 wz--n-  24.00g      0 
  loan_vg   1   0   0 wz--n- <10.00g <10.00g
[root@labserver ~]# 
[root@labserver ~]# vgdisplay loan_vg
  --- Volume group ---
  VG Name               loan_vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       0 / 0   
  Free  PE / Size       2559 / <10.00 GiB
  VG UUID               9sxJjU-Hx3X-eq08-hLvR-fij8-Vmp8-1IaDXW
   
[root@labserver ~]# 

  # When VG is created, internally small blocks are created and each block is called PE ( Physical Extent or Extent Size ). Default size of PE = PE Size = 4 MB
  # Larger the PE Size, the large Chunk of data can be read/write to the disk. i.e. I/O time reduces for more size of PE and performance increases
  # Suppose, we stored the file size of 1MB in a block, then remaining 3 MB is wasted.
  # But For larger PE block size, the storage wastage may occur and which increases the cost of the storage

  # Thus, tradeoffs: Performance Vs Cost
  # And Balance: Between Performance and Storage Wastage or Cost should be maintained.
 
  # PE is called Extent Size

  # We can remove existing vg and create vg with customed size

[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize   VFree  
  cs        1   3   0 wz--n-  24.00g      0 
  loan_vg   1   0   0 wz--n- <10.00g <10.00g
[root@labserver ~]#
 
[root@labserver ~]# vgremove loan_vg
  Volume group "loan_vg" successfully removed
[root@labserver ~]# 
[root@labserver ~]# vgs
  VG #PV #LV #SN Attr   VSize  VFree
  cs   1   3   0 wz--n- 24.00g    0 
[root@labserver ~]# 
[root@labserver ~]# vgcreate -s 8M loan_vg /dev/sdb          # ---> PE Block Size : 8MB
  Volume group "loan_vg" successfully created
[root@labserver ~]# 
[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  cs        1   3   0 wz--n- 24.00g    0 
  loan_vg   1   0   0 wz--n-  9.99g 9.99g
[root@labserver ~]# 
[root@labserver ~]# vgdisplay loan_vg
  --- Volume group ---
  VG Name               loan_vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               9.99 GiB
  PE Size               8.00 MiB
  Total PE              1279
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1279 / 9.99 GiB
  VG UUID               9jm5D6-1Jds-jhdU-6uX3-3qeS-3trQ-2Vc9jn
   
[root@labserver ~]# 

  # Slice of VG is created as the LV

		# 4. Create an LV of the required size in the above VG 

[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  cs        1   3   0 wz--n- 24.00g    0 
  loan_vg   1   0   0 wz--n-  9.99g 9.99g
[root@labserver ~]# 

[root@labserver ~]# lvcreate -L 4G -n education_loan_lv loan_vg
  Logical volume "education_loan_lv" created.
[root@labserver ~]# 
[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  cs        1   3   0 wz--n- 24.00g    0 
  loan_vg   1   1   0 wz--n-  9.99g 5.99g		# about ~6GB VG has free space of total ~10GB VG as 4GB LV is created above
[root@labserver ~]# 

[root@labserver ~]# lvs
  LV                VG      Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao---- 17.00g                                                    
  swap              cs      -wi-ao----  2.00g                                                    
  var               cs      -wi-ao----  5.00g                                                    
  education_loan_lv loan_vg -wi-a-----  4.00g                                                    
[root@labserver ~]# 

[root@labserver ~]# lvdisplay /dev/loan_vg/education_loan_lv
  --- Logical volume ---
  LV Path                /dev/loan_vg/education_loan_lv
  LV Name                education_loan_lv
  VG Name                loan_vg
  LV UUID                SxJywk-U2DZ-27qI-oRjp-JjOH-I4y2-56z4c3
  LV Write Access        read/write
  LV Creation host, time labserver, 2026-07-21 13:24:56 +0545
  LV Status              available
  # open                 0
  LV Size                4.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     16384
  Block device           253:3
   
[root@labserver ~]# 

[root@labserver ~]# lvcreate -L 2G -n home_loan_lv loan_vg
  Logical volume "home_loan_lv" created.
[root@labserver ~]# 
[root@labserver ~]# lvs
  LV                VG      Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao---- 17.00g                                                    
  swap              cs      -wi-ao----  2.00g                                                    
  var               cs      -wi-ao----  5.00g                                                    
  education_loan_lv loan_vg -wi-a-----  4.00g                                                    
  home_loan_lv      loan_vg -wi-a-----  2.00g                                                    
[root@labserver ~]# 
[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  cs        1   3   0 wz--n- 24.00g    0 
  loan_vg   1   2   0 wz--n-  9.99g 3.99g
[root@labserver ~]# 

[root@labserver ~]# lvdisplay /dev/loan_vg/home_loan_lv
  --- Logical volume ---
  LV Path                /dev/loan_vg/home_loan_lv
  LV Name                home_loan_lv
  VG Name                loan_vg
  LV UUID                6GLVy6-Bbcc-pyzX-FBBh-okER-2GbJ-Ui8h4F
  LV Write Access        read/write
  LV Creation host, time labserver, 2026-07-21 13:33:56 +0545
  LV Status              available
  # open                 0
  LV Size                2.00 GiB
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     16384
  Block device           253:4
   
[root@labserver ~]# 

[root@labserver ~]# vgdisplay
  --- Volume group ---
  VG Name               cs
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               24.00 GiB
  PE Size               4.00 MiB
  Total PE              6144
  Alloc PE / Size       6144 / 24.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               XgikYq-Hroz-5YCd-ljIp-D3lv-TQQc-XPBkJT
   
  --- Volume group ---
  VG Name               loan_vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               9.99 GiB
  PE Size               8.00 MiB
  Total PE              1279
  Alloc PE / Size       768 / 6.00 GiB
  Free  PE / Size       511 / 3.99 GiB
  VG UUID               9jm5D6-1Jds-jhdU-6uX3-3qeS-3trQ-2Vc9jn
   
[root@labserver ~]# 

  # Here, Free  PE / Size       511 / 3.99 GiB    ---> means 4GB vgspace is empty or 511 PE are empty and each PE Size is 8MB

  # To create the lv of size 800MB, we can use either commands:

[root@labserver ~]# # lvcreate -L 800M -n car_loan_lv loan_vg          # directly size 800M 

[root@labserver ~]# # lvcreate -l 100 -n car_loan_lv loan_vg           # number of PE blocks ---> 100*8  = 800 MB

[root@labserver ~]# lvcreate -l 100 -n car_loan_lv loan_vg
  Logical volume "car_loan_lv" created.
[root@labserver ~]# 

[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  17.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-a----- 800.00m                                                    
  education_loan_lv loan_vg -wi-a-----   4.00g                                                    
  home_loan_lv      loan_vg -wi-a-----   2.00g                                                    
[root@labserver ~]# 

		# 5. Format the Logical Volumes ( LVs ) with appropriate filesystems

[root@labserver ~]# mkfs -t xfs /dev/loan_vg/education_loan_lv
meta-data=/dev/loan_vg/education_loan_lv isize=512    agcount=4, agsize=262144 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=1048576, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
[root@labserver ~]# 

[root@labserver ~]# mkfs -t ext4 /dev/loan_vg/car_loan_lv 
mke2fs 1.47.1 (20-May-2024)
Creating filesystem with 204800 4k blocks and 51296 inodes
Filesystem UUID: 2b8ef911-b08e-4584-8251-6271423bfdf7
Superblock backups stored on blocks: 
	32768, 98304, 163840

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[root@labserver ~]# 

[root@labserver ~]# mkf
mkfifo       mkfs.cramfs  mkfs.ext2    mkfs.ext4    mkfs.xfs     
mkfs         mkfs.erofs   mkfs.ext3    mkfs.minix   
[root@labserver ~]# 

[root@labserver ~]# mkfs -t vfat /dev/loan_vg/car_loan_lv 
mkfs: failed to execute mkfs.vfat: No such file or directory
[root@labserver ~]# 
[root@labserver ~]# dnf install dosfstools
...
[root@labserver ~]# 

[root@labserver ~]# mkfs -t vfat /dev/loan_vg/car_loan_lv 
mkfs.fat 4.2 (2021-01-31)
[root@labserver ~]# 

[aadarsha@labserver ~]$ date
Wed Jul 22 01:10:18 PM +0545 2026
[aadarsha@labserver ~]$ whoami
aadarsha
[aadarsha@labserver ~]$ su - root
Password: 
Last login: Wed Jul 22 13:08:50 +0545 2026 on pts/0
[root@labserver ~]# 

[root@labserver ~]# lsblk
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                           8:0    0 25.1G  0 disk 
├─sda1                        8:1    0    2M  0 part 
├─sda2                        8:2    0    1G  0 part /boot
└─sda3                        8:3    0   24G  0 part 
  ├─cs-root                 253:0    0   17G  0 lvm  /
  ├─cs-swap                 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var                  253:5    0    5G  0 lvm  /var
sdb                           8:16   0   10G  0 disk 
├─loan_vg-education_loan_lv 253:2    0    4G  0 lvm  
├─loan_vg-home_loan_lv      253:3    0    2G  0 lvm  
└─loan_vg-car_loan_lv       253:4    0  800M  0 lvm  
sdc                           8:32   0    5G  0 disk 
sr0                          11:0    1 1024M  0 rom  
[root@labserver ~]# 

[root@labserver ~]# blkid /dev/loan_vg/education_loan_lv 
/dev/loan_vg/education_loan_lv: UUID="1bdbe1fd-7497-4f33-864b-6aca80e918b8" BLOCK_SIZE="512" TYPE="xfs"
[root@labserver ~]# 

[root@labserver ~]# blkid /dev/loan_vg/home_loan_lv 
[root@labserver ~]# 

[root@labserver ~]# mkfs.ext4 /dev/loan_vg/home_loan_lv 
mke2fs 1.47.1 (20-May-2024)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: 91161fcc-3955-4b60-b9da-74f5d6fedbb0
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

[root@labserver ~]# 

[root@labserver ~]# blkid /dev/loan_vg/home_loan_lv 
/dev/loan_vg/home_loan_lv: UUID="91161fcc-3955-4b60-b9da-74f5d6fedbb0" BLOCK_SIZE="4096" TYPE="ext4"
[root@labserver ~]# 

[root@labserver ~]# blkid /dev/loan_vg/car_loan_lv 
/dev/loan_vg/car_loan_lv: UUID="1118-0869" BLOCK_SIZE="512" TYPE="vfat"
[root@labserver ~]# 

[root@labserver ~]# lsblk -f
NAME                        FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                                          
├─sda1                                                                                                       
├─sda2                      xfs                        9f8e3f3e-f509-45f6-9fdf-d60e45d81cf2    574.8M    40% /boot
└─sda3                      LVM2_member LVM2 001       yTRxQ4-u1PY-BoMa-HJ3T-ONbT-fcId-z5lEoL                
  ├─cs-root                 xfs                        b36b9ac1-df90-470c-92a9-ac5acaafacea     15.5G     9% /
  ├─cs-swap                 swap        1              7d4ec80b-c095-487b-8ef7-d984232ca220                  [SWAP]
  └─cs-var                  xfs                        dda09a6b-dc33-4229-b517-b49f47416287      4.8G     4% /var
sdb                         LVM2_member LVM2 001       fZcef2-fDJa-ObCO-zrBj-D5FW-uSYg-NAvPSs                
├─loan_vg-education_loan_lv xfs                        1bdbe1fd-7497-4f33-864b-6aca80e918b8                  
├─loan_vg-home_loan_lv      ext4        1.0            91161fcc-3955-4b60-b9da-74f5d6fedbb0                  
└─loan_vg-car_loan_lv       vfat        FAT32          1118-0869                                             
sdc                                                                                                          
sr0                                                                                                          
[root@labserver ~]# 

		# 6. Mount the formatted Logical Volumes (LVs)

[root@labserver ~]# mkdir -p /abcbank/eduloan
[root@labserver ~]# mkdir -p /abcbank/homeloan
[root@labserver ~]# mkdir -p /abcbank/carloan

[root@labserver ~]# ls /abcbank/
carloan  eduloan  homeloan
[root@labserver ~]# 

 # For Temporary mount  --> unmounted when the system is poweroff

[root@labserver ~]# mount /dev/loan_vg/home_loan_lv /abcbank/homeloan/
 
[root@labserver ~]# df -h
Filesystem                        Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                17G  1.5G   16G   9% /
devtmpfs                          830M     0  830M   0% /dev
tmpfs                             853M     0  853M   0% /dev/shm
tmpfs                             341M  4.9M  337M   2% /run
tmpfs                             1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                         960M  386M  575M  41% /boot
/dev/mapper/cs-var                5.0G  181M  4.8G   4% /var
tmpfs                             1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                             171M  4.0K  171M   1% /run/user/1000
/dev/mapper/loan_vg-home_loan_lv  2.0G   24K  1.8G   1% /abcbank/homeloan			# --> mounted
[root@labserver ~]# 

 
 # Permanent Mount  ---> for automatic mounting at boot
 
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

# Format

# <partition name/UUID>                <mount-point - directory>   <file-system type>   <mount - options>   0 0

/dev/loan_vg/education_loan_lv         /abcbank/eduloan               xfs                 defaults          0 0
/dev/loan_vg/home_loan_lv              /abcbank/homeloan              ext4                defaults          0 0
UUID="1118-0869"                       /abcbank/carloan               vfat                defaults          0 0
[root@labserver ~]#

[root@labserver ~]# systemctl daemon-reload

[root@labserver ~]# mount -a
 
[root@labserver ~]# df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     17G  1.5G   16G   9% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
/dev/mapper/loan_vg-home_loan_lv       2.0G   24K  1.8G   1% /abcbank/homeloan
/dev/mapper/loan_vg-education_loan_lv  4.0G  110M  3.9G   3% /abcbank/eduloan
/dev/mapper/loan_vg-car_loan_lv        799M  4.0K  799M   1% /abcbank/carloan
[root@labserver ~]# 

[root@labserver ~]# lsblk
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                           8:0    0 25.1G  0 disk 
├─sda1                        8:1    0    2M  0 part 
├─sda2                        8:2    0    1G  0 part /boot
└─sda3                        8:3    0   24G  0 part 
  ├─cs-root                 253:0    0   17G  0 lvm  /
  ├─cs-swap                 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var                  253:5    0    5G  0 lvm  /var
sdb                           8:16   0   10G  0 disk 
├─loan_vg-education_loan_lv 253:2    0    4G  0 lvm  /abcbank/eduloan
├─loan_vg-home_loan_lv      253:3    0    2G  0 lvm  /abcbank/homeloan
└─loan_vg-car_loan_lv       253:4    0  800M  0 lvm  /abcbank/carloan
sdc                           8:32   0    5G  0 disk 
sr0                          11:0    1 1024M  0 rom  
[root@labserver ~]#  

		# 7. Now, We can keep data on mounted LVs

[root@labserver ~]# ls /abcbank/
carloan  eduloan  homeloan

[root@labserver ~]# cd /abcbank/homeloan/

[root@labserver homeloan]# pwd
/abcbank/homeloan

[root@labserver homeloan]# vi homeloan_data

[root@labserver homeloan]# ls
homeloan_data  lost+found

 # To make this LV full, let's use <fallocate> command

[root@labserver homeloan]# fallocate -l 1G homeloan_bigfile1
[root@labserver homeloan]# ls
homeloan_bigfile1  homeloan_data  lost+found

[root@labserver homeloan]# df -h .
Filesystem                        Size  Used Avail Use% Mounted on
/dev/mapper/loan_vg-home_loan_lv  2.0G  1.1G  804M  57% /abcbank/homeloan

[root@labserver homeloan]# fallocate -l 800M homeloan_bigfile2
[root@labserver homeloan]# ls
homeloan_bigfile1  homeloan_bigfile2  homeloan_data  lost+found
[root@labserver homeloan]# df -h .
Filesystem                        Size  Used Avail Use% Mounted on
/dev/mapper/loan_vg-home_loan_lv  2.0G  1.8G  3.5M 100% /abcbank/homeloan
[root@labserver homeloan]#

[root@labserver homeloan]# cd ../carloan/
[root@labserver carloan]# pwd
/abcbank/carloan
[root@labserver carloan]# ls
[root@labserver carloan]# df -h .
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/loan_vg-car_loan_lv  799M  4.0K  799M   1% /abcbank/carloan
[root@labserver carloan]# 

[root@labserver carloan]# fallocate -l 600M carloan_bigfile1
[root@labserver carloan]# ls
carloan_bigfile1
[root@labserver carloan]# df -h .
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/loan_vg-car_loan_lv  799M  601M  199M  76% /abcbank/carloan
[root@labserver carloan]# 

[root@labserver carloan]# cd ../eduloan/
[root@labserver eduloan]# pwd
/abcbank/eduloan
[root@labserver eduloan]# ls
[root@labserver eduloan]# df -h .
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/loan_vg-education_loan_lv  4.0G  110M  3.9G   3% /abcbank/eduloan
[root@labserver eduloan]# 

[root@labserver eduloan]# fallocate -l 3G eduloan_bigfile1
[root@labserver eduloan]# ls
eduloan_bigfile1
[root@labserver eduloan]# df -h .
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/loan_vg-education_loan_lv  4.0G  3.2G  851M  79% /abcbank/eduloan
[root@labserver eduloan]# 

[root@labserver eduloan]# vi eduloan_data
[root@labserver eduloan]# ls
eduloan_bigfile1  eduloan_data
[root@labserver eduloan]# 

[root@labserver eduloan]# df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     17G  1.5G   16G   9% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
/dev/mapper/loan_vg-home_loan_lv       2.0G  1.8G  3.5M 100% /abcbank/homeloan
/dev/mapper/loan_vg-education_loan_lv  4.0G  3.2G  851M  79% /abcbank/eduloan
/dev/mapper/loan_vg-car_loan_lv        799M  601M  199M  76% /abcbank/carloan
[root@labserver eduloan]#

 # Expanding the storage of the system on the fly ( while system is running ) 

 		# 8. Expanding the storage of LVs

[root@labserver eduloan]# cd
[root@labserver ~]#

[root@labserver ~]# df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     17G  1.5G   16G   9% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
/dev/mapper/loan_vg-home_loan_lv       2.0G  1.8G  3.5M 100% /abcbank/homeloan
/dev/mapper/loan_vg-education_loan_lv  4.0G  3.2G  851M  79% /abcbank/eduloan
/dev/mapper/loan_vg-car_loan_lv        799M  601M  199M  76% /abcbank/carloan
[root@labserver ~]# 


[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  17.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----   4.00g                                                    
  home_loan_lv      loan_vg -wi-ao----   2.00g                                                    
[root@labserver ~]# 

[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  cs        1   3   0 wz--n- 24.00g    0 
  loan_vg   1   3   0 wz--n-  9.99g 3.21g
[root@labserver ~]# 

 # Expand size of education_loan_lv to 6G from 4G i.e +2G 

 # Existing loan_vg has the free storage size of 3.21G , So, (2G) is easily possible to expand for education_loan_lv

[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  17.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----   4.00g                                                    
  home_loan_lv      loan_vg -wi-ao----   2.00g                                                    
[root@labserver ~]# 
[root@labserver ~]# # lvextend -L +2G /dev/loan_vg/education_loan_lv 
[root@labserver ~]# # lvextend -L 6G /dev/loan_vg/education_loan_lv 
[root@labserver ~]# 
[root@labserver ~]# lvextend -L 6G /dev/loan_vg/education_loan_lv 
  Size of logical volume loan_vg/education_loan_lv changed from 4.00 GiB (512 extents) to 6.00 GiB (768 extents).
  Logical volume loan_vg/education_loan_lv successfully resized.
[root@labserver ~]# 
[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  17.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----   6.00g  							      # LSize increased to 6.00g                                                   
  home_loan_lv      loan_vg -wi-ao----   2.00g                                                    
[root@labserver ~]# 

[root@labserver ~]# df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     17G  1.5G   16G   9% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
/dev/mapper/loan_vg-home_loan_lv       2.0G  1.8G  3.5M 100% /abcbank/homeloan
/dev/mapper/loan_vg-education_loan_lv  4.0G  3.2G  851M  79% /abcbank/eduloan					# Here, Still Size: 4.0G, since the extended LV is not formatted 
/dev/mapper/loan_vg-car_loan_lv        799M  601M  199M  76% /abcbank/carloan
[root@labserver ~]# 
[root@labserver ~]#

[root@labserver ~]# blkid /dev/loan_vg/education_loan_lv 
/dev/loan_vg/education_loan_lv: UUID="1bdbe1fd-7497-4f33-864b-6aca80e918b8" BLOCK_SIZE="512" TYPE="xfs"
[root@labserver ~]# 

 # NOTE: 
 # If you format whole LV using <mkfs> --> the existing data within the LV will be lost

[root@labserver ~]# blkid /dev/loan_vg/education_loan_lv 
/dev/loan_vg/education_loan_lv: UUID="1bdbe1fd-7497-4f33-864b-6aca80e918b8" BLOCK_SIZE="512" TYPE="xfs"
[root@labserver ~]# 

[root@labserver ~]# # resize2fs /dev/loan_vg/education_loan_lv         					# for filesystem TYPE="ext_2/3/4"

[root@labserver ~]# # xfs_growfs /dev/loan_vg/education_loan_lv        					# for filesystem TYPE="xfs"

[root@labserver ~]# xfs_growfs /dev/loan_vg/education_loan_lv 
meta-data=/dev/mapper/loan_vg-education_loan_lv isize=512    agcount=4, agsize=262144 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=1048576, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
data blocks changed from 1048576 to 1572864
[root@labserver ~]# 

[root@labserver ~]# df -h 
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     17G  1.5G   16G   9% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/loan_vg-car_loan_lv        799M  601M  199M  76% /abcbank/carloan
/dev/mapper/loan_vg-education_loan_lv  6.0G  3.2G  2.8G  53% /abcbank/eduloan                                        # Here, Size: increased to 6.0G from 4.0G, after the extended LV is formatted
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
/dev/mapper/loan_vg-home_loan_lv       2.0G  1.8G  3.5M 100% /abcbank/homeloan
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
[root@labserver ~]# 

 # Data remains as it was after formatting only extended LV
 
[root@labserver ~]# ls /abcbank/eduloan/
eduloan_bigfile1  eduloan_data
[root@labserver ~]# 
[root@labserver ~]# cat /abcbank/eduloan/eduloan_data 
this file contains the education loan data ...
[root@labserver ~]# 

 # Expand size of education_loan_lv to 6G from 4G i.e +2G 

 # Existing loan_vg has the free storage size of 3.21G , So, (2G) is easily possible to expand for education_loan_lv
 
[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  17.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----   6.00g                                                    
  home_loan_lv      loan_vg -wi-ao----   2.00g                                                    
[root@labserver ~]# 

[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  cs        1   3   0 wz--n- 24.00g    0 
  loan_vg   1   3   0 wz--n-  9.99g 1.21g
[root@labserver ~]# 


 		# 9. Expanding the storage of LVs when the existing VG do not have the sufficient size

 # Increase size of LV education_loan_lv from 6.00g to 10.00g ( i.e. +4G )
 
 # But existing VG loan_vg has VFree: 1.21g ( free space) 
 
 # So, new LUN or partitioned disk should be added to increase the size of VG
 
[root@labserver ~]# lsblk 
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                           8:0    0 25.1G  0 disk 
├─sda1                        8:1    0    2M  0 part 
├─sda2                        8:2    0    1G  0 part /boot
└─sda3                        8:3    0   24G  0 part 
  ├─cs-root                 253:0    0   17G  0 lvm  /
  ├─cs-swap                 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var                  253:2    0    5G  0 lvm  /var
sdb                           8:16   0   10G  0 disk 
├─loan_vg-education_loan_lv 253:3    0    6G  0 lvm  /abcbank/eduloan
├─loan_vg-home_loan_lv      253:4    0    2G  0 lvm  /abcbank/homeloan
└─loan_vg-car_loan_lv       253:5    0  800M  0 lvm  /abcbank/carloan
sdc                           8:32   0    5G  0 disk 					# this SATA disk (LUN) --> sdc can be added to loan_vg Volume Group
sr0                          11:0    1 1024M  0 rom  
[root@labserver ~]# 

 # NOTE:
 # For the better performace, the storage device type of same type should be added, for example: sdb and sdc should have same type i.e SATA.
 # If one is nvme other should also be nvme type.
 
[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  cs        1   3   0 wz--n- 24.00g    0 
  loan_vg   1   3   0 wz--n-  9.99g 1.21g
[root@labserver ~]# 
[root@labserver ~]# vgextend loan_vg /dev/sdc         					# Here, only the certain portion of the disk can be extended
  Physical volume "/dev/sdc" successfully created.
  Volume group "loan_vg" successfully extended
[root@labserver ~]#
 
[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  cs        1   3   0 wz--n- 24.00g    0 
  loan_vg   2   3   0 wz--n- 14.98g 6.20g
[root@labserver ~]# 

[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  17.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----   6.00g                                                    
  home_loan_lv      loan_vg -wi-ao----   2.00g                                                    
[root@labserver ~]# 
[root@labserver ~]# lvextend -L 10G -r /dev/loan_vg/education_loan_lv                      # -r  --> automatically resizes and formats the extended LV same as existing LV
  File system xfs found on loan_vg/education_loan_lv mounted at /abcbank/eduloan.
  Size of logical volume loan_vg/education_loan_lv changed from 6.00 GiB (768 extents) to 10.00 GiB (1280 extents).
  Extending file system xfs to 10.00 GiB (10737418240 bytes) on loan_vg/education_loan_lv...
xfs_growfs /dev/loan_vg/education_loan_lv
meta-data=/dev/mapper/loan_vg-education_loan_lv isize=512    agcount=6, agsize=262144 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=1572864, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
data blocks changed from 1572864 to 2621440
xfs_growfs done
  Extended file system xfs on loan_vg/education_loan_lv.
  Logical volume loan_vg/education_loan_lv successfully resized.
[root@labserver ~]# 
[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  17.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----  10.00g                                                    
  home_loan_lv      loan_vg -wi-ao----   2.00g                                                    
[root@labserver ~]# 
[root@labserver ~]# df -h 
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     17G  1.5G   16G   9% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/loan_vg-car_loan_lv        799M  601M  199M  76% /abcbank/carloan
/dev/mapper/loan_vg-education_loan_lv   10G  3.3G  6.8G  33% /abcbank/eduloan
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
/dev/mapper/loan_vg-home_loan_lv       2.0G  1.8G  3.5M 100% /abcbank/homeloan
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
[root@labserver ~]# 

[root@labserver ~]# lsblk 
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                           8:0    0 25.1G  0 disk 
├─sda1                        8:1    0    2M  0 part 
├─sda2                        8:2    0    1G  0 part /boot
└─sda3                        8:3    0   24G  0 part 
  ├─cs-root                 253:0    0   17G  0 lvm  /
  ├─cs-swap                 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var                  253:2    0    5G  0 lvm  /var
sdb                           8:16   0   10G  0 disk 
├─loan_vg-education_loan_lv 253:3    0   10G  0 lvm  /abcbank/eduloan
├─loan_vg-home_loan_lv      253:4    0    2G  0 lvm  /abcbank/homeloan
└─loan_vg-car_loan_lv       253:5    0  800M  0 lvm  /abcbank/carloan
sdc                           8:32   0    5G  0 disk 
└─loan_vg-education_loan_lv 253:3    0   10G  0 lvm  /abcbank/eduloan
sr0                          11:0    1 1024M  0 rom  
[root@labserver ~]# 

  # Check and Verify the existing data

[root@labserver ~]# 
[root@labserver ~]# ls /abcbank/eduloan/
eduloan_bigfile1  eduloan_data
[root@labserver ~]# 
 
 # NOTE:
 # XFS supports                    --> ( Extend only )
 # Ext (Ext2/3/4) supports both    --> ( Extend & Shrink )
 # VFAT / FAT32                    --> No standard resize support on Linux (recreate or use specialized tools)
 
[root@labserver ~]# blkid /dev/loan_vg/education_loan_lv 
/dev/loan_vg/education_loan_lv: UUID="1bdbe1fd-7497-4f33-864b-6aca80e918b8" BLOCK_SIZE="512" TYPE="xfs"
[root@labserver ~]# 
 
[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  cs        1   3   0 wz--n- 24.00g    0 
  loan_vg   2   3   0 wz--n- 14.98g 2.20g
[root@labserver ~]#
 
[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  17.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----  10.00g                                                    
  home_loan_lv      loan_vg -wi-ao----   2.00g                                                    
[root@labserver ~]# 

 # Increase the size of home_loan_lv by 2G
 
[root@labserver ~]# lvextend -L +2G /dev/loan_vg/home_loan_lv 
  Size of logical volume loan_vg/home_loan_lv changed from 2.00 GiB (256 extents) to 4.00 GiB (512 extents).
  Logical volume loan_vg/home_loan_lv successfully resized.
[root@labserver ~]# 

[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  17.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----  10.00g                                                    
  home_loan_lv      loan_vg -wi-ao----   4.00g                                                    
[root@labserver ~]# 

[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree  
  cs        1   3   0 wz--n- 24.00g      0 
  loan_vg   2   3   0 wz--n- 14.98g 208.00m
[root@labserver ~]#

[root@labserver ~]# df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     17G  1.5G   16G   9% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/loan_vg-car_loan_lv        799M  601M  199M  76% /abcbank/carloan
/dev/mapper/loan_vg-education_loan_lv   10G  3.3G  6.8G  33% /abcbank/eduloan
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
/dev/mapper/loan_vg-home_loan_lv       2.0G  1.8G  3.5M 100% /abcbank/homeloan
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
[root@labserver ~]# 

[root@labserver ~]# blkid /dev/loan_vg/home_loan_lv 
/dev/loan_vg/home_loan_lv: UUID="91161fcc-3955-4b60-b9da-74f5d6fedbb0" BLOCK_SIZE="4096" TYPE="ext4"
[root@labserver ~]# 

[root@labserver ~]# resize2fs /dev/loan_vg/home_loan_lv 						
resize2fs 1.47.1 (20-May-2024)
Filesystem at /dev/loan_vg/home_loan_lv is mounted on /abcbank/homeloan; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/loan_vg/home_loan_lv is now 1048576 (4k) blocks long.

[root@labserver ~]#

 # NOTE:
 
 #  < resize2fs /dev/loan_vg/home_loan_lv > should be manually typed on each boot, so it is BEST to use:
 
 #  < lvextend -L +2G -r /dev/loan_vg/home_loan_lv >  --> it automatically does resize2fs also

[root@labserver ~]# df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     17G  1.5G   16G   9% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/loan_vg-car_loan_lv        799M  601M  199M  76% /abcbank/carloan
/dev/mapper/loan_vg-education_loan_lv   10G  3.3G  6.8G  33% /abcbank/eduloan
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
/dev/mapper/loan_vg-home_loan_lv       3.9G  1.8G  1.9G  49% /abcbank/homeloan
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
[root@labserver ~]#

 # Check and Verify the data
 
[root@labserver ~]# ls /abcbank/homeloan/
homeloan_bigfile1  homeloan_bigfile2  homeloan_data  lost+found
[root@labserver ~]# 
[root@labserver ~]# cat /abcbank/homeloan/homeloan_data 
this file contains home loans data and information ...
[root@labserver ~]# 


 # Attatching disk on the existing VG --> cs
 
[root@labserver ~]# df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     17G  1.5G   16G   9% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/loan_vg-car_loan_lv        799M  601M  199M  76% /abcbank/carloan
/dev/mapper/loan_vg-education_loan_lv   10G  3.3G  6.8G  33% /abcbank/eduloan
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
/dev/mapper/loan_vg-home_loan_lv       3.9G  1.8G  1.9G  49% /abcbank/homeloan
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
[root@labserver ~]# 

[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree  
  cs        1   3   0 wz--n- 24.00g      0 
  loan_vg   2   3   0 wz--n- 14.98g 208.00m
[root@labserver ~]# 

[root@labserver ~]# lsblk
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                           8:0    0 25.1G  0 disk 
├─sda1                        8:1    0    2M  0 part 
├─sda2                        8:2    0    1G  0 part /boot
└─sda3                        8:3    0   24G  0 part 
  ├─cs-root                 253:0    0   17G  0 lvm  /
  ├─cs-swap                 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var                  253:2    0    5G  0 lvm  /var
sdb                           8:16   0   10G  0 disk 
├─loan_vg-education_loan_lv 253:3    0   10G  0 lvm  /abcbank/eduloan
├─loan_vg-home_loan_lv      253:4    0    4G  0 lvm  /abcbank/homeloan
└─loan_vg-car_loan_lv       253:5    0  800M  0 lvm  /abcbank/carloan
sdc                           8:32   0    5G  0 disk 
├─loan_vg-education_loan_lv 253:3    0   10G  0 lvm  /abcbank/eduloan
└─loan_vg-home_loan_lv      253:4    0    4G  0 lvm  /abcbank/homeloan
sr0                          11:0    1 1024M  0 rom  
[root@labserver ~]#
 
[root@labserver ~]# vgdisplay cs
  --- Volume group ---
  VG Name               cs
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               24.00 GiB
  PE Size               4.00 MiB
  Total PE              6144
  Alloc PE / Size       6144 / 24.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               XgikYq-Hroz-5YCd-ljIp-D3lv-TQQc-XPBkJT
   
[root@labserver ~]# 

 # First adding another 4 GB virtual disk that will appear as /dev/sdd

[root@labserver ~]# poweroff 
[root@labserver ~]# Connection to 192.168.254.2 closed by remote host.
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ VBoxManage showvminfo "labserver"
Name:                        labserver
...
Storage Controllers:
#0: 'IDE', Type: PIIX4, Instance: 0, Ports: 2 (max 2), Bootable
  Port 0, Unit 0: Empty
#1: 'SATA', Type: IntelAhci, Instance: 0, Ports: 3 (max 30), Bootable
  Port 0, Unit 0: UUID: 34ff0371-e9fd-494e-9e84-5f715826d250
    Location: "/home/aadarkdk/VirtualBox VMs/labserver/labserver.vdi"
  Port 1, Unit 0: UUID: eb54ffe5-1c33-4f34-b117-3db6422925b1
    Location: "/home/aadarkdk/VirtualBox VMs/labserver/extra10g.vdi"
  Port 2, Unit 0: UUID: 0118fcb3-e346-4220-87a7-e3147c3486ab
    Location: "/home/aadarkdk/VirtualBox VMs/labserver/extra5g.vdi"
...
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ VBoxManage createmedium disk \
  --filename "$HOME/VirtualBox VMs/labserver/extra4g.vdi" \
  --size 4096 \
  --format VDI
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Medium created. UUID: c549df15-dc3c-49e0-800a-d5b1f3e34ca3
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ VBoxManage storagectl "labserver" \
  --name "SATA" \
  --portcount 4
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ VBoxManage storageattach "labserver" \
  --storagectl "SATA" \
  --port 3 \
  --device 0 \
  --type hdd \
  --medium "$HOME/VirtualBox VMs/labserver/extra4g.vdi"
aadarkdk@pop-os:~$
 
aadarkdk@pop-os:~$ VBoxManage showvminfo "labserver"
Name:                        labserver
...
Storage Controllers:
#0: 'IDE', Type: PIIX4, Instance: 0, Ports: 2 (max 2), Bootable
  Port 0, Unit 0: Empty
#1: 'SATA', Type: IntelAhci, Instance: 0, Ports: 4 (max 30), Bootable
  Port 0, Unit 0: UUID: 34ff0371-e9fd-494e-9e84-5f715826d250
    Location: "/home/aadarkdk/VirtualBox VMs/labserver/labserver.vdi"
  Port 1, Unit 0: UUID: eb54ffe5-1c33-4f34-b117-3db6422925b1
    Location: "/home/aadarkdk/VirtualBox VMs/labserver/extra10g.vdi"
  Port 2, Unit 0: UUID: 0118fcb3-e346-4220-87a7-e3147c3486ab
    Location: "/home/aadarkdk/VirtualBox VMs/labserver/extra5g.vdi"
  Port 3, Unit 0: UUID: c549df15-dc3c-49e0-800a-d5b1f1e34ca3
    Location: "/home/aadarkdk/VirtualBox VMs/labserver/extra4g.vdi"
...
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ VBoxManage storageattach "labserver" \
  --storagectl "SATA" \
  --port 3 \
  --device 0 \
  --medium none
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ VBoxManage storageattach "labserver" \
  --storagectl "SATA" \
  --port 3 \
  --device 0 \
  --type hdd \
  --medium "$HOME/VirtualBox VMs/labserver/extra4g.vdi"
aadarkdk@pop-os:~$

aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Wed Jul 22 19:06:56 2026
[aadarsha@labserver ~]$ 
[aadarsha@labserver ~]$ whoami
aadarsha
[aadarsha@labserver ~]$ su - root
Password: 
Last login: Wed Jul 22 18:59:23 +0545 2026 on pts/0
[root@labserver ~]# 
[root@labserver ~]# lsblk
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                           8:0    0 25.1G  0 disk 
├─sda1                        8:1    0    2M  0 part 
├─sda2                        8:2    0    1G  0 part /boot
└─sda3                        8:3    0   24G  0 part 
  ├─cs-root                 253:0    0   17G  0 lvm  /
  ├─cs-swap                 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var                  253:5    0    5G  0 lvm  /var
sdb                           8:16   0   10G  0 disk 
├─loan_vg-education_loan_lv 253:2    0   10G  0 lvm  /abcbank/eduloan
├─loan_vg-home_loan_lv      253:3    0    4G  0 lvm  /abcbank/homeloan
└─loan_vg-car_loan_lv       253:4    0  800M  0 lvm  /abcbank/carloan
sdc                           8:32   0    5G  0 disk 
├─loan_vg-education_loan_lv 253:2    0   10G  0 lvm  /abcbank/eduloan
└─loan_vg-home_loan_lv      253:3    0    4G  0 lvm  /abcbank/homeloan
sdd                           8:48   0    4G  0 disk 
sr0                          11:0    1 1024M  0 rom  
[root@labserver ~]# 

[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize  VFree  
  cs        1   3   0 wz--n- 24.00g      0 
  loan_vg   2   3   0 wz--n- 14.98g 208.00m
[root@labserver ~]# 

[root@labserver ~]# vgextend cs /dev/sdd
  Physical volume "/dev/sdd" successfully created.
  Volume group "cs" successfully extended
[root@labserver ~]# 

[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize   VFree  
  cs        2   3   0 wz--n- <28.00g  <4.00g
  loan_vg   2   3   0 wz--n-  14.98g 208.00m
[root@labserver ~]# 

[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  17.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----  10.00g                                                    
  home_loan_lv      loan_vg -wi-ao----   4.00g                                                    
[root@labserver ~]# 

[root@labserver ~]# lvextend -L +3G -r /dev/cs/root 
  File system xfs found on cs/root mounted at /.
  Size of logical volume cs/root changed from 17.00 GiB (4352 extents) to 20.00 GiB (5120 extents).
  Extending file system xfs to 20.00 GiB (21474836480 bytes) on cs/root...
xfs_growfs /dev/cs/root
meta-data=/dev/mapper/cs-root    isize=512    agcount=4, agsize=1114112 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=1
         =                       exchange=0   metadir=0
data     =                       bsize=4096   blocks=4456448, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1, parent=0
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
         =                       rgcount=0    rgsize=0 extents
         =                       zoned=0      start=0 reserved=0
data blocks changed from 4456448 to 5242880
xfs_growfs done
  Extended file system xfs on cs/root.
  Logical volume cs/root successfully resized.
[root@labserver ~]# 

[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  20.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----  10.00g                                                    
  home_loan_lv      loan_vg -wi-ao----   4.00g                                                    
[root@labserver ~]# 

[root@labserver ~]# df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     20G  1.6G   19G   8% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/loan_vg-car_loan_lv        799M  601M  199M  76% /abcbank/carloan
/dev/mapper/loan_vg-home_loan_lv       3.9G  1.8G  1.9G  49% /abcbank/homeloan
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
/dev/mapper/loan_vg-education_loan_lv   10G  3.3G  6.8G  33% /abcbank/eduloan
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
[root@labserver ~]# 

 # Data was not impacted and the storage was increased on the fly ( while the system was in use )
 
[root@labserver ~]# ls /
abcbank  afs  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@labserver ~]#
 
 # NOTE:
  # Create a LV with 20 extents (PEs)
  # PE Size * no. of PEs (extents)  =  Disk Size
  # 8M * 20  = 160M

 # Create using any technique:
 
[root@labserver ~]# vgdisplay loan_vg
  --- Volume group ---
  VG Name               loan_vg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  8
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               14.98 GiB
  PE Size               8.00 MiB
  Total PE              1918
  Alloc PE / Size       1892 / 14.78 GiB
  Free  PE / Size       26 / 208.00 MiB
  VG UUID               9jm5D6-1Jds-jhdU-6uX3-3qeS-3trQ-2Vc9jn
   
[root@labserver ~]# 

[root@labserver ~]# vgs
  VG      #PV #LV #SN Attr   VSize   VFree   
  cs        2   3   0 wz--n- <28.00g 1020.00m
  loan_vg   2   3   0 wz--n-  14.98g  208.00m
[root@labserver ~]#

[root@labserver ~]# # lvcreate -L 160M -n newlv1 loan_vg 		 # -L  -->  size of the required LV
[root@labserver ~]# 

[root@labserver ~]# # lvcreate -l 20 -n newlv1 loan_vg                   # -l  -->  No. of extents
[root@labserver ~]# 

[root@labserver ~]# lvcreate -l 20 -n newlv1 loan_vg
  Logical volume "newlv1" created.
[root@labserver ~]# 

[root@labserver ~]# lvs
  LV                VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root              cs      -wi-ao----  20.00g                                                    
  swap              cs      -wi-ao----   2.00g                                                    
  var               cs      -wi-ao----   5.00g                                                    
  car_loan_lv       loan_vg -wi-ao---- 800.00m                                                    
  education_loan_lv loan_vg -wi-ao----  10.00g                                                    
  home_loan_lv      loan_vg -wi-ao----   4.00g                                                    
  newlv1            loan_vg -wi-a----- 160.00m                                                    
[root@labserver ~]# 

 # Logical Volume Clean UP
 
[root@labserver ~]# df -h
Filesystem                             Size  Used Avail Use% Mounted on
/dev/mapper/cs-root                     20G  1.6G   19G   8% /
devtmpfs                               830M     0  830M   0% /dev
tmpfs                                  853M     0  853M   0% /dev/shm
tmpfs                                  341M  4.9M  337M   2% /run
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2                              960M  386M  575M  41% /boot
/dev/mapper/loan_vg-car_loan_lv        799M  601M  199M  76% /abcbank/carloan
/dev/mapper/loan_vg-home_loan_lv       3.9G  1.8G  1.9G  49% /abcbank/homeloan
/dev/mapper/cs-var                     5.0G  181M  4.8G   4% /var
/dev/mapper/loan_vg-education_loan_lv   10G  3.3G  6.8G  33% /abcbank/eduloan
tmpfs                                  1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                                  171M  4.0K  171M   1% /run/user/1000
[root@labserver ~]# 
[root@labserver ~]# umount /abcbank/carloan/
[root@labserver ~]# umount /abcbank/homeloan/
[root@labserver ~]# umount /abcbank/eduloan/
[root@labserver ~]# 
[root@labserver ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cs-root   20G  1.6G   19G   8% /
devtmpfs             830M     0  830M   0% /dev
tmpfs                853M     0  853M   0% /dev/shm
tmpfs                341M  4.9M  337M   2% /run
tmpfs                1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            960M  386M  575M  41% /boot
/dev/mapper/cs-var   5.0G  181M  4.8G   4% /var
tmpfs                1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                171M  4.0K  171M   1% /run/user/1000
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

# Format

# <partition name/UUID>                <mount-point - directory>   <file-system type>   <mount - options>   0 0

/dev/loan_vg/education_loan_lv         /abcbank/eduloan               xfs                 defaults          0 0
/dev/loan_vg/home_loan_lv              /abcbank/homeloan              ext4                defaults          0 0
UUID="1118-0869"                       /abcbank/carloan               vfat                defaults          0 0
[root@labserver ~]# 
[root@labserver ~]# vi /etc/fstab 
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
[root@labserver ~]# lvremove /dev/loan_vg/car_loan_lv 
Do you really want to remove active logical volume loan_vg/car_loan_lv? [y/n]: y
  Logical volume "car_loan_lv" successfully removed.
[root@labserver ~]# 

[root@labserver ~]# lvremove /dev/loan_vg/home_loan_lv 
Do you really want to remove active logical volume loan_vg/home_loan_lv? [y/n]: y
  Logical volume "home_loan_lv" successfully removed.
[root@labserver ~]# 

[root@labserver ~]# lvremove /dev/loan_vg/education_loan_lv 
Do you really want to remove active logical volume loan_vg/education_loan_lv? [y/n]: y
  Logical volume "education_loan_lv" successfully removed.
[root@labserver ~]# 
[root@labserver ~]# lvs
  LV     VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   cs      -wi-ao----  20.00g                                                    
  swap   cs      -wi-ao----   2.00g                                                    
  var    cs      -wi-ao----   5.00g                                                    
  newlv1 loan_vg -wi-a----- 160.00m                                                    
[root@labserver ~]# 
[root@labserver ~]# vgremove loan_vg
Do you really want to remove volume group "loan_vg" containing 1 logical volumes? [y/n]: y
Do you really want to remove active logical volume loan_vg/newlv1? [y/n]: y
  Logical volume "newlv1" successfully removed.
  Volume group "loan_vg" successfully removed
[root@labserver ~]# 
[root@labserver ~]# lvs
  LV   VG Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root cs -wi-ao---- 20.00g                                                    
  swap cs -wi-ao----  2.00g                                                    
  var  cs -wi-ao----  5.00g                                                    
[root@labserver ~]# 
[root@labserver ~]# vgs
  VG #PV #LV #SN Attr   VSize   VFree   
  cs   2   3   0 wz--n- <28.00g 1020.00m
[root@labserver ~]# 
[root@labserver ~]# lsblk 
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 25.1G  0 disk 
├─sda1        8:1    0    2M  0 part 
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   24G  0 part 
  ├─cs-root 253:0    0   20G  0 lvm  /
  ├─cs-swap 253:1    0    2G  0 lvm  [SWAP]
  └─cs-var  253:5    0    5G  0 lvm  /var
sdb           8:16   0   10G  0 disk 
sdc           8:32   0    5G  0 disk 
sdd           8:48   0    4G  0 disk 
└─cs-root   253:0    0   20G  0 lvm  /
sr0          11:0    1 1024M  0 rom  
[root@labserver ~]# 
[root@labserver ~]# wipefs --all /dev/sdb
/dev/sdb: 8 bytes were erased at offset 0x00000218 (LVM2_member): 4c 56 4d 32 20 30 30 31
[root@labserver ~]# 
[root@labserver ~]# wipefs --all /dev/sdc
/dev/sdc: 8 bytes were erased at offset 0x00000218 (LVM2_member): 4c 56 4d 32 20 30 30 31
[root@labserver ~]#  

[root@labserver ~]# lsblk -f
NAME        FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                          
├─sda1                                                                                       
├─sda2      xfs                        9f8e3f3e-f509-45f6-9fdf-d60e45d81cf2    574.8M    40% /boot
└─sda3      LVM2_member LVM2 001       yTRxQ4-u1PY-BoMa-HJ3T-ONbT-fcId-z5lEoL                
  ├─cs-root xfs                        b36b9ac1-df90-470c-92a9-ac5acaafacea     18.4G     8% /
  ├─cs-swap swap        1              7d4ec80b-c095-487b-8ef7-d984232ca220                  [SWAP]
  └─cs-var  xfs                        dda09a6b-dc33-4229-b517-b49f47416287      4.8G     4% /var
sdb                                                                                          
sdc                                                                                          
sdd         LVM2_member LVM2 001       zOXJ2Z-t2ZN-LBCp-Ndco-r6bs-Ld0H-lV2PvB                
└─cs-root   xfs                        b36b9ac1-df90-470c-92a9-ac5acaafacea     18.4G     8% /
sr0                                                                                          
[root@labserver ~]# 

[root@labserver ~]# poweroff
[root@labserver ~]# Connection to 192.168.254.2 closed by remote host.
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$ 
 
 # NOTE:
   # It is always better practice to take the BACKUP while performing expansion and shrink of the disk.
```