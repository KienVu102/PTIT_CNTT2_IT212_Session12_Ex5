# FILE 02: MÃ NGUỒN TRƯỚC KHI REFACTOR (MÃ NGUỒN CŨ)

### Lớp EkycServiceImpl.java (Chưa tối ưu)
```java
package com.abcbank.ekyc.service.impl;

import com.abcbank.ekyc.dto.AccountRegisterRequest;
import com.abcbank.ekyc.dto.AccountRegisterResponse;
import com.abcbank.ekyc.entity.AccountEntity;
import com.abcbank.ekyc.repository.AccountRepository;
import com.abcbank.ekyc.service.EkycService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class EkycServiceImpl implements EkycService {

    @Autowired
    private AccountRepository accountRepository;

    @Override
    public AccountRegisterResponse registerEkycAccount(AccountRegisterRequest request) {
        // --- CODE SMELL 1: VALIDATION THỦ CÔNG (Vi phạm Single Responsibility Principle - SRP) ---
        if (request.getFullName() == null || request.getFullName().trim().isEmpty()) {
            throw new RuntimeException("Họ và tên không được để trống");
        }
        if (request.getPhone() == null || !request.getPhone().matches("^0[0-9]{9}$")) {
            throw new RuntimeException("Số điện thoại không đúng định dạng Việt Nam");
        }
        if (request.getCitizenId() == null || request.getCitizenId().length() != 12) {
            throw new RuntimeException("Số CCCD phải bao gồm chính xác 12 ký tự số");
        }

        // --- CODE SMELL 2: THIẾU LOGGING ĐỂ TRUY VẾT ---
        // Không có dòng log nào ghi nhận hệ thống đang xử lý cho ai, khi lỗi không biết đường vết.

        // Kiểm tra trùng lặp
        boolean isExist = accountRepository.existsByCitizenIdOrPhone(request.getCitizenId(), request.getPhone());
        if (isExist) {
            // Tự ném RuntimeException chung chung, Controller phải dùng try-catch thủ công
            throw new RuntimeException("LỖI NGHIỆP VỤ: Số CCCD hoặc Số điện thoại đã được đăng ký trên hệ thống!");
        }

        // Tạo mới tài khoản
        AccountEntity entity = new AccountEntity();
        entity.setFullName(request.getFullName());
        entity.setPhone(request.getPhone());
        entity.setEmail(request.getEmail());
        entity.setCitizenId(request.getCitizenId());
        entity.setAccountNumber("999" + System.currentTimeMillis() % 10000000);
        entity.setStatus("ACTIVE");

        AccountEntity saved = accountRepository.save(entity);

        AccountResponse response = new AccountResponse();
        response.setAccountId(saved.getId());
        response.setAccountNumber(saved.getAccountNumber());
        response.setStatus(saved.getStatus());

        return response;
    }
}
