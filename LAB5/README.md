# README - Lab 5: Thiết lập mô hình tường lửa pfSense

**Họ và tên sinh viên:** Nguyễn Phú Thọ
**Mã số sinh viên:** 1150080158
**Tên bài Lab:** Lab 5 - Thiết lập mô hình tường lửa pfSense

## Nội dung đã thực hiện
- Khởi tạo và cấu hình máy ảo pfSense trên nền tảng VMware với 3 interface: WAN (Bridged/NAT), LAN (Host-only) và DMZ (LAN Segment).
- Thiết lập địa chỉ IP tĩnh cho mạng LAN (10.0.0.1/8) và vùng DMZ (172.16.0.1/16).
- Triển khai máy ảo Windows Server đóng vai trò Domain Controller thuộc vùng LAN (10.0.0.2).
- Cấu hình Outbound NAT ở chế độ Hybrid Outbound NAT.
- Thiết lập luật tường lửa (Firewall Rules) và thực hiện kiểm thử thành công 3 tình huống:
  1. **Tình huống 1:** Chặn ping (ICMP) nhưng vẫn cho phép phân giải tên miền (DNS) và lướt web (HTTP/HTTPS) từ mạng LAN.
  2. **Tình huống 2:** Chỉ cho phép một host cụ thể (Domain Controller) ra Internet, chặn các host còn lại trong mạng LAN.
  3. **Tình huống 3:** Cô lập vùng DMZ, chặn kết nối từ DMZ vào mạng LAN nhưng vẫn cho phép DMZ ra Internet.

## Kết quả thực hiện
- Hệ thống mạng ảo hóa hoạt động ổn định, các dải mạng LAN và DMZ được định tuyến đúng qua pfSense.
- Các thiết bị trong LAN và DMZ nhận diện và giao tiếp được với Gateway.
- Các rule Firewall đã hoạt động chính xác theo kịch bản: chặn thành công các luồng ping trái phép, cho phép truy cập web bình thường và cô lập an toàn vùng DMZ. 

## Các lưu ý cần thiết để giảng viên có thể kiểm tra hoặc chạy lại bài
- **Môi trường ảo hóa:** Bài Lab được thực hiện trên VMware.
- **Ánh xạ card mạng (Network Adapter):**
  - Cổng WAN (em0): Cấu hình Bridged (hoặc NAT Network nếu mạng vật lý trùng dải).
  - Cổng LAN (em1): Cấu hình Host-only (VMnet1), dải 10.0.0.0/8.
  - Cổng DMZ (em2): Cấu hình LAN Segment (tên `dmz-net`), dải 172.16.0.0/16.
- **Tài khoản pfSense WebGUI:** 
  - Username: `admin`
  - Password: `<Nhập_mật_khẩu_pfSense_bạn_đã_đổi>`
- **Tài khoản Windows Server:**
  - Username: `Administrator`
  - Password: `<Nhập_mật_khẩu_Windows_Server_của_bạn>`
- **Lưu ý khi test rule:** pfSense là tường lửa theo trạng thái (stateful). Khi kiểm tra các kịch bản rule mới, cần vào `Diagnostics` -> `States` -> `Reset States` để xóa các kết nối cũ trước khi ping/curl thử nghiệm.
