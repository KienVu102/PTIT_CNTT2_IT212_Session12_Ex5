# 🚀 FILE 03: Mã nguồn sau khi Refactor

> *File này chứa toàn bộ cấu trúc mã nguồn mới đã tách lớp, cấu hình Global Exception, DTO lỗi và Logging chỉn chu.*

---

## 📋 Tổng quan

Sau khi được review, hệ thống được cấu trúc lại thành các lớp chuyên biệt để đảm bảo tính **mở rộng** và **dễ bảo trì**.

| # | Thành phần | Mô tả |
|---|-----------|-------|
| 1 | `DuplicateResourceException.java` | Custom Exception cho lỗi trùng lặp dữ liệu |
| 2 | `ErrorResponse.java` | DTO chuẩn hóa cấu trúc phản hồi lỗi |
| 3 | `AccountRegisterRequest.java` | Request DTO tích hợp Validation |
| 4 | `EkycServiceImpl.java` | Tầng Service tinh gọn & có Logging |
| 5 | `GlobalExceptionHandler.java` | Bộ điều hướng ngoại lệ tập trung |

---

## 1. 🔴 Custom Exception — `DuplicateResourceException.java`

> Khai báo ngoại lệ tùy chỉnh để xử lý trường hợp trùng lặp tài nguyên trong hệ thống.

```java
package com.abcbank.ekyc.exception;

public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) {
        super(message);
    }
}
```

---

## 2. 📦 DTO Lỗi Chuẩn Hóa — `ErrorResponse.java`

> Định dạng cấu trúc phản hồi lỗi thống nhất, bao gồm trạng thái HTTP, thông điệp và danh sách chi tiết lỗi.

```java
package com.abcbank.ekyc.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;
import java.util.Map;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String error;
    private String message;
    private LocalDateTime timestamp;
    private Map<String, String> details; // Chứa danh sách các trường bị lỗi Validation độc lập
}
```

---

## 3. ✅ Request DTO với Validation — `AccountRegisterRequest.java`

> Tách logic Validation ra khỏi Service, đưa toàn bộ ràng buộc dữ liệu vào lớp DTO đầu vào.

```java
package com.abcbank.ekyc.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Pattern;
import lombok.Data;

@Data
public class AccountRegisterRequest {

    @NotBlank(message = "Họ và tên không được để trống")
    private String fullName;

    @NotBlank(message = "Số điện thoại không được để trống")
    @Pattern(regexp = "^0[3|5|7|8|9][0-9]{8}$", message = "Số điện thoại không đúng định dạng nhà mạng Việt Nam")
    private String phone;

    @NotBlank(message = "Email không được để trống")
    @Email(message = "Email không đúng định dạng cấu trúc quốc tế")
    private String email;

    @NotBlank(message = "Số CCCD không được để trống")
    @Pattern(regexp = "^[0-9]{12}$", message = "Số CCCD phải bao gồm chính xác 12 ký tự số")
    private String citizenId;
}
```

---

## 4. ⚙️ Tầng Service tinh gọn & Logging — `EkycServiceImpl.java`

> Tầng xử lý nghiệp vụ sử dụng `@Slf4j` để ghi log đầy đủ ở từng bước của tiến trình đăng ký.

```java
package com.abcbank.ekyc.service.impl;

import com.abcbank.ekyc.dto.AccountRegisterRequest;
import com.abcbank.ekyc.dto.AccountRegisterResponse;
import com.abcbank.ekyc.entity.AccountEntity;
import com.abcbank.ekyc.exception.DuplicateResourceException;
import com.abcbank.ekyc.repository.AccountRepository;
import com.abcbank.ekyc.service.EkycService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.UUID;

@Service
@Slf4j // Tích hợp SLF4J Logging thông qua Lombok
public class EkycServiceImpl implements EkycService {

    @Autowired
    private AccountRepository accountRepository;

    @Override
    public AccountRegisterResponse registerEkycAccount(AccountRegisterRequest request) {
        // [LOG] Bắt đầu tiếp nhận tiến trình nghiệp vụ
        log.info("Starting eKYC registration process for customer: {}, Phone: {}",
                request.getFullName(), request.getPhone());

        // Kiểm tra trùng lặp thông tin từ Database
        boolean isExist = accountRepository.existsByCitizenIdOrPhone(
                request.getCitizenId(), request.getPhone());

        if (isExist) {
            log.error("Registration failed. CitizenId [{}] or Phone [{}] already exists.",
                    request.getCitizenId(), request.getPhone());
            throw new DuplicateResourceException(
                    "LỖI NGHIỆP VỤ: Số CCCD hoặc Số điện thoại đã được đăng ký trên hệ thống!");
        }

        // Bản đồ hóa dữ liệu từ Request DTO sang DB Entity
        AccountEntity entity = AccountEntity.builder()
                .fullName(request.getFullName())
                .phone(request.getPhone())
                .email(request.getEmail())
                .citizenId(request.getCitizenId())
                .accountNumber("102" + String.format("%07d",
                        Math.abs(UUID.randomUUID().hashCode() % 10000000)))
                .status("ACTIVE")
                .build();

        AccountEntity saved = accountRepository.save(entity);
        log.info("Successfully created bank account [{}] for customer ID: {}",
                saved.getAccountNumber(), saved.getId());

        return AccountRegisterResponse.builder()
                .accountId(saved.getId())
                .accountNumber(saved.getAccountNumber())
                .status(saved.getStatus())
                .build();
    }
}
```

---

## 5. 🛡️ Bộ điều hướng ngoại lệ tập trung — `GlobalExceptionHandler.java`

> Sử dụng `@RestControllerAdvice` để bắt và xử lý tập trung **mọi** ngoại lệ trong toàn bộ ứng dụng.

```java
package com.abcbank.ekyc.exception;

import com.abcbank.ekyc.dto.ErrorResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    /**
     * 1. Xử lý lỗi Validation dữ liệu đầu vào
     * Ví dụ: Trống tên, Sai định dạng số điện thoại
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });

        log.warn("Input validation failed: {}", errors);

        ErrorResponse errorResponse = ErrorResponse.builder()
                .status(HttpStatus.BAD_REQUEST.value())
                .error("Validation Error")
                .message("Dữ liệu đầu vào không hợp lệ")
                .timestamp(LocalDateTime.now())
                .details(errors)
                .build();

        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }

    /**
     * 2. Xử lý lỗi Nghiệp vụ
     * Ví dụ: Trùng lặp căn cước công dân hoặc số điện thoại
     */
    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ErrorResponse> handleDuplicateResourceException(DuplicateResourceException ex) {
        log.error("Business logic error occurred: {}", ex.getMessage());

        ErrorResponse errorResponse = ErrorResponse.builder()
                .status(HttpStatus.CONFLICT.value()) // HTTP 409 Conflict
                .error("Conflict Data Error")
                .message(ex.getMessage())
                .timestamp(LocalDateTime.now())
                .build();

        return new ResponseEntity<>(errorResponse, HttpStatus.CONFLICT);
    }
}
```

---

> 💡 **Ghi chú:** Toàn bộ mã nguồn trên tuân theo nguyên tắc **Clean Code** — mỗi lớp chỉ đảm nhận một trách nhiệm duy nhất (Single Responsibility Principle).
