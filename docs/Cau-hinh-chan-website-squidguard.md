## Hướng dẫn Cấu hình chặn truy cập HTTP/HTTPS trong pfSense

### Menu

- [1. Giới thiệu](#1)
- [2. Các bước tiến hành](#2)
	- [2.1 Tạo CA](#41)
	- [2.2 Cài đặt gói Squid và SquidGuard](#42)
	- [2.3 Cấu hình Squid](#43)
	- [2.4 Cấu hình SquidGuard](#44)
- [3. Hướng dẫn thêm CA lên máy client](#3)
- [4. Kiểm tra](#4)
- [5. Tham khảo](#5)
- [Bonus: Cấu hình chặn HTTPS không cần cài CA](#6)


<a name="1" />
	
### 1. Giới thiệu

[pfSense](https://www.pfsense.org)® là một ứng dụng miễn phí, mã nguồn mở được chỉnh sửa từ bản phân phối FreeBSD; nó có tính năng như một router, một firewall quản lý và cấu hình trên giao diện Web thân thiện. Thêm vào đó, nó còn cung cấp các gói để mở rộng thêm tính năng cho người sử dụng có nhiều tùy chọn trong quá trình sử dụng mà dung lượng tăng lên không đáng kể.

Trong bài viết này, chúng tôi sẽ hướng dẫn các bạn cấu hình một Proxy server sử dụng pfSense để chặn truy cập vào một số website có sử dụng HTTPS (HTTP + SSL) như Facebook, YouTube,... Dưới đây là phần hướng dẫn chi tiết.

<a name="2" />

### 2. Các bước tiến hành

<a name="21" />

#### 2.1 Tạo CA

Như chúng ta đã biết, HTTPS sử dụng CA để xác thực. Chúng ta tạo CA internal này và cài lên các máy mà Firewall quản lý.

Thao tác như sau: **System > Cert. Manager**

<img src="/images/CA-1.png" />

Chọn **CAs > New**

<img src="/images/CA-2.png" />

Điền tên cho CA và chọn **Method: Create an internal Certificate Authority**, sau đó điền đầy đủ thông tin của bạn vào các trường.

<img src="/images/CA-3.png" />

Nhấn vào **SAVE** để lưu lại.

<img src="/images/CA-4.png" />

Thông tin của CA mới được hiển thị.

<img src="/images/CA-5.png" />

Nhấp vào biểu tượng hình sao (**Export CA**) để tải CA về và copy vào các máy client.

<img src="/images/CA-6.png" />

<a name="22" />

#### 2.2 Cài đặt gói Squid và SquidGuard

Trong phần này, chúng ta sẽ cài đặt gói `squid` - một Proxy Server khá nổi tiếng. Thao tác như sau:

- **Bước 1**: Chúng ta vào **System > Package Manager** 

<img src="/images/package-1.png" />

- **Bước 2**: Tìm kiếm gói cài đặt

Chọn tab **Available Package**, gõ `squid` vào ô tìm kiếm và bấm **Search**

<img src="/images/package-2.png" />

- **Bước 3**: Cài đặt 2 gói `squid` và `squidGuard`

	- Chúng cài đặt lần lượt 2 gói `squid` và `squidGuard` bằng cách bấm vào nút `+ Install`. Trong hình, chúng tôi cài mẫu gói `squid`.
	
	- Bấm **Confirm** để xác nhận cho việc cài đặt.
	
	<img src="/images/package-3.png" />

	- Chờ vài phút, `squid` đã cài xong.
	
	<img src="/images/package-4.png" />
	
	- Tương tự, chúng ta tìm kiếm và xác nhận cài đặt gói `squidGuard`.
	
	<img src="/images/package-5.png" />
	
	<img src="/images/package-6.png" />

Chúng ta chuyển sang bước tiếp theo để cấu hình `squid`.

<a name="23" />

#### 2.3 Cấu hình `squid`

Tiếp đến, chúng ta sẽ cấu hình cơ bản cho `squid` - Proxy Server.

- **Bước 1**: Mở phần cấu hình của `squid`

	Trên Menu, chúng ta chọn **Services > Squid Proxy Server**

	<img src="/images/squid-1.png" />

- **Bước 2**: Cấu hình **Local Cache**
	
	Chọn tab **Local Cache**. Tại đây, chúng ta có thể chỉnh dung lượng lưu trữ cho cache, không lưu trữ cache từ IP nào,... Tuy nhiên, chúng tôi sẽ sử dụng những thông số mặc định. Kéo xuống dưới và chọn **SAVE**.
	
	<img src="/images/squid-3.png" />
	
	<img src="/images/squid-4.png" />
	
- **Bước 3**: Cấu hình `squid`

	- Kích hoạt `squid`
	
	Chọn tab **Gerenal**, tích vào ô "**Check to enable the Squid proxy.**"

	<img src="/images/squid-5.png" />
	
	- Cấu hình "Transparent Proxy Settings", tùy chọn này cho phép tất cả các luồng traffic từ client sẽ qua cổng 80 thay vì 3218 mặc định. Vì vậy, không phải khai báo cấu hình trên Client.
	
	<img src="/images/squid-9.png" />
	
	- Bật tính năng **SSL Man In the Middle Filtering** và chọn CA mới tạo ở Bước 1.	
	
	<img src="/images/squid-6.png" />
	
	- Bật tính năng ghi Log và cấu hình log xoay vòng trong 7 ngày.
	
	<img src="/images/squid-7.png" />
	
	- Lưu lại thông tin và cấu hình
	
	<img src="/images/squid-8.png" />
	
<a name="24" />

#### 2.4 Cấu hình `squidGuard`

Sau khi hoàn thành phần cấu hình `squid`, chúng ta sẽ cấu hình `squidGuard` để xác định những danh sách những địa chỉ nào được phép, địa chỉ nào bị cấm truy cập.
	
<img src="/images/sg-1.png" />

- **Bước 1**: Tạo danh sách các website bị cấm truy cập

	Chọn tab **Target categories > Add** 
	
	<img src="/images/sg-2.png" />
	
	Đặt tên cho Category và liệt kê các domain bị chặn. Trong ví dụ này, chúng tôi chặn 2 domain là `facebook.com` và `youtube.com`.
	
	<img src="/images/sg-3.png" />
	
	Cuộn chuột xuống bên dưới tích vào tùy chọn ghi lại Log.
	
	<img src="/images/sg-4.png" />
	
- **Bước 2**: Áp dụng **Deny** cho category

	Chọn tab **Common ACL** và áp dụng chính sách **Deny**

	<img src="/images/sg-5.png" />
	
	Bật tùy chọn ghi log và lưu lại cấu hình.
	
	<img src="/images/sg-6.png" />

- **Bước 3**: Kích hoạt `squidGuard`

	Tích vào tùy chọn và bấm vào `Apply`.
	
	**Lưu ý**: Mỗi khi có thay đổi ở phần **Target categories**, chúng ta phải bấm vào nút `Apply` để áp dụng những thay đổi.
	
	<img src="/images/sg-7.png" />
	
	`squidGuard` đã được kích hoạt.
	
	<img src="/images/sg-8.png" />
	
- **Bước 4**: Cấu hình thêm về `squidGuard` ghi log

	<img src="/images/sg-9.png" />
	
	Bấm **SAVE** để lưu lại.
	
	<img src="/images/sg-10.png" />

<a name="3" />
	
### 3. Hướng dẫn thêm CA lên máy client

#### 3.1 Cài CA lên máy

Thông thường các trình duyệt Web đều sử dụng chung một kho CA. Chúng ta làm theo các bước sau để thêm một CA vào 'kho' đó.

Sau khi copy CA mà chúng ta đã tạo lên client. Chúng ta mở CA lên:

<img src="/images/client1.png" />

Bấm vào **Install Certificate...**

<img src="/images/client2.png" />

Một cửa sổ chào mừng xuất hiện. Bấm **Next** để tiếp tục

<img src="/images/client3.png" />

Chọn **Place all certificates in the following store**, bấm vào **Browse...**

<img src="/images/client4.png" />

Chọn thư mục **Trust Root Certificate Authorities**

<img src="/images/client5.png" />

Bấm **Next** để tiếp tục

<img src="/images/client6.png" />

Bấm **Finish** để hoàn tất quá trình.

<img src="/images/client7.png" />

Bấm **Yes** để xác nhận cảnh báo.

<img src="/images/client8.png" />

Thông báo thêm thành công CA. Bấm **OK** và **OK** để thoát.

<img src="/images/client9.png" /></br>

<img src="/images/client10.png" />

#### 3.2 Cài CA vào Firefox

Khởi động **Firefox**, **Menu > Options** (**Tùy chọn**)

<img src="/images/ff1.png" />

Tìm kiếm từ khóa `ca` và kéo xuống phần **Certificates** (**Chứng thư**) và bấm vào **View Certificate** (**Chứng thư**)

<img src="/images/ff2.png" />

Chọn tab **Authorities** (**Nhà thẩm định**), bấm vào **Import...** (**Nhập**)

<img src="/images/ff3.png" />

Chọn file CA đã tải từ Server**

<img src="/images/ff4.png" />

Đánh dấu các tùy chọn **Trust**

<img src="/images/ff5.png" />

<a name="4" />

### 4. Kiểm tra

- Khi truy cập vào `facebook.com`

<img src="/images/test-1.png" />

- Khi truy cập vào `youtube.com`

<img src="/images/test-2.png" />

- Khi truy cập vào `google.com`
	
<img src="/images/test-3.png" />

<a name="5" />

### Bonus:

- Chọn SSL/MITM Mode: http://prntscr.com/iici4h
- Chọn CA: http://prntscr.com/iicihs