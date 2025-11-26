# Hướng dẫn Trunking – VTP – VLAN trên Cisco Packet Tracer

Tài liệu này giúp sinh viên tự tay triển khai mô hình Trunking, VTP và VLAN với giao diện trực quan trong Cisco Packet Tracer. Mỗi bước đều có mục tiêu rõ ràng, lệnh mẫu và cách kiểm tra nhằm đảm bảo cấu hình đúng ngay lần đầu.

## 1. Mục tiêu học tập
- Hiểu ý nghĩa của từng thành phần: VLAN dùng để tách mạng logic, Trunk dùng để mang nhiều VLAN qua cùng một đường truyền, VTP dùng để đồng bộ cơ sở dữ liệu VLAN giữa các switch.
- Thực hành cấu hình đầy đủ trên Switch Server và các Switch Client.
- Gán VLAN cho từng PC và kiểm thử kết nối thông qua lệnh ping.

## 2. Sơ đồ thiết bị và đánh số
| Thiết bị | Số lượng | Vai trò | Ghi chú |
| --- | --- | --- | --- |
| Switch 2960 | 4 | 1 Server (Switch 0), 3 Client (Switch 1-3) | Đặt tên trùng với số thứ tự trên Packet Tracer để dễ theo dõi |
| PC | 9 | PC0-2 gắn với Switch 1, PC3-5 gắn với Switch 2, PC6-8 gắn với Switch 3 | Mỗi nhóm 3 PC thuộc các VLAN khác nhau |

> **Lưu ý:** Luôn đặt label cho từng cổng trên sơ đồ để hạn chế nhầm lẫn trong quá trình thực hành.

## 3. Kết nối vật lý (Layer 1)
### 3.1 Server ↔ Client
| Từ | Đến | Cổng Server | Cổng Client |
| --- | --- | --- | --- |
| Switch 0 | Switch 1 | Fa0/1 | Fa0/1 |
| Switch 0 | Switch 2 | Fa0/2 | Fa0/1 |
| Switch 0 | Switch 3 | Fa0/3 | Fa0/1 |

### 3.2 Client ↔ PC
- Switch 1 ↔ PC0, PC1, PC2: dùng các cổng Fa0/2 → Fa0/4 trên switch, nối vào cổng FastEthernet của từng PC.
- Switch 2 ↔ PC3, PC4, PC5: tương tự Switch 1.
- Switch 3 ↔ PC6, PC7, PC8: tương tự Switch 1.

> **Mẹo kiểm tra nhanh:** Nhấp đôi vào từng dây để chắc chắn không có đường dây đỏ (down) trước khi cấu hình.

## 4. Cấu hình Switch Server (Switch 0)
### 4.1 Truy cập chế độ cấu hình
```
Switch>enable
Switch#configure terminal
```

### 4.2 Tạo VLAN
1. Lặp lại khối lệnh sau cho từng VLAN cần tạo (ví dụ 10, 20, 30):
   ```
   Switch(config)#vlan 10
   Switch(config-vlan)#name TaiChinh
   Switch(config-vlan)#exit
   ```
2. Dùng `show vlan brief` để xác nhận danh sách VLAN mới xuất hiện.

   **Ví dụ kết quả `show vlan brief` trên Switch Server:**
   ```
   VLAN Name                             Status    Ports
   ---- -------------------------------- --------- -------------------------------
   1    default                          active    Fa0/4-Fa0/24, Gi0/1-Gi0/2
   10   TaiChinh                         active    
   20   KinhDoanh                        active    
   30   NhanSu                           active    
   1002 fddi-default                     active    
   1003 token-ring-default               active    
   1004 fddinet-default                  active    
   1005 trnet-default                    active    
   ```

> **Giải thích:** VLAN giúp cô lập lưu lượng theo phòng ban (Tài chính, Kinh doanh, Nhân sự). Việc đặt tên rõ ràng giúp giảm sai sót khi gán cổng Access.

### 4.3 Thiết lập đường trunk đến các Client
1. Chọn đúng cổng uplink đang nối với Client.
   ```
   Switch(config)#interface fastEthernet0/1
   Switch(config-if)#switchport mode trunk
   Switch(config-if)#exit
   ```
2. Lặp lại cho Fa0/2 và Fa0/3.
3. Kiểm tra lại bằng `show interfaces trunk` hoặc `show vlan brief` (các cổng trunk sẽ không còn hiển thị trong VLAN 1).

### 4.4 Bật VTP ở chế độ Server
1. Đặt chế độ, domain và mật khẩu thống nhất:
   ```
   Switch(config)#vtp mode server
   Switch(config)#vtp domain abc.com.vn
   Switch(config)#vtp password 123456
   ```
2. Xác nhận bằng `show vtp status`.

   **Ví dụ kết quả `show vtp status` trên Switch Server:**
   ```
   VTP Version capable             : 1 to 2
   VTP version running             : 1
   VTP Domain Name                 : abc.com.vn
   VTP Pruning Mode                : Disabled
   VTP Traps Generation            : Disabled
   Device ID                       : 0004.9A1A.BB00
   Configuration last modified by  : 0.0.0.0 at 3-1-93 00:17:31

   Feature VLAN:
   --------------
   VTP Operating Mode              : Server
   Number of existing VLANs        : 8
   Configuration Revision          : 3
   MD5 digest                      : 0xF5 0x6F 0x40 0x9A 0xDD ...
   ```

> **Checklist kiểm tra Server:**
> - `show vlan brief` liệt kê VLAN 10/20/30.
> - `show interfaces trunk` cho thấy Fa0/1-0/3 ở mode trunk.
> - `show vtp status` báo chế độ Server và đúng domain/password.

## 5. Cấu hình từng Switch Client (Switch 1-3)
Thực hiện lần lượt cho từng switch. Các bước giống nhau, chỉ khác ở việc gán cổng Access.

### 5.1 Đặt VTP Mode = Client
```
Switch>enable
Switch#configure terminal
Switch(config)#vtp mode client
Switch(config)#vtp domain abc.com.vn
Switch(config)#vtp password 123456
```
- Dùng `show vtp status` để chắc chắn số lượng VLAN đồng bộ với Server (Configuration Revision giống nhau).

**Ví dụ kết quả `show vtp status` trên Switch Client:**
```
VTP Version capable             : 1 to 2
VTP version running             : 1
VTP Domain Name                 : abc.com.vn
VTP Pruning Mode                : Disabled
Device ID                       : 0060.3ED2.0700

Feature VLAN:
--------------
VTP Operating Mode              : Client
Number of existing VLANs        : 8
Configuration Revision          : 3
MD5 digest                      : 0xF5 0x6F 0x40 0x9A 0xDD ...
```

### 5.2 Gán cổng Access vào VLAN
1. Xác định PC thuộc VLAN nào (tham khảo bảng bên dưới).
2. Trên từng cổng uplink tới PC, cấu hình:
   ```
   Switch(config)#interface fastEthernet0/2
   Switch(config-if)#switchport mode access
   Switch(config-if)#switchport access vlan 10
   Switch(config-if)#exit
   ```
3. Kiểm chứng bằng `show vlan brief` – cổng nào đã gán đúng sẽ nằm trong cột Ports của VLAN tương ứng.

**Ví dụ kết quả `show vlan brief` trên Switch Client:**
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5-Fa0/24, Gi0/1-Gi0/2
10   TaiChinh                         active    Fa0/2
20   KinhDoanh                        active    Fa0/3
30   NhanSu                           active    Fa0/4
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```

#### Gợi ý phân bổ VLAN
| Switch Client | PC | VLAN đề xuất | Mục đích |
| --- | --- | --- | --- |
| Switch 1 | PC0 | 10 | Tài chính |
| Switch 1 | PC1 | 20 | Kinh doanh |
| Switch 1 | PC2 | 30 | Nhân sự |
| Switch 2 | PC3 | 10 | Tài chính |
| Switch 2 | PC4 | 20 | Kinh doanh |
| Switch 2 | PC5 | 30 | Nhân sự |
| Switch 3 | PC6 | 10 | Tài chính |
| Switch 3 | PC7 | 20 | Kinh doanh |
| Switch 3 | PC8 | 30 | Nhân sự |

> **Mẹo xử lý lỗi:** Nếu `show vlan brief` trên Client không thấy VLAN 10/20/30, hãy kiểm tra lại domain/password VTP và trạng thái trunk trên Switch Server.

## 6. Cấu hình PC
### 6.1 Gán địa chỉ IP tĩnh
- Trên từng PC: `Desktop → IP Configuration` và đặt IP theo dải VLAN tương ứng (ví dụ 192.168.1.x cho VLAN 10, 192.168.2.x cho VLAN 20, 192.168.3.x cho VLAN 30).
- Giữ Subnet Mask mặc định `/24` (255.255.255.0). Gateway chưa cần trong phạm vi bài lab này.

### 6.2 Kiểm thử kết nối
1. Mở `Desktop → Command Prompt` và sử dụng `ping` giữa các PC cùng VLAN (ví dụ PC0 ping PC3, cả hai thuộc VLAN 10).
2. Các gói tin cùng VLAN phải nhận được `Reply`. Nếu khác VLAN, trạng thái sẽ là `Request timed out` (đúng với mô hình chưa có Router-on-a-stick).
3. Nếu không ping được trong cùng VLAN, kiểm tra lại: dây cắm, VLAN access, địa chỉ IP.

## 7. Tổng kết nhanh
- VLAN giúp cô lập broadcast domain; từng switch cần đồng bộ VLAN để tránh tạo thủ công nhiều lần.
- Trunk đảm bảo một đường uplink có thể mang toàn bộ VLAN giữa Server và Client.
- VTP giúp quản lý tập trung VLAN tại Switch Server và tự động phân phối xuống Client.
- Hoàn tất bài lab khi: VLAN xuất hiện đồng bộ, trunk ở trạng thái up, PC cùng VLAN ping được nhau.

Chúc bạn thực hành thành công! Hãy lưu lại file Packet Tracer ở từng mốc (chưa cấu hình, sau khi cấu hình switch, sau khi cấu hình PC) để dễ dàng quay lại nếu cần.