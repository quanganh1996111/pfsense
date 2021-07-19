# Cấu hình Pfsense Site to Site với mô hình Pfsense HA

## Mô hình

<img src="https://imgur.com/2l8tiCd.png">

## Phần 1. Chuẩn bị

Cần chuẩn bị:

- 2 Cụm đã cấu hình HA cho pfSense.

- 2 Cụm pfSense đều đã cấu hình mode TAP.

### Mô hình cụm

#### Cụm 1 (Site01):

- pfsense-ha01-site01:

    - `vtnet0`: 172.16.2.20

    - `vtnet1`: 10.10.40.30 - VLAN40

    - `vtnet2`: 10.10.50.20 - VLAN50

- pfsense-ha02-site01:

    - `vtnet0`: 172.16.2.21

    - `vtnet1`: 10.10.40.31 - VLAN40

    - `vtnet2`: 10.10.50.21 - VLAN50

- IP VIP:

    - VIP WAN: 172.16.2.22

    - VIP LAN: 10.10.40.32

#### Cụm 2 (Site02):

- pfsense-ha01-site02:

    - `vtnet0`: 172.16.3.20

    - `vtnet1`: 10.10.41.30 - VLAN41

    - `vtnet2`: 10.10.50.30 - VLAN50

- pfsense-ha02-site02:

    - `vtnet0`: 172.16.3.21

    - `vtnet1`: 10.10.41.31 - VLAN41

    - `vtnet2`: 10.10.50.31 - VLAN50

- IP VIP:

    - VIP WAN: 172.16.3.22

    - VIP LAN: 10.10.41.32

## Phần 2. Cấu hình VPN IPSec tại Cụm 1 (Site01)

### Bước 1: Truy cập vào IP VIP WAN Cụm 1

http://172.16.2.22/

#### Chọn `VPN` > `IPsec`

<img src="https://imgur.com/iYqB6nl.png">

<img src="https://imgur.com/hvRQO77.png">

Tại mục `General Information`

- `Key Exchange version`: `IKEv1`

- `Internet Protocol`: `IPv4`

- `Interface`: 172.16.2.22 (Site01-VIP) - Đây là địa chỉ VIP WAN của cụm 1

- `Remote Gateway`: 172.16.3.22 - IP VIP WAN của cụm 2

- `Description`: VPN Site-to-Site HA

<img src="https://imgur.com/exfDUAX.png">

Tại mục `Phase 1 Proposal (Authentication)`

- `Authentication Method`: `Mutual PSK`

- `Negotiation mode`: `Main`

- `My identifier`: `My IP address`

- `Peer identifier`: `Peer IP address`

- Click vào `Generate new Pres-Shared Key` để sinh giá trị `Pre-Shared Key`:

`9766ec4cbe17c4cae8b81d4330eeef19e83bbe90f95040d846a74dc9`

**Lưu ý**: Phần Key này sẽ cần phải lưu lại để cấu hình Cụm 2 (Site02), yêu cầu key phải giống nhau.

<img src="https://imgur.com/p25fzFx.png">

Tại mục `Phase 1 Proposal (Encryption Algorithm)`: `Encryption Algorithm`

- `Algorithm`: `3DES`

- `Hash`: `MD5`

- `DH Group`: `14 (2048 bit)`

<img src="https://imgur.com/vwkQOao.png">

#### Chọn `Show Phase 2 Entries`

<img src="https://imgur.com/OCx9pg4.png">

Tại mục `General Information`

- `Mode`: `Tunnel IPv4`

- `Local Network` : 10.10.40.0 - VLAN40 Cụm 1

- `NAT/BINAT translation`: `None`

- `Remote Network`: 10.10.41.0 - VLAN41 Cụm 2

- `Description`: Site01-to-Site02

<img src="https://imgur.com/PS4nBOR.png">

Tại mục `Phase 2 Proposal (SA/Key Exchange)`

- `Encryption Algorithms`: `3DES`

- `Hash Algorithms`: `MD5`

- `PFS key group`: `14 (2048 bit)`

Tại mục `Expiration and Replacement`

- `Lifetime`: `3600`

Tại mục `Advanced Configuration`

- `Automatically ping host`: 10.10.41.35 - IP VLAN41 Cụm 2

<img src="https://imgur.com/4IOdfbf.png">

<img src="https://imgur.com/1csSjWj.png">

Kết quả:

<img src="https://imgur.com/7Q9RNKK.png">

### Bước 2: Cấu hình Firewall cho VPN Site to Site tại interface WAN

Chọn `Firewall` > `Rules`

<img src="https://imgur.com/NLegF4d.png">

<img src="https://imgur.com/EMe8elB.png">

Tại mục `Edit Firewall Rule`

- `Action`: `Pass`

- `Interface`: `WAN`

- `Address Family`: `IPv4`

- `Protocol`: `TCP/UDP`

Tại mục `Destination`: `Destination Port Range`

- `From`: `IPsec NAT-T (4500)`

- `To`: `IPsec NAT-T (4500)`

<img src="https://imgur.com/1T0aRMb.png">

<img src="https://imgur.com/kXz7ctC.png">

Tại mục `Edit Firewall Rule`

- `Action`: `Pass`

- `Interface`: `WAN`

- `Address Family`: `IPv4`

- `Protocol`: `TCP/UDP`

Tại mục `Destination`: `Destination Port Range`

- `From`: `ISAKMP (500)`

- `To`: `ISAKMP (500)`

<img src="https://imgur.com/eWRxrwv.png">

<img src="https://imgur.com/YbiAJuE.png">

### Bước 3: Cấu hình Firewall cho VPN Site to Site tại interface IPsec

Chọn `IPsec` > `Add`

<img src="https://imgur.com/a9aeK1U.png">

Tại mục `Edit Firewall Rule`

- `Action`: `Pass`

- `Interface`: `IPSec`

- `Address Family`: `IPv4`

- `Protocol`: `Any`

Tại mục `Source`:

- `Source` - `Network`: 10.10.41.0/24 Dải VLAN41 tại Cụm 2

<img src="https://imgur.com/WzrnJ3A.png">

<img src="https://imgur.com/gb58C1z.png">

### Bước 4: Bước Kiểm tra trạng thái VPN Site to Site IPsec

<img src="https://imgur.com/cb9SL9s.png">

<img src="https://imgur.com/E93ThgK.png">

**Lưu ý**: Do chưa cấu hình trên Cụm 2 nên trạng thái vẫn là `CONNECTING`

## Phần 3. Cấu hình VPN IPSec tại Cụm 2 (Site02)

Bước 1: Thiết lập VPN Site to Site qua IPSec

Truy cập vào VIP WAN của Cụm 2: http://172.16.3.22/

Chọn `VPN` > `IPsec`

<img src="https://imgur.com/zBnfL4S.png">

<img src="https://imgur.com/G9KAPqA.png">

Tại mục `General Information`

- `Key Exchange version`: `IKEv1`

- `Internet Protocol`: `IPv4`

- `Interface` : 172.16.3.22 (Site02-VIP) - VIP WAN Cụm 2

- `Remote Gateway`: 172.16.2.22 - VIP WAN Cụm 1

- `Description`: VPN Site-to-Site HA

<img src="https://imgur.com/8uWt2Zp.png">

Tại mục `Phase 1 Proposal (Authentication)`

- `Authentication Method`: `Mutual PSK`

- `Negotiation mode`: `Main`

- `My identifier`: `My IP address`

- `Peer identifier`: `Peer IP address`

- Giá trị `Pre-Shared Key` phải copy key từ Cụm 1 ở phần trên.

`9766ec4cbe17c4cae8b81d4330eeef19e83bbe90f95040d846a74dc9`

<img src="https://imgur.com/1HQEbgY.png">

Tại mục `Phase 1 Proposal (Encryption Algorithm)`: `Encryption Algorithm`

- `Algorithm - 3DES`

- `Hash - MD5`

- `DH Group: 14 (2048 bit)`

<img src="https://imgur.com/bbId56X.png">

#### Chọn `Show Phase 2 Entries`

<img src="https://imgur.com/2m8t319.png">

<img src="https://imgur.com/jkdFyGH.png">

Tại mục `General Information`

- `Mode: Tunnel IPv4`

- `Local Network`: `Network` - 10.10.41.0/24 - Dải VLAN41 trên Cụm 2

- `NAT/BINAT translation: None`

- `Remote Network`: `Network` - 10.10.40.0/24 - Dải VLAN40 trên Cụm 1

- `Description`: VLAN41 to VLAN40

<img src="https://imgur.com/M1c0yyM.png">

Tại mục `Phase 2 Proposal (SA/Key Exchange)`

- `Encryption Algorithms: 3DES`

- `Hash Algorithms: MD5`

- `PFS key group: 14 (2048 bit)`

Tại mục `Expiration and Replacement`

- `Lifetime: 3600`

<img src="https://imgur.com/IxGnxwP.png">

Tại mục `Advanced Configuration`

- `Automatically ping host`: 10.10.40.35 - IP VLAN40 Cụm 1

Kết quả:

<img src="https://imgur.com/aU02GgV.png">

### Bước 2: Cấu hình Firewall cho VPN Site to Site tại interface WAN

Chọn `Firewall` > `Rules`

<img src="https://imgur.com/L1R4SZw.png">

<img src="https://imgur.com/RmglCpc.png">

Tại mục `Edit Firewall Rule`

- `Action: Pass`

- `Interface: WAN`

- `Address Family: IPv4`

- `Protocol: TCP/UDP`

Tại mục `Destination`: `Destination Port Range`

- `From: IPsec NAT-T (4500)`

- `To: IPsec NAT-T (4500)`

<img src="https://imgur.com/ToOM8r8.png">

<img src="https://imgur.com/YNCyojM.png">

Tại mục `Edit Firewall Rule`

- `Action: Pass`

- `Interface: WAN`

- `Address Family: IPv4`

- `Protocol: TCP/UDP`

Tại mục `Destination`: `Destination Port Range`

- `From: ISAKMP (500)`

- `To: ISAKMP (500)`

<img src="https://imgur.com/WtrYNqJ.png">

<img src="https://imgur.com/yxFuQZ7.png">

### Bước 3: Cấu hình Firewall cho VPN Site to Site tại interface IPsec

Chọn `IPsec > Add`

<img src="https://imgur.com/oOkIBq0.png">

Tại mục `Edit Firewall Rule`

- `Action: Pass`

- `Interface: IPSec`

- `Address Family: IPv4`

- `Protocol: Any`

Tại mục Source:

- `Source - Network` - 10.10.40.0/24 - Dải VLAN40 tại Cụm 1

<img src="https://imgur.com/FY0058F.png">

Kết quả:

<img src="https://imgur.com/ejxG41S.png">

### Bước 4: Bước Kiểm tra trạng thái VPN Site to Site IPsec

Chọn `Status > IPsec`

- Tại Cụm 1:

<img src="https://imgur.com/aLabzNG.png">

- Tại Cụm 2:

<img src="https://imgur.com/x1PvAZi.png">

## Phần 5. Kiểm tra tính HA PFSENSE với VPN Site-to-Site

- Tiến hành tắt node `pfsense-ha01-site01` trên Cụm 1:

<img src="https://imgur.com/4gz1soR.png">

<img src="https://imgur.com/Pf7zSwW.png">

- Sẽ mất từ 1-5p để pfSense cập nhật và tính toán lại được đi.

Kết quả ghi nhận pfSense vẫn kết nối 2 Site với nhau:

<img src="https://imgur.com/A3rzQmq.png">

<img src="https://imgur.com/r8aqzo3.png">