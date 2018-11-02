### Khi bạn gõ `https://dantri.com.vn/` vào thanh tìm kiếm trên google chrome

ip client: 10.10.10.174
doman client: influxdb.com
ip dns server local: 10.10.10.173
doman server: minhkma.com

<img src='https://i.imgur.com/YSat6Fr.png'>

- Client query ipv4 và ipv6 của tên miền `dantri.com.vn` 

<img src='https://i.imgur.com/skB1AJX.png'>

- Trên DNS server local chưa có thông tin của tên miền `dantri.com.vn` nên nó sẽ tiếp tục query đển dns root để hỏi thông tin.

```
16:04:02.194744 IP minhkma.com.31357 > M.ROOT-SERVERS.NET.domain: 34763% [1au] NS? . (28)
16:04:02.194838 IP minhkma.com.43583 > M.ROOT-SERVERS.NET.domain: 65507% [1au] A? dantri.com.vn. (42)
16:04:02.195004 IP minhkma.com.21549 > M.ROOT-SERVERS.NET.domain: 54637% [1au] AAAA? dantri.com.vn. (42)

```
<img src='https://i.imgur.com/WDw1Mx1.png'>

- Ở đây root server có địa chỉ IP là `202.12.27.33`

- Root server trả về thông tin tên miền `dantri.com.vn` được quản lý bởi các máy chủ:

```

    Authoritative nameservers
        vn: type NS, class IN, ns g.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            Name Server: g.dns-servers.vn
        vn: type NS, class IN, ns a.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: a.dns-servers.vn
        vn: type NS, class IN, ns c.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: c.dns-servers.vn
        vn: type NS, class IN, ns f.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: f.dns-servers.vn
        vn: type NS, class IN, ns e.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: e.dns-servers.vn
        vn: type NS, class IN, ns d.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: d.dns-servers.vn
        vn: type NS, class IN, ns b.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: b.dns-servers.vn
        vn: type DS, class IN
            Name: vn
            Type: DS(Delegation Signer) (43)
            Class: IN (0x0001)
            Time to live: 86400
            Data length: 24
            Key id: 0xba0b
            Algorithm: RSA/SHA-256 (8)
            Digest Type: SHA-1 (1)
            Digest: 0457d87762782ccc3d355d6aa87e28c37fb293c9
        vn: type DS, class IN
            Name: vn
            Type: DS(Delegation Signer) (43)
            Class: IN (0x0001)
            Time to live: 86400
            Data length: 36
            Key id: 0xba0b
            Algorithm: RSA/SHA-256 (8)
            Digest Type: SHA-256 (2)
            Digest: 009f879f8dbab6a453eaccbff323354e72715e4ba8f77131...
        vn: type DS, class IN
            Name: vn
            Type: DS(Delegation Signer) (43)
            Class: IN (0x0001)
            Time to live: 86400
            Data length: 24
            Key id: 0x0cbc
            Algorithm: RSA/SHA-256 (8)
            Digest Type: SHA-1 (1)
            Digest: 913c652b4006deac045b945f4ffcbc1065645421
        vn: type DS, class IN
            Name: vn
            Type: DS(Delegation Signer) (43)
            Class: IN (0x0001)
            Time to live: 86400
            Data length: 36
            Key id: 0x0cbc
            Algorithm: RSA/SHA-256 (8)
            Digest Type: SHA-256 (2)
            Digest: 5a58c19af266077ffe16c2668812796fa8193661af569dbc...
        vn: type RRSIG, class IN
            Name: vn
            Type: RRSIG (46)
            Class: IN (0x0001)
            Time to live: 86400
            Data length: 275
            Type Covered: DS(Delegation Signer) (43)
            Algorithm: RSA/SHA-256 (8)
            Labels: 1
            Original TTL: 86400 (1 day)
            Signature Expiration: Nov 15, 2018 12:00:00.000000000 +07
            Signature Inception: Nov  2, 2018 11:00:00.000000000 +07
            Key Tag: 2134
            Signer's name: <Root>
            Signature: f069a4db905f767b55f154d8775076a93d6952fe977e49b3...
    Additional records
        a.dns-servers.vn: type A, class IN, addr 194.0.1.18
            Name: a.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 194.0.1.18
        b.dns-servers.vn: type A, class IN, addr 203.119.73.105
            Name: b.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 203.119.73.105
        c.dns-servers.vn: type A, class IN, addr 203.119.38.105
            Name: c.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 203.119.38.105
        d.dns-servers.vn: type A, class IN, addr 203.119.44.105
            Name: d.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 203.119.44.105
        e.dns-servers.vn: type A, class IN, addr 203.119.60.105
            Name: e.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 203.119.60.105
        f.dns-servers.vn: type A, class IN, addr 203.119.68.105
            Name: f.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 203.119.68.105
        g.dns-servers.vn: type A, class IN, addr 204.61.216.115
            Name: g.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 204.61.216.115
        a.dns-servers.vn: type AAAA, class IN, addr 2001:678:4::12
            Name: a.dns-servers.vn
            Type: AAAA (IPv6 Address) (28)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            AAAA Address: 2001:678:4::12
        b.dns-servers.vn: type AAAA, class IN, addr 2001:dc8:1:2::105
            Name: b.dns-servers.vn
            Type: AAAA (IPv6 Address) (28)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            AAAA Address: 2001:dc8:1:2::105
        c.dns-servers.vn: type AAAA, class IN, addr 2001:dc8:c000:7::105
            Name: c.dns-servers.vn
            Type: AAAA (IPv6 Address) (28)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            AAAA Address: 2001:dc8:c000:7::105
        f.dns-servers.vn: type AAAA, class IN, addr 2001:dc8:d000:2::105
            Name: f.dns-servers.vn
            Type: AAAA (IPv6 Address) (28)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            AAAA Address: 2001:dc8:d000:2::105
        g.dns-servers.vn: type AAAA, class IN, addr 2001:500:14:6115:ad::1
            Name: g.dns-servers.vn
            Type: AAAA (IPv6 Address) (28)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            AAAA Address: 2001:500:14:6115:ad::1
        <Root>: type OPT
            Name: <Root>
            Type: OPT (41)
            UDP payload size: 4096
            Higher bits in extended RCODE: 0x00
            EDNS0 version: 0
            Z: 0x8000
                1... .... .... .... = DO bit: Accepts DNSSEC security RRs
                .000 0000 0000 0000 = Reserved: 0x0000
            Data length: 0
```

- DNS server local tiếp tục query tới máy chủ `d.dns-servers.vn.domain` có địa chỉ ip `203.119.44.105` 

<img src='https://i.imgur.com/LYgO4ja.png'>

- Máy chủ `d.dns-servers.vn.domain` trả về cho DNS server local thống tin authoritative nameservers

<img src='https://i.imgur.com/O3DknzW.png'>

- DNS server tìm kiếm địa chỉ IP của máy chủ `ns7.synerfy.vn` có địa chỉ IP là `123.30.51.247` từ máy chủ `ns2.vdconline.vn` có địa chỉ IP`123.30.176.243`

<img src='https://i.imgur.com/Y6qW1FD.png'>

- DNS server query `dantri.com.vn` với máy chủ `ns7.synerfy.vn` có địa chỉ IP là `123.30.51.247` vừa có. `ns7.synerfy.vn` trả lời lại địa chỉ ipv4 của `dantri.com.vn`

```
Frame 39: 258 bytes on wire (2064 bits), 258 bytes captured (2064 bits)
    Encapsulation type: Ethernet (1)
    Arrival Time: Nov  2, 2018 16:04:02.444771000 +07
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1541149442.444771000 seconds
    [Time delta from previous captured frame: 0.000398000 seconds]
    [Time delta from previous displayed frame: 0.000398000 seconds]
    [Time since reference or first frame: 0.251647000 seconds]
    Frame Number: 39
    Frame Length: 258 bytes (2064 bits)
    Capture Length: 258 bytes (2064 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: eth:ethertype:ip:udp:dns]
    [Coloring Rule Name: UDP]
    [Coloring Rule String: udp]
Ethernet II, Src: Vmware_22:dd:cf (00:0c:29:22:dd:cf), Dst: Vmware_b7:29:3c (00:50:56:b7:29:3c)
Internet Protocol Version 4, Src: 123.30.51.247, Dst: 10.10.10.173
User Datagram Protocol, Src Port: 53, Dst Port: 2764
Domain Name System (response)
    Transaction ID: 0x3d33
    Flags: 0x8400 Standard query response, No error
    Questions: 1
    Answer RRs: 4
    Authority RRs: 3
    Additional RRs: 4
    Queries
    Answers
        dantri.com.vn: type A, class IN, addr 123.30.151.72
            Name: dantri.com.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 300
            Data length: 4
            Address: 123.30.151.72
        dantri.com.vn: type A, class IN, addr 123.30.151.72
            Name: dantri.com.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 300
            Data length: 4
            Address: 123.30.151.72
        dantri.com.vn: type A, class IN, addr 14.225.10.14
            Name: dantri.com.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 300
            Data length: 4
            Address: 14.225.10.14
        dantri.com.vn: type A, class IN, addr 123.30.151.72
            Name: dantri.com.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 300
            Data length: 4
            Address: 123.30.151.72
    Authoritative nameservers
        dantri.com.vn: type NS, class IN, ns ns7.synerfy.vn
            Name: dantri.com.vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 14
            Name Server: ns7.synerfy.vn
        dantri.com.vn: type NS, class IN, ns ns8.synerfy.vn
            Name: dantri.com.vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 6
            Name Server: ns8.synerfy.vn
        dantri.com.vn: type NS, class IN, ns ns6.synerfy.vn
            Name: dantri.com.vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 6
            Name Server: ns6.synerfy.vn
    Additional records
        ns7.synerfy.vn: type A, class IN, addr 123.30.51.247
            Name: ns7.synerfy.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 4
            Address: 123.30.51.247
        ns8.synerfy.vn: type A, class IN, addr 210.245.87.125
            Name: ns8.synerfy.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 4
            Address: 210.245.87.125
        ns6.synerfy.vn: type A, class IN, addr 173.193.24.114
            Name: ns6.synerfy.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 4
            Address: 173.193.24.114
        <Root>: type OPT
            Name: <Root>
            Type: OPT (41)
            UDP payload size: 1024
            Higher bits in extended RCODE: 0x00
            EDNS0 version: 0
            Z: 0x0000
                0... .... .... .... = DO bit: Cannot handle DNSSEC security RRs
                .000 0000 0000 0000 = Reserved: 0x0000
            Data length: 0
    [Request In: 33]
    [Time: 0.002556000 seconds]
```

- Cuối cùng DNS server local trả lại thông tin cho client thông tin tên miền `dantri.com.vn`

```
51	0.400378	10.10.10.173	10.10.10.174	DNS	167	Standard query response 0x65d5 A dantri.com.vn A 123.30.151.72 A 14.225.10.14 NS ns7.synerfy.vn NS ns6.synerfy.vn NS ns8.synerfy.vn
```
