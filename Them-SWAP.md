## Hướng dẫn thêm SWAP vào pfSense

- **Bước 1**: Tạo một Disk ảo bằng lệnh `dd`

	Trong ví dụ, chúng tôi sẽ tạo 1 file với dung lượng 1GB.

	```sh
	dd if=/dev/zero of=/usr/swap1 bs=1m count=1024
	```

- **Bước 2**: Phần quyền cho file

	```sh
	chmod 0600 /usr/swap1
	```

- **Bước 3**: Liệt kê các MD (Memory Device) đang có trên hệ thống

	```sh
	mdconfig -l
	md0
	```

- **Bước 4**: Khai báo swap trong file `/etc/fstab`

	```sh
	echo 'md1	none	swap	sw,file=/usr/swap1	0	0' >> /etc/fstab
	```

	**Chú ý**: `md1` là tên gán cho MD.  

- **Bước 5**: Kích hoạt SWAP tức thời

	```sh
	swapon -aq
	```

	- Xem lại thông tin SWAP
	
	```sh
	[2.4.2-RELEASE][admin@fw.hlcorp.lab]/root: swapinfo -k
	Device          1K-blocks     Used    Avail Capacity
	/dev/gptid/286a4064-d329-11e7-a    419840        0   419840     0%
	/dev/md2          1048576        0  1048576     0%
	Total             1468416        0  1468416     0%
	```
	
### Tham khảo:

- https://www.freebsd.org/doc/handbook/adding-swap-space.html
- https://www.cyberciti.biz/faq/create-a-freebsd-swap-file/
