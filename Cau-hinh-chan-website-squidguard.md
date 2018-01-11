## Hướng dẫn Cấu hình chặn truy cập HTTP/HTTPS trong pfSense

### Menu

- [1. Giới thiệu](#1)
- [2. Các bước tiến hành](#2)
	- [2.1 Tạo CA](#41)
	- [2.2 Cài đặt gói Squid và SquidGuard](#42)
	- [2.3 Cấu hình Squid](#43)
	- [2.4 Cấu hình SquidGuard](#44)
- [3. Kiểm tra](#3)
- [4. Tham khảo](#4)


<a name="1" />
	
### 1. Giới thiệu

[pfSense](https://www.pfsense.org)® là một ứng dụng miễn phí, mã nguồn mở được chỉnh sửa từ bản phân phối FreeDSD; nó có tính năng như một router, một firewall quản lý và cấu hình trên giao diện Web thân thiện. Thêm vào đó, nó còn cung cấp các gói để mở rộng thêm tính năng cho người sử dụng có nhiều tùy chọn trong quá trình sử dụng mà dung lượng tăng lên không đáng kể.

Trong bài viết này, chúng tôi sẽ hướng dẫn các bạn cấu hình một Proxy sêrver sử dụng pfSense để chặn truy cập vào một số website có sử dụng HTTPS (HTTP + SSL) như Facebook, YouTube,... Dưới đây là phần hướng dẫn chi tiết.

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


<a name="24" />
#### 2.4