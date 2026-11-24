# Lab 9 — Remote DNS Cache Poisoning

Họ và tên: Nguyễn Phú Thọ
MSSV: 1150080158
Lớp: CNPM2
Môn: An toàn bảo mật HTTT
Link youtube:
Link video: 



Em xin phép thầy thông cảm, máy em yếu, RAM không đủ nên không chạy nổi 5 máy ảo cùng lúc. Chỗ nào chưa làm được em ghi rõ lý do, không bịa ảnh hay bịa kết quả.

## Đã làm được

Phần C: chụp snapshot pfSense tên LAB10_START_FROM_LAB9. Thêm card ATTACKNET cho pfSense, IP 10.10.0.1/24. Kali mới đặt IP 10.10.0.200 bằng nmcli, thêm 2 route qua 10.10.0.1, đủ công cụ scapy, wireshark, tcpdump. Làm đủ 12 rule firewall, DNSLAB 7 rule, ATTACKNET 5 rule, đã Apply và Reset States.

Phần D: sao lưu cấu hình Lab 8, chép 2 file baseline cho BIND, restart BIND chạy được. Cài dnspython cho DNS-SERVER và DMZ-WEB bằng NAT tạm rồi tắt NAT lại. Chạy AUTH-DNS trên 172.16.6.20:53 và 5300, PID 1257. VICTIM dig www.example.lab10.test ra 172.16.6.10, baseline đúng.

Phần I làm trước để bù ảnh: bỏ dòng query-source khỏi BIND, restart BIND chạy lại PID 1674, không còn resolver mô phỏng. Tcpdump thấy port nguồn random, ví dụ 42838, 60245, 56709, không còn 33333.

Phần K: đã trả lời ngắn gọn đủ 12 câu hỏi, kèm ghi chú ảnh theo 7 mục báo cáo.

## Chưa làm được

Phần E: chưa bật được Kali làm máy thứ 5. Ngày 24/11 lúc 11:03, VMware báo Not enough physical memory khi đang chạy pfSense, DNS-SERVER, DMZ-WEB, VICTIM. Không có ảnh mục 3.

Phần F và G: chưa làm, vì cũng cần đủ 5 máy cùng lúc. Không có ảnh mục 4, mục 5, mục 6.

Phần H: chưa làm, vì chưa có kết quả G để so sánh.

Phần I.5: chưa tắt rule UDP 5300 và gỡ ATTACKNET, vì em giữ lại để lần sau thử tấn công tiếp.

Phần L: báo cáo Word chưa viết xong. Ảnh mục 1, 2, 7 đã có, ảnh mục 3, 4, 5, 6 chưa có.

Lỗi script thầy: AUTH-DNS chỉ nhận query từ 10.9.0.10:33333, trong khi BIND dùng port ngẫu nhiên nên dig timeout dù BIND đang chạy. Em sửa tạm bằng cách thêm query-source port 33333 vào BIND để baseline chạy được, rồi bỏ dòng này ra ở mục 7. Vì vậy sau khi hardening, dig baseline qua BIND lại timeout, ảnh restore dig về 172.16.6.10 không lấy được, em ghi rõ ở đây.

## Lưu ý khi chạy lại

5 VM trên VMware: pfSense, DNS-SERVER 10.9.0.10, VICTIM 10.9.0.100, Kali 10.10.0.200, DMZ-WEB 172.16.6.10, AUTH-DNS 172.16.6.20. Tắt DC01 và WEB-PROXY trong suốt Lab 9.

Kali bị xóa và làm lại ngày 24/11, coi như máy mới, đã chép lại 3 file attacker vào lab10 và lab10-fake. IP Kali đặt bằng nmcli nên còn sau reboot, nhưng 2 route đặt tay mất khi tắt máy, bật lại phải gõ lại. IP phụ 172.16.6.20 trên DMZ-WEB cũng mất khi reboot, bật lại phải gõ lại.

Shared Folder hiện trong vmware-hgfsclient vẫn phải mount tay mới thấy file. Sau khi sửa rule phải Apply rồi Reset States mới test lại.

Lần sau làm lại E, F, G: reboot máy thật, đóng bớt app, giảm RAM từng VM xuống 1024 MB, hoặc bật VMware swap rồi bật đủ 5 máy.
