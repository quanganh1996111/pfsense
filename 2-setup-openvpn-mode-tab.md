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

- Cài đặt thành công:

<img src="https://imgur.com/FWL9ez5.png">

## Phần 4. Thiết lập OPENVPN

#### Bước 1: Chọn `VPN > OpenVPN`

<img src="https://imgur.com/UCDb7EB.png">

<img src="https://imgur.com/ipzhE9w.png">

#### Bước 2: Tạo mới server OpenVPN

Chọn Server

- Tại mục `General Information`:

    - `Server mode > Remote Access ( SSL/TLS + User Auth )`

    - `Protocol > UDP IPv4 and IPv6 on all interfaces (multihome)`

    - `Device mode > tap - Layer 2 Tap Mode`

    - `Local port: 1194`

<img src="https://imgur.com/joyzjgA.png">

- Tại mục Cryptographic Settings

    - Chọn `TLS Configuration > Use a TLS Key`

    - Chọn `TLS Configuration > Automatically generate a TLS Key`

    - Chọn `Peer Certificate Authority > anhtqlab` **LƯU Ý**: ĐÂY LÀ CERT TẠO TỪ PHẦN 1

    - Chọn `Server certificate > ca-server` **LƯU Ý**: ĐÂY LÀ CERT TẠO TỪ PHẦN 1

    - Chọn `Data Encryption Algorithms > AES-256-CBC (256 bit key, 128 bit block)` 

    - Chọn `Enable NCP > Enable Negotiable Cryptographic Parameters`

    - Tại `Fallback Data Encryption Algorithm - Chọn AES-256-CBC (256bit key, 128 bit block)`

<img src="https://imgur.com/HfUTW5S.png">

<img src="https://imgur.com/mJiCgKV.png">

- Tại Tunner Settings

    - Chọn `Bridge DHCP: Allow client on the bridge to obtain DHCP`

    - `Bridge Interface: LAN`

    - `Server Bridge DHCP Start: 10.10.40.10`
    
    - `Server Bridge DHCP End: 10.10.40.100`

<img src="https://imgur.com/DQmA7BQ.png">

- `Custom Options` thêm cấu hình: `push "route 10.10.40.0 255.255.255.0"`

<img src="https://imgur.com/9vXGOcE.png">

##### Kết quả sau khi cấu hình

<img src="https://imgur.com/hCoDUDo.png">

