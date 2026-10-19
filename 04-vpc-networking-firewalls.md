# 04. Amazon VPC, Subnet Routing & Network Security

## 1. Nền tảng VPC & Dải địa chỉ CIDR
- Kích thước VPC cho phép từ `/16` (65,536 IP) đến `/28` (16 IP).
- **Quy tắc 5 địa chỉ IP bị AWS giữ lại trong mọi Subnet:**
  - `.0`: Địa chỉ mạng (Network address).
  - `.1`: Bộ định tuyến ảo VPC (VPC Router).
  - `.2`: Máy chủ DNS (Route 53 Resolver).
  - `.3`: Dành cho tương lai của AWS.
  - `.255`: Địa chỉ Broadcast mạng.
  - *(Subnet `/24` có 256 IP -> Khả dụng thực tế: 251 IP).*

## 2. Public Subnet vs Private Subnet & Cổng kết nối
- **Local Route:** Mọi Route Table đều có sẵn dòng `VPC-CIDR -> local`, cho phép các Subnet trong cùng 1 VPC nói chuyện trực tiếp với nhau.
- **Public Subnet:** Route Table có tuyến `0.0.0.0/0 -> igw-xxxx` (Internet Gateway).
- **Private Subnet:** Tuyến mặc định `0.0.0.0/0 -> nat-xxxx` (NAT Gateway).
- **NAT Gateway:**
  - Cho phép máy ở Private Subnet tải bản vá, gọi API ngoài; chặn chiều ngược lại từ Internet.
  - **Bắt buộc đặt trong Public Subnet và gắn Elastic IP**.
  - Tính sẵn sàng cao (HA): NAT Gateway bị khóa theo 1 AZ. Kiến trúc HA chuẩn bắt buộc phải tạo **1 NAT Gateway trên mỗi AZ**.

## 3. So sánh Security Group và Network ACL (NACL)

| Tiêu chí | Security Group (SG) | Network ACL (NACL) |
|---|---|---|
| **Cấp áp dụng** | Card mạng ảo (ENI / Instance) | Ranh giới Subnet |
| **Trạng thái** | **Stateful** (Cho vào là tự động cho ra) | **Stateless** (Cần mở cả Inbound và Outbound) |
| **Loại rule** | **Chỉ có ALLOW** | **Hỗ trợ cả ALLOW và DENY** |
| **Đánh giá** | Đánh giá toàn bộ rules cùng lúc | Đánh giá tuần tự theo số từ bé đến lớn (Rule 100...) |
| **Cổng tạm thời** | Tự xử lý ngầm | Bắt buộc mở Outbound Ephemeral Ports (1024-65535) |

- **Từ khóa nhận diện:**
  - `Block a single malicious IP`: Dùng **Network ACL** (NACL Inbound Deny).
  - `Allow traffic without managing IPs`: Dùng **Security Group Reference/Chaining** (Source = `sg-xxxx`).
