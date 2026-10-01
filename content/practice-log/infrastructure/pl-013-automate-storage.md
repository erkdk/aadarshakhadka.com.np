---
title: "PL - 013 — Automating Linux Storage Management with Ansible"
date: 2026-10-01
draft: false
---

### Attatching 5GB on each testserver and prodserver vm:
A dedicated 5 GiB virtual disk is attached to each VM to provide additional storage for the disk-management exercise.
```
aadarkdk@pop-os:~$ VBoxManage controlvm testserver acpipowerbutton
aadarkdk@pop-os:~$ VBoxManage controlvm prodserver acpipowerbutton

aadarkdk@pop-os:~$ VBoxManage list runningvms
"node" {734df242-0e74-4437-a378-c829764b7d10}
"dev1" {21aa3e65-2a67-408f-b924-3475b21e70a0}
"dev2" {c4798a4a-7bd1-4289-8f1f-037924cbabf1}

aadarkdk@pop-os:~$ VBoxManage createmedium disk --filename "$HOME/VirtualBox VMs/testserver/testserver-data.vdi" --size 5120 --format VDI
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Medium created. UUID: 5fac21fc-55ec-4e17-b738-c3f863141424
aadarkdk@pop-os:~$ VBoxManage storageattach testserver --storagectl "SATA" --port 1 --device 0 --type hdd --medium "$HOME/VirtualBox VMs/testserver/testserver-data.vdi"

aadarkdk@pop-os:~$ VBoxManage createmedium disk --filename "$HOME/VirtualBox VMs/prodserver/prodserver-data.vdi" --size 5120 --format VDI
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Medium created. UUID: bf403542-e047-46f3-9a10-c2878782e9e3
aadarkdk@pop-os:~$ VBoxManage storageattach prodserver --storagectl "SATA" --port 1 --device 0 --type hdd --medium "$HOME/VirtualBox VMs/prodserver/prodserver-data.vdi"
```
----

### Verify the Newly Attached Data Disks
The newly attached disks are verified on both managed nodes using `lsblk`. The additional disk is expected to appear as `/dev/sdb` with a capacity of approximately 5 GiB.
```
[cnode@testserver ~]$ hostname
testserver
[cnode@testserver ~]$ hostname -I
192.168.254.18 
[cnode@testserver ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk 
├─sda1        8:1    0    1M  0 part 
├─sda2        8:2    0    2G  0 part /boot
└─sda3        8:3    0   18G  0 part 
  ├─cs-root 253:0    0   16G  0 lvm  /
  └─cs-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0    5G  0 disk 
sr0          11:0    1 1024M  0 rom  
[cnode@testserver ~]$ 

[cnode@prodserver ~]$ hostname
prodserver
[cnode@prodserver ~]$ hostname -I
192.168.254.19 
[cnode@prodserver ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk 
├─sda1        8:1    0    1M  0 part 
├─sda2        8:2    0    2G  0 part /boot
└─sda3        8:3    0   18G  0 part 
  ├─cs-root 253:0    0   16G  0 lvm  /
  └─cs-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0    5G  0 disk 
sr0          11:0    1 1024M  0 rom  
[cnode@prodserver ~]$ 
```
---

### Verify Ansible Connectivity and Node Configuration
Before performing storage operations, Ansible connectivity and privilege escalation are verified for all managed nodes.
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

[cnode@control-node ~]$ 
```
---

### Disk Management Workflow
The standard disk-management workflow consists of three primary stages:
- Partition the disk.
- Create a filesystem on the partition.
- Mount the filesystem at the required mount point.

### Review Ansible Module Documentation
The relevant Ansible module documentation is reviewed before implementing the playbook. The following modules are used to automate the disk-management workflow:

1. `community.general.parted` — creates and manages disk partitions.
2. `community.general.filesystem` — creates the filesystem on the partition.
3. `ansible.builtin.file` — creates the mount-point directory.
4. `ansible.posix.mount` — mounts the filesystem and manages persistent mount configuration.


```
# 1. Create a disk partition ( fdisk/gdisk )

[cnode@control-node ansible-lab]$ ansible-doc parted
...

/EXAMPLES


# 2. Create the filesystem on the disk partition ( mkfs )

[cnode@control-node ansible-lab]$ ansible-doc filesystem
...

/EXAMPLES 


# 3. Create a directory or mount point ( mkdir /backup )

[cnode@control-node ansible-lab]$ ansible-doc file
...

/EXAMPLES 


# 4. Mount the disk partition ( mount /dev/sdb1 /backup )

[cnode@control-node ansible-lab]$ ansible-doc mount
...

/EXAMPLES
```
----

### Configure the Ansible Environment
The Ansible configuration defines the inventory location, remote connection user, and privilege-escalation settings required to manage the target nodes.
```
[cnode@control-node ~]$ cd ansible-lab/
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ cat ansible.cfg 
[defaults]
inventory = ./inventory
remote_user = cnode

[privilege_escalation]
become = true
become_method = sudo
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
[cnode@control-node ansible-lab]$ vim diskmanage.yml
[cnode@control-node ansible-lab]$ cat diskmanage.yml
---
# Disk Management
- name: Configure disk partition, filesystem, and mount point
  hosts: testprod

  tasks:
    # Create a 2 GiB partition on the newly attached data disk.
    - name: Create a 2 GiB partition on the data disk
      community.general.parted:
       device: /dev/sdb
       number: 1
       state: present
       part_end: 2GiB

    # Create an XFS filesystem on the newly created partition.     
    - name: Create an XFS filesystem on the data partition
      community.general.filesystem:
       fstype: xfs
       dev: /dev/sdb1

    # Create the directory that serves as the mount point.         
    - name: Create the /backup mount point
      ansible.builtin.file:
       path: /backup
       state: directory
       mode: '0755'

    # Mount the partition and persist the configuration in /etc/fstab.     
    - name: Mount the data partition at /backup
      ansible.posix.mount:
       path: /backup
       src: /dev/sdb1
       fstype: xfs
       opts: defaults
       state: mounted
[cnode@control-node ansible-lab]$ 
```

---
### Validate, Perform a Check-Mode, Apply Playbook

```
[cnode@control-node ansible-lab]$ ls
ansible.cfg  diskmanage.yml  inventory
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check diskmanage.yml 
[WARNING]: Collection community.general does not support Ansible version 2.16.19

playbook: diskmanage.yml
[cnode@control-node ansible-lab]$ ansible-playbook --check diskmanage.yml
[WARNING]: Collection community.general does not support Ansible version 2.16.19

PLAY [Configure disk disk partition, filesystem, and mount point] *******************

TASK [Gathering Facts] **************************************************************
ok: [prodserver]
ok: [testserver]

TASK [Create a 2 GiB partition on the data disk] ************************************
changed: [testserver]
changed: [prodserver]

TASK [Create an XFS filesystem on the data partition] *******************************
fatal: [prodserver]: FAILED! => {"changed": false, "msg": "Device /dev/sdb1 not found."}
fatal: [testserver]: FAILED! => {"changed": false, "msg": "Device /dev/sdb1 not found."}

PLAY RECAP **************************************************************************
prodserver                 : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
testserver                 : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ ansible-playbook diskmanage.yml
[WARNING]: Collection community.general does not support Ansible version 2.16.19

PLAY [Configure disk disk partition, filesystem, and mount point] *******************

TASK [Gathering Facts] **************************************************************
ok: [prodserver]
ok: [testserver]

TASK [Create a 2 GiB partition on the data disk] ************************************
changed: [testserver]
changed: [prodserver]

TASK [Create an XFS filesystem on the data partition] *******************************
changed: [testserver]
changed: [prodserver]

TASK [Create the /backup mount point] ***********************************************
changed: [testserver]
changed: [prodserver]

TASK [Mount the data partition at /backup] ******************************************
changed: [testserver]
changed: [prodserver]

PLAY RECAP **************************************************************************
prodserver                 : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 
```
---

### Verify the Disk Configuration
```
[cnode@testserver ~]$ lsblk 
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk 
├─sda1        8:1    0    1M  0 part 
├─sda2        8:2    0    2G  0 part /boot
└─sda3        8:3    0   18G  0 part 
  ├─cs-root 253:0    0   16G  0 lvm  /
  └─cs-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0    5G  0 disk 
└─sdb1        8:17   0    2G  0 part /backup
sr0          11:0    1 1024M  0 rom  
[cnode@testserver ~]$ 

[cnode@prodserver ~]$ lsblk 
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk 
├─sda1        8:1    0    1M  0 part 
├─sda2        8:2    0    2G  0 part /boot
└─sda3        8:3    0   18G  0 part 
  ├─cs-root 253:0    0   16G  0 lvm  /
  └─cs-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0    5G  0 disk 
└─sdb1        8:17   0    2G  0 part /backup
sr0          11:0    1 1024M  0 rom  
[cnode@prodserver ~]$ 

[cnode@testserver ~]$ cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Mon Aug 31 13:00:07 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=c370d097-c751-4a89-b76e-8e508b4ccca8 /                       xfs     defaults        0 0
UUID=5ff5e58c-12bc-4c3b-8e43-c35b6d494838 /boot                   xfs     defaults        0 0
UUID=0e9d3aa4-acc8-4332-a58d-14fe2d85699f none                    swap    defaults        0 0
/dev/sdb1 /backup xfs defaults 0 0
[cnode@testserver ~]$ 

[cnode@prodserver ~]$ cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Mon Aug 31 13:00:07 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=c370d097-c751-4a89-b76e-8e508b4ccca8 /                       xfs     defaults        0 0
UUID=5ff5e58c-12bc-4c3b-8e43-c35b6d494838 /boot                   xfs     defaults        0 0
UUID=0e9d3aa4-acc8-4332-a58d-14fe2d85699f none                    swap    defaults        0 0
/dev/sdb1 /backup xfs defaults 0 0
[cnode@prodserver ~]$
```

---
---
---


# LVM Storage Management with Ansible

## 1. Overview

Logical Volume Manager (LVM) provides flexible storage management by adding an abstraction layer between physical storage and filesystems.

The basic LVM hierarchy is:

```text
Physical Disk
     │
     ▼
Partition
     │
     ▼
Physical Volume (PV)
     │
     ▼
Volume Group (VG)
     │
     ▼
Logical Volume (LV)
     │
     ▼
Filesystem
     │
     ▼
Mount Point
```

For this lab, the storage layout is:

```text
/dev/sdb
├── /dev/sdb1          2 GiB    existing XFS filesystem
│
└── /dev/sdb2          ~2.4 GiB
       │
       └── PV
            │
            └── VG: testvg
                 ├── LV: testlv1    1 GiB    XFS
                 │       └── /backup-lv1
                 │
                 └── LV: testlv2    400 MiB  ext4
                         └── /backup-lv2
```

The procedure is automated with Ansible and is executed against both `testserver` and `prodserver`.

### Lab Sesion:

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

[cnode@control-node ~]$ 
```

---


make CORRECTION BELOW

```


---

# 2. Lab Environment

The target servers initially contain a 5-GiB secondary disk:

```text
/dev/sdb
├── /dev/sdb1    2 GiB    mounted at /backup
└── remaining space      available for LVM
```

The operating-system disk is `/dev/sda` and must not be modified by this procedure.

Before making storage changes, verify the disks on every target:

```bash
lsblk
sudo fdisk -l
```

Example:

```text
sda     20G
├─sda1   1M
├─sda2   2G    /boot
└─sda3  18G
  ├─cs-root  16G  /
  └─cs-swap   2G  [SWAP]

sdb      5G
└─sdb1   2G    /backup
```

> **Important:** Device names such as `/dev/sdb` are environment-dependent. Always verify the target disk before applying a storage playbook.

---

# 3. Initial LVM State

Before the configuration, the existing volume group is:

```bash
sudo vgs
sudo lvs
```

Example:

```text
VG   #PV  #LV  VSize
cs     1    2  <18.00g
```

The secondary disk has no LVM configuration yet:

```text
/dev/sdb
└── /dev/sdb1
```

The objective is to create a second partition from the remaining space:

```text
/dev/sdb2
```

and use it as an LVM physical volume.

---

# 4. Required Ansible Collections

The playbook uses:

* `community.general.parted`
* `community.general.lvg`
* `community.general.lvol`
* `community.general.filesystem`
* `ansible.posix.mount`

These modules are provided by Ansible collections rather than `ansible-core`.

Check the installed collections:

```bash
ansible-galaxy collection list
```

Install or update the required collections:

```bash
ansible-galaxy collection install community.general ansible.posix
```

For production documentation, pin tested collection versions in your automation environment rather than relying on whatever happens to be installed on the control node.

---

# 5. Inventory

Example inventory:

```ini
[testprod]
testserver
prodserver
```

The playbook targets:

```yaml
hosts: testprod
```

---

# 6. Creating the LVM Storage

## 6.1 Recommended Playbook

The following version preserves the intent of the original lab while making the configuration clearer and more maintainable.

```yaml
---
- name: Configure LVM storage
  hosts: testprod
  become: true

  vars:
    lvm_disk: /dev/sdb
    lvm_partition: /dev/sdb2

    volume_group: testvg

    logical_volumes:
      - name: testlv1
        size: 1g
        filesystem: xfs
        mount_point: /backup-lv1

      - name: testlv2
        size: 400m
        filesystem: ext4
        mount_point: /backup-lv2

  tasks:

    - name: Create LVM partition using remaining disk space
      community.general.parted:
        device: "{{ lvm_disk }}"
        number: 2
        state: present
        part_start: 2500MiB
        part_end: 100%
        flags:
          - lvm

    - name: Create volume group
      community.general.lvg:
        vg: "{{ volume_group }}"
        pvs: "{{ lvm_partition }}"

    - name: Create logical volumes
      community.general.lvol:
        vg: "{{ volume_group }}"
        lv: "{{ item.name }}"
        size: "{{ item.size }}"
        state: present
      loop: "{{ logical_volumes }}"

    - name: Create filesystems
      community.general.filesystem:
        fstype: "{{ item.filesystem }}"
        dev: "/dev/{{ volume_group }}/{{ item.name }}"
      loop: "{{ logical_volumes }}"

    - name: Create mount point directories
      ansible.builtin.file:
        path: "{{ item.mount_point }}"
        state: directory
        owner: root
        group: root
        mode: "0755"
      loop: "{{ logical_volumes }}"

    - name: Mount logical volumes and persist configuration
      ansible.posix.mount:
        path: "{{ item.mount_point }}"
        src: "/dev/{{ volume_group }}/{{ item.name }}"
        fstype: "{{ item.filesystem }}"
        opts: defaults
        state: mounted
      loop: "{{ logical_volumes }}"
```

`community.general.parted` supports `part_end: 100%`, which is appropriate when the partition should consume the remaining disk space. The module also supports the LVM partition flag.

---

# 7. Why `part_end: 100%` Is Preferred Here

The original playbook used:

```yaml
part_start: 2500MiB
part_end: 5000MiB
```

This creates a partition with a fixed endpoint.

For example:

```text
/dev/sdb
0                    2500 MiB                5000 MiB
|-------------------------|------------------------|
                          └── /dev/sdb2
```

This works for the current 5-GiB disk, but it is not truly "remaining space" logic.

Using:

```yaml
part_end: 100%
```

means:

```text
/dev/sdb
0                    2500 MiB                  end
|-------------------------|----------------------|
                          └── /dev/sdb2
```

This allows the partition to consume the remaining available disk space.

---

# 8. Why `pesize: 8M` Was Removed

The original playbook contained:

```yaml
pesize: 8M
```

This is valid, but it is not required for this lab.

LVM uses physical extents to allocate space inside a volume group. Unless a specific physical-extent size is required by the storage design, allowing the module/system default keeps the playbook simpler.

The `community.general.lvg` documentation specifies a default PE size and notes that PE size is not modified for an already-existing VG.

If a production design requires a specific PE size, document the reason explicitly:

```yaml
pesize: 8M
```

Do not include it merely because it appeared in a lab exercise.

---

# 9. Logical Volumes

The playbook creates:

```text
testvg
├── testlv1    1 GiB
└── testlv2    400 MiB
```

The remaining capacity stays available in the volume group.

This is visible with:

```bash
sudo vgs
sudo lvs
```

Example:

```text
VG      #PV  #LV  VSize    VFree
testvg    1    2  <2.44g   <1.05g
```

This is an important advantage of LVM: the unused capacity can later be allocated to an existing or new logical volume.

---

# 10. Filesystems

The logical volumes are formatted as follows:

| Logical Volume |    Size | Filesystem | Mount Point   |
| -------------- | ------: | ---------- | ------------- |
| `testlv1`      |   1 GiB | XFS        | `/backup-lv1` |
| `testlv2`      | 400 MiB | ext4       | `/backup-lv2` |

The filesystem is created after the LV exists.

For example:

```yaml
community.general.filesystem:
  fstype: xfs
  dev: /dev/testvg/testlv1
```

The filesystem module creates the filesystem on the specified block device.

> **Warning:** Formatting a device destroys existing filesystem data. Never use `force: true` unless overwriting the device is intentional.

---

# 11. Persistent Mounts

The `ansible.posix.mount` task:

```yaml
state: mounted
```

both mounts the filesystem and manages the persistent `/etc/fstab` configuration. The module is specifically designed to control active and configured mount points.

The resulting configuration is similar to:

```text
/dev/mapper/testvg-testlv1 /backup-lv1 xfs  defaults 0 0
/dev/mapper/testvg-testlv2 /backup-lv2 ext4 defaults 0 0
```

For production systems, UUID-based `/etc/fstab` entries are generally preferable when the environment requires stable filesystem identification.

Verify filesystem UUIDs with:

```bash
sudo blkid
```

and verify the resulting mounts with:

```bash
findmnt
```

---

# 12. Syntax Validation

Before making changes:

```bash
ansible-playbook --syntax-check lvcreate.yml
```

A successful result should resemble:

```text
playbook: lvcreate.yml
```

A collection compatibility warning such as:

```text
Collection community.general does not support Ansible version 2.16.19
```

should not be ignored in a production-quality automation environment.

The playbook may still execute successfully, as it did in this lab, but the Ansible/collection versions should be aligned and tested before publication or production use.

---

# 13. Execute the Playbook

Run:

```bash
ansible-playbook lvcreate.yml
```

The expected result is:

```text
PLAY RECAP
testserver   : ok=10  changed=9  unreachable=0  failed=0
prodserver   : ok=10  changed=9  unreachable=0  failed=0
```

For an existing environment, first use check mode where supported:

```bash
ansible-playbook lvcreate.yml --check
```

---

# 14. Verification

After the playbook completes, verify the complete storage stack.

## 14.1 Verify partitions and mounts

```bash
lsblk -f
```

Expected structure:

```text
sdb
├─sdb1
└─sdb2
   ├─testvg-testlv1    /backup-lv1
   └─testvg-testlv2    /backup-lv2
```

## 14.2 Verify the volume group

```bash
sudo vgs
```

Expected:

```text
VG      #PV  #LV  VSize    VFree
testvg    1    2  <2.44g   <1.05g
```

## 14.3 Verify logical volumes

```bash
sudo lvs
```

Expected:

```text
LV       VG      LSize
testlv1  testvg  1.00g
testlv2  testvg  400.00m
```

## 14.4 Verify filesystems

```bash
lsblk -f
```

or:

```bash
sudo blkid
```

## 14.5 Verify mounts

```bash
findmnt /backup-lv1
findmnt /backup-lv2
```

## 14.6 Verify persistent configuration

```bash
grep -E 'backup-lv1|backup-lv2' /etc/fstab
```

---

# 15. Final Result

The final storage hierarchy is:

```text
/dev/sdb
│
├── /dev/sdb1
│     └── XFS
│          └── /backup
│
└── /dev/sdb2
      │
      └── PV
           │
           └── testvg
                │
                ├── testlv1
                │     └── XFS
                │          └── /backup-lv1
                │
                └── testlv2
                      └── ext4
                           └── /backup-lv2
```

The remaining capacity in `testvg` is available for future storage expansion.

---

# 16. Expanding the Disk and LVM Storage

Disk expansion follows this sequence:

```text
Increase virtual/physical disk
          │
          ▼
Resize partition
          │
          ▼
Resize PV
          │
          ▼
Resize VG
          │
          ▼
Extend LV
          │
          ▼
Grow filesystem
```

The exact procedure depends on where the free capacity is located.

For this lab, if `/dev/sdb` is expanded from 5 GiB to a larger size and `/dev/sdb2` is the final partition, the general process is:

```text
/dev/sdb
├── sdb1       existing 2 GiB
└── sdb2       LVM partition
               ↑
               extend to disk end
```

First inspect the disk:

```bash
lsblk
sudo parted /dev/sdb print
```

The partition can be extended with `community.general.parted` using `part_end: 100%` and `resize: true`.

Then resize the physical volume:

```bash
sudo pvresize /dev/sdb2
```

Verify:

```bash
sudo pvs
sudo vgs
```

The newly available capacity should appear as free space in the VG.

---

# 17. Extending an Existing Logical Volume

If `testvg` contains free space, extend an LV.

For example, to add 500 MiB to `testlv1`:

```bash
sudo lvextend -L +500M /dev/testvg/testlv1
```

Then grow the filesystem.

For XFS:

```bash
sudo xfs_growfs /backup-lv1
```

XFS can be grown while mounted, but it cannot be reduced.

For ext4:

```bash
sudo resize2fs /dev/testvg/testlv2
```

Alternatively, Ansible's `community.general.lvol` can resize the LV and, where supported, resize the underlying filesystem using:

```yaml
resizefs: true
```

The module supports filesystem resizing for XFS and ext2/ext3/ext4 among other supported filesystems.

---

# 18. Recommended Ansible Expansion Pattern

For an existing XFS LV:

```yaml
- name: Extend logical volume and filesystem
  community.general.lvol:
    vg: testvg
    lv: testlv1
    size: +500M
    resizefs: true
```

The `+` syntax means "increase by this amount."

Be aware that relative `+` sizing is not idempotent in the same way as specifying an absolute desired size.

For repeatable production automation, define the desired final size rather than blindly adding capacity on every execution.

---

# 19. Important Operational Rules

### Do

* Verify the target disk before modifying partitions.
* Use `lsblk`, `pvs`, `vgs`, `lvs`, and `findmnt` during validation.
* Keep backups before modifying production storage.
* Test the playbook with check mode where appropriate.
* Pin and test Ansible collection versions.
* Use descriptive variables instead of repeating device names.
* Leave unused VG capacity available when future expansion is expected.

### Do not

* Assume `/dev/sdb` is always the correct disk.
* Format an existing device without verifying it.
* use `force: true` casually.
* Reduce an XFS filesystem; XFS does not support shrinking.
* Hard-code a disk endpoint such as `5000MiB` when the actual requirement is "use all remaining space."
* Ignore Ansible collection compatibility warnings in production automation.

---

# 20. Key Takeaway

The complete LVM automation workflow is:

```text
Partition
   ↓
PV
   ↓
VG
   ↓
LV
   ↓
Filesystem
   ↓
Mount
   ↓
/etc/fstab
```

For expansion:

```text
Disk
 ↓
Partition
 ↓
PV
 ↓
VG
 ↓
LV
 ↓
Filesystem
```

Each layer must be expanded in the correct order. Increasing the disk alone does not automatically increase the filesystem capacity.

This separation of storage layers is the fundamental concept to understand when administering LVM with Ansible.
```

---

make CORRECTION ABOVE


```
[cnode@control-node ~]$ cd ansible-lab/
[cnode@control-node ansible-lab]$
[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory
[cnode@control-node ansible-lab]$ vim lvcreate.yml
[cnode@control-node ansible-lab]$ 

[cnode@control-node ansible-lab]$ ls
ansible.cfg  inventory  lvcreate.yml
[cnode@control-node ansible-lab]$ vim lvcreate.yml 

[cnode@control-node ansible-lab]$ cat lvcreate.yml 
---
# LVM Playbook
- name: Play to create LV and filesystems
  hosts: testprod
  become: true

  tasks:
    - name: Create partition 2 on the remaining space of data disk
      community.general.parted:
        device: /dev/sdb
        number: 2
        state: present
        part_start: 2500MiB
        part_end: 5000MiB

    - name: Create a volume group using /dev/sdb2
      community.general.lvg:
        vg: testvg
        pvs: /dev/sdb2
        pesize: 8M

    - name: Create a logical volume testlv1 (1 GiB)
      community.general.lvol:
        vg: testvg
        lv: testlv1
        size: 1g
     
    - name: Create a logical volume testlv2 (400 MiB)
      community.general.lvol:
        vg: testvg
        lv: testlv2
        size: 400m

    - name: Create an XFS filesystem on testlv1
      community.general.filesystem:
        fstype: xfs
        dev: /dev/mapper/testvg-testlv1

    - name: Create an ext4 filesystem on testlv2
      community.general.filesystem:
        fstype: ext4
        dev: /dev/mapper/testvg-testlv2

    - name: Create mount point directories
      ansible.builtin.file:
        path: "{{ item }}"
        state: directory
        mode: '0755'
      loop:
        - /backup-lv1
        - /backup-lv2

    - name: Mount testlv1 at /backup-lv1 and persist in fstab
      ansible.posix.mount:
        path: /backup-lv1
        src: /dev/mapper/testvg-testlv1
        fstype: xfs
        opts: defaults
        state: mounted
 
    - name: Mount testlv2 at /backup-lv2 and persist in fstab
      ansible.posix.mount:
        path: /backup-lv2
        src: /dev/mapper/testvg-testlv2
        fstype: ext4
        opts: defaults
        state: mounted
[cnode@control-node ansible-lab]$ 
```
---

```
```
[cnode@testserver ~]$ sudo vgs
  VG #PV #LV #SN Attr   VSize   VFree
  cs   1   2   0 wz--n- <18.00g    0 
[cnode@testserver ~]$ sudo lvs
  LV   VG Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root cs -wi-ao---- <16.00g                                                    
  swap cs -wi-ao----   2.00g                                                    
[cnode@testserver ~]$ cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Mon Aug 31 13:00:07 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=c370d097-c751-4a89-b76e-8e508b4ccca8 /                       xfs     defaults        0 0
UUID=5ff5e58c-12bc-4c3b-8e43-c35b6d494838 /boot                   xfs     defaults        0 0
UUID=0e9d3aa4-acc8-4332-a58d-14fe2d85699f none                    swap    defaults        0 0
/dev/sdb1 /backup xfs defaults 0 0
[cnode@testserver ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk 
├─sda1        8:1    0    1M  0 part 
├─sda2        8:2    0    2G  0 part /boot
└─sda3        8:3    0   18G  0 part 
  ├─cs-root 253:0    0   16G  0 lvm  /
  └─cs-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0    5G  0 disk 
└─sdb1        8:17   0    2G  0 part /backup
sr0          11:0    1 1024M  0 rom  
[cnode@testserver ~]$ 

[cnode@prodserver ~]$ sudo vgs
  VG #PV #LV #SN Attr   VSize   VFree
  cs   1   2   0 wz--n- <18.00g    0 
[cnode@prodserver ~]$ sudo lvs
  LV   VG Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root cs -wi-ao---- <16.00g                                                    
  swap cs -wi-ao----   2.00g                                                    
[cnode@prodserver ~]$ cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Mon Aug 31 13:00:07 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=c370d097-c751-4a89-b76e-8e508b4ccca8 /                       xfs     defaults        0 0
UUID=5ff5e58c-12bc-4c3b-8e43-c35b6d494838 /boot                   xfs     defaults        0 0
UUID=0e9d3aa4-acc8-4332-a58d-14fe2d85699f none                    swap    defaults        0 0
/dev/sdb1 /backup xfs defaults 0 0
[cnode@prodserver ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   20G  0 disk 
├─sda1        8:1    0    1M  0 part 
├─sda2        8:2    0    2G  0 part /boot
└─sda3        8:3    0   18G  0 part 
  ├─cs-root 253:0    0   16G  0 lvm  /
  └─cs-swap 253:1    0    2G  0 lvm  [SWAP]
sdb           8:16   0    5G  0 disk 
└─sdb1        8:17   0    2G  0 part /backup
sr0          11:0    1 1024M  0 rom  
[cnode@prodserver ~]$ 
```
---

```
[cnode@control-node ansible-lab]$ ansible-playbook --syntax-check lvcreate.yml 
[WARNING]: Collection community.general does not support Ansible version 2.16.19

playbook: lvcreate.yml
[cnode@control-node ansible-lab]$ ansible-playbook lvcreate.yml 
[WARNING]: Collection community.general does not support Ansible version 2.16.19

PLAY [Play to create LV and filesystems] ********************************************

TASK [Gathering Facts] **************************************************************
ok: [testserver]
ok: [prodserver]

TASK [Create partition 2 on the remaining space of data disk] ***********************
changed: [prodserver]
changed: [testserver]

TASK [Create a volume group using /dev/sdb2] ****************************************
changed: [prodserver]
changed: [testserver]

TASK [Create a logical volume testlv1 (1 GiB)] **************************************
changed: [prodserver]
changed: [testserver]

TASK [Create a logical volume testlv2 (400 MiB)] ************************************
changed: [prodserver]
changed: [testserver]

TASK [Create an XFS filesystem on testlv1] ******************************************
changed: [prodserver]
changed: [testserver]

TASK [Create an ext4 filesystem on testlv2] *****************************************
changed: [testserver]
changed: [prodserver]

TASK [Create mount point directories] ***********************************************
changed: [prodserver] => (item=/backup-lv1)
changed: [testserver] => (item=/backup-lv1)
changed: [prodserver] => (item=/backup-lv2)
changed: [testserver] => (item=/backup-lv2)

TASK [Mount testlv1 at /backup-lv1 and persist in fstab] ****************************
changed: [prodserver]
changed: [testserver]

TASK [Mount testlv2 at /backup-lv2 and persist in fstab] ****************************
changed: [testserver]
changed: [prodserver]

PLAY RECAP **************************************************************************
prodserver                 : ok=10   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
testserver                 : ok=10   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[cnode@control-node ansible-lab]$ 
```

---

### After

```
[cnode@testserver ~]$ lsblk
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                  8:0    0   20G  0 disk 
├─sda1               8:1    0    1M  0 part 
├─sda2               8:2    0    2G  0 part /boot
└─sda3               8:3    0   18G  0 part 
  ├─cs-root        253:0    0   16G  0 lvm  /
  └─cs-swap        253:1    0    2G  0 lvm  [SWAP]
sdb                  8:16   0    5G  0 disk 
├─sdb1               8:17   0    2G  0 part /backup
└─sdb2               8:18   0  2.4G  0 part 
  ├─testvg-testlv1 253:2    0    1G  0 lvm  /backup-lv1
  └─testvg-testlv2 253:3    0  400M  0 lvm  /backup-lv2
sr0                 11:0    1 1024M  0 rom  
[cnode@testserver ~]$ cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Mon Aug 31 13:00:07 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=c370d097-c751-4a89-b76e-8e508b4ccca8 /                       xfs     defaults        0 0
UUID=5ff5e58c-12bc-4c3b-8e43-c35b6d494838 /boot                   xfs     defaults        0 0
UUID=0e9d3aa4-acc8-4332-a58d-14fe2d85699f none                    swap    defaults        0 0
/dev/sdb1 /backup xfs defaults 0 0
/dev/mapper/testvg-testlv1 /backup-lv1 xfs defaults 0 0
/dev/mapper/testvg-testlv2 /backup-lv2 ext4 defaults 0 0
[cnode@testserver ~]$ sudo vgs
  VG     #PV #LV #SN Attr   VSize   VFree 
  cs       1   2   0 wz--n- <18.00g     0 
  testvg   1   2   0 wz--n-  <2.44g <1.05g
[cnode@testserver ~]$ sudo lvs
  LV      VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    cs     -wi-ao---- <16.00g                                                    
  swap    cs     -wi-ao----   2.00g                                                    
  testlv1 testvg -wi-ao----   1.00g                                                    
  testlv2 testvg -wi-ao---- 400.00m                                                    
[cnode@testserver ~]$ 

[cnode@prodserver ~]$ lsblk
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                  8:0    0   20G  0 disk 
├─sda1               8:1    0    1M  0 part 
├─sda2               8:2    0    2G  0 part /boot
└─sda3               8:3    0   18G  0 part 
  ├─cs-root        253:0    0   16G  0 lvm  /
  └─cs-swap        253:1    0    2G  0 lvm  [SWAP]
sdb                  8:16   0    5G  0 disk 
├─sdb1               8:17   0    2G  0 part /backup
└─sdb2               8:18   0  2.4G  0 part 
  ├─testvg-testlv1 253:2    0    1G  0 lvm  /backup-lv1
  └─testvg-testlv2 253:3    0  400M  0 lvm  /backup-lv2
sr0                 11:0    1 1024M  0 rom  
[cnode@prodserver ~]$ cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Mon Aug 31 13:00:07 2026
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=c370d097-c751-4a89-b76e-8e508b4ccca8 /                       xfs     defaults        0 0
UUID=5ff5e58c-12bc-4c3b-8e43-c35b6d494838 /boot                   xfs     defaults        0 0
UUID=0e9d3aa4-acc8-4332-a58d-14fe2d85699f none                    swap    defaults        0 0
/dev/sdb1 /backup xfs defaults 0 0
/dev/mapper/testvg-testlv1 /backup-lv1 xfs defaults 0 0
/dev/mapper/testvg-testlv2 /backup-lv2 ext4 defaults 0 0
[cnode@prodserver ~]$ sudo vgs
  VG     #PV #LV #SN Attr   VSize   VFree 
  cs       1   2   0 wz--n- <18.00g     0 
  testvg   1   2   0 wz--n-  <2.44g <1.05g
[cnode@prodserver ~]$ sudo lvs
  LV      VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    cs     -wi-ao---- <16.00g                                                    
  swap    cs     -wi-ao----   2.00g                                                    
  testlv1 testvg -wi-ao----   1.00g                                                    
  testlv2 testvg -wi-ao---- 400.00m                                                    
[cnode@prodserver ~]$ 
```

### Increasing disk
