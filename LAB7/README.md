# Lab 7 — Web Protection trên pfSense + WEB-PROXY

- **Họ và tên sinh viên:** Nguyễn Phú Thọ
- **Mã số sinh viên:** 1150080158
- **Lớp:** CNPM2
- **Môn:** ATBM_HTTT
- **Báo cáo Word:** `Lab7_CNPM2_1150080158_NguyenPhuTho-2.docx` (dàn bài A→I, ảnh chèn ở C, D, E, F, không chèn dồn vào I.1)
- **File hướng dẫn theo:** `HUONG_DAN_LAB7.md`

## 1. Nội dung đã thực hiện

- **Phần A — Chuẩn bị:** giải nén bộ tài nguyên, kiểm tra 3 máy cũ còn sống (DC01 ipconfig đủ .10/.21/.22, ping 10.6.0.1 và 172.16.6.10 thông, DMZ-WEB 172.16.6.10/24, WebGUI từ máy thật vào được 192.168.56.254), tải xong ISO Ubuntu 24.04.
- **Phần B — Tạo WEB-PROXY:** máy ảo 2 GB RAM, 2 CPU, 20 GB disk, card LAN Segment `lab6-lan`, cài Ubuntu Server 24.04 xong, đăng nhập lần đầu bằng tài khoản `tholab7`.
- **Phần B3 — IP tĩnh WEB-PROXY:** IP 10.6.0.30/24, Gateway 10.6.0.1, ping gateway thông.
- **Phần C1 — Cài Squid + E2Guardian + ClamAV:** cả 3 đều `active (running)`, ClamAV DB up-to-date.
- **Phần C2 — Khóa Squid:** Squid chỉ nghe ở 127.0.0.1:3128.
- **Phần C3 — E2Guardian chính:** nghe ở cổng 8080, chuyển tiếp về Squid 3128, đã bật `contentscanner` ở dòng 494 trong `e2guardian.conf`, `enablessl = off`.
- **Phần C4 — Nhóm lọc:** `reportinglevel = 3`, `bypass = 300`, `contentscanexceptions = off`.
- **Phần D2 — DNS lab7.test:** script `Add-LAB7-DNS.ps1` tạo đủ 13 bản ghi A về 172.16.6.10, `nslookup` qua 10.6.0.10 trả đúng.
- **Phần E1 — Alias pfSense:** LAB7_CLIENTS (10.6.0.21, 10.6.0.22), LAB7_PROXY (10.6.0.30), WEB_PORTS (80, 443), MAIL_PORTS (25, 465, 587, 110, 995, 143, 993), đã Apply.
- **Phần E2 — Ruleset LAN:** đủ 8 dòng theo thứ tự 1, 2, 2b, 3, 4, 5, 6, 7, đã Apply. Lưu ý UI: "LAN net" = "LAN subnets", "Single host or alias" = "Address or Alias", cổng 8080 chọn "(other)".
- **Phần F1 — Danh sách E2Guardian:** `bannedsitelist` 8 tên, `exceptionsitelist` 3 tên, đã có từ phần C.
- **Phần I.2 — Câu hỏi phân tích:** đã có ý chính cho đủ 10 câu, em viết lại bằng lời của mình.

## 2. Kết quả thực hiện và những chỗ chưa làm được vì máy yếu

- **Đã xác minh xong, có ảnh thật:** dịch vụ WEB-PROXY (C.1, C.2), DNS 13 bản ghi (C.4), Alias (D.1), ruleset LAN (D.2).
- **Chưa làm được — Phần A2:** backup `config-truoc-lab7.xml` chưa xác nhận xong, cần làm lại trên WebGUI.
- **Chưa làm được — Phần A3:** snapshot `LAB7_START` cho 3 máy cũ chưa xác nhận xong.
- **Chưa làm được — Phần A4:** tắt Captive Portal Lab 6 chưa xác nhận xong.
- **Chưa làm được — Phần B3 DNS:** lúc cài đặt DNS để tạm 8.8.8.8 vì DC01 đang tắt, cần đổi về 10.6.0.10 sau khi bật lại DC01.
- **Chưa làm được — Phần D1:** chưa tạo xong 13 trang trên DMZ-WEB. Script 74 dòng dán qua clipboard bị cắt thành 30/59/87 dòng, lệch EOF, `bash -n` lỗi dòng 70. Lệnh trực tiếp cũng bị rối khi `sudo` hỏi mật khẩu và khi `sudo dd` ghi file 100/110 MB làm máy đơ. Kiểm tra cuối: `curl` normal vẫn ra trang nginx mặc định, chưa có thư mục `download/files/`.
- **Chưa làm được — Phần E3:** chưa kiểm thử xong chống bypass. Cần mở đủ pfSense + WEB-PROXY + DMZ-WEB + DC01, máy em không mở nổi, lại thêm D1 chưa xong nên test normal qua proxy chưa xác minh được.
- **Chưa làm được — Phần F2:** chưa kiểm thử xong hard block, warning/bypass, education. Lý do giống E3: không mở được đủ máy cùng lúc để test trình duyệt qua proxy.
- **Chưa làm được — Phần G:** schedule 60 phút, EICAR qua proxy, giới hạn file 100/110 MB đều chưa có kết quả kiểm thử, vì phụ thuộc D1 và F2 ở trên. EICAR khi làm lại phải dùng file `eicar.com.txt`, không dùng `eicar.com`, nếu không log chỉ ghi chặn đuôi file chứ không ghi detection virus.
- **Chưa làm được — Phần HTTPS:** HTTPS inspection là mục tùy chọn, em xin bỏ qua.
- **Lý do chung cho các phần chưa làm được:** máy em đời cũ, chỉ khoảng 5 GB RAM, không chạy nổi 4 VM cùng lúc, đặc biệt không chạy nổi pfSense + WEB-PROXY + DC01 cùng lúc. Cách chạy lại em đã tính là chia từng đợt 2-3 VM, hoặc dùng Edge trên máy thật qua proxy 10.6.0.30:8080 thay cho DC01, nhưng em chưa thực hiện xong bước này.

## 3. Lưu ý để kiểm tra hoặc chạy lại bài

- **Môi trường:** 4 VM trên VMware Workstation — `pfSense-LAB6`, `DC01` (Windows 11, kiêm Client-A/B), `DMZ-WEB` (Ubuntu Server 22.04), `WEB-PROXY` (Ubuntu Server 24.04). Không dùng VirtualBox.
- **Bảng địa chỉ IP:**

  | Thiết bị | Interface | IP |
  |---|---|---|
  | pfSense | WAN (VMnet8) | DHCP |
  | pfSense | MGMT (VMnet1) | 192.168.56.254/24 (WebGUI) |
  | pfSense | LAN (lab6-lan) | 10.6.0.1/24 |
  | pfSense | DMZ (lab6-dmz) | 172.16.6.1/24 |
  | DC01 | LAN | 10.6.0.10 (+ 2 IP phụ .21/.22 giả lập Client-A/B) |
  | DMZ-WEB | DMZ | 172.16.6.10/24 |
  | WEB-PROXY | LAN (lab6-lan) | 10.6.0.30/24, GW 10.6.0.1, DNS 10.6.0.10 |

- **Thứ tự bật VM:** pfSense → WEB-PROXY → DMZ-WEB → DC01 (DC01 nặng nhất, bật cuối). Khi máy yếu, chia từng đợt chỉ 2-3 VM, không bật cả 4 cùng lúc.
- **Lưu ý quan trọng:**
  - Sau ruleset E2, ping từ DC01 tới 10.6.0.1 và 172.16.6.10 timeout là bình thường (luật 7 Block Any, không có luật ICMP) — dùng `nslookup` và `curl` qua proxy thay cho ping.
  - DMZ-WEB bị cô lập, `apt update` lỗi DNS là bình thường — DMZ đã có sẵn nginx 1.18.0 và curl 7.81.0 từ Lab 6, bỏ 2 dòng apt trong script.
  - Dán file dài qua clipboard hay bị cắt — chuyển file qua Shared Folder của VMware, hoặc gõ từng đoạn nhỏ, chạy `sudo -v` trước khi dán khối lệnh có `sudo`.
  - Sau mỗi lần sửa rule phải bấm **Apply Changes**; khi test không đổi sau khi đổi rule, vào **Diagnostics → States → Reset** rồi test lại.
  - Kiểm thử phía client trên DC01 bằng trình duyệt với proxy 10.6.0.30:8080, hoặc bằng `curl.exe --interface 10.6.0.21` / `--interface 10.6.0.22` kèm `-x http://10.6.0.30:8080` khi test qua proxy.
