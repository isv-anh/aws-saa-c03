# 05. VPC Connectivity & Endpoints

## 1. VPC Peering vs AWS Transit Gateway

| Tiêu chí | VPC Peering | AWS Transit Gateway |
|---|---|---|
| **Mô hình** | Điểm - Điểm (Point-to-Point) | Trục trung tâm (Hub-and-Spoke) |
| **Bắc cầu (Transitive)** | **Không hỗ trợ** (Cấm bắc cầu A-B-C) | **Hỗ trợ định tuyến bắc cầu** |
| **Quy mô** | Thích hợp cho 2 - 5 VPC | Hàng trăm/nghìn VPC, đa account |
| **Kết nối On-prem** | Không tập trung | Tích hợp VPN và Direct Connect Gateway |
| **Chi phí** | Miễn phí tạo kết nối | Tính phí theo giờ attachment + data processed |

- **Điều kiện VPC Peering:** Không trùng dải IP (No overlapping CIDRs); kết nối 1-1 không bắc cầu.

## 2. VPC Endpoints (Kết nối dịch vụ không qua Internet)
- Giúp máy chủ Private Subnet gọi dịch vụ AWS qua mạng riêng, không tốn phí NAT Gateway, an toàn bảo mật.

| Đặc tính | Gateway Endpoint | Interface Endpoint (AWS PrivateLink) |
|---|---|---|
| **Dịch vụ hỗ trợ** | **Chỉ 2 dịch vụ: S3 và DynamoDB** | Hầu hết dịch vụ còn lại (Bedrock, SQS...) + S3 |
| **Triển khai** | Cấu hình Target trong Route Table | Cắm card mạng ảo (ENI) có Private IP vào Subnet |
| **Chi phí** | **HOÀN TOÀN MIỄN PHÍ ($0)** | **Có tính phí theo giờ + $/GB dữ liệu** |
| **Truy cập On-prem** | Không hỗ trợ qua VPN/Direct Connect | Hỗ trợ truy cập từ On-prem qua Private IP |

- **Tín hiệu đề thi:**
  - `Access S3 / DynamoDB from private subnet with LOWEST cost` -> **Gateway VPC Endpoint**.
  - `Access S3 from On-premises over Direct Connect` -> **Interface VPC Endpoint for S3**.
