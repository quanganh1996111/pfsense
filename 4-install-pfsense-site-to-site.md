# Cấu hình Pfsense Site to Site (Không HA)

## Phần 1. Mô hình hoạt động

<img src="https://imgur.com/djFmihb.png">

### Chuẩn bị

Cấu hình 2 VM pfSense bao gồm:

- IP Public, IP VLAN khác dải.

- Cấu hình OPENVPN Mode TAP.

Cấu hình Cụm 1:

- VM pfSense-site01:

    - `vtnet0`: 172.16.2.16 - WAN

    - `vtnet1`: 10.10.40.7 - VLAN40

- VM CentOS 7: IP 10.10.40.15

<img src="https://imgur.com/UqHWHmt.png">

Cấu hình Cụm 2:

- VM pfSense-site02:

    - `vtnet0`: 172.16.3.11 - WAN

    - `vtnet1`: 10.10.41.7 - VLAN40

- VM CentOS 7: IP 10.10.41.15

<img src="https://imgur.com/QnMQwyN.png">

## Phần 2. Cấu hình VPN IPSec tại Cụm 1

**Thực hiện trên pfSense-site01**

### Bước 1: Thiết lập VPN Site to Site qua IPSec

#### Chọn `VPN > IPsec`

<img src="https://imgur.com/n2WhXr3.png">

<img src="https://imgur.com/dpU7Xfx.png">

Tại mục `General Information`:

- `Key Exchange version`: `IKEv1`

- `Internet Protocol`: `IPv4`

- `Interface`: `WAN`

- `Remote Gateway`: 172.16.3.11 - Địa chỉ Public pfSense-site02

- `Description`: VPN to Site pfSense-site02

<img src="https://imgur.com/lTAvLoy.png">

Tại mục `Phase 1 Proposal (Authentication)`

- `Authentication Method`: `Mutual PSK`

- `My identifier`: `My IP address`

- `Peer identifier`: `Peer IP address`

- Click vào `Generate new Pres-Shared Key` để sinh giá trị `Pre-Shared Key`

<img src="https://imgur.com/NNxYtZb.png">

Tại mục `Phase 1 Proposal (Encryption Algorithm)`

- Cấu hình `Encryption Algorithm`

    - `Algorithm - 3DES`
    
    - `Hash MD5`
    
    - `DH Group`: `14 (2048 bit)`

Sau đó tiến hành `Save` lại.

<img src="https://imgur.com/qxRqGCV.png">

#### Chọn `Show Phase 2 Entries`

<img src="https://imgur.com/Q0D8pdm.png">

<img src="https://imgur.com/eVuAsnK.png">

Tại mục `General Information`

- `Mode`: `Tunnel IPv4`

- `Local Network`: Network - `10.10.40.0 / 24` - Dải địa chỉ VLAN40 tại Cụm 1

- `NAT/BINAT translation`: `None`

- `Remote Network`: Network - `10.10.41.0 / 24` - Dải địa chỉ VLAN41 tại Cụm 2

- `Description`: `VLAN40 to VLAN41`

<img src="https://imgur.com/w8JduHa.png">

Tại mục `Phase 2 Proposal (SA/Key Exchange)`

- `Encryption Algorithms`: `3DES`

- `Hash Algorithms`: `MD5`

- `PFS key group`: `14 (2048 bit)`

Tại mục `Expiration and Replacement`

- `Lifetime`: `3600`

Tại mục `Advanced Configuration`

- `Automatically ping host`: `10.10.41.20` IP VLAN41 tại Cụm 2

<img src="https://imgur.com/QJTjA59.png">

<img src="https://imgur.com/cHpUkAv.png">

Kết quả:

<img src="https://imgur.com/VzpN731.png">

### Bước 2: Cấu hình Firewall cho VPN Site to Site tại interface WAN

Chọn `Firewall` > `Rules`

<img src="https://imgur.com/Qhb5dIy.png">

<img src="https://imgur.com/iU0sSZB.png">

#### Tại giao diện cấu hình Rule

- Tại mục `Edit Firewall Rule`

    - `Action`: `Pass`

    - `Interface`: `WAN`

    - `Address Family`: `IPv4`

    - `Protocol`: `TCP/UDP`

- Tại mục `Destination`: `Destination Port Range`

    - `From`: `IPsec NAT-T (4500)`
    
    - `To`: `IPsec NAT-T (4500)`

- Sau đó `Save` lại.

<img src="https://imgur.com/wHGSYsE.png">

<img src="https://imgur.com/UT20mI7.png">

Tiếp tục `Add` thêm Rule:

<img src="https://imgur.com/lxnpowN.png">

Tại giao diện cấu hình Rule

- Tại mục `Edit Firewall Rule`

    - `Action`: `Pass`

    - `Interface`: `WAN`
    
    - `Address Family`: `IPv4`

    - `Protocol`: `TCP/UDP`

- Tại mục `Destination`:

    - `Destination Port Range`

    - `From`: `ISAKMP (500)`

    - `To`: `ISAKMP (500)`

- Chọn `Save`

<img src="https://imgur.com/Rdza1vL.png">

<img src="https://imgur.com/BqXeZAQ.png">

#### Bước 3: Cấu hình Firewall cho VPN Site to Site tại interface IPsec

<img src="https://imgur.com/LscK7vy.png">

Tại mục `Edit Firewall Rule`

- `Action`: `Pass`

- `Interface`: `IPSec`

- `Address Family`: `IPv4`

- `Protocol`: `Any`

Tại mục `Source`:

- `Source` - Network - 10.10.41.0/24 - Cho phép dải VLAN41 được phép kết nối qua VPN IPsec

Chọn `Save`

<img src="https://imgur.com/ohElGqu.png">

<img src="https://imgur.com/68svXjU.png">

#### Bước 4: Bước Kiểm tra trạng thái VPN Site to Site IPsec

