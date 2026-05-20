# FIT4012 Lab 7 - Báo cáo 1 trang: SHA-256

**Họ và tên:** Đỗ Đăng Thành  
**MSSV:** FIT4016-18-01  
**Lớp:** FIT4012

## 1. Mục tiêu / Objective

Bài thực hành nhằm phân tích và triển khai thuật toán SHA-256, bao gồm các mục tiêu cụ thể:
- Hiểu và triển khai thuật toán băm SHA-256 từ đầu
- Chạy chương trình băm chuỗi và kiểm tra với known answer test vectors
- Kiểm tra toàn vẹn file bằng SHA-256
- Triển khai băm mật khẩu và nghiên cứu vấn đề bảo mật
- Cải tiến bằng salt để tránh việc hai người có cùng mật khẩu tạo ra cùng hash

## 2. Cách làm / Approach

Nhóm đã thực hiện các bước sau:

- **Biên dịch và chạy `sha_procedure.cpp`**: Chương trình cung cấp chế độ `--self-test` để kiểm tra với các test vector đã biết, `--hash-string` để băm chuỗi từ tham số dòng lệnh, và `--hash-file` để băm nội dung file.

- **Kiểm thử SHA-256 bằng known answer test vector**: Sử dụng các test vector chuẩn từ FIPS 180-4 cho chuỗi rỗng "", "abc", và "hello FIT4012 SHA" để xác minh tính đúng đắn của implementation.

- **Viết/chạy chương trình kiểm tra toàn vẹn file (`file_integrity.cpp`)**: Chương trình cho phép tính hash của file và so sánh với hash mong đợi, phát hiện file bị thay đổi.

- **Viết/chạy chương trình băm mật khẩu (`password_hash.cpp`)**: Chế độ `register` để lưu hash mật khẩu, chế độ `login` để kiểm tra mật khẩu.

- **Bổ sung salt (`salted_password_hash.cpp`)**: Tạo salt ngẫu nhiên 16 bytes (32 ký tự hex), lưu theo định dạng `salt:hash` để cùng một mật khẩu nhưng mỗi lần đăng ký tạo ra hash khác nhau.

## 3. Kết quả / Result

### 3.1. Hash của chuỗi `abc`:
```
ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
```

### 3.2. Hash của file mẫu trước khi sửa:
```
5ee62dc925a9958dbd6732c570a23c7f65a8c11066e889b15068cfb4bf1a0bd9
```

### 3.3. Kết quả kiểm tra file sau khi sửa nội dung:
```
[FAIL] File was changed or expected hash is incorrect
expected: 5ee62dc925a9958dbd6732c570a23c7f65a8c11066e889b15068cfb4bf1a0bd9
actual  : c232e5627e703ee5b311e0df8520b3d10dc8867b27636bb58fe58eb1fb9d6acb
```
→ **Phát hiện thành công file bị tamper.**

### 3.4. Kết quả đăng nhập với mật khẩu đúng:
```
[PASS] Login success
```

### 3.5. Kết quả đăng nhập với mật khẩu sai:
```
[FAIL] Login failed: wrong password
```

### 3.6. Hai bản ghi `salt:hash` của cùng một mật khẩu có giống nhau không?
**KHÔNG giống nhau.** Mỗi lần register tạo ra salt ngẫu nhiên khác nhau, dẫn đến hash khác nhau hoàn toàn:
```
test_password_salted_1.hash: (salt1:hash1)
test_password_salted_2.hash: (salt2:hash2)
```
→ **Cùng mật khẩu "same-password" nhưng hai file hash khác nhau do salt khác nhau.**

### 3.7. Kết quả test suite:
```
[PASS] SHA programs compile successfully.
[PASS] SHA-256 known answer tests passed.
[PASS] Tamper / flip 1 byte negative test passed.
[PASS] Password hash and wrong password negative test passed.
[PASS] Salted password test passed.
```

## 4. Kết luận / Conclusion

### 4.1. SHA-256 giúp phát hiện thay đổi dữ liệu như thế nào?
SHA-256 tạo ra một "dấu vân tay" số 256-bit cho dữ liệu. Chỉ cần thay đổi 1 bit trong dữ liệu gốc, hash kết quả sẽ thay đổi hoàn toàn (hiệu ứng avalanche). Điều này cho phép phát hiện mọi thay đổi dữ liệu bằng cách so sánh hash hiện tại với hash ban đầu.

### 4.2. Vì sao cần salt khi lưu hash mật khẩu?
Salt ngăn chặn các cuộc tấn công bằng rainbow table (bảng tra trước các hash phổ biến). Khi mỗi người dùng có một salt khác nhau:
- Cùng mật khẩu nhưng tạo ra hash khác nhau
- Kẻ tấn công không thể dùng chung một rainbow table cho nhiều user
- Buộc kẻ tấn công phải brute-force từng tài khoản riêng lẻ

### 4.3. Vì sao SHA-256 demo trong lab chưa nên dùng trực tiếp cho hệ thống xác thực thật?
- **Tốc độ quá nhanh**: SHA-256 được thiết kế để tính nhanh, nhưng mật khẩu cần hàm băm chậm để chống brute-force
- **Không có cơ chế làm chậm**: Không như bcrypt, scrypt, Argon2id có thể điều chỉnh chi phí tính toán
- **Dễ bị tấn công bằng GPU/ASIC**: SHA-256 có thể tính hàng tỷ lần/giây trên phần cứng chuyên dụng
- **Khuyến nghị**: Sử dụng Argon2id (winner của Password Hashing Competition), bcrypt, hoặc scrypt cho hệ thống thật

---

**Ghi chú**: Code và minh chứng chi tiết được lưu trong thư mục `logs/my-run.log`.