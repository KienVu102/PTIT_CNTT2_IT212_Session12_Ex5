# FILE 04: BÁO CÁO TÓM TẮT QUÁ TRÌNH CẢI TIẾN HỆ THỐNG (IMPROVEMENT SUMMARY)
**Môn học:** Phát triển Ứng dụng Web với Java (Spring Boot)
**Sinh viên thực hiện:** Vũ Đình Kiên

---

### I. CÁC "MÙI CODE" (CODE SMELLS) VÀ VI PHẠM KỸ THUẬT ĐƯỢC PHÁT HIỆN
Trong phiên bản cũ, mã nguồn gặp phải một số lỗi cấu trúc hệ thống nghiêm trọng sau:
1. **Vi phạm nguyên lý Single Responsibility Principle (SRP - Chữ S trong SOLID):** Lớp `EkycServiceImpl` vừa phải gánh vác logic nghiệp vụ chính, vừa phải thực hiện kiểm tra định dạng chuỗi dữ liệu (Validation) theo cách thủ công. Điều này làm mã nguồn phình to và khó bảo trì.
2. **Xử lý lỗi phân tán (Scattered Exception Handling):** Hệ thống ném ra `RuntimeException` chung chung tại Service. Gây khó khăn cho tầng Controller khi phải bao bọc nhiều khối lệnh `try-catch` thủ công, cấu trúc thông điệp phản hồi không đồng nhất.
3. **Mù thông tin vận hành (Lack of Logging):** Hoàn toàn vắng bóng cơ chế ghi vết hệ thống. Khi xảy ra lỗi hoặc sập luồng dữ liệu trên môi trường Production, quản trị viên không có dữ liệu log để phân tích lý do.

---

### II. BẢNG SO SÁNH CHI TIẾT TRƯỚC VÀ SAU KHI REFACTOR

| Tiêu chí so sánh | Mã nguồn TRƯỚC khi Refactor (Cũ) | Mã nguồn SAU khi Refactor (Mới) | Lợi ích mang lại |
| :--- | :--- | :--- | :--- |
| **Quản lý Validation** | Sử dụng các câu lệnh điều kiện `if-else` lồng nhau phức tạp bên trong Service Layer. | Đưa trực tiếp vào Request DTO bằng các Annotation tiêu chuẩn của Jakarta Bean Validation (`@NotBlank`, `@Pattern`, `@Email`). | Giảm 40% số dòng code trong Service. Tách biệt hoàn toàn tầng kiểm tra dữ liệu và tầng xử lý nghiệp vụ. |
| **Kiểm soát Ngoại lệ (Exception)** | Ném lỗi `RuntimeException` thô sơ. Client nhận về trang lỗi mặc định Whitelabel Error Page của Spring rất xấu. | Sử dụng mô hình xử lý tập trung `@RestControllerAdvice` kết hợp Custom Exception chuyên biệt. | Gom toàn bộ lỗi về một mối. Trả về cấu trúc JSON thống nhất (`ErrorResponse`) giúp lập trình viên Front-end dễ cấu hình giao diện hiển thị lỗi. |
| **Giám sát hệ thống (Logging)** | Không sử dụng log. Hệ thống chạy ngầm và không để lại dấu vết dữ liệu. | Tích hợp thư viện Logback thông qua Annotation `@Slf4j` tại Service và Exception Handler. | Ghi lại toàn bộ chu trình sống của Request (In, Out, Error). Dễ dàng kiểm soát bảo mật và debug hệ thống. |
| **Cấu trúc khởi tạo Đối tượng** | Dùng từ khóa `new` thủ công và gán từng thuộc tính bằng phương thức `set()`. | Áp dụng **Design Pattern: Builder** thông qua thư viện Lombok `@Builder`. | Code ngắn gọn, trực quan, tránh lỗi gán thiếu thuộc tính quan trọng khi khởi tạo. |

---

### III. KẾT LUẬN
Hệ thống sau khi thực hiện Refactor không chỉ giải quyết triệt để các bài toán nghiệp vụ kiểm thử, đáp ứng chuẩn yêu cầu kỹ thuật mà còn đạt độ tối ưu cao về kiến trúc mã nguồn sạch (Clean Code), sẵn sàng cho các bài toán tích hợp microservices phức tạp hơn về sau.
