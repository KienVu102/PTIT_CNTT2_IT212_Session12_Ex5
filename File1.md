# FILE 01: PROMPT YÊU CẦU CODE REVIEW & REFACTOR
**Dự án:** API Đăng ký mở tài khoản trực tuyến (eKYC) - ABC Bank
**Đối tượng:** Thầy/Cô và Hội đồng chấm bài chuyên môn

---

### Nội dung Prompt đã gửi cho AI (Antigravity):

"Chào bạn, tôi là sinh viên năm 2 ngành Công nghệ thông tin. Tôi đang phát triển một hệ thống API eKYC Đăng ký mở tài khoản ngân hàng bằng Spring Boot. 

Dưới đây là mã nguồn phiên bản đầu tiên của tôi (chạy được nhưng chưa tối ưu). Tôi muốn bạn đóng vai một Chuyên gia Code Review (Lead Developer) để giúp tôi thực hiện các công việc sau:

1. **Phân tích Code:** Chỉ ra các Code Smell, Technical Debt hoặc điểm vi phạm nguyên lý SOLID trong đoạn code cũ của tôi.
2. **Thực hiện Refactor mã nguồn theo chuẩn Clean Code:**
   - Tách logic Validation ra khỏi Service chính (Sử dụng Spring Validation `@Valid` và Custom Validation nếu cần).
   - Xây dựng cơ chế Global Exception Handler sử dụng `@RestControllerAdvice` để gom toàn bộ các lỗi (Validation, Business logic) về chung một cấu trúc JSON trả về duy nhất (`ErrorResponse`).
   - Thêm cơ chế Logging bằng SLF4J (`@Slf4j`) tại các điểm quan trọng: Khi bắt đầu nhận request, khi xảy ra ngoại lệ, và khi xử lý thành công để phục vụ truy vết lỗi (Traceability).
3. **Lập bảng báo cáo tóm tắt:** So sánh chi tiết cấu trúc "Trước khi Refactor" và "Sau khi Refactor" để thấy rõ những cải tiến kỹ thuật.
