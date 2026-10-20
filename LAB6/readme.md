# Lab 6 — Firewall pfSense + Active Directory + Chính sách truy cập

- **Họ và tên sinh viên:** Nguyễn Phú Thọ
- **Mã số sinh viên:** 1150080158
- **Lớp:** CNPM2
- **Môn:** An toàn hệ thống thông tin
- Link youtube: https://www.youtube.com/@akali6054

## 1. Nội dung đã thực hiện

- **Dựng topology riêng cho Lab 6** (không tái sử dụng Lab 5): pfSense 4 card mạng (WAN/MGMT/LAN/DMZ), Windows Server 2025 làm Domain Controller `DC01` (forest `lab6.test`), Ubuntu Server làm `DMZ-WEB` chạy nginx với 4 IP mô phỏng 4 nhóm website.
- **Tích hợp Active Directory vào pfSense qua LDAP**: tạo Authentication Server `LAB6-AD` (bind bằng service account `svc_pfsense`), kiểm thử xác thực thành công 2 tài khoản `sv_a`, `sv_b`.
- **Captive Portal** `LAB6_USERS` trên interface LAN, xác thực bằng tài khoản domain qua LDAP; whitelist 3 IP (DC01 + 2 IP client giả lập) khỏi portal sau khi demo.
- **Alias** (8 alias: CLIENT_A/B, POLICY_CLIENTS, GENERAL/JOB/SOCIAL/RESTRICTED_SITE, WEB_PORTS) và **Schedule** (LUNCH_WINDOW, WORK_HOURS, TEST_NOW).
- **7 kịch bản firewall**: chặn site theo user, cấm site hạn chế, giới hạn theo khung giờ, chỉ cho web trong giờ hành chính, cô lập DMZ khỏi LAN, so sánh Block/Reject, đọc Firewall Log và State Table.
- **Chính sách tổng hợp (Phần H)**: gộp toàn bộ thành một ruleset duy nhất trên tab LAN theo đúng thứ tự ưu tiên.

## 2. Kết quả thực hiện

- Toàn bộ kiểm thử trong 7 kịch bản đều cho kết quả đúng thiết kế (xem Bảng kết quả kiểm thử trong báo cáo).
- 12 câu hỏi phân tích (Phần J) đã được trả lời kèm ảnh minh chứng ở Phụ lục A.
- File `config.xml` của pfSense được xuất ra và nộp kèm (Diagnostics → Backup & Restore).

## 3. Lưu ý để kiểm tra hoặc chạy lại bài

- **Môi trường:** 3 VM trên VMware Workstation — `pfSense-LAB6`, `DC01` (Windows Server 2025), `DMZ-WEB` (Ubuntu Server 22.04). Không dùng VirtualBox.
- **Bảng địa chỉ IP:**

  | Thiết bị | Interface | IP |
  |---|---|---|
  | pfSense | WAN (VMnet8) | DHCP |
  | pfSense | MGMT (VMnet1) | 192.168.56.254/24 (WebGUI) |
  | pfSense | LAN (lab6-lan) | 10.6.0.1/24 |
  | pfSense | DMZ (lab6-dmz) | 172.16.6.1/24 |
  | DC01 | LAN | 10.6.0.10 (+ 2 IP phụ .21/.22 giả lập Client-A/B) |
  | DMZ-WEB | DMZ | 172.16.6.10 (+ 3 IP phụ .20/.30/.40 cho 3 site) |

- **Thứ tự bật VM:** pfSense → DC01 (chờ AD khởi động xong) → DMZ-WEB.
- **Lưu ý quan trọng:**
  - 3 IP phụ của DMZ-WEB (`.20/.30/.40`) được thêm bằng lệnh `ip addr` nên **mất khi reboot** — bật lại VM phải chạy lại 3 lệnh `sudo ip addr add`.
  - DC01 phải luôn chạy trong lúc kiểm tra vì AD/LDAP và Captive Portal cần domain online.
  - Windows Server 2025 mặc định ép LDAP signing: phải tắt policy "Domain controller: LDAP server signing requirements Enforcement" (Disabled) thì bind LDAP thường qua cổng 389 mới thành công.
  - Kiểm thử phía client chạy trên DC01 bằng lệnh `curl.exe --interface 10.6.0.21` (vai Client-A) hoặc `--interface 10.6.0.22` (vai Client-B).
  - Sau mỗi lần sửa rule phải bấm **Apply Changes**; khi kết quả test không đổi sau khi đổi rule, vào **Diagnostics → States → Reset** để xóa kết nối cũ rồi test lại.
