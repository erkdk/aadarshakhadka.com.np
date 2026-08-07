---
title: "PL - 019 — SELinux Security Management & Policy Control"
date: 2026-08-07
draft: false
---

#### Terminal Session 
```
 # SELinux ( Security Enhanced Linux )

  # Modes of SELinux
   # - Enforcing
   # - Permission
   # - Disabled
 
| Mode           | Policy Enforced | Violations Logged | Use Case                                |                              
|----------------|-----------------|-------------------|-----------------------------------------|
|  Enforcing     | Yes             | Yes               | Normal/production systems               |                             
|  Permissive    | No              | Yes               | Testing and troubleshooting             |                              
|  Disabled      | No              | No                | SELinux is turned off (not recommended) | 

aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Thu Aug  6 05:14:23 2026 from 192.168.254.32

[aadarsha@lab ~]$ whoami
aadarsha

[aadarsha@lab ~]$ date
Thu Aug  6 05:18:47 AM +0545 2026

[aadarsha@lab ~]$ getenforce
Enforcing
 
 # Changing SELinux modes

  # Temporary

[aadarsha@lab ~]$ getenforce
Enforcing

[aadarsha@lab ~]$ setenforce 0
setenforce:  security_setenforce() failed:  Permission denied

[aadarsha@lab ~]$ su - root
Password: 
Last login: Thu Aug  6 05:14:54 +0545 2026 on pts/0

[root@lab ~]# getenforce
Enforcing

[root@lab ~]# setenforce 0

[root@lab ~]# getenforce
Permissive
 
[root@lab ~]# setenforce 1
 
[root@lab ~]# getenforce
Enforcing
 
 # Permanently
 
[root@lab ~]# cat /etc/selinux/config 
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://docs.fedoraproject.org/en-US/quick-docs/getting-started-with-selinux/#getting-started-with-selinux-selinux-states-and-modes
#
# NOTE: In earlier Fedora kernel builds, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
[root@lab ~]# 

[root@lab ~]# vi /etc/selinux/config

[root@lab ~]# cat /etc/selinux/config 
...
SELINUX=permissive
...
[root@lab ~]# 

[root@lab ~]# getenforce
Enforcing

  # After modifying SELINUX, to implement and persist the change, reboot the system (Need to do reboot to reload)
  # the policy of SELinux is applied during boot time
  
[root@lab ~]# reboot
[root@lab ~]# Connection to 192.168.254.2 closed by remote host.
Connection to 192.168.254.2 closed.
 
aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Thu Aug  6 05:18:39 2026 from 192.168.254.32

[aadarsha@lab ~]$ su - root
Password: 
Last login: Thu Aug  6 05:23:35 +0545 2026 on pts/0

[root@lab ~]# getenforce 
Permissive

[root@lab ~]# cat /.autorelabel
cat: /.autorelabel: No such file or directory
[root@lab ~]# 

[root@lab ~]# vi /etc/selinux/config 

[root@lab ~]# cat /etc/selinux/config 
...
SELINUX=permissive
...
[root@lab ~]# 

 # changing SELINUX to enforcing
 
[root@lab ~]# vi /etc/selinux/config 

[root@lab ~]# cat /etc/selinux/config 
...
#
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
[root@lab ~]# 

[root@lab ~]# getenfoce

[root@lab ~]# getenforce
Permissive

[root@lab ~]# touch /.autorelabel

[root@lab ~]# cat /.autorelabel 

[root@lab ~]# reboot
[root@lab ~]# Connection to 192.168.254.2 closed by remote host.
Connection to 192.168.254.2 closed.
aadarkdk@pop-os:~$ 

aadarkdk@pop-os:~$ ssh aadarsha@192.168.254.2
aadarsha@192.168.254.2's password: 
Last login: Thu Aug  6 05:29:27 2026 from 192.168.254.32
[aadarsha@lab ~]$ 

  # Note: while changing to enforcing mode must create /.autorelabel unlike permissive mode

[aadarsha@lab ~]$ getenforce
Enforcing
[aadarsha@lab ~]$ 

 # Security context of SELinux

[root@lab ~]# touch file1

[root@lab ~]# mkdir dir1
[root@lab ~]# 
 
[root@lab ~]# ls -Z
    system_u:object_r:admin_home_t:s0 anaconda-ks.cfg
unconfined_u:object_r:admin_home_t:s0 dir1
unconfined_u:object_r:admin_home_t:s0 file1
[root@lab ~]# 

[root@lab ~]# ls -Z file1 
unconfined_u:object_r:admin_home_t:s0 file1

 # Format of SELinux Security Context

   # User:Role:Type:Sensitivity

   # Type

   # Effects of Create, Copy & Move Operations on SELinux

     # - Create: When a file is created inside a directory then the file inherits the SELinux security context of the parent directory

     # - Copy: When a file is copied from one directory to another directory then file takes SELinux security context of the destination.

     # - Move: When a file is moved from one directory to another directory then the file still retains its original SELinux Security context

     # - Copy: When a file is copied from one directory to another directory then file takes SELinux security context of the destination directory


 # To show the effect of SELinux context, we need the service
 # Let's use httpd

[root@lab ~]# rpm -q httpd
package httpd is not installed

[root@lab ~]# yum -y install httpd
...
Installed:
  apr-1.7.5-3.el10.x86_64                       apr-util-1.6.3-23.el10.x86_64                
  apr-util-lmdb-1.6.3-23.el10.x86_64            apr-util-openssl-1.6.3-23.el10.x86_64        
  centos-logos-httpd-100.5-1.el10.noarch        httpd-2.4.63-14.el10.x86_64                  
  httpd-core-2.4.63-14.el10.x86_64              httpd-filesystem-2.4.63-14.el10.noarch       
  httpd-tools-2.4.63-14.el10.x86_64             mailcap-2.1.54-8.el10.noarch                 
  mod_http2-2.0.29-4.el10.x86_64                mod_lua-2.4.63-14.el10.x86_64                

Complete!
[root@lab ~]# 
 
[root@lab ~]# systemctl status httpd
○ httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:httpd.service(8)
[root@lab ~]# 

[root@lab ~]# systemctl start httpd

[root@lab ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: active (running) since Thu 2026-08-06 05:45:04 +0545; 5s ago
...
[root@lab ~]# systemctl enable httpd
Created symlink '/etc/systemd/system/multi-user.target.wants/httpd.service' → '/usr/lib/systemd/system/httpd.service'.
[root@lab ~]# 

[root@lab ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: disabled)
     Active: active (running) since Thu 2026-08-06 05:45:04 +0545; 17s ago
...
[root@lab ~]# 

[root@lab ~]# cd /var/www/html/
[root@lab html]# ls
[root@lab html]# 
[root@lab html]# vi index.html

[root@lab html]# ls
index.html
[root@lab html]# cat index.html 
<h1> Httpd Service is running </h1>
[root@lab html]# 

[root@lab html]# ls -dZ /var/www/html/
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
[root@lab html]# 

[root@lab html]# ls -Z index.html 
unconfined_u:object_r:httpd_sys_content_t:s0 index.html
[root@lab html]# 

 # the created file index.html has the security context same as that of it's parent directory /var/www/html
 
[root@lab html]# systemctl is-enabled httpd
enabled

[root@lab html]# systemctl reload httpd

[root@lab html]# firewall-cmd --list-all | grep http
[root@lab html]# 
[root@lab html]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 5050/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@lab html]# 

[root@lab html]# firewall-cmd --permanent --add-service=http
success

[root@lab html]# firewall-cmd --reload
success

[root@lab html]# firewall-cmd --list-all
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client http ssh
  ports: 5050/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[root@lab html]#
 
[root@lab html]# ls
index.html
[root@lab html]# 

[root@lab html]# curl http://localhost
<h1> Httpd Service is running </h1>
[root@lab html]# 

[root@lab html]# curl 0
<h1> Httpd Service is running </h1>
[root@lab html]# 

[root@lab html]# curl 127.0.0.1
<h1> Httpd Service is running </h1>
[root@lab html]# 

[root@lab html]# hostname -I
192.168.254.2 
 
[root@lab html]# vi index.html 

[root@lab html]# curl 127.0.0.1
<h1> Httpd Service is running </h1>
<h3> Testing the Security context of SELinux </h3>
[root@lab html]# 

[root@lab html]# cd
[root@lab ~]# pwd
/root

[root@lab ~]# vi index.html

[root@lab ~]# cat index.html 
<h1> this is the file created on /root </h1>

<p> it's security context will be according to it's parent home directory </p>

[root@lab ~]# 

[root@lab ~]# ls -dZ /root/
system_u:object_r:admin_home_t:s0 /root/
 
[root@lab ~]# ls -Z index.html 
unconfined_u:object_r:admin_home_t:s0 index.html

[root@lab ~]# cp /root/index.html /var/www/html/
cp: overwrite '/var/www/html/index.html'? y
[root@lab ~]# 

[root@lab ~]# ls -Z /var/www/html/index.html 
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
[root@lab ~]# 
 # In above, security context is according to destination's parent directory

[root@lab ~]# ls -dZ /var/www/html/
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
[root@lab ~]# 

[root@lab ~]# curl 0
<h1> this is the file created on /root </h1>

<p> it's security context will be according to it's parent home directory </p>

[root@lab ~]# 

 # For the context of move:

[root@lab ~]# history

 1067  pwd
 1068  vi index.html 
 1069  ls -Z index.html 
 1070  ls -dZ . 
 1071  mv index.html /var/www/html/
 1072  curl 0
 1073  # This is the error page because of SELinux Security context.
 1074  clear
 1075  history
[root@lab ~]# 

[root@lab ~]# setenforce 0

[root@lab ~]# getenforce 
Permissive

[root@lab ~]# curl 0
<h1> this is the file created on /root </h1>

<p> it's security context will be according to it's parent home directory </p>

<p> While moving: security context is carried on </p>
[root@lab ~]# 

 # while removing the SELinux to permissive from enforcing  ---> bad practice in prod env

[root@lab ~]# cd /var/www/html/

[root@lab html]# ls
index.html

[root@lab html]# ls -dZ .
system_u:object_r:httpd_sys_content_t:s0 .

[root@lab html]# ls -Z index.html 
unconfined_u:object_r:admin_home_t:s0 index.html

 # different security context

   # Chaning SELinux Security Context of a File/Dir

   # chcon -t <Security Context Type> <file/dir>
 
[root@lab html]# pwd
/var/www/html

[root@lab html]# ls -dZ .
system_u:object_r:httpd_sys_content_t:s0 .

[root@lab html]# chcon -t httpd_sys_content_t index.html 

[root@lab html]# ls -Z index.html 
unconfined_u:object_r:httpd_sys_content_t:s0 index.html

[root@lab html]# chcon --reference=/var/www/html index.html 
 
 # OR

[root@lab html]# chcon --reference=/var/www/html index.html 

[root@lab html]# curl 0
<h1> this is the file created on /root </h1>

<p> it's security context will be according to it's parent home directory </p>

<p> While moving: security context is carried on </p>
[root@lab html]# 

[root@lab html]# getenforce 
Permissive

[root@lab html]# setenforce 1

[root@lab html]# getenforce 
Enforcing
 
[root@lab html]# curl 0
<h1> this is the file created on /root </h1>

<p> it's security context will be according to it's parent home directory </p>

<p> While moving: security context is carried on </p>
[root@lab html]# 

[root@lab html]# cd
[root@lab ~]# 
[root@lab ~]# cd -
/var/www/html
[root@lab html]# 
[root@lab html]# curl 0:80
<h1> this is the file created on /root </h1>

<p> it's security context will be according to it's parent home directory </p>

<p> While moving: security context is carried on </p>
[root@lab html]# 

[root@lab html]# vi /etc/httpd/conf/httpd.conf 

[root@lab html]# grep Listen /etc/httpd/conf/httpd.conf 
# Listen: Allows you to bind Apache to specific IP addresses and/or
# Change this to Listen on a specific IP address, but note that if
#Listen 12.34.56.78:80
Listen 80
[root@lab html]# 

[root@lab html]# vi /etc/httpd/conf/httpd.conf                            # Modified port to 8080

[root@lab html]# grep Listen /etc/httpd/conf/httpd.conf 
# Listen: Allows you to bind Apache to specific IP addresses and/or
# Change this to Listen on a specific IP address, but note that if
#Listen 12.34.56.78:80
# Listen 80
Listen 8080
[root@lab html]# 

[root@lab html]# systemctl restart httpd
 
[root@lab html]# ls /var/log/httpd/
access_log  error_log
[root@lab html]# 

[root@lab html]# curl 0:80
curl: (7) Failed to connect to 0.0.0.0 port 80 after 0 ms: Could not connect to server
[root@lab html]# 

[root@lab html]# curl 0:8080
<h1> this is the file created on /root </h1>

<p> it's security context will be according to it's parent home directory </p>

<p> While moving: security context is carried on </p>
[root@lab html]# 

[root@lab html]# getenforce 
Enforcing
[root@lab html]# 

 # semanage port -a -t http_port_t -p 8080 tcp
 
[root@lab html]# # semanage port -a -t http_port_t -p tcp 8080

[root@lab html]# firewall-cmd --permanent --add-port=8080
Error: INVALID_PORT: bad port (most likely missing protocol), correct syntax is portid[-portid]/protocol
[root@lab html]# 

[root@lab html]# firewall-cmd --permanent --add-port=8080/tcp
success

[root@lab html]# firewall-cmd --reload
success

[root@lab html]# firewall-cmd --list-ports
5050/tcp 8080/tcp

[root@lab html]# semanage port -a -t http_port_t -p tcp 8080
Port tcp/8080 already defined, modifying instead
[root@lab html]# 

[root@lab html]# curl http://localhost:8080
<h1> this is the file created on /root </h1>

<p> it's security context will be according to it's parent home directory </p>

<p> While moving: security context is carried on </p>
[root@lab html]# 
 
[aadarsha@lab ~]$ whoami
aadarsha

[aadarsha@lab ~]$ su - root
Password: 
Last login: Fri Aug  7 05:13:01 +0545 2026 on pts/0

 # SELinux Booleans

[root@lab ~]# getenforce
Enforcing

[root@lab ~]# getsebool -a         # list all boolean parameters
abrt_anon_write --> off
abrt_handle_event --> on
abrt_upload_watch_anon_write --> on
auditadm_exec_content --> on
authlogin_nsswitch_use_ldap --> off
authlogin_radius --> off
authlogin_yubikey --> off
cdrecord_read_content --> off
cluster_can_network_connect --> off
cluster_manage_all_files --> off
cluster_use_execmem --> off
colord_use_nfs --> off
condor_tcp_network_connect --> off
corecmd_bin_sys_resource --> off
cron_can_relabel --> off
cron_system_cronjob_use_shares --> off
cron_userdomain_transition --> on
cups_execmem --> off
daemons_dontaudit_scheduling --> on
daemons_dump_core --> off
daemons_enable_cluster_mode --> off
daemons_use_tcp_wrapper --> off
daemons_use_tty --> off
dbadm_exec_content --> on
dbadm_manage_user_files --> off
dbadm_read_user_files --> off
deny_bluetooth --> off
deny_execmem --> off
deny_ptrace --> off
dhcpc_exec_iptables --> off
dhcpd_use_ldap --> off
dnsmasq_use_ipset --> off
domain_can_mmap_files --> off
domain_can_write_kmsg --> off
domain_fd_use --> on
domain_kernel_load_modules --> on
fcron_crond --> off
fenced_can_network_connect --> off
fenced_can_ssh --> off
fips_mode --> on
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
git_cgi_enable_homedirs --> off
git_cgi_use_cifs --> off
git_cgi_use_nfs --> off
git_session_bind_all_unreserved_ports --> off
git_session_users --> off
git_system_enable_homedirs --> off
git_system_use_cifs --> off
git_system_use_nfs --> off
gitosis_can_sendmail --> off
glance_api_can_network --> off
glance_use_execmem --> off
glance_use_fusefs --> off
global_ssp --> off
gluster_anon_write --> off
gluster_export_all_ro --> off
gluster_export_all_rw --> on
gluster_use_execmem --> off
gpg_web_anon_write --> off
gssd_read_tmp --> on
guest_exec_content --> on
haproxy_connect_any --> off
httpd_anon_write --> off
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off
httpd_can_connect_mythtv --> off
httpd_can_connect_zabbix --> off
httpd_can_manage_courier_spool --> off
httpd_can_network_connect --> off
httpd_can_network_connect_cobbler --> off
httpd_can_network_connect_db --> off
httpd_can_network_memcache --> off
httpd_can_network_redis --> off
httpd_can_network_relay --> off
httpd_can_sendmail --> off
httpd_dbus_avahi --> off
httpd_dbus_sssd --> off
httpd_dontaudit_search_dirs --> off
httpd_enable_cgi --> on
httpd_enable_ftp_server --> off
httpd_enable_homedirs --> off
httpd_execmem --> off
httpd_graceful_shutdown --> off
httpd_manage_ipa --> off
httpd_mod_auth_ntlm_winbind --> off
httpd_mod_auth_pam --> off
httpd_read_user_content --> off
httpd_run_ipa --> off
httpd_run_preupgrade --> off
httpd_run_stickshift --> off
httpd_serve_cobbler_files --> off
httpd_setrlimit --> off
httpd_ssi_exec --> off
httpd_sys_script_anon_write --> off
httpd_tmp_exec --> off
httpd_tty_comm --> off
httpd_unified --> off
httpd_use_cifs --> off
httpd_use_fusefs --> off
httpd_use_gpg --> off
httpd_use_nfs --> off
httpd_use_opencryptoki --> off
httpd_use_openstack --> off
httpd_use_sasl --> off
httpd_verify_dns --> off
icecast_use_any_tcp_ports --> off
init_audit_control --> on
init_create_dirs --> on
irc_use_any_tcp_ports --> off
irqbalance_run_unconfined --> off
irssi_use_full_network --> off
kdumpgui_run_bootloader --> off
keepalived_connect_any --> off
kerberos_enabled --> on
ksmtuned_use_cifs --> off
ksmtuned_use_nfs --> off
logadm_exec_content --> on
logging_syslogd_append_public_content --> off
logging_syslogd_can_sendmail --> off
logging_syslogd_list_non_security_dirs --> off
logging_syslogd_run_nagios_plugins --> off
logging_syslogd_run_unconfined --> off
logging_syslogd_use_tty --> on
login_console_enabled --> on
logrotate_read_inside_containers --> off
logrotate_use_cifs --> off
logrotate_use_fusefs --> off
logrotate_use_nfs --> off
logwatch_can_network_connect_mail --> off
lsmd_plugin_connect_any --> off
mcelog_client --> off
mcelog_exec_scripts --> on
mcelog_foreground --> off
mcelog_server --> off
mmap_low_allowed --> off
mount_anyfile --> on
mozilla_plugin_bind_unreserved_ports --> off
mozilla_plugin_can_network_connect --> on
mozilla_plugin_use_bluejeans --> off
mozilla_plugin_use_gps --> off
mozilla_plugin_use_spice --> off
mozilla_read_content --> off
mpd_enable_homedirs --> off
mpd_use_cifs --> off
mpd_use_nfs --> off
mysql_connect_any --> off
mysql_connect_http --> off
named_tcp_bind_http_port --> off
named_write_master_zones --> on
neutron_can_network --> off
nfs_export_all_ro --> on
nfs_export_all_rw --> on
nfsd_anon_write --> off
nis_enabled --> off
nscd_use_shm --> on
openshift_use_nfs --> off
polipo_connect_all_unreserved --> off
polipo_session_bind_all_unreserved_ports --> off
polipo_session_users --> off
polipo_use_cifs --> off
polipo_use_nfs --> off
polyinstantiation_enabled --> off
postfix_local_write_mail_spool --> on
postgresql_can_rsync --> off
postgresql_selinux_transmit_client_label --> off
postgresql_selinux_unconfined_dbadm --> on
postgresql_selinux_users_ddl --> on
pppd_can_insmod --> off
pppd_for_user --> off
racoon_read_shadow --> off
radius_use_jit --> off
redis_enable_notify --> off
rngd_execmem --> off
rpcd_use_fusefs --> off
rsync_anon_write --> off
rsync_client --> off
rsync_export_all_ro --> off
rsync_full_access --> off
rsync_sys_admin --> off
samba_create_home_dirs --> off
samba_domain_controller --> off
samba_enable_home_dirs --> off
samba_export_all_ro --> off
samba_export_all_rw --> off
samba_load_libgfapi --> off
samba_portmapper --> off
samba_run_unconfined --> off
samba_share_fusefs --> off
samba_share_nfs --> off
sanlock_enable_home_dirs --> off
sanlock_use_fusefs --> off
sanlock_use_nfs --> off
sanlock_use_samba --> off
saslauthd_read_shadow --> off
screen_allow_session_sharing --> off
secadm_exec_content --> on
secure_mode --> off
secure_mode_insmod --> off
secure_mode_policyload --> off
selinuxuser_direct_dri_enabled --> on
selinuxuser_execheap --> off
selinuxuser_execmod --> on
selinuxuser_execstack --> on
selinuxuser_mysql_connect_enabled --> off
selinuxuser_ping --> on
selinuxuser_postgresql_connect_enabled --> off
selinuxuser_rw_noexattrfile --> on
selinuxuser_share_music --> off
selinuxuser_tcp_server --> off
selinuxuser_udp_server --> off
selinuxuser_use_ssh_chroot --> off
smartmon_3ware --> off
smbd_anon_write --> off
spamassassin_can_network --> off
spamd_enable_home_dirs --> on
spamd_update_can_network --> off
squid_bind_snmp_port --> off
squid_connect_any --> on
squid_use_tproxy --> off
ssh_chroot_rw_homedirs --> off
ssh_keysign --> off
ssh_sysadm_login --> off
ssh_use_tcpd --> off
sslh_can_bind_any_port --> off
sslh_can_connect_any_port --> off
sssd_access_kernel_keys --> off
sssd_connect_all_unreserved_ports --> off
sssd_use_usb --> off
staff_exec_content --> on
staff_use_svirt --> off
swift_can_network --> off
sysadm_exec_content --> on
systemd_socket_proxyd_bind_any --> off
systemd_socket_proxyd_connect_any --> off
telepathy_connect_all_ports --> off
telepathy_tcp_connect_generic_network_ports --> on
tftp_anon_write --> off
tftp_home_dir --> off
tmpreaper_use_cifs --> off
tmpreaper_use_nfs --> off
tmpreaper_use_samba --> off
tomcat_can_network_connect_db --> off
tomcat_read_rpm_db --> off
tomcat_use_execmem --> off
unconfined_chrome_sandbox_transition --> on
unconfined_dyntrans_all --> off
unconfined_login --> on
unconfined_mozilla_plugin_transition --> on
unprivuser_use_svirt --> off
use_ecryptfs_home_dirs --> off
use_fusefs_home_dirs --> off
use_lpd_server --> off
use_nfs_home_dirs --> off
use_samba_home_dirs --> off
use_virtualbox --> on
user_exec_content --> on
varnishd_connect_any --> off
virt_hooks_unconfined --> off
virt_lockd_blk_devs --> off
virt_qemu_ga_manage_ssh --> off
virt_qemu_ga_read_nonsecurity_files --> off
virt_qemu_ga_run_unconfined --> off
virt_read_qemu_ga_data --> off
virt_rw_qemu_ga_data --> off
virt_sandbox_share_apache_content --> off
virt_sandbox_use_all_caps --> on
virt_sandbox_use_audit --> on
virt_sandbox_use_fusefs --> off
virt_sandbox_use_mknod --> off
virt_sandbox_use_netlink --> off
virt_sandbox_use_sys_admin --> off
virt_transition_userdomain --> off
virt_use_comm --> off
virt_use_execmem --> off
virt_use_fusefs --> off
virt_use_glusterd --> off
virt_use_nfs --> off
virt_use_pcscd --> off
virt_use_pulseaudio --> off
virt_use_rawip --> off
virt_use_samba --> off
virt_use_sanlock --> off
virt_use_usb --> on
virt_use_xserver --> off
virtqemud_use_execmem --> on
webadm_manage_user_files --> off
webadm_read_user_files --> off
wine_mmap_zero_ignore --> off
xdm_bind_vnc_tcp_port --> off
xdm_exec_bootloader --> off
xdm_manage_bootloader --> on
xdm_sysadm_login --> off
xdm_write_home --> off
xen_use_nfs --> off
xend_run_blktap --> on
xend_run_qemu --> on
xguest_connect_network --> on
xguest_exec_content --> on
xguest_mount_media --> on
xguest_use_bluetooth --> on
xserver_clients_write_xshm --> off
xserver_execmem --> off
xserver_object_manager --> off
zarafa_setrlimit --> off
zoneminder_anon_write --> off
zoneminder_run_sudo --> off
[root@lab ~]# 
[root@lab ~]# 
[root@lab ~]# getsebool -a | grep httpd
httpd_anon_write --> off
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off
httpd_can_connect_mythtv --> off
httpd_can_connect_zabbix --> off
httpd_can_manage_courier_spool --> off
httpd_can_network_connect --> off
httpd_can_network_connect_cobbler --> off
httpd_can_network_connect_db --> off
httpd_can_network_memcache --> off
httpd_can_network_redis --> off
httpd_can_network_relay --> off
httpd_can_sendmail --> off
httpd_dbus_avahi --> off
httpd_dbus_sssd --> off
httpd_dontaudit_search_dirs --> off
httpd_enable_cgi --> on
httpd_enable_ftp_server --> off
httpd_enable_homedirs --> off
httpd_execmem --> off
httpd_graceful_shutdown --> off
httpd_manage_ipa --> off
httpd_mod_auth_ntlm_winbind --> off
httpd_mod_auth_pam --> off
httpd_read_user_content --> off
httpd_run_ipa --> off
httpd_run_preupgrade --> off
httpd_run_stickshift --> off
httpd_serve_cobbler_files --> off
httpd_setrlimit --> off
httpd_ssi_exec --> off
httpd_sys_script_anon_write --> off
httpd_tmp_exec --> off
httpd_tty_comm --> off
httpd_unified --> off
httpd_use_cifs --> off
httpd_use_fusefs --> off
httpd_use_gpg --> off
httpd_use_nfs --> off
httpd_use_opencryptoki --> off
httpd_use_openstack --> off
httpd_use_sasl --> off
httpd_verify_dns --> off
[root@lab ~]# 
[root@lab ~]# 
[root@lab ~]# getsebool -a | grep connect
cluster_can_network_connect --> off
condor_tcp_network_connect --> off
fenced_can_network_connect --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
haproxy_connect_any --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off
httpd_can_connect_mythtv --> off
httpd_can_connect_zabbix --> off
httpd_can_network_connect --> off
httpd_can_network_connect_cobbler --> off
httpd_can_network_connect_db --> off
keepalived_connect_any --> off
logwatch_can_network_connect_mail --> off
lsmd_plugin_connect_any --> off
mozilla_plugin_can_network_connect --> on
mysql_connect_any --> off
mysql_connect_http --> off
polipo_connect_all_unreserved --> off
selinuxuser_mysql_connect_enabled --> off
selinuxuser_postgresql_connect_enabled --> off
squid_connect_any --> on
sslh_can_connect_any_port --> off
sssd_connect_all_unreserved_ports --> off
systemd_socket_proxyd_connect_any --> off
telepathy_connect_all_ports --> off
telepathy_tcp_connect_generic_network_ports --> on
tomcat_can_network_connect_db --> off
varnishd_connect_any --> off
xguest_connect_network --> on
[root@lab ~]# 

 # Example use case: 
  # while configuring load balancers or deploying application and allowing access, we may need to turn on Booleans
  # also for some security purposes, we may need to turn off Booleans

[root@lab ~]# getsebool httpd_can_network_connect
httpd_can_network_connect --> off

 # Enable SELinux Boolean Parameter

[root@lab ~]# setsebool -P httpd_can_network_connect on

[root@lab ~]# getsebool httpd_can_network_connect
httpd_can_network_connect --> on
 
[root@lab ~]# getsebool -a | grep ftp
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off
[root@lab ~]# 
```