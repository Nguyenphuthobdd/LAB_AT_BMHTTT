# Báo cáo Lab 1: Bắt gói tin Telnet và SSH

* **Họ và tên: Nguyễn Phú Thọ
* **Mã số sinh viên: 1150080158
* **Môn học:** An toàn Hệ thống thông tin

---

## 1. Nội dung đã thực hiện
* Thiết lập môi trường và cấu hình tài khoản thực hành trên hệ thống.
* Thực hiện kết nối và bắt gói tin **Telnet** (cổng 23) bằng Wireshark; phân tích kết quả để chứng minh điểm yếu lộ thông tin (plaintext) ngay cả khi đổi mật khẩu phức tạp.
* Nghiên cứu, tìm hiểu cơ chế hoạt động, cấu trúc gói tin và tính bảo mật của **SSH** (cổng 22) thông qua tài liệu chuyên môn và phân tích đối chiếu.
* *Lưu ý về môi trường thực hành SSH:* Do điều kiện phần cứng và giới hạn tương thích của hệ điều hành trên máy ảo cá nhân không hỗ trợ cài đặt hoặc kích hoạt ổn định dịch vụ OpenSSH theo phương pháp truyền thống, em đã tập trung phân tích sâu về mặt lý thuyết chuyên môn, cấu trúc mã hóa phiên truyền và các thuộc tính CIA của giao thức SSH để hoàn thành các câu hỏi yêu cầu trong bài Lab.

## 2. Kết quả đạt được
* Đã thu thập và phân tích hoàn thiện các hình ảnh minh họa cho phiên làm việc Telnet bằng công cụ Wireshark.
* Làm rõ sự khác biệt cốt lõi về mặt bảo mật giữa giao thức không mã hóa (Telnet) và giao thức bảo mật (SSH).
* Hoàn thành đầy đủ hệ thống câu hỏi phân tích lý thuyết và thực nghiệm theo yêu cầu đề bài.

## 3. Cấu trúc thư mục nộp bài
* `baocao.docx`: File báo cáo chi tiết kèm link video quay lại quá trình thực hành.
* Thư mục chứa các hình ảnh chụp màn hình kết quả bắt gói tin Wireshark.
