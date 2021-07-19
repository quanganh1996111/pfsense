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

## Phần 3. Cấu hình VIP trên 2 Node PFSENSE

**Thực hiện trên node1 HA-pfSense-01**

### Bước 1: Cấu hình VIP cho đường WAN

Chọn `Firewall` > `Virtual IPs`

<img src="https://imgur.com/DyowDOr.png">

<img src="https://imgur.com/hGSHIuD.png">

- Tại màn tạo với VIP:

    - `Type`: `CARP`

    - `Interface`: `WAN`

    - `Addresses`: 172.16.2.30 /20
    
    - `VHID Group`: 65

    - `Description`: VIP Public

<img src="https://imgur.com/rRhstvz.png">

<img src="https://imgur.com/Y7slNy2.png">

**Lưu ý:** Chỉ cấu hình trên HA-pfSense-01, config sẽ được đồng bộ sang HA-pfSense-02

- Kiểm tra trên Node Backup HA-pfSense-02:

<img src="https://imgur.com/MT3K2lJ.png">

<img src="https://imgur.com/Dh67WH4.png">

### Bước 2: Cấu hình VIP cho đường LAN

Bước này tương tự bước 1, tuy nhiên cần cấu hình 2 Node pfSense chạy dải LAN trên cùng một VLAN trước đó.

Ở bài này đã cấu hình sẵn 2 dải `VLAN40 - 10.10.40.x/24` cho HA-pfSense-01 và `VLAN41 - 10.10.41.x/24` cho HA-pfSense-01 nên không cấu hình được.

### Bước 3: Kiểm tra trạng thái VIP

- Thực hiện trên Node HA-pfSense-01:

<img src="https://imgur.com/16ocScr.png">

<img src="https://imgur.com/hckWjqw.png">

- Trạng thái trên Node HA-pfSense-02 là `BACKUP`:

<img src="https://imgur.com/XTQb6gE.png">

## Phần 4. Cấu hình lại bảng NAT cho pfSense

**Thực hiện trên Node HA-pfSense-01**

### Bước 1: Thay đổi chế độ NAT

<img src="https://imgur.com/lKfIL1F.png">

<img src="https://imgur.com/KlT5DQ3.png">

Chọn `Manual Outbound NAT rule generation (AON - Advanced Outbound NAT)` > Chọn `Save`

<img src="https://imgur.com/Z1kIfHu.png">

<img src="https://imgur.com/h777QhL.png">

<img src="https://imgur.com/J8E4SC0.png">

### Bước 2: Điều chỉnh Traffic sẽ đi qua IP VIP

**Cấu hình lại traffic sẽ đi qua IP VIP WAN thay vì IP Public. Sẽ cấu hình trên các interface LAN**

- Cấu hình rule `Auto created rule for ISAKMP - LAN to WAN` 

    - Cấu hình lại `Translation` > `Address` thành VIP PUBLIC `172.16.2.30`

<img src="https://imgur.com/BOxE6CB.png">

<img src="https://imgur.com/9QdAABX.png">

- Cấu hình rule `Auto created rule - LAN to WAN`

    - Cấu hình lại `Translation` > `Address` thành VIP PUBLIC `172.16.2.30`

<img src="https://imgur.com/tf9OgIL.png">

- Kết quả:

<img src="https://imgur.com/SaEeg9e.png">

## Phần 5. Kiểm tra

### Truy cập tới Pfsense thông qua VIP

- Truy cập pfSense thông qua IP VIP http://172.16.2.30/

<img src="https://imgur.com/rSgZZXS.png">

- Thực hiện tắt Node 1 HA-pfSense-01 và vẫn truy cập qua IP VIP thành công

<img src="https://imgur.com/fi8LrHS.png">