# Hướng dẫn cấu hình OpenVPN mode TAP

## Yêu cầu đối với VM

- 2 Card mạng, 1 LAN, 1 WAN (Có IP Public).

- Đường WAN sẽ sử dụng để Client được kết nối tới. **IP WAN: 172.16.2.48**

- Đường LAN là đường quản trị, các Server sẽ được SSH qua dảy mạng LAN. **Dải IP LAN 10.10.40.x**

## Phần 1. Thiết lập Certificate

### Khởi tạo CA cho OpenVPN

CA này sẽ xác thực tất cả các certificate của server VPN và user VPN khi kết nối tới PFSense OpenVPN.

#### Bước 1: Chọn `System > Cert. Manager`

<img src="https://imgur.com/zyFkYQ2.png">

<img src="https://imgur.com/zZoLaqA.png">

#### Bước 2: Điền các thông tin cho Certificate

- Tại `Create / Edit CA`:

    - `Descriptive name: ca-user`

    - `Method: Create an internal Certificate Authority`

<img src="https://imgur.com/62KruRJ.png">

- Tại `Internal Certificate Authority` điền đầy đủ thông tin, sau đó `save`:

<img src="https://imgur.com/lktCZE5.png">

- Kết quả:

<img src="https://imgur.com/DiwsGlB.png">

#### Bước 4: Tạo `Certificates > Add`

<img src="https://imgur.com/8ZtjxG3.png">

- Tại `Add/Sign a New Certificate`:

    - `Method: Create an internal Certificate`

    - `Descriptive name: ca-server`

- Tại `Certificate Attributes`

    - `Certificate Type: Server Certificate`

<img src="https://imgur.com/hcLI4jI.png">

<img src="https://imgur.com/42yY8sD.png">

## Phần 2. Tạo User Client

### Thêm mới User cho OpenVPN

#### Bước 1: `System -> User Manager`

Đây là User sẽ được kết nối VPN tới

<img src="https://imgur.com/uKQotdV.png">

<img src="https://imgur.com/dcc5SAr.png">

#### Bước 2: Tại User Properties

Nhập thông tin tài khoản tại `Username / Password`

Chọn `Certificate - Click to create a user certificate`

## Phần 3. Cài gói Openvpn export client

#### Bước 1: Chọn `System > Package Manager`

<img src="https://imgur.com/tFIbnBe.png">

#### Bước 2: Chọn `Available Packages`

Tại Search term nhập `openvpn` > Chọn `Search`. Sau khi có kết quả, chọn `Install` tại `openvpn-client-export`

<img src="https://imgur.com/neUTBwT.png">

<img src="https://imgur.com/EqqWNMg.png">

