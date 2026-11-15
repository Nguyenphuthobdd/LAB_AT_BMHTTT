# Lab 8 — Local DNS Attack (DNS Spoofing & Cache Poisoning)

- **Họ và tên sinh viên:** Nguyễn Phú Thọ
- **Mã số sinh viên:** 1150080158
- **Lớp:** CNPM2
- **Môn:** An toàn bảo mật HTTT
- Link youtube: https://www.youtube.com/@akali6054
- Link Video 



## 1. Nội dung đã thực hiện

- **Phần A–B — Tổng quan và mô hình:** đã kiểm tra `sha256sum -c SHA256SUMS.txt` → **18/18 file OK**. Mạng mới `DNSLAB 10.9.0.0/24` (LAN Segment `dns-lab`) thêm vào pfSense-LAB6, không phá Lab 6/7. APP-TEST `172.16.6.20` là IP phụ sẵn trên DMZ-WEB từ Lab 6, không tạo VM mới.
- **Phần C — Dựng mạng DNS-LAB:**
  - **C.3:** gán interface mới `OPT3 = em4 (00:0c:29:7e:8c:2c)` thành `DNSLAB`, IP tĩnh `10.9.0.1/24`, Gateway để None.
  - **C.4:** ruleset DNSLAB đủ 5 rule đúng thứ tự và đúng Protocol, đã Apply: 1 Pass DNS TCP/UDP 53 (10.9.0.10 → 172.16.6.20), 2 Pass HTTP TCP 80 (10.9.0.0/24 → 172.16.6.10), 3 Block LAN Any + Log, 4 Block MGMT Any + Log, 5 Block Any + Log.
  - **C.5.1 — DNS-SERVER:** VM Ubuntu Server 24.04, RAM 2048 MB, disk 20 GB, 2 card NAT + LAN Segment `dns-lab`, username `tho333` (lúc cài gõ `tho333`, không phải `tho`), hostname `dns-server`, đã cài OpenSSH server.
  - **C.5.2 — VICTIM:** VM Ubuntu Server 24.04, RAM 2048 MB, disk 20 GB, username `tho333`, hostname thực tế là `tho333` (prompt `tho333@tho333`), 2 card NAT + `dns-lab` thêm đủ từ đầu.
  - **C.5.3 — Kali (ATTACKER):** VM VMware làm sẵn `kali-linux-2026.2-vmware-amd64`, đăng nhập `kali` / `kali`, thêm card LAN Segment `dns-lab`, giữ card NAT.
- **Phần D — BIND9 trên DNS-SERVER:**
  - **D.1:** IP tĩnh `10.9.0.10/24` cho card `ens37` (file `/etc/netplan/50-dnslab.yaml`, thêm `metric: 200` để NAT metric 100 vẫn là đường ra Internet chính), `ip neigh` thấy gateway `10.9.0.1` đúng MAC pfSense, `ping 8.8.8.8` 3/3 Reply.
  - **D.2:** `named -v` ra **BIND 9.18.39-0ubuntu0.24.04.9-Ubuntu**.
  - **D.3:** chép đủ 4 file qua Shared Folder VMware (`named.conf.options` 504, `named.conf.local` 324, `db.lab9.test` 313, `db.10.9.0` 216), có file `.bak` sao lưu.
  - **D.4:** `named-checkconf` không lỗi, 2 `named-checkzone` đều `OK` (serial 2026101101), `systemctl status bind9` **active (running)**.
  - **D.5:** `dig @127.0.0.1 www.lab9.test +short` ra `172.16.6.10`, `dig @127.0.0.1 -x 10.9.0.100 +short` ra `victim.lab9.test.` — sau khi sửa lỗi file thầy: đổi tên zone ngược trong `named.conf.local` từ `9.10.in-addr.arpa` thành **`0.9.10.in-addr.arpa`** (đúng mạng /24, khớp các bản ghi `10/100/200` trong `db.10.9.0`).
  - **D.6:** tắt card NAT, chỉ còn mạng lab: `ens33 DOWN`, `ens37 UP 10.9.0.10/24`, `ping 8.8.8.8` 100% loss là đúng (đã cô lập).
- **Phần E — VICTIM và ATTACKER:**
  - **E.1 — VICTIM:** IP tĩnh `10.9.0.100/24` cho `ens37` (file mẫu ghi `enp0s3` nên phải sửa thành `ens37`), DNS duy nhất `10.9.0.10`, `ping 10.9.0.10` 3/3 Reply, `dig www.lab9.test` ra `172.16.6.10`, sau đó tắt NAT: `ens33 DOWN`, `ens37 UP 10.9.0.100/24`.
  - **E.2 — Kali:** IP tĩnh `10.9.0.200/24` cho `eth1`, `python3-scapy 2.7.0`, `wireshark 4.6.6-1`, `tcpdump 4.99.6-2` đã có sẵn bản mới nhất, chép `dns_spoof_lab9.py` vào `~/lab9/` và `fake_index.html` vào `~/lab9-fake/index.html`, `python3 -m py_compile` ra **`script OK`**.
  - **E.3 — Web giả:** `sudo python3 -m http.server 80 --directory ~/lab9-fake`, trên VICTIM `curl http://10.9.0.200` ra trang **`LAB9 FAKE SITE` / `LAB 9 - TRANG MÔ PHỎNG BỊ CHUYỂN HƯỚNG`**.
- **Phần G — Task 1 (sửa file `/etc/hosts`):** trên VICTIM sao lưu `/etc/hosts` sang `/etc/hosts.bak`, thêm dòng `10.9.0.200 www.lab9.test`, kiểm tra `getent hosts www.lab9.test` ra `10.9.0.200`, `curl http://www.lab9.test` ra trang fake. Sau đó khôi phục ngay bằng `cp /etc/hosts.bak /etc/hosts`, `cat` kiểm tra không còn dòng `10.9.0.200`.
- **Phần J.1 — Khôi phục:** đã `rndc flush` trên DNS-SERVER và `resolvectl flush-caches` trên VICTIM, `/etc/hosts` đã khôi phục sạch, chụp snapshot **`LAB8_DONE`** cho pfSense-LAB6. Kali đã tắt nên script spoof + web giả coi như đã dừng, AUTH-DNS trên DMZ-WEB chưa từng chạy nên không có gì để dừng.
- **Phần L.1 — Câu hỏi phân tích:** đã có ý trả lời ngắn gọn cho đủ 10 câu, kèm vị trí chèn hình và ghi chú chỗ chưa có hình (Task 2/3, baseline).

## 2. Kết quả thực hiện và những chỗ chưa làm được vì máy yếu / VMware

- **Đã xác minh xong, có ảnh thật:** ruleset DNSLAB (C.4), IP 3 máy đúng bảng (C.5/D.1/E.1/E.2), BIND9 checkzone + status (D.4), dig xuôi/ngược (D.5), web giả (E.3), Task 1 getent + curl + khôi phục hosts (G + J.1).
- **Chưa làm được — Phần C.1:** snapshot `LAB8_START` cho pfSense (và Kali) trước khi thêm card DNSLAB chưa xác nhận xong.
- **Chưa làm được — Phần C.4:** bước Reset States (Diagnostics → States → Reset) sau khi Apply ruleset chưa xác nhận xong.
- **Chưa làm được — Phần F (Baseline):** em xin bỏ qua vì máy không bật nổi 5 VM cùng lúc. Chưa có ảnh F.1 (AUTH-DNS listening), F.2 (`dig www.external.lab9.test` ra `172.16.6.10` trên cả 2 máy), F.3 (Wireshark thấy 2 query).
- **Chưa làm được — Phần H (Task 2, Direct DNS Spoofing):** script đã chạy đúng (`listening for www.external.lab9.test`, `mode= victim iface= eth1`), đã flush cache 2 đầu, nhưng trên VICTIM `dig www.external.lab9.test +short` 2 lần đều ra trống, terminal Kali không in dòng `forged`. Đã chẩn đoán: Kali không thấy query — `tcpdump -i eth1 -n udp port 53` chạy trong lúc VICTIM dig mà không hiện gói nào. Đã thử `sudo ip link set eth1 promisc on` (`ip link show eth1` hiện `PROMISC`) rồi chạy lại script và dig lại, vẫn fail. Nguyên nhân là VMware LAN Segment chặn unicast tới Kali, bật PROMISC trong guest chưa đủ, cần sửa file `.vmx` của Kali thêm `ethernet1.noPromisc = "FALSE"` rồi bật lại máy mới nghe được — em chưa thực hiện bước này.
- **Chưa làm được — Phần I (Task 3, Cache Poisoning):** chưa làm, vì Task 3 cũng cần Kali nghe được query DNS-SERVER → AUTH-DNS như Task 2, và cần DMZ-WEB chạy AUTH-DNS (chưa bật). Không có ảnh cache dump (`rndc dumpdb`) và ảnh dig sau khi dừng script vẫn ra `10.9.0.200`.
- **Chưa làm được — Phần J.2 (Hardening):** chưa làm (xóa dòng `send-cookie no` trong `named.conf.options`, restart BIND9 rồi lặp lại Task 3), vì Task 3 chưa xong.
- **Chưa làm được — Phần L (Báo cáo Word):** file `CNPM2-Lab8_1150080158-NguyenPhuTho.docx` chưa viết, mới chỉ có ý trả lời 10 câu hỏi ở L.1. Ảnh bắt buộc L.2 còn thiếu: (4) baseline, (6) Task 2, (7) Task 3, (8) sau flush về `172.16.6.10` cho `external.lab9.test`.
- **Chưa chẩn đoán xong — WebGUI pfSense:** lúc 18:48 WebGUI `https://192.168.56.254` không vào được, em hỏi có cần restart pfSense không, chưa làm bước kiểm tra `ping 192.168.56.254` và trạng thái VM trên VMware.
- **Lý do chung cho các phần chưa làm được:** máy em đời cũ, không chạy nổi 5 VM (pfSense + DNS-SERVER + VICTIM + Kali + DMZ-WEB) cùng lúc, nên phải chia từng đợt 3–4 VM và bỏ qua phần F. Riêng Task 2/3 kẹt thêm ở điểm VMware: LAN Segment không có nút Promiscuous Mode Allow VMs như VirtualBox, phải sửa `.vmx` mới nghe được unicast.

## 3. Lưu ý để kiểm tra hoặc chạy lại bài

- **Môi trường:** 5 VM trên VMware Workstation — `pfSense-LAB6`, `DNS-SERVER` (Ubuntu Server 24.04), `VICTIM` (Ubuntu Server 24.04), `Kali` (Kali Linux 2026.2), `DMZ-WEB` (Ubuntu Server 22.04, dùng lại từ Lab 6). Không dùng VirtualBox. Tắt `DC01` và `WEB-PROXY` (Lab 7) trong suốt Lab 8.
- **Bảng địa chỉ IP:**

  | Thiết bị | Interface | IP |
  |---|---|---|
  | pfSense | WAN (VMnet8) | DHCP |
  | pfSense | MGMT (VMnet1) | 192.168.56.254/24 (WebGUI) |
  | pfSense | LAN (lab6-lan) | 10.6.0.1/24 |
  | pfSense | DMZ (lab6-dmz) | 172.16.6.1/24 |
  | pfSense | DNSLAB (LAN Segment `dns-lab`, em4) | 10.9.0.1/24 |
  | DNS-SERVER | dns-lab (ens37) | 10.9.0.10/24, GW 10.9.0.1, DNS 10.9.0.10 |
  | VICTIM | dns-lab (ens37) | 10.9.0.100/24, GW 10.9.0.1, DNS 10.9.0.10 |
  | Kali | NAT (eth0) | 192.168.250.132/24 (DHCP) |
  | Kali | dns-lab (eth1) | 10.9.0.200/24 |
  | DMZ-WEB | DMZ | 172.16.6.10/24 (+ IP phụ .20 cho AUTH-DNS) |

- **Thứ tự bật VM:** pfSense → DNS-SERVER → VICTIM → Kali → DMZ-WEB (chỉ bật DMZ-WEB khi cần chạy AUTH-DNS ở F.1). Khi máy yếu, chia từng đợt chỉ 3–4 VM, không bật cả 5 cùng lúc.
- **Lưu ý quan trọng:**
  - IP `10.9.0.200` của Kali đặt bằng lệnh `sudo ip addr add` nên **mất khi reboot** — bật lại Kali phải đặt lại IP, rồi mới chạy web giả + script. **Đừng gõ thêm `ip route add default via 10.9.0.1`** — lệnh này làm `apt update` lỗi Network is unreachable vì pfSense chặn đường đó, mà test trong cùng mạng `10.9.0.0/24` thì không cần gateway.
  - Ping gateway `10.9.0.1` từ DNS-SERVER/VICTIM timeout là bình thường (ruleset C.4 không có rule ICMP, rule 5 Block Any chặn hết) — kiểm chứng bằng `ip neigh` (thấy MAC pfSense), `ip route` và `ping 8.8.8.8` qua NAT thay cho ping gateway.
  - File thầy cho lệch zone ngược: zone khai `9.10.in-addr.arpa` nhưng bản ghi PTR trong `db.10.9.0` chỉ ghi `100` (kiểu /24) — phải sửa tên zone trong `named.conf.local` thành `0.9.10.in-addr.arpa` rồi checkzone lại + restart bind9 mới dig ngược ra `victim.lab9.test.`.
  - Shared Folder VMware hiện trong `vmware-hgfsclient` vẫn chưa đủ — phải mount tay bằng `sudo mkdir -p /mnt/hgfs && sudo mount -t fuse.vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other` rồi mới `ls /mnt/hgfs/` thấy `LAB9_Resources`. Chuyển file nguyên vẹn qua Shared Folder, không dán script dài qua clipboard.
  - Sau mỗi lần sửa rule phải bấm **Apply Changes**; khi test không đổi sau khi đổi rule, vào **Diagnostics → States → Reset** rồi test lại.
  - Để làm lại Task 2/3: tắt Kali, sửa file `.vmx` của Kali thêm dòng `ethernet1.noPromisc = "FALSE"`, bật lại Kali, đặt lại IP `10.9.0.200`, bật `ip link set eth1 promisc on`, chạy lại web giả + script, rồi flush cache 2 đầu (`rndc flush` + `resolvectl flush-caches`) trước khi dig.

