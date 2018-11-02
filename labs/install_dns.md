# Hướng dẫn cài đặt DNS server trên CentOS7

## Mô hình 

- Primary (Master) DNS Server:
    + OS: CentOS7
    + IP: 10.10.10.173
    + Tắt firewalld và selinux 
- Client:
    + OS: CentOS7
    + IP: 10.10.10.174

## Cài đặt DNS server 

- Install bind9

    ``yum install bind bind-utils -y``

- Chỉnh sửa file conf

    ``vi /etc/named.conf``

- Sửa theo file cấu hình này:

```
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// See the BIND Administrator's Reference Manual (ARM) for details about the
// configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {
	listen-on port 53 { 10.10.10.173;};
	#listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	allow-query     { 10.10.10.0/24;};

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

	dnssec-enable yes;
	dnssec-validation yes;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.iscdlv.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

#zone "." IN {
#	type hint;
#	file "named.ca";
#};

zone "com" IN {
type master;
file "forward.com";
allow-update { none; };
};

zone "10.10.10.in-addr.arpa" IN {
type master;
file "reverse.com";
allow-update { none; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```

- Tạo cở sở dữ liệu cho các zone 

``vi /var/named/forward.com``

Thêm các dòng sau:

```sh 
$TTL 86400
@   IN  SOA     minhkma.com. nguyenvanminhkma.gmail.com. (
        2018110201  ;Serial
        3600        ;Refresh
        1800        ;Retry
        604800      ;Expire
        86400       ;Minimum TTL
)
@       IN  NS          minhkma.com.
@       IN  A           10.10.10.173
@       IN  A           10.10.10.174
minhkma       IN  A     10.10.10.173
influxdb      IN  A     10.10.10.174
```

``vi /var/named/reverse.com``

Thêm các dòng sau:

```sh
$TTL 86400
@   IN  SOA     minhkma.com. nguyenvanminhkma.gmail.com. (
        2018110201  ;Serial
        3600        ;Refresh
        1800        ;Retry
        604800      ;Expire
        86400       ;Minimum TTL
)
@       IN  NS          minhkma.com.
@       IN  PTR         com.
minhkma           IN  A   10.10.10.173
influxdb          IN  A   10.10.10.174
173     IN  PTR         minhkma.com.
174     IN  PTR         influxdb.com.
```

- Khởi động service DNS

```sh
systemctl enable named
systemctl start named
```

- Tắt firewalld và selinux 

```sh
sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/sysconfig/selinux
sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/selinux/config
setenforce 0

systemctl disable firewalld
systemctl stop firewalld
```

- Trỏ DNS của client về DNS server

``vi /etc/sysconfig/network-scripts/ifcfg-ens160``

sửa dòng `DNS1=10.10.10.173`

- Kiểm tra lại

```
[root@rabbitmq ~]# nslookup minhkma.com
Server:		10.10.10.173
Address:	10.10.10.173#53

Name:	minhkma.com
Address: 10.10.10.173

[root@rabbitmq ~]# nslookup influxdb.com
Server:		10.10.10.173
Address:	10.10.10.173#53

Name:	influxdb.com
Address: 10.10.10.174

[root@rabbitmq ~]# nslookup 10.10.10.173
Server:		10.10.10.173
Address:	10.10.10.173#53

173.10.10.10.in-addr.arpa	name = minhkma.com.

[root@rabbitmq ~]# nslookup 10.10.10.174
Server:		10.10.10.173
Address:	10.10.10.173#53

174.10.10.10.in-addr.arpa	name = influxdb.com.

```

- Sử dụng tcpdump để trace DNS:

```
[root@rabbitmq ~]# tcpdump -i ens160 udp port 53 -w dns.pcap
tcpdump: listening on ens160, link-type EN10MB (Ethernet), capture size 262144 bytes
^C4 packets captured
6 packets received by filter
0 packets dropped by kernel
[root@rabbitmq ~]# tcpdump -r dns.pcap 
reading from file dns.pcap, link-type EN10MB (Ethernet)
15:23:24.698700 IP influxdb.com.42119 > minhkma.com.domain: 11212+ A? minhkma.com. (29)
15:23:24.698729 IP influxdb.com.42119 > minhkma.com.domain: 35592+ AAAA? minhkma.com. (29)
15:23:24.698998 IP minhkma.com.domain > influxdb.com.42119: 35592* 0/1/0 (88)
15:23:24.699025 IP minhkma.com.domain > influxdb.com.42119: 11212* 1/1/0 A 10.10.10.173 (59)
```

- client `influxdb.com` sẽ gửi hai bản tin truy vấn địa chỉ ipv4 và ipv6 của tên miền `minhkma.com`
- server gửi cho client địa chỉ ipv4 và ipv6


Sử dụng wireshark để  xem chi tiết từng gói tin hơn

<img src='https://i.imgur.com/WdOt2QE.png'>

- Query:
    
    ``minhkma.com: type A, class IN``

- Answers:

    ``minhkma.com: type A, class IN, addr 10.10.10.173``

OKE =))))
