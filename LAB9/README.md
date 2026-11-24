# Báo cáo thực hành Lab 9 — Remote DNS Cache Poisoning

- **Họ và tên sinh viên:** Nguyễn Phú Thọ
- **Mã số sinh viên:** 1150080158
- **Lớp:** CNPM2
- **Môn:** An toàn bảo mật HTTT
- Link kênh youtube: https://www.youtube.com/@akali6054
- Link video: https://youtu.be/bd0-m4a4Ga4
 ----------
 ----------
 ----------
 em xin phép thầy thông cảm, máy em yếu, RAM không đủ nên không chạy nổi 5 máy ảo cùng lúc. Chỗ nào chưa làm được em ghi rõ lý do, không bịa ảnh hay bịa kết quả.

## 1. Nội dung đã thực hiện

- **Phần A–B — Tổng quan và mô hình:** Lab 9 kế thừa toàn bộ hạ tầng Lab 8 là pfSense-LAB6, DNS-SERVER 10.9.0.10, VICTIM 10.9.0.100, DMZ-WEB 172.16.6.10, thêm subnet mới ATTACKNET 10.10.0.0/24, Kali chuyển sang 10.10.0.200, Promiscuous = Deny, không sniff DNSLAB nữa. Tài nguyên Lab 9 ở trong Downloads, thư mục Tool_ATBMHTTT, LAB10, LAB10_Resources, đủ 01 tới 07 và file README_FIRST.txt.
- **Phần C — Chuyển từ Lab 8 sang Lab 9:**
  - **C.1:** tắt hết VM, chụp snapshot pfSense-LAB6 tên LAB10_START_FROM_LAB9, mô tả Truoc Lab 9, tu Lab 8.
  - **C.2:** thêm Adapter 6 = LAN Segment attack-net cho pfSense, trong WebGUI hiện thành em5, MAC 00:0c:29:7e:8c:36, gán thành OPT4, đổi tên ATTACKNET, IP tĩnh 10.10.0.1/24, Gateway None, Save và Apply. WebGUI lúc đầu vào bằng http không được, ping 192.168.56.254 vẫn 4/4 Reply, phải vào bằng https://192.168.56.254 mới được.
  - **C.3:** Kali mới thêm Network Adapter 2 = LAN Segment attack-net, card 1 NAT, card 2 attack-net. Đặt IP bằng nmcli, con-name attacknet, IP 10.10.0.200/24, vì dùng ip addr add bị NetworkManager ghi đè mất. Thêm route 10.9.0.0/24 và 172.16.6.0/24 qua 10.10.0.1. Ping gateway 10.10.0.1 timeout 100% là kỳ vọng vì không có rule ICMP, ip neigh thấy đúng MAC pfSense em5 00:0c:29:7e:8c:36. Công cụ đủ: python3-scapy 2.7.0, wireshark 4.6.6-1, tcpdump 4.99.6-2. Kali bị xóa và làm lại ngày 24/11, coi như máy mới hoàn toàn, mất hết lab9, lab9-fake, IP đặt tay cũ, Shared Folder, file attacker Lab 8. Đã chép lại kaminsky_remote_lab10.py và evil_dns_lab10.py vào lab10, fake_index.html vào lab10-fake/index.html, py_compile ra script OK.
  - **C.4:** làm đủ 12 rule firewall đúng thứ tự, đã Apply và Reset States. DNSLAB 7 rule: Pass DNS-SERVER 10.9.0.10 tới AUTH-DNS 172.16.6.20:53, Pass VICTIM tới DMZ-WEB 172.16.6.10:80, Pass DNS-SERVER tới Kali 10.10.0.200:53 sau poison, Pass VICTIM tới Kali 10.10.0.200:80 Fake Site, Block sang LAN 10.6.0.0/24 có Log, Block sang MGMT 192.168.56.0/24, Block Any còn lại. ATTACKNET 5 rule: Pass Kali 10.10.0.200 tới DNS-SERVER 10.9.0.10:53, Pass Kali tới AUTH-DNS 172.16.6.20:5300 race relay, Block sang LAN 10.6.0.0/24 có Log, Block sang MGMT có Log, Block Any còn lại có Log.
- **Phần D — Chuẩn bị DNS-SERVER và AUTH-DNS:**
  - **D.1:** sao lưu cấu hình Lab 8 vào bind-backup-lab8, đủ 4 file named.conf.local, named.conf.options, db.lab9.test, db.10.9.0. Chép 2 file baseline, named.conf.options.baseline có recursion chỉ 10.9.0.0/24, dnssec-validation auto, validate-except example.lab10.test, named.conf.local.baseline forward example.lab10.test tới 172.16.6.20. named-checkconf không lỗi, restart BIND9 active running PID 1847. Chép vuln_resolver_lab10.py về thư mục home. Lúc đầu thiếu dnspython, báo ModuleNotFoundError, đã bật NAT tạm ens33 192.168.250.130, cài xong python3-dnspython 2.6.1, dnsutils đã có sẵn 1:9.18.39, rồi tắt NAT lại để cô lập, ens33 DOWN, ens37 UP 10.9.0.10/24.
  - **D.2:** DMZ-WEB đăng nhập lab6, thêm lại IP phụ 172.16.6.20/24 trên ens33, IP phụ này mất khi reboot giống Lab 6, kiểm tra không còn process AUTH-DNS cũ của Lab 8. Chép auth_dns_relay_lab10.py về thư mục home qua Shared Folder, py_compile ra script OK. Lúc đầu DMZ-WEB cũng thiếu dnspython, đã thêm card NAT tạm ens37 192.168.250.133, phải xóa default qua 172.16.6.1 mới ping 8.8.8.8 thông, cài xong python3-dnspython 2.1.0, rồi gỡ card NAT, thêm lại IP phụ .20. Chạy AUTH-DNS bằng nohup, PID 1257, nghe 172.16.6.20:53 và 5300, in ra LAB10 AUTH-DNS on 172.16.6.20:53, www.example.lab10.test -> 172.16.6.10, random labels NXDOMAIN after 3.0s.
  - **D.3:** VICTIM 10.9.0.100/24, DNS trỏ về stub 127.0.0.53. dig @10.9.0.10 www.example.lab10.test ra 172.16.6.10, baseline đúng, ảnh này dán mục 2 báo cáo. Lúc test baseline có gặp lỗi: AUTH-DNS chỉ nhận query từ 10.9.0.10:33333, trong khi BIND dùng port ngẫu nhiên, tcpdump thấy BIND hỏi từ port 51044 nên bị AUTH-DNS bỏ qua, dig timeout dù BIND đang chạy và gói đã tới DMZ-WEB. Em đã sửa tạm bằng cách thêm query-source address 10.9.0.10 port 33333 vào named.conf.options để baseline chạy được, named-checkconf chỉ cảnh báo port is deprecated, đúng như tài liệu nói cấu hình ép cổng nguồn cũ đã bị deprecate, rồi bỏ dòng này ra ở mục 7.
- **Phần I (làm trước để bù ảnh) — Khôi phục + hardening:**
  - **I.1:** bỏ dòng query-source khỏi named.conf.options, named-checkconf không lỗi, restart BIND9 active running PID 1674. ps aux grep vuln chỉ thấy dòng grep, không còn resolver mô phỏng nào chạy.
  - **I.2:** recursion ACL trong file config chỉ cho 10.9.0.0/24, Kali ATTACKNET không có quyền recursion.
  - **I.3:** quan sát source-port randomization bằng tcpdump trên DNS-SERVER. Lần 1, 6 gói hỏi root, port 53935, 56649, 40092, 46578, 51902, 50917. Lần 2, 3 gói sang AUTH-DNS hỏi abc123.example.lab10.test, port 42838, 60245, 56709. Tất cả đều khác nhau, không có gói nào là 33333 nữa.
  - **I.4:** dnssec-validation auto, riêng example.lab10.test được validate-except vì là namespace .test nội bộ.
- **Phần K — Câu hỏi phân tích:** đã trả lời ngắn gọn đủ 12 câu, kèm ghi chú ảnh theo 7 mục báo cáo, câu nào có ảnh, câu nào chưa có đều ghi rõ.

## 2. Những chỗ chưa làm được

- **Phần E, F, G, H:** chưa làm được vì máy không bật nổi 5 VM cùng lúc, VMware báo Not enough physical memory khi bật Kali. Không có ảnh mục 3, 4, 5, 6.
- **Phần I.5:** chưa tắt rule UDP 5300 và gỡ ATTACKNET, vì em giữ lại để lần sau thử tấn công tiếp.
- **Phần L:** báo cáo Word chưa viết xong. Ảnh mục 1, 2, 7 đã có, ảnh mục 3, 4, 5, 6 chưa có.

## 3. Lưu ý để kiểm tra hoặc chạy lại bài

- **Môi trường:** 5 VM trên VMware — pfSense, DNS-SERVER, VICTIM, Kali, DMZ-WEB. Không dùng VirtualBox. Tắt DC01 và WEB-PROXY trong suốt Lab 9.
- **Bảng địa chỉ IP:**

  | Thiết bị | IP |
  |---|---|
  | pfSense ATTACKNET | 10.10.0.1/24 |
  | DNS-SERVER | 10.9.0.10/24 |
  | VICTIM | 10.9.0.100/24 |
  | Kali | 10.10.0.200/24 |
  | DMZ-WEB | 172.16.6.10/24 |
  | AUTH-DNS | 172.16.6.20:53 + :5300 |

- **Lưu ý quan trọng:**
  - IP Kali đặt bằng nmcli nên còn sau reboot, nhưng 2 route đặt tay mất khi tắt máy, bật lại phải gõ lại.
  - IP phụ 172.16.6.20 trên DMZ-WEB mất khi reboot, bật lại phải gõ lại.
  - Shared Folder hiện trong vmware-hgfsclient vẫn phải mount tay mới thấy file.
  - Sau khi sửa rule phải Apply rồi Reset States mới test lại.
  - Lần sau làm lại E, F, G: reboot máy thật, đóng bớt app, giảm RAM từng VM xuống 1024 MB, hoặc bật VMware swap rồi bật đủ 5 máy.
