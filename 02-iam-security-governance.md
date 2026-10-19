# 02. AWS IAM, Security & Multi-Account Governance

## 1. Quản lý tài khoản Root (AWS Root User)
- Sinh ra cùng email tạo tài khoản, nắm toàn quyền tối cao.
- **Tác vụ duy nhất chỉ Root làm được:** Đóng account, đổi email/thông tin thanh toán, đổi Support Plan, gỡ policy khóa nhầm.
- **Best Practices:** Xóa toàn bộ Access Key/Secret Key; Bật MFA phần cứng/ảo; Khóa mật khẩu và không dùng cho công việc thường ngày.

## 2. Bốn thực thể IAM cốt lõi
- **IAM User:** Đại diện cho 1 người dùng hoặc ứng dụng tương tác lâu dài.
- **IAM Group:** Tập hợp các User. Không thể lồng group vào group; không thể dùng làm Principal thực thi.
- **IAM Role:** Cấp quyền tạm thời (**Temporary Credentials**) qua dịch vụ **AWS STS**. Không có password/static key.
  - *EC2 Instance Profile:* Vỏ bọc gán Role vào EC2, tự xoay vòng token qua metadata `169.254.169.254`.
  - *Cross-Account:* Account A gọi `sts:AssumeRole` sang Role của Account B (Role có Trust Policy cho phép Account A).
  - *External ID:* Điều kiện `sts:ExternalId` trong Trust Policy chống lỗ hổng Confused Deputy khi thuê SaaS bên thứ 3.
- **IAM Policy (JSON):**
  - Cấu trúc: `Version`, `Statement` (`Effect`: Allow/Deny, `Action`, `Resource`, `Condition`).
  - Đánh giá: **Explicit Deny > Explicit Allow > Implicit Deny (Mặc định chặn)**.

## 3. AWS Organizations & Service Control Policies (SCP)
- SCP là rào chắn trần (Guardrails), chỉ quy định quyền tối đa được phép, không trực tiếp cấp quyền.
- Áp dụng từ Management Account xuống các OU và Member Account (kể cả Root của Member Account cũng bị khống chế).
- **Ngoại lệ:** SCP không có tác dụng lên chính Management Account.

## 4. Bộ ba phản ứng sự cố gần thời gian thực (Near Real-time)
- **AWS CloudTrail:** Ghi log API (Ai, làm gì, khi nào, từ IP nào).
- **Amazon EventBridge:** Lọc sự kiện JSON (Event Pattern).
- **Action Target:** Kích hoạt **Amazon SNS** (bắn email cảnh báo) hoặc **AWS Lambda** (tự động rollback, đóng cổng vi phạm).
