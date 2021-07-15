# Hướng dẫn cấu hình Pfsense HA

## Mục đích

Tăng khả năng sẵn sàng cho Dịch vụ cung cấp VPS của pfSense. Nếu 1 node down vẫn có 1 node dự phòng.

### Các bước cấu hình

- Cấu hình CARP (Common Address Redundancy Protocol): Giao thức tạo VIP trên Pfsense

- Pfsync : Đồng bộ Firewall Pfsense

- XMLRPC sync : Đồng bộ cấu hình OpenVPN

#### Lưu ý:

Các node pfSense phải có chung 1 phiên bản.

## Phần 1. Mô hình

**Chuẩn bị:**

- Cài đặt 2 VM pfSense: [Hướng dẫn cài đặt](https://github.com/quanganh1996111/pfsense/blob/main/1-install-pfsense.md)

- 1 IP PUBLIC làm VIP cho 2 VM Pfsense

- Mỗi VM gồm 3 dải IP:

    - `vtnet0`: Dải WAN, dải kết nối IP Public.

    - `vtnet1`: Dải LAN, dải kết nối các máy chủ nội bộ cấp VPN.

    - `vtnet2`: Dải SYNC, dải đồng bộ dữ liệu giữ 2 Node Pfsense.

<img src="https://imgur.com/yKNLjGr.png">

**Cấu hình IP của các VM pfSense:**

- HA-pfSense-01:

    - `vtnet0`: 172.16.2.48 - WAN

    - `vtnet1`: 10.10.40.5 - VLAN40

    - `vtnet2`: 10.10.50.5 - SYNC

<img src="https://imgur.com/WduX0OB.png">

- HA-pfSense-01:

    - `vtnet0`: 172.16.2.15 - WAN

    - `vtnet1`: 10.10.41.5 - VLAN40

    - `vtnet2`: 10.10.50.6 - SYNC

<img src="https://imgur.com/pJdhPqD.png">

## Phần 2. Cấu hình pfSync

### Bước 1: Cấu hình Rule cho Interface SYNC

**Interface SYNC** là interface sử dụng để Pfsense đồng bộ dữ liệu qua lại

**Lưu ý:** Thực hiện trên cả HA-pfSense-01 và HA-pfSense-02

Cấu hình Allow All trên `Interface SYNC`

Chọn `Firewall` > `Rules`

<img src="https://imgur.com/BK7gchq.png">

<img src="https://imgur.com/p6urYjg.png">

### Bước 2: Cấu hình High Availability trên node Master

**Thực hiện trên node HA-Pfsense-CMC-1**

Chọn `System` > `High Avail. Sync`

<img src="https://imgur.com/VC1shJR.png">

- Tại `State Synchronization Settings (pfsync)`

    - Chọn `Synchronize states` : `pfsync transfers state insertion, update, and deletion messages between firewalls`.

    - `Synchronize Interface`: `SYNC`

    - `pfsync Synchronize Peer IP`: 10.10.50.6 (IP Sync của HA-pfSense-02)

<img src="https://imgur.com/lf4tbjQ.png">

- Tại `Configuration Synchronization Settings (XMLRPC Sync)`

    - `Synchronize Config to IP`: 10.10.50.6 (IP Sync của HA-pfSense-02)

    - `Remote System Username` : `admin` (Tài khoản truy cập HA-pfSense-02)

    - `Remote System Password` : `xxxxx` (Mật khẩu của tài khoản admin truy cập HA-pfSense-02)

    - Chọn `Toggle All`.

    - Sau đó `Save`.

<img src="https://imgur.com/Z3ymmfp.png">

### Bước 3: Cấu hình High Availability trên node Backup

**Thực hiện trên Node HA-pfSense-02**

Chọn `System` > `High Avail. Sync`

<img src="https://imgur.com/xs1oLH8.png">

- Tại `State Synchronization Settings (pfsync)`

    - Chọn `Synchronize states` : `pfsync transfers state insertion, update, and deletion messages between firewalls`

    - `Synchronize Interface`: `SYNC`

    - `pfsync Synchronize Peer IP` : 10.10.50.5 (IP Sync của HA-pfSense-01)

**Lưu ý**: Noded Backup bỏ qua cấu hình `Configuration Synchronization Settings (XMLRPC Sync)`

<img src="https://imgur.com/0LKyDzh.png">

### Bước 4: Kiểm tra cấu hình đồng bộ

**Thực hiện trên node HA-pfSense-01**

Chọn `Firewall` > `Aliases`

<img src="https://imgur.com/aC7bvPe.png">

<img src="https://imgur.com/DBbxOx4.png">

Nhập `name` tùy ý:

<img src="https://imgur.com/pZPt4Qi.png">

Kết quả ở Node 1 HA-pfSense-01:

<img src="https://imgur.com/Beg11R5.png">

Kiểm tra trên Node 2 HA-pfSense-02:

<img src="https://imgur.com/6sbnTPC.png">

<img src="https://imgur.com/PmqfyVY.png">

**Lưu ý:**

- Nếu cấu hình đúng, khi tạo mới config Firewall, VPN trên node HA-Pfsense-CMC-1 các cấu hình sẽ tự được đồng bộ sang HA-Pfsense-CMC-2

- Nếu cấu hình sai, kiểm tra cấu hình Firewall interface SYNC, hoặc kiểm tra log hệ thống

<img src="https://imgur.com/yoLD7Gf.png">

<img src="https://imgur.com/XTJfco9.png">