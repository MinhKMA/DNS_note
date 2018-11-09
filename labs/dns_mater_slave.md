# Mô hình

<img src='https://i.imgur.com/NoFB20I.png'>

Mô tả:

- DNS master server:
    + IP: 10.10.11.171
    + OS: CentOS 7
    + name: master.com

- DNS slave server:
    + IP: 10.10.11.172
    + OS: CentOS 7
    + name: slave.com

- Client
    + IP: 10.10.11.173
    + OS: CentOS 7
    + name: client.com

# Các bước cài đặt: 

- Vì trên môi trường lab nên tôi đã tắt selinux và firewalld

    ```
    sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/sysconfig/selinux
    sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/selinux/config
    setenforce 0

    systemctl stop firewalld
    systemctl disable firewalld
    ```
- Nếu bạn sử dụng firewalld thì phải mở port 53 cho giao thức DNS hoạt động

    ```
    firewall-cmd --permanent --add-port=53/tcp
    firewall-cmd --reload
    ```
## Cài đặt các gói cần thiết:

- Trên tất cả node DNS master server và DNS slave client:

    - Update:

        ```
        yum update -y
        yum install bind bind-utils -y
        ```

    - Phân quyền: 

        ```
        chgrp named -R /var/named
        chown -v root:named /etc/named.conf
        ```

## Cấu hình node DNS master server:

- Chỉnh sửa file cấu hình:

    `vim /etc/named.conf`

- Như sau:

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
        listen-on port 53 { 127.0.0.1; 10.10.11.171; }; ### Master DNS IP ###
        listen-on-v6 port 53 { ::1; };
        directory 	"/var/named";
        dump-file 	"/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { localhost; 10.10.11.0/24; }; ### IP Range ###
        allow-transfer{ localhost; 10.10.11.172; }; ### Slave DNS IP ###

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

    zone "." IN {
        type hint;
        file "named.ca";
    };

    zone "com" IN {
    type master;
    file "forward.com";
    allow-update { none; };
    };

    zone "10.10.11.in-addr.arpa" IN {
    type master;
    file "reverse.com";
    allow-update { none; };
    };

    include "/etc/named.rfc1912.zones";
    include "/etc/named.root.key";
    ```

- Tạo cở sở dữ liệu cho các zones

    + `forward.com`
    
        ```vim /var/named/forward.com```

    + Thêm các dòng sau 
    
        ```
        $TTL 86400
        @   IN  SOA     master.com. nguyenvanminhkma.gmail.com. (
                2018110903  ;Serial
                3600        ;Refresh
                1800        ;Retry
                604800      ;Expire
                86400       ;Minimum TTL
        )
        @       IN  NS          master.com.
        @       IN  NS          slave.com.
        @       IN  A           10.10.11.171
        @       IN  A           10.10.11.172
        master       IN  A     10.10.11.171
        slave        IN  A     10.10.11.172
        ```

    + `reverse.com`

        ```vim /var/named/reverse.com```

    + Thêm các dòng sau

        ```
        $TTL 86400
        @   IN  SOA     master.com. nguyenvanminhkma.gmail.com. (
                2018110903  ;Serial
                3600        ;Refresh
                1800        ;Retry
                604800      ;Expire
                86400       ;Minimum TTL
        )
        @       IN  NS          master.com.
        @       IN  NS          slave.com.
        @       IN  PTR         com.
        master           IN  A   10.10.11.171
        slave            IN  A   10.10.11.172
        171     IN  PTR         master.com.
        172     IN  PTR         slave.com.
        ```

## Cấu hình node DNS slave server 

- Chỉnh sửa file cấu hình:

    `vim /etc/named.conf`

- Như sau:

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
        listen-on port 53 { 127.0.0.1; 10.10.11.172; };
        listen-on-v6 port 53 { ::1; };
        directory 	"/var/named";
        dump-file 	"/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { localhost; 10.10.11.0/24; };

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

    zone "." IN {
        type hint;
        file "named.ca";
    };

    zone "com" IN {
    type slave;
    file "slaves/forward.com";
    masters { 10.10.11.171; };
    };

    zone "10.10.11.in-addr.arpa" IN {
    type slave;
    file "slaves/reverse.com";
    masters { 10.10.11.171; };
    };

    include "/etc/named.rfc1912.zones";
    include "/etc/named.root.key";
    ```

- Tạo cở sở dữ liệu cho các zones

    ```
    [root@slave ~]# ls /var/named/slaves/
    forward.com  reverse.com
    ```

    + 2 file tương tự như phía DNS master server


## Kiểm tra lại file cấu hình và khởi động dịch vụ:

- DNS master server:

    ```
    [root@master ~]# named-checkconf /etc/named.conf
    [root@master ~]# named-checkzone com /var/named/forward.com
    zone com/IN: loaded serial 2018110903
    OK
    [root@master ~]# named-checkzone com /var/named/reverse.com
    zone com/IN: loaded serial 2018110903
    OK
    ```

- DNS slave server:

    ```
    [root@slave ~]# named-checkconf /etc/named.conf 
    [root@slave ~]# named-checkzone com /var/named/slaves/reverse.com
    zone com/IN: loaded serial 2018110903
    OK
    [root@slave ~]# named-checkzone com /var/named/slaves/forward.com
    zone com/IN: loaded serial 2018110903
    OK
    ```
- Nếu đã OK hết, Trên tất cả node master và slave:

    ```
    systemctl enable named
    systemctl start named
    ```

## Cập nhập một recode mới trên node master

- Trên node master update zones:

    + Update `forward.com`

        ```vim /var/named/forward.com```

        ```
        $TTL 86400
        @   IN  SOA     master.com. nguyenvanminhkma.gmail.com. (
                2018110904  ;Serial
                3600        ;Refresh
                1800        ;Retry
                604800      ;Expire
                86400       ;Minimum TTL
        )
        @       IN  NS          master.com.
        @       IN  NS          slave.com.
        @       IN  A           10.10.11.171
        @       IN  A           10.10.11.172
        @       IN  A           10.10.11.173
        master       IN  A     10.10.11.171
        slave        IN  A     10.10.11.172
        client       IN  A     10.10.11.173
        ```

    + Update `reverse.com`

        ```vim /var/named/reverse.com```
        ```
        $TTL 86400
        @   IN  SOA     master.com. nguyenvanminhkma.gmail.com. (
                2018110904  ;Serial
                3600        ;Refresh
                1800        ;Retry
                604800      ;Expire
                86400       ;Minimum TTL
        )
        @       IN  NS          master.com.
        @       IN  NS          slave.com.
        @       IN  PTR         com.
        master           IN  A   10.10.11.171
        slave            IN  A   10.10.11.172
        client           IN  A   10.10.11.173
        171     IN  PTR         master.com.
        172     IN  PTR         slave.com.
        173     IN  PTR         client.com.
        ```
    
    + Thêm record của client và sửa thông Serial
    + Thông số  Serial theo format `năm/tháng/ngày/lần_thay_đổi_thứ_mấy`

- Kiểm tra log message của nodo slave xem đã transfer update từ master về chưa

    <img src="https://i.imgur.com/L4h52vm.png">

- Như vậy là đã thành công. 

