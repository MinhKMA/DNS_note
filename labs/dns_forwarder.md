# DNS forwarding:

- [1. Khái niệm](#1)
- [2. Tại sao lại cần có DNS forwarding](#2)
    + [2.1 Recursive Name Query](#2.1)
    + [2.2 Iterative Name Query](#2.2)
    + [2.3 Ưu điểm khi sử dụng DNS forwarding](#2.3)
- [3. Phân tích gói tin khi có DNS forwarding](#3)

Đôi khi chúng ta có sự nhầm lẫn giữa chuyển tiếp DNS và chuyển hướng HTTP hoặc sử dụng bản ghi CNAME để chỉ định bí danh DNS.

```
DNS forwarding exclusively refers to the process where specific DNS requests are forwarded to a designated DNS server for resolution.
```

Và nó không phải là giải pháp để chuyển hướng miền này sang tên miền khác, mà bạn sẽ sử dụng điều hướng HTTP. Cũng không hữu ích khi tạo bí danh subdomain cho tên miền khác: đó là công việc của bản ghi CNAME

<a name="1"></a>
## 1. Khái niệm:

DNS forwarding là quá trình các truy vấn được xử lý bởi một máy chủ được chỉ định thay vì xử lý trên máy chủ DNS server local. Thông thường các máy chủ DNS xử lý phân giải địa chỉ trong mạng lan được cấu hình forward requests tới một máy chủ DNS bên ngoài chuyên để xử lý. 

Khi quyết định cách allocate DNS resources trong một mạng, điều quan trọng là phải tách biệt giữa external và internal DNS. Vì tất các DNS server được cấu hình để xử lý phân giải cả external và internal sẽ làm ảnh hưởng đến hiệu năng và security cho network.

<a name="2"></a>
## 2. Tại sao lại cần có DNS forwarding

Trước tiên DNS name queries có hai cách thức: đệ quy và lặp lại. (recursive and iterative)

Trong lúc phân tích các cách thức truy vấn tôi có đề cập đến các khái niệm DNS server, DNS root, DNS authoritative

- `Root server` : Chúng là một bộ máy chủ định danh cố định duy trì danh sách các máy chủ định danh (master/slave) có thẩm quyền cho mọi miền đã đăng ký.

- `Authoritative server` : Máy chủ master/slave phụ cho một miền cụ thể đã được cấu hình bởi quản trị viên có thông tin tên máy chủ cho tên miền đó. Thông tin về các máy chủ này được thêm vào các máy chủ gốc khi tên miền được đăng ký.

- `DNS server`: Chính là máy chủ DNS trong local dùng để caching và forwarding kết quả và truy vấn. 
<a name="2.1"></a>
### 2.1 Recursive Name Query

Truy vấn đệ qui là một loại truy vấn, trong đó máy chủ DNS nhận truy vấn của bạn sẽ xử lý tất cả các công việc của việc tìm kiếm kết qủa và trả về cho bạn. Trong quá trình xử lý, DNS server có thể truy vấn đến nhiều máy chủ DNS ngoài internet thay bạn để tìm kiếm kết quả.

Hãy cùng tìm hiểu quá trình truy vấn đệ qui thông qua ví dụ sau: 

Giả sử bạn muốn trình duyệt tìm kiếm `www.example.com` và file `resolve.conf` khai báo như sau:

```
[root@myvm ~]# cat /etc/resolv.conf
nameserver 172.16.200.30
nameserver 172.16.200.31
```

File `resolve.conf` có nghĩa là máy chủ DNS có địa chỉ ip là 172.16.200.30 & 31. Dù bạn sử dụng ứng dụng nào, hệ điều hành sẽ gửi các truy vấn DNS tới hai máy chủ DNS đó.

- Bước 1: Khi bạn nhập `www.example.com` trong trình duyệt. Vì vậy, trình giải quyết của hệ điều hành sẽ gửi một truy vấn DNS cho bản ghi A tới máy chủ DNS 172.16.200.30.

- Bước 2: Máy chủ DNS 172.16.200.30 khi nhận được truy vấn sẽ tới file cache để tìm kiếm địa chỉ IP cho domain `www.example.com`. Nhưng nó không tồn tại :(

- Bước 3: Vì câu trả lời không có sẵn trong máy chủ DNS 172.16.200.30, Máy chủ này gửi một truy vấn tới một trong những máy chủ DNS root để tìm kiếm kết quả. ***Now an important fact to note here is that root server's are always iterative servers.***

- Bước 4: Máy chủ DNS root sẽ trả về một danh sách máy chủ chịu trách nhiệm xử lý .COM gTLD's.

- Bước 5: Máy chủ DNS 172.16.200.30 sẽ chọn một trong những máy chủ .COM gTLD từ danh sách máy chủ của DNS root giới thiệu để truy vấn kết qủa cho `www.example.com`

- Bước 6: Tương tự như máy chủ root, các máy chủ gTLD là thực hiện các truy vấn lặp. Vì vậy, nó trả lại máy chủ DNS 172.16.200.30 với danh sách các địa chỉ IP của máy chủ DNS chịu trách nhiệm về tên miền (máy chủ tên có thẩm quyền cho tên miền) www.example.com.

- Bước 7: Lần này cũng máy chủ DNS của chúng tôi sẽ chọn một trong các IP từ danh sách máy chủ định danh được cung cấp và truy vấn bản ghi A cho www.example.com. Máy chủ tên có thẩm quyền được truy vấn, sẽ trả lời lại với bản ghi A như sau:

    ```www.example.com = <XXX:XX:XX:XX> (Some IP address)```

- Bước 8: Máy chủ DNS của chúng tôi 172.16.200.30 sẽ trả về máy client của bạn với cặp tên miền và ip (và bất kỳ thông tin nào khác nếu có). Bây giờ trình duyệt sẽ gửi yêu cầu đến địa chỉ IP đã cho, cho trang web `www.example.com`

Biểu đồ dưới đây có thể làm cho khái niệm rõ ràng.

<img src="https://i.imgur.com/i07G1m7.png">

Như bạn có thể thấy từ hình trên. Máy chủ DNS của tôi (172.16.200.30) truy vấn thông qua các máy chủ DNS khác thay cho client của tôi.

**Note: Bạn cũng thể tắt truy vấn đệ qui trong file `named.conf ở` option `recursion no`**

```Làm thế nào để DNS server chọn một trong danh sách servers được gửi để truy vấn?```

- Ở trường hợp trên, bạn có thể thấy máy chủ DNS 172.16.200.30 đã phải chọn một máy chủ từ một danh sách để truy vấn.

- Ví dụ có 13 máy chủ root (UHMM! khi tôi nói 13 máy chủ root, 13 là số lượng địa chỉ phổ dụng. Có hàng trăm máy chủ tại các vị trí khác nhau trên thế giới. Các địa chỉ máy chủ root này là anycasted addresses.), máy chủ root sẽ được truy vấn, cho câu trả lời?

- Hầu hết các máy chủ DNS sử dụng một thuật toán để chọn một trong danh sách các máy chủ để phân phối thời giản tải và phản hồi.

- BIND là một phần mềm được sử dụng rộng rãi và phổ biến nhất. Nó sử dụng một công nghệ gọi là RTT metric(Round Trip Time metric). Sử dụng kỹ thuật này, máy chủ sẽ kiểm tra RTT mỗi máy chủ root và chọn máy chủ có RTT thấp hơn.

<a name="2.2"></a>
### 2.2 Iterative Name Query

Trước khi bắt đầu giải thích cho truy vấn lặp lại. Một điều quan trọng cần lưu ý là, tất cả các máy chủ DNS phải hỗ trợ truy vấn lặp lại (không phải đệ quy).

Trong truy vấn lặp, máy chủ thay vì tìm kiếm kết quả cuối cho truy vấn của bạn mà sẽ trả về một referral đến máy chủ DNS khác có thể tìm kiếm được kết quả. 

Nhưng nếu máy chủ DNS 172.16.200.30 không phải là một máy chủ đệ quy (có nghĩa là nó lặp đi lặp lại), nó sẽ cho chúng ta câu trả lời nếu nó có trong hồ sơ của nó. Nếu không thì  sẽ giới thiệu cho tôi các máy chủ root (nó sẽ không tự truy vấn máy chủ root và các máy chủ khác).

Hãy cùng xem các bước sau:

- Bước 1: Khi bạn nhập `www.example.com` trong trình duyệt. Vì vậy, trình giải quyết của hệ điều hành sẽ gửi một truy vấn DNS cho bản ghi A tới máy chủ DNS 172.16.200.30.

- Bước 2: Máy chủ DNS 172.16.200.30 khi nhận được truy vấn sẽ tới file cache để tìm kiếm địa chỉ IP cho domain `www.example.com`. Nhưng nó không tồn tại :(

- Bước 3: Bây giờ thay vì truy vấn máy chủ root, máy chủ DNS của bạn sẽ trả lời bạn với giới thiệu đến máy chủ root. Bây giờ trình hệ điều hành của bạn, sẽ truy vấn các máy chủ root để tìm câu trả lời.

- Bây giờ các bước còn lại đều giống nhau. Sự khác biệt duy nhất trong truy vấn lặp lại là

    + Nếu máy chủ DNS không có kết quả, nó sẽ không truy vấn bất kỳ máy chủ nào khác để có kết quả, nhưng thay vào đó nó sẽ trả lại thông tin máy chủ DNS root
    + Trong truy vấn lặp lại, công việc tìm kiếm kết quả sẽ nằm ở resolver của hệ điều hành của bạn. 

<img src="https://i.imgur.com/o5hxioe.png">

<a name="2.3"></a>
### Tại sao lại cần DNS forwarding

- Thông tin DNS server local có thể được hiển thị trên Internet. Sẽ tốt hơn nếu có sự tách biệt chặt chẽ giữa DNS nội bộ và bên ngoài. Việc phơi bày các DNS server local trên Internet mở sẽ tạo ra lỗ hổng bảo mật và quyền riêng tư tiềm ẩn.

- Không có DNS forwarding, tất cả các máy chủ DNS sẽ truy vấn các giải pháp DNS bên ngoài nếu chúng không có địa chỉ truy vấn được lưu trong bộ nhớ cache. Điều này có thể dẫn đến lưu lượng truy cập mạng quá mức. Bằng cách chỉ định máy chủ DNS như một forwarder, máy chủ đó chịu trách nhiệm cho tất cả các giải pháp DNS bên ngoài và có thể xây dựng bộ nhớ cache của các địa chỉ bên ngoài, giảm nhu cầu truy vấn phân giải đệ quy và giảm lưu lượng truy cập. Đối với các công ty nhỏ hơn với băng thông có sẵn hạn chế, DNS forwarding có thể tăng hiệu quả của network bằng cách giảm mức sử dụng băng thông và cải thiện tốc độ đáp ứng của requests DNS.

<a name="3"></a>
## 3. Phân tích gói tin:

Bắt gói tin để kiểm chứng:

- Mô hình:

    + DNS server: 10.10.11.171
    + DNS client: 10.10.11.172
    + DNS forwarding: 10.10.11.173

- Kịch bản: Đứng trên máy client query `dantri.com.vn`

- Trên DNS server local:

```
tcpdump -i ens160 udp port 53
10:54:44.988678 IP 10.10.11.172.53803 > master.domain: 18118+ A? dantri.com.vn. (31)
10:54:44.988981 IP 10.10.11.173.38294 > google-public-dns-a.google.com.domain: 18700+ PTR? 172.11.10.10.in-addr.arpa. (43)
10:54:44.989878 IP master.30949 > 10.10.11.173.domain: 41063+% [1au] A? dantri.com.vn. (42)
10:54:44.990354 IP master.36913 > google-public-dns-a.google.com.domain: 41272+ PTR? 172.11.10.10.in-addr.arpa. (43)
10:54:44.990885 IP 10.10.11.173.52734 > 72.18.c1ad.ip4.static.sl-reverse.com.domain: 9491% [1au] A? dantri.com.vn. (42)
10:54:45.050693 IP google-public-dns-a.google.com.domain > master.36913: 41272 NXDomain 0/0/0 (43)
10:54:45.050906 IP google-public-dns-a.google.com.domain > 10.10.11.173.38294: 18700 NXDomain 0/0/0 (43)
10:54:45.051497 IP master.34357 > google-public-dns-a.google.com.domain: 19882+ PTR? 114.24.193.173.in-addr.arpa. (45)
10:54:45.051679 IP 10.10.11.173.44149 > google-public-dns-a.google.com.domain: 48579+ PTR? 114.24.193.173.in-addr.arpa. (45)
10:54:45.109979 IP google-public-dns-a.google.com.domain > 10.10.11.173.44149: 48579 1/0/0 PTR 72.18.c1ad.ip4.static.sl-reverse.com. (95)
10:54:45.113045 IP google-public-dns-a.google.com.domain > master.34357: 19882 1/0/0 PTR 72.18.c1ad.ip4.static.sl-reverse.com. (95)
10:54:45.311853 IP 72.18.c1ad.ip4.static.sl-reverse.com.domain > 10.10.11.173.52734: 9491*- 4/3/4 A 123.30.151.72, A 123.30.151.72, A 14.225.10.14, A 123.30.151.72 (216)
10:54:45.312087 IP 10.10.11.173.domain > master.30949: 41063 2/3/4 A 123.30.151.72, A 14.225.10.14 (184)
10:54:45.312817 IP master.domain > 10.10.11.172.53803: 18118 2/7/12 A 123.30.151.72, A 14.225.10.14 (439)
```

- Trên đó chúng ta có thể thấy tất cả các truy vấn DNS server sẽ đẩy hết cho DNS forwarding xử lý. Sau khi DNS forwarding có kết quả sẽ trả lại cho DNS server. 