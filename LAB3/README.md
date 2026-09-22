# Báo cáo Thực hành Lab 3: Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

- **Sinh viên:** Nguyễn Phú Thọ
- **MSSV:** 1150080158
- **Trường:** Trường Đại học Tài nguyên và Môi trường TP.HCM
- **Phiên bản máy ảo:** Windows 11 25H2 x64 (Build 26200.9445) trên VMware Workstation Pro[cite: 1]

## 1. Nội dung đã thực hiện
Trong bài Lab này, em đã thiết lập môi trường cô lập an toàn trên máy ảo và hoàn thành toàn bộ 7 tình huống giả lập các mối đe dọa an toàn thông tin, cụ thể[cite: 1]:
*   **Thiết lập Baseline:** Cài đặt các công cụ cần thiết (Python, Wireshark, bộ Sysinternals) và trích xuất trạng thái hệ thống ban đầu[cite: 1].
*   **TH1 (Đánh giá rủi ro):** Lập bảng phân tích tài sản, lỗ hổng, rủi ro và phân loại 5 tình huống an toàn thông tin[cite: 1].
*   **TH2 (Malware):** Tạo file kiểm thử EICAR để xác minh tính năng bảo vệ theo thời gian thực của Microsoft Defender[cite: 1].
*   **TH3 (Tấn công xác thực):** Giả lập đăng nhập sai mật khẩu, đổi mật khẩu và trích xuất các log xác thực (Event ID 4624, 4625, 4648) để phân tích[cite: 1].
*   **TH4 (Backdoor/Persistence):** Cấu hình cơ chế tự khởi động (Registry Run, Scheduled Task), mở cổng mạng 8080 trên localhost, sau đó dùng Sysmon, Autoruns và Process Explorer để truy vết[cite: 1].
*   **TH5 (Sniffing/MITM):** Dùng Wireshark bắt các gói tin HTTP (dạng văn bản rõ) và HTTPS (đã mã hóa) để so sánh trực quan[cite: 1].
*   **TH6 (DoS/DDoS/Mail Bombing):** Chạy script giả lập tải cục bộ để hiểu về DoS, đồng thời phân tích các file log CSV để nhận diện nguồn DDoS và hành vi dội bom thư[cite: 1].
*   **TH7 (Social Engineering):** Phân tích mẫu email Phishing offline để tìm ra các chỉ dấu lừa đảo và phân loại các kỹ thuật tấn công phi kỹ thuật[cite: 1].
*   **Cleanup:** Gỡ bỏ toàn bộ cơ chế Persistence, đóng cổng mạng, đối chiếu lại Autoruns và xuất mã băm SHA-256 cho toàn bộ bằng chứng[cite: 1].

## 2. Kết quả thực hiện
*   **Đánh giá:** **PASS** toàn bộ các tình huống[cite: 1].
*   **Bằng chứng:** Đã thu thập đủ 11 ảnh chụp màn hình (từ H1 đến H11) thực hiện trực tiếp trên máy ảo cá nhân, khớp với mốc thời gian (timestamp) sinh ra trong các file log[cite: 1].
*   **Dữ liệu toàn vẹn:** Đã xuất file `evidence_sha256.csv` chứa mã băm của mọi tài liệu bằng chứng để đảm bảo dữ liệu không bị chỉnh sửa sau khi thực hành[cite: 1].
*   Hệ thống máy ảo đã được làm sạch an toàn (Defender giữ bật, cổng mạng đã đóng, không còn file EICAR) trước khi đóng gói báo cáo[cite: 1].

## 3. Các lưu ý dành cho giảng viên khi kiểm tra
Để thuận tiện cho thầy/cô trong quá trình kiểm tra và chấm điểm, em xin lưu ý một số điểm sau:
*   **Môi trường mạng:** Toàn bộ quá trình chạy script gây tải (DoS) và giả lập tấn công đều được em giới hạn nghiêm ngặt ở mục tiêu `127.0.0.1` (localhost) trên mạng Host-only[cite: 1]. Em chỉ bật mạng NAT vài giây ở TH5 để lấy gói tin HTTPS từ một tên miền mẫu, sau đó lập tức cô lập lại máy ảo[cite: 1].
*   **Xác minh bằng chứng:** Thầy/cô có thể dùng file `evidence_sha256.csv` để đối chiếu chéo (checksum) với các file log .txt và .csv trong thư mục báo cáo nhằm xác nhận tính nguyên bản của dữ liệu[cite: 1].
*   **Tuân thủ an toàn Repository:** Em không tải lên GitHub bất kỳ tệp thực thi nào (.exe), công cụ cài đặt hay file giả lập mã độc (EICAR) để tránh vi phạm chính sách quét mã độc tự động của nền tảng[cite: 1]. File báo cáo Word đính kèm đã che/ẩn các thông tin định danh cá nhân theo đúng quy định[cite: 1].
