# 01. AWS Global Infrastructure & Shared Responsibility Model

## 1. Hạ tầng toàn cầu (AWS Global Infrastructure)
- **AWS Region (Khu vực địa lý):**
  - Vị trí địa lý độc lập hoàn toàn trên thế giới (vd: `ap-southeast-1` tại Singapore).
  - Mỗi Region chứa tối thiểu **3 Availability Zones (AZs)**.
  - Tiêu chí chọn Region: Pháp lý/Chủ quyền dữ liệu (Data Sovereignty) > Độ trễ (Latency) > Chi phí (Pricing) > Dịch vụ sẵn có.
- **Availability Zone - AZ (Vùng sẵn sàng):**
  - Gồm một hoặc nhiều Data Center tách biệt về điện, mạng và làm mát.
  - Kết nối với nhau qua cáp quang băng thông cao, độ trễ cực thấp (< 10ms).
  - Tên AZ (`us-east-1a`) có thể khác nhau giữa các account; dùng `AZ ID` (vd: `use1-az1`) để xác định đúng cụm vật lý.
- **Edge Location / Point of Presence (PoP):**
  - Trạm trung gian lưu bộ nhớ đệm (caching) toàn cầu cho **Amazon CloudFront** và **AWS Global Accelerator**. Không dùng để chạy máy chủ tính toán.

## 2. Mô hình trách nhiệm chung (Shared Responsibility Model)
- **Security OF the Cloud (AWS chịu trách nhiệm):**
  - Bảo vệ hạ tầng vật lý, data center, máy chủ, mạng truyền dẫn, lớp ảo hóa (hypervisor).
  - Vá lỗi hệ điều hành và phần mềm cho các dịch vụ Managed (RDS, DynamoDB, S3).
- **Security IN the Cloud (Khách hàng chịu trách nhiệm):**
  - Dữ liệu khách hàng, cấu hình mã hóa (at-rest, in-transit).
  - Quản lý danh tính, phân quyền IAM, tài khoản Root, MFA.
  - Hệ điều hành máy chủ EC2, vá lỗi ứng dụng, cấu hình tường lửa (Security Group, NACL).

## 3. Tín hiệu nhận diện đề thi SAA
- `High Availability (HA) / Fault Tolerance`: Triển khai phân tán trên **>= 2 AZs (Multi-AZ)**.
- `Disaster Recovery (DR) / Regional Outage`: Triển khai đa vùng **Multi-Region**.
- `Low latency globally`: **Amazon CloudFront**.
