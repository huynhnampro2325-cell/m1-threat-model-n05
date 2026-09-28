# MỐC M1: MÔ HÌNH ĐE DỌA (THREAT MODEL)

**Đề tài:** Chủ đề B - Nền tảng định danh tập trung cho ba dịch vụ
**Nhóm:** N05
**Ngày cập nhật:** 28/09/2026

---

## 1. Tổng quan hệ thống

- **Identity Provider (IdP):** Keycloak đóng vai trò trung tâm, cấp phát mã thông báo OIDC (OIDC Tokens) cho 3 ứng dụng (Relying Parties).
- **Ứng dụng:** 3 dịch vụ độc lập tiếp nhận token từ Keycloak để xác thực người dùng.
- **Phân quyền:** Hệ thống có 2 vai trò (user, admin). Vai trò `admin` bắt buộc xác thực hai yếu tố (2FA) bằng TOTP.

## 2. Tài sản cần bảo vệ

- Mã thông báo OIDC: Access Token, ID Token, Refresh Token.
- Thông tin xác thực người dùng: Credentials, mã TOTP Seed.
- Cấu hình hệ thống Keycloak: Realm, Client, Client Secrets.

## 3. Mô hình đe dọa và Biện pháp giảm thiểu

| # | Mối đe dọa | Kịch bản tấn công | Biện pháp giảm thiểu |
|---|---|---|---|
| 1 | Chiếm đoạt phiên (Session Hijacking) | Kẻ tấn công đánh cắp Access/Refresh Token qua XSS, malware hoặc sniffing mạng. | Cấu hình token lifespan ngắn, bắt buộc HTTPS/TLS, SameSite cookies nghiêm ngặt. |
| 2 | Mã thông báo sống quá lâu (Long-lived Tokens) | Refresh token có thời hạn quá dài, nếu bị lộ sẽ cho phép kẻ tấn công duy trì truy cập lâu dài. | Cấu hình Refresh Token Max Reuse = 0, thiết lập SSO Session Idle/Max timeout hợp lý, bật tính năng thu hồi token (Revocation). |
| 3 | Máy chủ tài nguyên không kiểm tra đối tượng nhận (Audience Mismatch) | Kẻ tấn công lấy token hợp lệ của Ứng dụng A, gửi đến Ứng dụng B. Ứng dụng B không kiểm tra trường `aud` trong token nên chấp nhận. | Cấu hình bắt buộc kiểm tra trường `aud` (Audience) tại tất cả các Resource Server. Cấu hình Token Exchange hoặc Audience mapper chính xác trên Keycloak. |
| 4 | Lạm dụng quy trình khôi phục mật khẩu (Password Reset Abuse) | Kẻ tấn công lợi dụng chức năng "Quên mật khẩu" để spam email, dò tìm tài khoản tồn tại (User Enumeration) hoặc brute-force mã OTP. | Vô hiệu hóa thông báo lỗi tiết lộ tài khoản tồn tại. Áp dụng Rate Limiting cho API reset password. Yêu cầu TOTP khi thực hiện reset mật khẩu cho admin. |

## 4. Bằng chứng dự kiến sẽ nộp (M2 - M4)

- [ ] Tệp cấu hình Keycloak (đã lọc bỏ client secrets).
- [ ] Ca kiểm thử (Test case) từ chối token có thuật toán `none` (Thuật toán rỗng).
- [ ] Ca kiểm thử từ chối token sai đối tượng nhận (`aud`).
- [ ] Bảng theo dõi vòng đời mã thông báo (Token Lifespan Table).

---

*Ghi chú: Mô hình đe dọa này là cơ sở để nhóm triển khai cấu hình thực tế trên Keycloak ở Mốc M2.*
