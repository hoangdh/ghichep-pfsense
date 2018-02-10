## Hướng dẫn giám sát pfSense qua check_mk (OMD)

### Menu

- [1. Giới thiệu](#1)
- [2. Cài đặt check_mk_agent lên pfSense](#2)
- [3. Cấu hình SNMP trên pfSense](#3)
- [4. Thêm pfSense vào check_mk](#4)
- [5. Tham khảo](#5)

<a name="1" />
	
### 1. Giới thiệu

Việc giám sát hiệu năng hệ thống là một điều vô cùng quan trọng. Nó giúp chúng ta biết được tình trạng của các máy chủ trong hệ thống, tự động gửi cảnh báo mỗi khi có lỗi. Trong bài viết này, chúng tôi sẽ hướng dẫn các bạn giám sát pfSense thông qua [check_mk](https://github.com/hoangdh/meditech-ghichep-omd).

<a name="2" />

### 2. Cài đặt check_mk_agent lên pfSense

Sử dụng SSH để cấu hình pfSense.

- **Bước 1**: Cài gói `bash`

	`check_mk_agent` là một script viết bằng `bash`. Như vậy, chúng ta phải cài gói `bash` trên pfSense. 

	```sh
	pkg install -y bash
	```

- **Bước 2**: Tạo 2 thư mục đặc biệt

	```sh
	mkdir -p /opt/bin/
	mkdir -p /opt/etc/xinetd.d
	```
	
- **Bước 3**: Tải agent và phân quyền

	```sh
	curl --output /opt/bin/check_mk_agent 'https://git.mathias-kettner.de/git/?p=check_mk.git;a=blob_plain;f=agents/check_mk_agent.freebsd;hb=HEAD'
	chmod +x /opt/bin/check_mk_agent
	```

- **Bước 4**: Tạo file cấu hình cho `check-mk`

	- Chúng ta tạo file cấu hình tại `/opt/etc/xinetd.d/check_mk`.
	
	```sh
	vi /opt/etc/xinetd.d/check_mk
	```
	
	- Nội dung file

	```sh
	# +------------------------------------------------------------------+
	# |             ____ _               _        __  __ _  __           |
	# |            / ___| |__   ___  ___| | __   |  \/  | |/ /           |
	# |           | |   | '_ \ / _ \/ __| |/ /   | |\/| | ' /            |
	# |           | |___| | | |  __/ (__|   (    | |  | | . \            |
	# |            \____|_| |_|\___|\___|_|\_\___|_|  |_|_|\_\           |
	# |                                                                  |
	# | Copyright Mathias Kettner 2014             mk@mathias-kettner.de |
	# +------------------------------------------------------------------+
	#
	# This file is part of Check_MK.
	# The official homepage is at http://mathias-kettner.de/check_mk.
	#
	# check_mk is free software;  you can redistribute it and/or modify it
	# under the  terms of the  GNU General Public License  as published by
	# the Free Software Foundation in version 2.  check_mk is  distributed
	# in the hope that it will be useful, but WITHOUT ANY WARRANTY;  with-
	# out even the implied warranty of  MERCHANTABILITY  or  FITNESS FOR A
	# PARTICULAR PURPOSE. See the  GNU General Public License for more de-
	# ails.  You should have  received  a copy of the  GNU  General Public
	# License along with GNU Make; see the file  COPYING.  If  not,  write
	# to the Free Software Foundation, Inc., 51 Franklin St,  Fifth Floor,
	# Boston, MA 02110-1301 USA.
	 
	service check_mk
	{
		type           = UNLISTED
		port           = 6556
		socket_type    = stream
		protocol       = tcp
		wait           = no
		user           = root
		server         = /opt/bin/check_mk_agent
	 
		# If you use fully redundant monitoring and poll the client
		# from more then one monitoring servers in parallel you might
		# want to use the agent cache wrapper:&lt;br /&gt;
	 
		#server         = /usr/bin/check_mk_caching_agent
	 
		# configure the IP address(es) of your Nagios server here:
		#only_from      = 127.0.0.1 10.0.20.1 10.0.20.2
	 
		# Don't be too verbose. Don't log every check. This might be
		# commented out for debugging. If this option is commented out
		# the default options will be used for this service.
		log_on_success =
	 
		disable        = no
	}
	```

- **Bước 5**: Chỉnh sửa file cấu hình của "Filter Reload", để thêm file cấu hình của `check_mk`

	Chúng ta tìm đến dòng có chứa ` fclose($xinetd_fd);` và chèn thêm dòng sau vào trước nó trong file cấu hình `/etc/inc/filter.inc`
	
	```sh
	vi /etc/inc/filter.inc 
	```
	
	Thêm dòng sau:
	
	```sh
	fwrite($xinetd_fd, "includedir /opt/etc/xinetd.d"); 
	```
	
	<img src="/images/monitor-1.png" />
	
	Lưu lại và thoát khỏi `vi`.	

- **Bước 6**: Nạp lại các filter thông qua GUI (**Status > Filter Reload**)

	<img src="/images/monitor-2.png" />
	
	<img src="/images/monitor-3.png" />
	
	<img src="/images/monitor-4.png" />

	**Chú ý**: Mở port **6556** của FW để cho phép check_mk có thể truy cập để lấy dữ liệu.

<a name="3" />

### 3. Kích hoạt SNMP trên pfSense

- **Bước 1**: Trên Dashboard chọn **Services > SNMP**

	<img src="/images/snmp1.png" />
	
- **Bước 2**: Khai báo thông tin

	<img src="/images/snmp2.png" />
	
	**Giải thích:**
	
	- `(1)` Kích hoạt SNMP
	- `(2)` Khai báo **System Location** (Tùy chọn)
	- `(3)` Khai báo **System Contact** (Tùy chọn)
	- `(4)` Khai báo **Community String** - QUAN TRỌNG: Chuỗi này sẽ được sử dụng để xác thực ở `check-mk`.

- **Bước 3**: Lưu lại cấu hình

	<img src="/images/snmp3.png" />
	
	Thông báo cấu hình thành công!
	
	<img src="/images/snmp4.png" />
	
<a name="4" />

### 4. Thêm pfSense vào check_mk

Bây giờ, chúng ta đăng nhập vào `check-mk` và thêm một host mới. 

<img src="/images/monitor-5.png" />

Chọn *Agent type* là **Dual: Check_MK Agent + SNMP** và điền `Community String` mà bạn khai báo ở pfSense vào `SNMP credentials`. Sau đó bấm **Save & go to Services**

<img src="/images/monitor-6.png" />

<a name="5" />

### 5. Tham khảo

- https://forum.pfsense.org/index.php?PHPSESSID=1envonkn9s2qss1gg8fb3kd7h2&topic=111517.0
- https://openschoolsolutions.org/pfsense-monitoring-check-mk/