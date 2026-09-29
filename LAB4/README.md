# BÁO CÁO THỰC HÀNH LAB 4

- **Họ và tên sinh viên:** Nguyễn Phú Thọ[cite: 14]
- **Mã số sinh viên:** [1150080158][cite: 14]
- **Tên bài Lab:** Lab 4 - Khảo sát và đánh giá bề mặt mạng bằng Nmap[cite: 14]

## Nội dung đã thực hiện[cite: 14]
- Thiết lập môi trường mạng ảo riêng lập (Host-Only / VMnet1) trên phần mềm VMware giữa máy quét (Kali Linux) và máy mục tiêu (Metasploitable 2).
- Khám phá các thiết bị đang hoạt động trong mạng (Host discovery) bằng Nmap.
- Thực hành các kỹ thuật quét cổng TCP (TCP Connect, SYN scan, FIN, Xmas, NULL, ACK) và quét cổng UDP.
- Thực hiện định danh phiên bản dịch vụ (`-sV`) và hệ điều hành (`-O`, `-A`) của máy mục tiêu.
- Chạy các tập lệnh NSE (Nmap Scripting Engine) để rà soát thông tin và kiểm tra lỗ hổng SMB (MS17-010).
- Xuất kết quả quét lưu thành tệp văn bản.

## Kết quả thực hiện[cite: 14]
- Xác định được chính xác trạng thái các cổng mạng (open, closed, filtered) trên máy mục tiêu Metasploitable 2.
- Thu thập thành công các thông tin về phiên bản phần mềm, hệ điều hành và xuất tệp báo cáo hoàn chỉnh.
- Đã tổng hợp đầy đủ 8 ảnh minh chứng bắt buộc và hoàn thiện phần trả lời 10 câu hỏi phân tích vào phiếu báo cáo.

## Các lưu ý cần thiết để giảng viên có thể kiểm tra hoặc chạy lại bài[cite: 14]
- **Môi trường:** VMware Workstation.
- **Cấu hình mạng:** Sử dụng card mạng **VMnet1 (Host-only)** với dải IP `192.168.56.0/24`.
- **IP Máy quét (Kali Linux):** `192.168.56.129`.
- **IP Máy đích (Metasploitable 2):** `192.168.56.128`.
- **Lưu ý khi chạy lại lệnh:** Toàn bộ lệnh quét trong bài được ngắm vào mục tiêu là `192.168.56.128`. Để kiểm tra hoặc chạy lại, vui lòng thiết lập lại dải IP của Metasploitable 2 khớp với địa chỉ này, hoặc thay đổi IP trong câu lệnh cho phù hợp với môi trường thực tế.
