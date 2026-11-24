# Lab 9 — Remote DNS Cache Poisoning (Kaminsky)

- **Họ và tên sinh viên:** Nguyễn Phú Thọ
- **Mã số sinh viên:** 1150080158
- **Lớp:** CNPM2
- **Môn:** An toàn bảo mật HTTT
- Link youtube: 
- Link video: 



  
- em xin phép thầy thông cảm, máy em đời cũ, RAM không đủ nên không chạy nổi 5 máy ảo cùng lúc. Nhiều phần dưới đây em đã làm tới đâu thì ghi rõ tới đó, chỗ nào chưa làm được em ghi rõ lý do, không bịa ảnh hay bịa kết quả. Em mong thầy thông cảm và cho em nộp bài với phần đã làm được, kèm bảng so sánh và câu hỏi phân tích đầy đủ.

## 1. Nội dung đã thực hiện

- **Phần A–B — Tổng quan và mô hình:** Lab 9 kế thừa toàn bộ hạ tầng Lab 8 (pfSense-LAB6, DNS-SERVER 10.9.0.10, VICTIM 10.9.0.100, DMZ-WEB 172.16.6.10), thêm subnet mới `ATTACKNET 10.10.0.0/24`, Kali chuyển sang `10.10.0.200`, Promiscuous = Deny, không sniff DNSLAB nữa. Tài nguyên Lab 9 ở `C:\Users\Tho\Downloads\Tool_ATBMHTTT\LAB10\LAB10\LAB10_Resources` (đủ 01–07 + `README_FIRST.txt`).
- **Phần C — Chuyển từ Lab 8 sang Lab 9:**
  - **C.1:** tắt hết VM, chụp snapshot pfSense-LAB6 tên `LAB10_START_FROM_LAB9`, mô tả "Truoc Lab 9, tu Lab 8".
  - **C.2:** thêm Adapter 6 = LAN Segment `attack-net` cho pfSense, trong WebGUI hiện thành `em5 (00:0c:29:7e:8c:36)`, gán thành `OPT4`, đổi tên `ATTACKNET`, IP tĩnh `10.10.0.1/24`, Gateway None, Save + Apply. WebGUI lúc đầu vào bằng http không được, ping `192.168.56.254` vẫn 4/4 Reply — phải vào bằng `https://192.168.56.254` mới được.
  - **C.3:** Kali mới (xem lưu ý dưới) thêm Network Adapter 2 = LAN Segment `attack-net`, card 1 NAT, card 2 attack-net. Đặt IP bằng `nmcli con add type ethernet ifname eth1 con-name attacknet ipv4.addresses 10.10.0.200/24 ipv4.method manual` rồi `nmcli con up attacknet` (dùng `ip addr add` bị NetworkManager ghi đè mất). Thêm route `10.9.0.0/24` và `172.16.6.0/24 via 10.10.0.1 dev eth1`. Ping gateway `10.10.0.1` timeout 100% là kỳ vọng (không có rule ICMP), `ip neigh show dev eth1` thấy đúng MAC pfSense em5 `00:0c:29:7e:8c:36`. Công cụ đủ: `python3-scapy 2.7.0`, `wireshark 4.6.6-1`, `tcpdump 4.99.6-2`.
  - **C.4:** ruleset đủ 12 rule đúng thứ tự, đã Apply + Reset States:
    - **DNSLAB (7 rule):** 1 Pass DNS-SERVER 10.9.0.10 → AUTH-DNS 172.16.6.20:53, 2 Pass VICTIM → DMZ-WEB 172.16.6.10:80, 3 Pass DNS-SERVER 10.9.0.10 → Kali 10.10.0.200:53 (sau poison), 4 Pass VICTIM → Kali 10.10.0.200:80 (Fake Site), 5 Block sang LAN 10.6.0.0/24 + Log, 6 Block sang MGMT 192.168.56.0/24, 7 Block Any còn lại.
    - **ATTACKNET (5 rule):** 1 Pass Kali 10.10.0.200 → DNS-SERVER 10.9.0.10:53, 2 Pass Kali → AUTH-DNS 172.16.6.20:5300 (race relay), 3 Block sang LAN 10.6.0.0/24 + Log, 4 Block sang MGMT + Log, 5 Block Any còn lại + Log.
- **Phần D — Chuẩn bị DNS-SERVER và AUTH-DNS:**
  - **D.1 — Baseline BIND9:** sao lưu cấu hình Lab 8 vào `~/bind-backup-lab8/` (đủ 4 file `named.conf.local`, `named.conf.options`, `db.lab9.test`, `db.10.9.0`), chép 2 file baseline (`named.conf.options.baseline`: recursion chỉ `10.9.0.0/24`, `dnssec-validation auto`, `validate-except example.lab10.test`; `named.conf.local.baseline`: forward `example.lab10.test` → `172.16.6.20`), `named-checkconf` không lỗi, restart BIND9 active running PID 1847. Chép `vuln_resolver_lab10.py` về `~/`. Lúc đầu thiếu `dnspython` (ModuleNotFoundError) — đã bật NAT tạm (ens33 `192.168.250.130`), cài xong `python3-dnspython 2.6.1`, dnsutils đã có sẵn `1:9.18.39`, rồi tắt NAT lại để cô lập (ens33 DOWN, ens37 UP 10.9.0.10/24).
  - **D.2 — AUTH-DNS/RELAY trên APP-TEST (172.16.6.20):** DMZ-WEB đăng nhập `lab6`, thêm lại IP phụ `172.16.6.20/24` trên ens33 (IP phụ mất khi reboot, giống Lab 6), kiểm tra không còn process AUTH-DNS cũ của Lab 8, chép `auth_dns_relay_lab10.py` về `~/` qua Shared Folder, `python3 -m py_compile` ra `script OK`. Lúc đầu DMZ-WEB cũng thiếu `dnspython` — đã thêm card NAT tạm ens37 (`192.168.250.133`), phải xóa default via `172.16.6.1` mới ping `8.8.8.8` thông, cài xong `python3-dnspython 2.1.0`, rồi gỡ card NAT, thêm lại IP phụ .20. Chạy AUTH-DNS bằng `sudo nohup python3 ~/auth_dns_relay_lab10.py > ~/auth-dns.log 2>&1 &` (PID 1257), nghe `172.16.6.20:53` + `:5300`, in ra `LAB10 AUTH-DNS on 172.16.6.20:53; www.example.lab10.test -> 172.16.6.10; random labels NXDOMAIN after 3.0s`.
  - **D.3 — Baseline từ VICTIM:** VICTIM `10.9.0.100/24`, DNS trỏ về stub `127.0.0.53` (systemd-resolved). `dig @10.9.0.10 www.example.lab10.test +short` ra **`172.16.6.10`** — baseline đúng, ảnh này dán mục 2 báo cáo.
- **Phần I (làm trước, để bù ảnh cho mục 3–6) — Khôi phục + hardening:**
  - **I.1:** đã bỏ dòng `query-source` khỏi `/etc/bind/named.conf.options`, `named-checkconf` không lỗi, restart BIND9 active running PID 1674. `ps aux | grep vuln` chỉ thấy dòng grep — không còn resolver mô phỏng nào chạy.
  - **I.2:** recursion ACL trong file config chỉ cho `10.9.0.0/24`, Kali ATTACKNET không có quyền recursion.
  - **I.3:** quan sát source-port randomization bằng tcpdump trên DNS-SERVER: lần 1 (6 gói hỏi root) port `53935, 56649, 40092, 46578, 51902, 50917`; lần 2 (3 gói sang AUTH-DNS hỏi `abc123.example.lab10.test`) port `42838, 60245, 56709` — tất cả đều khác nhau, không có gói nào là `33333` nữa.
  - **I.4:** `dnssec-validation auto`, riêng `example.lab10.test` được `validate-except` vì là namespace `.test` nội bộ.
  - **I.5:** chưa làm (xem mục 2 dưới đây).
- **Phần K — Câu hỏi phân tích:** đã có ý trả lời ngắn gọn cho đủ 12 câu, kèm vị trí chèn hình theo 7 mục ảnh báo cáo và ghi chú chỗ chưa có hình (xem mục 2). Câu 3 tính xác suất batch K là `K / 65536` (batch 4096 → 1/16 ≈ 6,25%).

## 2. Kết quả thực hiện và những chỗ chưa làm được vì máy yếu

- **Đã xác minh xong, có ảnh thật:** snapshot `LAB10_START_FROM_LAB9` (C.1), ATTACKNET + Assignments (C.2), Kali IP + ARP + công cụ (C.3), ruleset DNSLAB 7 + ATTACKNET 5 (C.4), BIND baseline + checkconf + status (D.1), AUTH-DNS đang chạy (D.2), baseline VICTIM dig ra `172.16.6.10` (D.3, mục 2), BIND chạy lại + file config đã hardening + tcpdump port random (mục 7).
- **Chưa làm được — Phần E (Xác nhận tính "remote"):** chưa bật được Kali làm máy thứ 5. Lúc 11:03 ngày 24/11, VMware báo **"Not enough physical memory is available to power on this virtual machine"** khi đang chạy pfSense + DNS-SERVER + DMZ-WEB + VICTIM. Không có ảnh mục 3 (tcpdump trên Kali phải ra 0 packets, không thấy `10.9.0.10 ↔ 172.16.6.20`).
- **Chưa làm được — Phần F (Pha tấn công, resolver mô phỏng):** chưa làm, vì cũng cần đủ 5 máy cùng lúc (dừng BIND, chạy `vuln_resolver_lab10.py`, chạy evil DNS + fake web trên Kali).
- **Chưa làm được — Phần G (Task 1, Kaminsky):** chưa làm, vì chưa chạy được F. Không có ảnh mục 4 (AUTH-DNS `[PENDING]` / `[HIT]`, resolver `[CACHE]`), mục 5 (packet capture `10.9.0.10:33333 → 172.16.6.20:53` + gói giả trả về, NS = `ns.evil...`, glue A = `10.10.0.200`), mục 6 (VICTIM dig ra `10.10.0.200` + mở Fake Site).
- **Chưa làm được — Phần H (Task 2, phân tích cache):** chưa làm, vì chưa có kết quả G để so sánh trước/sau và trả lời câu hỏi bailiwick/glue bằng bằng chứng thực tế (chỉ trả lời bằng lý thuyết ở phần K).
- **Chưa làm được — Phần I.5 (Tắt ATTACKNET sau buổi lab):** chưa làm (chưa disable/xóa rule UDP 5300, chưa tắt/gỡ interface ATTACKNET), vì em vẫn giữ lại để lần sau có thể thử tấn công tiếp khi máy đủ RAM.
- **Chưa làm được — Phần L (Báo cáo Word):** file báo cáo chưa viết xong, mới chỉ có file README này + ý trả lời 12 câu ở phần K. Ảnh bắt buộc còn thiếu: mục 3, mục 4, mục 5, mục 6 (xem bảng ở mục 1). Ảnh mục 1, 2, 7 đã có.
- **Lý do chung cho các phần chưa làm được:** máy em đời cũ, RAM không đủ để chạy 5 VM cùng lúc (pfSense + DNS-SERVER + VICTIM + Kali + DMZ-WEB). Em đã chia từng đợt chỉ 3–4 VM để làm được tới D.3 và mục 7, nhưng tới phần E thì VMware báo hết bộ nhớ vật lý, không bật thêm Kali được. Em đã thử các cách giảm tải (tắt Kali khi chưa cần, tắt DMZ-WEB/VICTIM khi chưa cần), nhưng pha tấn công bắt buộc phải bật đủ 5 máy cùng lúc nên không thể lách được. Lần sau em sẽ thử reboot máy thật để giải phóng RAM, đóng bớt ứng dụng (Edge...), giảm RAM từng VM xuống 1024 MB, hoặc bật VMware swap (Preferences → Memory → Allow most VM memory to be swapped) rồi làm lại E/F/G.
- **Lỗi của tài liệu / script thầy (em ghi rõ để thầy biết, không phải do em làm sai):** script `auth_dns_relay_lab10.py` chỉ nhận query DNS từ đúng `10.9.0.10:33333` (xem dòng `if addr[0]!=RESOLVER or addr[1]!=RESOLVER_PORT: continue` trong `dns_loop`), trong khi BIND9 hiện đại dùng source port ngẫu nhiên (tcpdump trên DMZ-WEB thấy BIND hỏi từ port `51044` nên bị AUTH-DNS bỏ qua, dig timeout dù BIND đang chạy và gói đã tới DMZ-WEB). Em đã sửa tạm bằng cách thêm `query-source address 10.9.0.10 port 33333;` vào `/etc/bind/named.conf.options` để baseline D.3 chạy được (named-checkconf chỉ cảnh báo `port is deprecated`, đúng như tài liệu §F.1 nói cấu hình ép cổng nguồn cũ đã bị deprecate), rồi bỏ dòng này ra ở mục 7 để BIND random trở lại. Vì vậy, sau khi hardening, dig baseline qua BIND lại timeout (AUTH-DNS không nhận port ngẫu nhiên) — ảnh restore dig về `172.16.6.10` bằng BIND đã hardening không lấy được, em ghi rõ hạn chế này ở đây chứ không bịa ảnh.

## 3. Lưu ý để kiểm tra hoặc chạy lại bài

- **Môi trường:** 5 VM trên VMware Workstation — `pfSense-LAB6`, `DNS-SERVER` (Ubuntu Server 24.04, user `tho333`), `VICTIM` (Ubuntu Server 24.04, user `tho333`), `Kali` (Kali Linux 2026.2, user `kali` / `kali`), `DMZ-WEB` (Ubuntu Server 22.04, user `lab6`, dùng lại từ Lab 6). Không dùng VirtualBox. Tắt `DC01` và `WEB-PROXY` (Lab 7) trong suốt Lab 9.
- **Bảng địa chỉ IP:**

  | Thiết bị | Interface | IP |
  |---|---|---|
  | pfSense | WAN (VMnet8) | DHCP |
  | pfSense | MGMT (VMnet1) | 192.168.56.254/24 (WebGUI, vào bằng https) |
  | pfSense | LAN (lab6-lan) | 10.6.0.1/24 |
  | pfSense | DMZ (lab6-dmz) | 172.16.6.1/24 |
  | pfSense | DNSLAB (LAN Segment `dns-lab`, em4) | 10.9.0.1/24 |
  | pfSense | ATTACKNET (LAN Segment `attack-net`, em5, 00:0c:29:7e:8c:36) | 10.10.0.1/24 |
  | DNS-SERVER | dns-lab (ens37) | 10.9.0.10/24, GW 10.9.0.1 |
  | VICTIM | dns-lab (ens37) | 10.9.0.100/24, GW 10.9.0.1, DNS 10.9.0.10 |
  | Kali | NAT (eth0) | 192.168.250.132/24 (DHCP) |
  | Kali | attack-net (eth1, nmcli `attacknet`) | 10.10.0.200/24, GW 10.10.0.1 |
  | DMZ-WEB | DMZ (ens33) | 172.16.6.10/24 (+ IP phụ .20 cho AUTH-DNS) |
  | AUTH-DNS | APP-TEST (IP phụ trên DMZ-WEB) | 172.16.6.20:53 + :5300 |

- **Thứ tự bật VM:** pfSense → DNS-SERVER → DMZ-WEB → VICTIM → Kali (Kali bật cuối, chỉ khi cần E/F/G). Khi máy yếu, chia từng đợt chỉ 3–4 VM, không bật cả 5 cùng lúc — riêng pha tấn công E/F/G bắt buộc phải đủ 5 máy.
- **Lưu ý quan trọng:**
  - **Kali bị xóa và làm lại:** ngày 24/11 em lỡ xóa file Kali, phải giải nén lại từ `kali-linux-2026.2-vmware-amd64.7z` trong Downloads. Coi Kali Lab 9 như máy mới hoàn toàn: mất hết `~/lab9/`, `~/lab9-fake/`, IP đặt tay cũ, Shared Folder, file attacker Lab 8. Đã chép lại `kaminsky_remote_lab10.py` + `evil_dns_lab10.py` vào `~/lab10/`, `fake_index.html` vào `~/lab10-fake/index.html`, `python3 -m py_compile` ra `script OK`.
  - IP Kali `10.10.0.200` đặt bằng `nmcli` nên **còn sau reboot**, nhưng 2 route đặt tay (`10.9.0.0/24` và `172.16.6.0/24 via 10.10.0.1`) **mất khi tắt máy** — bật lại Kali phải gõ lại 2 route này. Ping gateway `10.10.0.1` timeout là kỳ vọng (không có rule ICMP) — kiểm chứng bằng `ip neigh` (thấy MAC pfSense em5).
  - IP phụ `172.16.6.20` trên DMZ-WEB đặt bằng `ip addr add` nên **mất khi reboot** — bật lại DMZ-WEB phải gõ lại `sudo ip addr add 172.16.6.20/24 dev ens33`.
  - AUTH-DNS chạy lần đầu bằng `sudo python3 ~/auth_dns_relay_lab10.py` ở terminal thường thì bị tắt khi chuyển terminal — phải chạy nền bằng `sudo nohup python3 ~/auth_dns_relay_lab10.py > ~/auth-dns.log 2>&1 &` (log bị Python buffer nên `cat ~/auth-dns.log` chỉ thấy dòng `nohup: ignoring input`, kiểm tra bằng `ps aux | grep auth_dns` và `sudo ss -tulnp | grep 53` thấy `172.16.6.20:53` + `:5300` mới chắc là đang chạy).
  - Cài `python3-dnspython` trên DNS-SERVER và DMZ-WEB đều phải bật NAT tạm, vì 2 máy này đang cô lập không có mạng. Riêng DMZ-WEB: thêm card NAT ens37 xong phải bật tay (`sudo ip link set ens37 up` + `sudo dhclient ens37`), rồi **xóa default via `172.16.6.1`** mới ping `8.8.8.8` thông (default cũ metric 0 thắng route NAT metric 50). Cài xong phải gỡ card NAT và thêm lại IP phụ .20.
  - Shared Folder VMware hiện trong `vmware-hgfsclient` vẫn chưa đủ — phải mount tay bằng `sudo mkdir -p /mnt/hgfs && sudo mount -t fuse.vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other` rồi mới `ls /mnt/hgfs/` thấy `LAB10_Resources`. Chuyển file nguyên vẹn qua Shared Folder, không dán script dài qua clipboard.
  - Sau mỗi lần sửa rule phải bấm **Apply Changes**; khi test không đổi sau khi đổi rule, vào **Diagnostics → States → Reset** rồi test lại. Bấm Reset xong trang xoay hoài là bình thường, bấm F5 refresh lại là được.
  - Để làm lại E/F/G khi máy đủ RAM: bật đủ 5 VM, kiểm tra lại IP 5 máy + BIND đang chạy + AUTH-DNS đang chạy nền (PID cũ đã tắt thì chạy lại bằng `nohup` như trên), rồi làm theo §E → §F → §G trong `LAB9_HuongDan.docx`. Nhớ: trước khi tấn công phải thêm lại dòng `query-source address 10.9.0.10 port 33333;` vào BIND **hoặc** dùng `vuln_resolver_lab10.py` thay BIND ở pha tấn công (theo §F.2: dừng BIND rồi chạy resolver mô phỏng, vì nó luôn phát upstream từ UDP 33333), nếu không AUTH-DNS sẽ bỏ qua query như đã gặp ở D.3.
