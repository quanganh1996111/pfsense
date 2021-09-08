# Cài đặt Pfsense cơ bản

## Phần 1: Chuẩn bị

### Tài file iso của pfsense

Tải file iso từ trang https://www.pfsense.org/download/

<img src="https://imgur.com/5TY2sgo.png">

Sau khi tải xong, tiến hành upload file iso lên Node KVM muốn tạo.

<img src="https://imgur.com/3eLMLD9.png">

### Chuẩn bị VM

Tài nguyên cấp cho Pfsense:

- RAM: 2GB.

- CPU: 2.

- Disk: 20GB.

Chuẩn bị VM để cài đặt Pfsense:

- Chuẩn bị 2 Card mạng: 1 đường LAN và 1 đường WAN (kết nối Internet).

- Đường WAN sẽ sử dụng để Client kết nối tới.

- Đường LAN để quản trị, có quyền SSH tới các VM thông qua LAN.

Mô hình lab bao gồm:

- Đường WAN sẽ đặt IP: 172.16.2.14

<img src="https://imgur.com/ur0HQrN">

- Dải VLAN40: 10.10.40.x đã được cấu hình trong bài: [Cấu hình VLAN với Bridge trên KVM](https://github.com/quanganh1996111/KVM/blob/main/documents/2-kvm-network/3-vlan-bridge.md)

## Phần 2. Cài đặt Pfsense

<img src="https://imgur.com/UfncOqR.png">

<img src="https://imgur.com/p3u73FW.png">

<img src="https://imgur.com/EXbL3cr.png">

<img src="https://imgur.com/Nfo57jJ.png">

<img src="https://imgur.com/8HFFyhe.png">

<img src="https://imgur.com/FQGBFdh.png">

Sau khi hoàn thành các bước trên, đợi 1-2 phút để VM khởi động lại.

## Phần 3: Thiết lập cấu hình cho Pfsense

### Bước 1: Bỏ qua thiết lập tự động cho VLANs

<img src="https://imgur.com/9JOEpei.png">

### Bước 2: Cấu hình VLAN thủ công

Bước này ta chọn cấu hình cho `vtnet0`

<img src="https://imgur.com/eJb7ObI.png">

Bỏ qua bước cấu hình cho `vtnet1`

<img src="https://imgur.com/HKhwQj2.png">

<img src="https://imgur.com/6vynvu2.png">

Sau đó đợi để Pfsense thiết lập cấu hình cơ bản.

### Bước 3: Cấu hình IP tĩnh cho đường WAN

- Chọn tùy chọn `2` để cấu hình IP

<img src="https://imgur.com/5aFMB8b.png">

```

- Đặt IP:

> 172.16.2.14

Đặt Prefix

>20

- Đặt Gateway:

>172.16.10.1

```

### Bước 4: Bỏ qua Cấu hình IPv6

<img src="https://imgur.com/EE3WTai.png">

<img src="https://imgur.com/veH3TgF.png">

### Bước 5: Thiết lập revert to HTTP

<img src="https://imgur.com/pOSMZfU.png">

### Thông báo thiết lập cấu hình thành công

<img src="https://imgur.com/kIwKgad.png">

Giao diện Pfsense sau khi hoàn thành:

Tài khoản mặc định để truy cập là `admin / pfsense`

<img src="https://imgur.com/ofNSYlK.png">

## Phần 4. Thiết lập cấu hình Pfsense qua giao diện.

<img src="https://imgur.com/0nIK7qG.png">

<img src="https://imgur.com/kefNIKJ.png">

- Thiết lập DNS Google:

<img src="https://imgur.com/FZzuynK.png">

- Thiết lập Timezone:

<img src="https://imgur.com/GmxJj3J.png">

- Chọn Next do đã cấu hình IP trước đó:

<img src="https://imgur.com/BXQkEYf.png">

- Thay đổi mật khẩu cho tài khoản `admin`:

<img src="https://imgur.com/94eg1p1.png">

- Reload lại để cập nhật thiết lập trước đó:

<img src="https://imgur.com/qHZfoa7.png">

- Thông báo thiết lập hoàn tất:

<img src="https://imgur.com/7G19Xow.png">

- Giao diện màn hình Dashboard của Pfsense:

<img src="https://imgur.com/36IfiQe.png">

### Tắt Hardware Checksum Offloading

<img src="https://imgur.com/mXmwp6N.png">

<img src="https://imgur.com/czRAqMO.png">

<img src="https://imgur.com/gTzM59q.png">

### Bổ sung Rule Firewall cho phép kết nối giao diện Pfsense qua IP Public

- Cho phép kết nối qua HTTP:

<img src="https://imgur.com/IqW1YX4.png">

<img src="https://imgur.com/ejHlASC.png">

<img src="https://imgur.com/b0XFDwt.png">

- Cho phép kết nối qua HTTPS:

<img src="https://imgur.com/fiCYJVS.png">

<img src="https://imgur.com/lza86ah.png">

- Chọn `Apply Changes` để lưu các thiết lập:

<img src="https://imgur.com/cJjKhfE.png">

<img src="https://imgur.com/bHtvXbn.png">

### Chuyển kết nối dashboard mặc định Pfsense tới HTTPS

<img src="https://imgur.com/kZVebOw.png">

<img src="https://imgur.com/ggy5mEa.png">

<img src="https://imgur.com/XdLPEjp.png">

### Enable card mạng LAN

<img src="https://imgur.com/rwxJOGu.png">

- Add card mạng `vtnet1`:

<img src="https://imgur.com/9WjfE93.png">

<img src="https://imgur.com/bTgXitN.png">

- Đặt IPv4 cho card mạng LAN: 10.10.40.5 sau đó `Save` lại:

<img src="https://imgur.com/PphXGZw.png">

- `Apply Changes` để xác nhận cấu hình:

<img src="https://imgur.com/RlYYyF1.png">

- Quay lại màn hình Dashboard đã nhận IP của 2 card mạng:

<img src="https://imgur.com/sQEu5Kw.png">