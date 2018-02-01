## Hướng dẫn thêm SWAP vào pfSense

- **Bước 1**: Tạo một Disk ảo bằng lệnh `dd`

Trong ví dụ, chúng tôi sẽ tạo 1 file với dung lượng 1GB.

```sh
dd if=/dev/zero of=/usr/swap0 bs=1m count=1024
```

- **Bước 2**: Phần quyền cho file

```sh
chmod 0600 /usr/swap0
```

- **Bước 3**: Khai báo swap trong file `/etc/rc.conf`

```sh
echo 'swapfile="/usr/swap0"' >> /etc/rc.conf
```

- **Bước 4**: Xem thông tin swap đang sử dụng trên hệ thống

```sh
swapinfo

Device          1K-blocks     Used    Avail Capacity
/dev/gptid/286a4064-d329-11e7-a    419840        0   419840     0%
Total             419840        0  419840     0%
```

- **Bước 5**: Liệt kê các MD (Memory Device) đang có trên hệ thống

```sh
mdconfig -l
md0 md1
```

- **Bước 6**: Kích hoạt MD và SWAP

Nhìn vào thông tin ở **Bước 4**, chúng ta thấy đã có 1 file SWAP trên hệ thống. Vì vậy ở câu lệnh này tùy chọn `-u` phải là `1`, đây là slot của swap.

```sh
mdconfig -a -t vnode -f /usr/swap0 -u 1 && swapon /dev/md1
```

### Tham khảo:

- https://www.freebsd.org/doc/handbook/adding-swap-space.html
- https://www.cyberciti.biz/faq/create-a-freebsd-swap-file/
