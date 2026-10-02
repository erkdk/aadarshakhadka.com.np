---
title: "DNS Fundamentals: Building a BIND Authoritative & Recursive DNS Server"
date: 2026-10-01
draft: false
---

### What is DNS?

DNS (Domain Name System) is a distributed naming system that translates human-readable names into network addresses and provides other information about network services.

#### Usecase:
- Forward Lookup:
```
Hostname / Domain Name --> IP Address
```
For example:
```
webserver.example.com --> 192.168.254.22
```
- Reverse Lookup:
DNS can also perform the reverse:
```
IP Address --> Hostname
```
For example:
```
192.168.254.22 --> webserver.example.com
```

### Lab session

```
aadarkdk@pop-os:~$ ssh aadarkhadka@192.168.254.22
...
[aadarkhadka@localhost ~]$ 

[aadarkhadka@localhost ~]$ hostname -I
192.168.254.22
[aadarkhadka@localhost ~]$ 

[aadarkhadka@localhost ~]$ hostname
localhost.localdomain
```
---

#### DNS Server Configuration (```dnsserver.lab.local```)
```
[aadarkhadka@localhost ~]$ sudo hostnamectl set-hostname dnsserver.lab.local

[aadarkhadka@localhost ~]$ hostname
dnsserver.lab.local
[aadarkhadka@localhost ~]$ sudo vi /etc/NetworkManager/system-connections/enp0s3.nmconnection
[aadarkhadka@dnsserver ~]$ sudo cat /etc/NetworkManager/system-connections/enp0s3.nmconnection
[connection]
id=enp0s3
uuid=0f33e541-9a7d-3e69-84be-79d2cf70211b
type=ethernet
autoconnect-priority=-999
interface-name=enp0s3
timestamp=1790485361

[ethernet]

[ipv4]
method=manual
address=192.168.254.20/24
gateway=192.168.254.254
dns=192.168.254.254;

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
[aadarkhadka@dnsserver ~]$ sudo nmcli connection reload
[aadarkhadka@dnsserver ~]$ sudo nmcli connection up enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)
[aadarkhadka@dnsserver ~]$ 
```
```
[aadarkhadka@dnsserver ~]$ hostname -I
192.168.254.20 
[aadarkhadka@dnsserver ~]$ ip route                                               # default gateway
default via 192.168.254.254 dev enp0s3 proto static metric 100 
192.168.254.0/24 dev enp0s3 proto kernel scope link src 192.168.254.20 metric 100 
[aadarkhadka@dnsserver ~]$

[aadarkhadka@dnsserver ~]$ nmcli -g IP4.ADDRESS,IP4.GATEWAY,IP4.DNS device show enp0s3
192.168.254.20/24
192.168.254.254
192.168.254.254
[aadarkhadka@dnsserver ~]$ 
```
---

#### Client Machine Configuration ( ```client.lab.local``` )
```
[aadarkhadka@localhost ~]$ hostname
localhost.localdomain
[aadarkhadka@localhost ~]$ hostname -I
192.168.254.3 
[aadarkhadka@localhost ~]$ sudo cat /etc/NetworkManager/system-connections/enp0s3.nmconnection
[sudo] password for aadarkhadka: 
[connection]
id=enp0s3
uuid=9b9b1a2f-5bed-3066-8c06-33633f2e8a85
type=ethernet
autoconnect-priority=-999
interface-name=enp0s3
timestamp=1790603728

[ethernet]

[ipv4]
method=auto

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
[aadarkhadka@localhost ~]$ sudo vi /etc/NetworkManager/system-connections/enp0s3.nmconnection
[aadarkhadka@dnsserver ~]$ sudo cat /etc/NetworkManager/system-connections/enp0s3.nmconnection
[connection]
id=enp0s3
uuid=0f33e541-9a7d-3e69-84be-79d2cf70211b
type=ethernet
autoconnect-priority=-999
interface-name=enp0s3
timestamp=1790485361

[ethernet]

[ipv4]
method=manual
address=192.168.254.20/24
gateway=192.168.254.254
dns=192.168.254.254;

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
[aadarkhadka@dnsserver ~]$ sudo nmcli connection reload
[aadarkhadka@dnsserver ~]$ sudo nmcli connection up enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)
[aadarkhadka@dnsserver ~]$ 
```

```
[aadarkhadka@localhost ~]$ sudo nmcli connection up enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
[aadarkhadka@localhost ~]$ sudo hostnamectl set-hostname client.lab.local
[aadarkhadka@localhost ~]$ hostname
client.lab.local

[aadarkhadka@client ~]$ hostname 
client.lab.local
[aadarkhadka@client ~]$ hostname -I
192.168.254.21 
[aadarkhadka@client ~]$
```
---

#### Web Server Configuration ( ```webserver.lab.local``` )
```
[aadarkhadka@localhost ~]$ hostname
localhost.localdomain
[aadarkhadka@localhost ~]$ hostname -I
192.168.254.23 
[aadarkhadka@localhost ~]$ sudo hostnamectl set-hostname webserver.lab.local
[sudo] password for aadarkhadka: 
Sorry, try again.
[sudo] password for aadarkhadka: 
[aadarkhadka@localhost ~]$ hostname
webserver.lab.local
[aadarkhadka@localhost ~]$ sudo vi /etc/NetworkManager/system-connections/enp0s3.nmconnection
[aadarkhadka@dnsserver ~]$ sudo cat /etc/NetworkManager/system-connections/enp0s3.nmconnection
[connection]
id=enp0s3
uuid=0f33e541-9a7d-3e69-84be-79d2cf70211b
type=ethernet
autoconnect-priority=-999
interface-name=enp0s3
timestamp=1790485361

[ethernet]

[ipv4]
method=manual
address=192.168.254.20/24
gateway=192.168.254.254
dns=192.168.254.254;

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
[aadarkhadka@dnsserver ~]$ sudo nmcli connection reload
[aadarkhadka@dnsserver ~]$ sudo nmcli connection up enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)
[aadarkhadka@dnsserver ~]$ 
```
```
[aadarkhadka@webserver ~]$ sudo nmcli connection up enp0s3
[sudo] password for aadarkhadka: 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
[aadarkhadka@webserver ~]$
```

```
[aadarkhadka@webserver ~]$ hostname -I
192.168.254.22 
[aadarkhadka@webserver ~]$ hostname
webserver.lab.local
[aadarkhadka@webserver ~]$ whoami
aadarkhadka
[aadarkhadka@webserver ~]$
```

---

- Similarly, I configured for ( ```dbserver.lab.local``` ) and ( ```fileserver.lab.local``` ).

---

### Configuring the Master DNS Server

#### Steps:

 1. Install required packages ( i.e. bind )

 2. Start and Enable the DNS Service  ( i.e. named )

 3. Allow DNS packets to enter through the firewall

 4. Configure DNS

  4.1. Configure DNS parameters: in /etc/named.conf

       cp /etc/named.conf /etc/named.conf.bak                             # Take backup before configuring

  4.2. Register Domain: in /etc/named.rfc1912.zones

       cp /etc/named.rfc1912.zones  /etc/named.rfc1912.zones.bak          # Take backup before configuring

  4.3. Check for the syntax errors in the config files (both files)
  
       named-checkconf
       
  4.4.  



---



```
[aadarkhadka@dnsserver ~]$ rpm -q bind bind-utils
package bind is not installed
package bind-utils is not installed
[aadarkhadka@dnsserver ~]$ sudo dnf install bind bind-utils -y

[aadarkhadka@dnsserver ~]$ rpm -q bind bind-utils
bind-9.18.33-28.el10.x86_64
bind-utils-9.18.33-28.el10.x86_64

[aadarkhadka@dnsserver ~]$ sudo systemctl start named
[aadarkhadka@dnsserver ~]$ sudo systemctl enable named
Created symlink '/etc/systemd/system/multi-user.target.wants/named.service' → '/usr/lib/systemd/system/named.service'.
[aadarkhadka@dnsserver ~]$ sudo systemctl is-active named
active
[aadarkhadka@dnsserver ~]$ sudo systemctl is-enabled named
enabled
[aadarkhadka@dnsserver ~]$ sudo ss -lntup | grep ':53'
udp   UNCONN 0      0                              127.0.0.1:53        0.0.0.0:*    users:(("named",pid=6125,fd=26))        
udp   UNCONN 0      0                              127.0.0.1:53        0.0.0.0:*    users:(("named",pid=6125,fd=25))        
udp   UNCONN 0      0                                  [::1]:53           [::]:*    users:(("named",pid=6125,fd=31))        
udp   UNCONN 0      0                                  [::1]:53           [::]:*    users:(("named",pid=6125,fd=32))        
tcp   LISTEN 0      10                             127.0.0.1:53        0.0.0.0:*    users:(("named",pid=6125,fd=29))        
tcp   LISTEN 0      10                             127.0.0.1:53        0.0.0.0:*    users:(("named",pid=6125,fd=27))        
tcp   LISTEN 0      10                                 [::1]:53           [::]:*    users:(("named",pid=6125,fd=33))        
tcp   LISTEN 0      10                                 [::1]:53           [::]:*    users:(("named",pid=6125,fd=34))        
[aadarkhadka@dnsserver ~]$ sudo less /etc/named.conf
```

---

```
[aadarkhadka@dnsserver ~]$ sudo vi /etc/named.conf
[aadarkhadka@dnsserver ~]$ sudo named-checkconf
/etc/named.conf:11: missing ';' before '}'
[aadarkhadka@dnsserver ~]$ sudo vi +11 /etc/named.conf
[aadarkhadka@dnsserver ~]$ sudo named-checkconf
[aadarkhadka@dnsserver ~]$ sudo cat /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
	listen-on port 53 { 127.0.0.1; 192.168.254.20; };
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file	"/var/named/data/named.secroots";
	recursing-file	"/var/named/data/named.recursing";
	allow-query     { localhost; 192.168.254.0/24; };
        allow-recursion { localhost; 192.168.254.0/24; };

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-validation yes;

	managed-keys-directory "/var/named/dynamic";
	geoip-directory "/usr/share/GeoIP";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";

	/* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
	include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

[aadarkhadka@dnsserver ~]$ sudo systemctl restart named
[aadarkhadka@dnsserver ~]$ sudo systemctl is-active named
active
[aadarkhadka@dnsserver ~]$ sudo ss -lntup | grep ':53'
udp   UNCONN 0      0                         192.168.254.20:53        0.0.0.0:*    users:(("named",pid=2009,fd=31))        
udp   UNCONN 0      0                         192.168.254.20:53        0.0.0.0:*    users:(("named",pid=2009,fd=32))        
udp   UNCONN 0      0                              127.0.0.1:53        0.0.0.0:*    users:(("named",pid=2009,fd=25))        
udp   UNCONN 0      0                              127.0.0.1:53        0.0.0.0:*    users:(("named",pid=2009,fd=26))        
udp   UNCONN 0      0                                  [::1]:53           [::]:*    users:(("named",pid=2009,fd=36))        
udp   UNCONN 0      0                                  [::1]:53           [::]:*    users:(("named",pid=2009,fd=35))        
tcp   LISTEN 0      10                        192.168.254.20:53        0.0.0.0:*    users:(("named",pid=2009,fd=33))        
tcp   LISTEN 0      10                        192.168.254.20:53        0.0.0.0:*    users:(("named",pid=2009,fd=34))        
tcp   LISTEN 0      10                             127.0.0.1:53        0.0.0.0:*    users:(("named",pid=2009,fd=27))        
tcp   LISTEN 0      10                             127.0.0.1:53        0.0.0.0:*    users:(("named",pid=2009,fd=28))        
tcp   LISTEN 0      10                                 [::1]:53           [::]:*    users:(("named",pid=2009,fd=38))        
tcp   LISTEN 0      10                                 [::1]:53           [::]:*    users:(("named",pid=2009,fd=37))        
[aadarkhadka@dnsserver ~]$ 
```

```
[aadarkhadka@dnsserver ~]$ dig @127.0.0.1 google.com

; <<>> DiG 9.18.33 <<>> @127.0.0.1 google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12145
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 978611b20ffebd16010000006abdc4e1a8cbc9848361aa11 (good)
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		290	IN	A	172.217.26.110

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Thu Oct 01 08:11:41 +0545 2026
;; MSG SIZE  rcvd: 83

[aadarkhadka@dnsserver ~]$ dig @192.168.254.20 google.com

; <<>> DiG 9.18.33 <<>> @192.168.254.20 google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49827
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 22b72d05a2e74578010000006abdc4e71c3c72083008c2ca (good)
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		284	IN	A	172.217.26.110

;; Query time: 0 msec
;; SERVER: 192.168.254.20#53(192.168.254.20) (UDP)
;; WHEN: Thu Oct 01 08:11:47 +0545 2026
;; MSG SIZE  rcvd: 83

[aadarkhadka@dnsserver ~]$ 

```

```
[aadarkhadka@client ~]$ dig @192.168.254.20 google.com

; <<>> DiG 9.18.33 <<>> @192.168.254.20 google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47856
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e84600fd07c29d15010000006abdc83ce3c08d386b1356cd (good)
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		300	IN	A	172.217.26.110

;; Query time: 98 msec
;; SERVER: 192.168.254.20#53(192.168.254.20) (UDP)
;; WHEN: Thu Oct 01 08:25:42 +0545 2026
;; MSG SIZE  rcvd: 83

[aadarkhadka@client ~]$ 
```

```
[aadarkhadka@dnsserver ~]$ sudo vi /etc/named.rfc1912.zones
[aadarkhadka@dnsserver ~]$ sudo cat /etc/named.rfc1912.zones
// named.rfc1912.zones:
//
// Provided by Red Hat caching-nameserver package 
//
// ISC BIND named zone configuration for zones recommended by
// RFC 1912 section 4.1 : localhost TLDs and address zones
// and https://tools.ietf.org/html/rfc6303
// (c)2007 R W Franks
// 
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// Note: empty-zones-enable yes; option is default.
// If private ranges should be forwarded, add 
// disable-empty-zone "."; into options
// 

zone "localhost.localdomain" IN {
	type primary;
	file "named.localhost";
	allow-update { none; };
};

zone "localhost" IN {
	type primary;
	file "named.localhost";
	allow-update { none; };
};

zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {
	type primary;
	file "named.loopback";
	allow-update { none; };
};

zone "1.0.0.127.in-addr.arpa" IN {
	type primary;
	file "named.loopback";
	allow-update { none; };
};

zone "0.in-addr.arpa" IN {
	type primary;
	file "named.empty";
	allow-update { none; };
};

zone "lab.local" IN {
    type master;
    file "lab.local.zone";
};
[aadarkhadka@dnsserver ~]$ sudo named-checkconf
[aadarkhadka@dnsserver ~]$ 
```

```
[aadarkhadka@dnsserver ~]$ sudo ls /var/named
data  dynamic  named.ca  named.empty  named.localhost  named.loopback  slaves
[aadarkhadka@dnsserver ~]$ sudo vi /var/named/lab.local.zone
[aadarkhadka@dnsserver ~]$ sudo cat /var/named/lab.local.zone
[sudo] password for aadarkhadka: 
$TTL 86400

@   IN   SOA   dnsserver.lab.local.   admin.lab.local. (
         2026100101 ; Serial
         3600       ; Refresh
         600        ; Retry
         86400      ; Expire
         3600       ; Minimum TTL
)

    IN   NS  dnsserver.lab.local.

dnsserver  IN   A   192.168.254.20
client     IN   A   192.168.254.21
webserver  IN   A   192.168.254.22
dbserver   IN   A   192.168.254.23
fileserver IN   A   192.168.254.24
[aadarkhadka@dnsserver ~]$ sudo named-checkzone lab.local /var/named/lab.local.zone
zone lab.local/IN: loaded serial 2026100101
OK
[aadarkhadka@dnsserver ~]$ 

[aadarkhadka@dnsserver ~]$ sudo ls -lZ /var/named/lab.local.zone
-rw-r--r--. 1 root root unconfined_u:object_r:named_zone_t:s0 432 Oct  1 09:25 /var/named/lab.local.zone
[aadarkhadka@dnsserver ~]$ sudo restorecon -v /var/named/lab.local.zone
[aadarkhadka@dnsserver ~]$ sudo ls -lZ /var/named/lab.local.zone
-rw-r--r--. 1 root root unconfined_u:object_r:named_zone_t:s0 432 Oct  1 09:25 /var/named/lab.local.zone
[aadarkhadka@dnsserver ~]$
```

```
[aadarkhadka@dnsserver ~]$ sudo systemctl restart named
[aadarkhadka@dnsserver ~]$ sudo systemctl is-active named
active
[aadarkhadka@dnsserver ~]$ dig @192.168.254.20 webserver.lab.local

; <<>> DiG 9.18.33 <<>> @192.168.254.20 webserver.lab.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8566
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 8330a2144f0df6e6010000006abdd7ea2f9c1586c57b23a6 (good)
;; QUESTION SECTION:
;webserver.lab.local.		IN	A

;; ANSWER SECTION:
webserver.lab.local.	86400	IN	A	192.168.254.22

;; Query time: 1 msec
;; SERVER: 192.168.254.20#53(192.168.254.20) (UDP)
;; WHEN: Thu Oct 01 09:32:54 +0545 2026
;; MSG SIZE  rcvd: 92

[aadarkhadka@dnsserver ~]$ dig @192.168.254.20 dbserver.lab.local

; <<>> DiG 9.18.33 <<>> @192.168.254.20 dbserver.lab.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45281
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 3e578fe9e910f9e8010000006abdd858b260e39b1f4e737f (good)
;; QUESTION SECTION:
;dbserver.lab.local.		IN	A

;; ANSWER SECTION:
dbserver.lab.local.	86400	IN	A	192.168.254.23

;; Query time: 0 msec
;; SERVER: 192.168.254.20#53(192.168.254.20) (UDP)
;; WHEN: Thu Oct 01 09:34:44 +0545 2026
;; MSG SIZE  rcvd: 91

[aadarkhadka@dnsserver ~]$ 

```

#### reverse zone

```
[aadarkhadka@dnsserver ~]$ sudo vi /etc/named.rfc1912.zones
[aadarkhadka@dnsserver ~]$ sudo cat /etc/named.rfc1912.zones
...
zone "lab.local" IN {
    type master;
    file "lab.local.zone";
};

zone "254.168.192.in-addr.arpa" IN {
    type master;
    file "254.168.192.rev";
};
[aadarkhadka@dnsserver ~]$ sudo named-checkconf

[aadarkhadka@dnsserver ~]$ sudo vi /var/named/254.168.192.rev
[aadarkhadka@dnsserver ~]$ sudo cat /var/named/254.168.192.rev
[sudo] password for aadarkhadka: 
$TTL 86400

@   IN   SOA   dnsserver.lab.local.   admin.lab.local.   (
         2026100101 ; Serial
         3600       ; Refresh
         600        ; Retry
         86400      ; Expire
         3600       ; Minimum TTL
)

    IN   NS   dnsserver.lab.local.

20  IN   PTR  dnsserver.lab.local.
21  IN   PTR  client.lab.local.
22  IN   PTR  webserver.lab.local.
23  IN   PTR  dbserver.lab.local.
24  IN   PTR  fileserver.lab.local.
[aadarkhadka@dnsserver ~]$ sudo named-checkzone 254.168.192.in-addr.arpa /var/named/254.168.192.rev
zone 254.168.192.in-addr.arpa/IN: loaded serial 2026100101
OK
[aadarkhadka@dnsserver ~]$

[aadarkhadka@dnsserver ~]$ sudo ls -lZ /var/named/254.168.192.rev
-rw-r--r--. 1 root root unconfined_u:object_r:named_zone_t:s0 432 Oct  1 10:13 /var/named/254.168.192.rev
[aadarkhadka@dnsserver ~]$ sudo restorecon -v /var/named/254.168.192.rev
[aadarkhadka@dnsserver ~]$ sudo ls -lZ /var/named/254.168.192.rev
-rw-r--r--. 1 root root unconfined_u:object_r:named_zone_t:s0 432 Oct  1 10:13 /var/named/254.168.192.rev
[aadarkhadka@dnsserver ~]$ 
```
---

```
[aadarkhadka@dnsserver ~]$ sudo named-checkconf
[aadarkhadka@dnsserver ~]$ sudo systemctl restart named
[aadarkhadka@dnsserver ~]$ sudo systemctl is-active named
active
[aadarkhadka@dnsserver ~]$ dig @192.168.254.20 webserver.lab.local

; <<>> DiG 9.18.33 <<>> @192.168.254.20 webserver.lab.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9488
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e215c693151b13bf010000006abde3439c9dfcb92b6b3f5d (good)
;; QUESTION SECTION:
;webserver.lab.local.		IN	A

;; ANSWER SECTION:
webserver.lab.local.	86400	IN	A	192.168.254.22

;; Query time: 2 msec
;; SERVER: 192.168.254.20#53(192.168.254.20) (UDP)
;; WHEN: Thu Oct 01 10:21:19 +0545 2026
;; MSG SIZE  rcvd: 92

[aadarkhadka@dnsserver ~]$ dig @192.168.254.20 -x 192.168.254.22

; <<>> DiG 9.18.33 <<>> @192.168.254.20 -x 192.168.254.22
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 60858
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 86a2369314d66c0d010000006abde36c3f70c3e3bf55d4d1 (good)
;; QUESTION SECTION:
;22.254.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
22.254.168.192.in-addr.arpa. 86400 IN	PTR	webserver.lab.local.

;; Query time: 0 msec
;; SERVER: 192.168.254.20#53(192.168.254.20) (UDP)
;; WHEN: Thu Oct 01 10:22:00 +0545 2026
;; MSG SIZE  rcvd: 117

[aadarkhadka@dnsserver ~]$ 
```

```
[aadarkhadka@client ~]$ sudo nmcli connection modify enp0s3 ipv4.dns "192.168.254.20"
[aadarkhadka@client ~]$ sudo nmcli connection down enp0s3
Connection 'enp0s3' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/2)
[aadarkhadka@client ~]$ sudo nmcli connection up enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
[aadarkhadka@client ~]$ nmcli -g IP4.ADDRESS,IP4.GATEWAY,IP4.DNS device show enp0s3
192.168.254.21/24
192.168.254.254
192.168.254.20
[aadarkhadka@client ~]$ dig webserver.lab.local

; <<>> DiG 9.18.33 <<>> webserver.lab.local
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36855
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: fa8d7aa47219af0d010000006abe241657ee262e7bff7a2e (good)
;; QUESTION SECTION:
;webserver.lab.local.		IN	A

;; ANSWER SECTION:
webserver.lab.local.	86400	IN	A	192.168.254.22

;; Query time: 2 msec
;; SERVER: 192.168.254.20#53(192.168.254.20) (UDP)
;; WHEN: Thu Oct 01 14:57:37 +0545 2026
;; MSG SIZE  rcvd: 92

[aadarkhadka@client ~]$

[aadarkhadka@client ~]$ dig -x 192.168.254.22

; <<>> DiG 9.18.33 <<>> -x 192.168.254.22
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6785
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 91d722ed0420e1dc010000006abe24da46d2d9664831e25d (good)
;; QUESTION SECTION:
;22.254.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
22.254.168.192.in-addr.arpa. 86400 IN	PTR	webserver.lab.local.

;; Query time: 1 msec
;; SERVER: 192.168.254.20#53(192.168.254.20) (UDP)
;; WHEN: Thu Oct 01 15:00:52 +0545 2026
;; MSG SIZE  rcvd: 117

[aadarkhadka@client ~]$ dig -x 192.168.254.23

; <<>> DiG 9.18.33 <<>> -x 192.168.254.23
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5981
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 362e0be45022443c010000006abe25c15810cb4bd51ceba3 (good)
;; QUESTION SECTION:
;23.254.168.192.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
23.254.168.192.in-addr.arpa. 86400 IN	PTR	dbserver.lab.local.

;; Query time: 1 msec
;; SERVER: 192.168.254.20#53(192.168.254.20) (UDP)
;; WHEN: Thu Oct 01 15:04:43 +0545 2026
;; MSG SIZE  rcvd: 116

[aadarkhadka@client ~]$ 
```

#### DNS Configuration on webserver
```
[aadarkhadka@webserver ~]$ sudo nmcli connection modify enp0s3 ipv4.dns "192.168.254.20"
[aadarkhadka@webserver ~]$ sudo nmcli connection down enp0s3
Connection 'enp0s3' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
[aadarkhadka@webserver ~]$ sudo nmcli connection up enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/7)
[aadarkhadka@webserver ~]$ nmcli -g IP4.ADDRESS,IP4.GATEWAY,IP4.DNS device show enp0s3
192.168.254.22/24
192.168.254.254
192.168.254.20
[aadarkhadka@webserver ~]$ 
```

#### DNS Configuration on dbserver
```
[aadarkhadka@dbserver ~]$ sudo nmcli connection modify enp0s3 ipv4.dns "192.168.254.20"
[aadarkhadka@webserver ~]$ sudo nmcli connection down enp0s3
Connection 'enp0s3' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
[aadarkhadka@dbserver ~]$ sudo nmcli connection up enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)
[aadarkhadka@dbserver ~]$ nmcli -g IP4.ADDRESS,IP4.GATEWAY,IP4.DNS device show enp0s3
192.168.254.23/24
192.168.254.254
192.168.254.20
[aadarkhadka@dbserver ~]$

```


#### DNS Configuration on fileserver
```
[aadarkhadka@fileserver ~]$ sudo nmcli connection modify enp0s3 ipv4.dns "192.168.254.20"
[aadarkhadka@fileserver ~]$ sudo nmcli connection down enp0s3
Connection 'enp0s3' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
[aadarkhadka@fileserver ~]$ sudo nmcli connection up enp0s3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/7)
[aadarkhadka@fileserver ~]$ nmcli -g IP4.ADDRESS,IP4.GATEWAY,IP4.DNS device show enp0s3
192.168.254.24/24
192.168.254.254
192.168.254.20
[aadarkhadka@fileserver ~]$ 
```

```
[aadarkhadka@client ~]$ getent hosts webserver.lab.local
192.168.254.22  webserver.lab.local
[aadarkhadka@client ~]$ getent hosts dbserver.lab.local
192.168.254.23  dbserver.lab.local
[aadarkhadka@client ~]$ getent hosts fileserver.lab.local
192.168.254.24  fileserver.lab.local
[aadarkhadka@client ~]$ getent hosts dnsserver.lab.local
192.168.254.20  dnsserver.lab.local
[aadarkhadka@client ~]$ 
```

```
[aadarkhadka@dnsserver ~]$ sudo named-checkconf
[aadarkhadka@dnsserver ~]$ sudo named-checkzone lab.local /var/named/lab.local.zone
zone lab.local/IN: loaded serial 2026100101
OK
[aadarkhadka@dnsserver ~]$ sudo named-checkzone 254.168.192.in-addr.arpa /var/named/254.168.192.rev
zone 254.168.192.in-addr.arpa/IN: loaded serial 2026100101
OK
[aadarkhadka@dnsserver ~]$ sudo systemctl is-active named
active
[aadarkhadka@dnsserver ~]$ sudo systemctl is-enabled named
enabled
[aadarkhadka@dnsserver ~]$ 
```
---
