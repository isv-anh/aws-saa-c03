# 03. Amazon EC2, Pricing Models & Storage Comparison

## 1. Phân loại dòng máy EC2 (Instance Types)
- **T / M (General Purpose):** Cân bằng CPU, RAM, Network; dòng T có CPU burstable credits.
- **C (Compute Optimized):** Xử lý tính toán cao, batch jobs, video encoding, game server.
- **R / X (Memory Optimized):** Cơ sở dữ liệu in-memory (Redis/Memcached), xử lý dữ liệu lớn (Big Data).
- **I / D (Storage Optimized):** Đọc ghi đĩa tuần tự/ngẫu nhiên cực nhanh, NoSQL DB cục bộ.
- **P / G (Accelerated Computing):** GPU, huấn luyện Machine Learning/Deep Learning, đồ họa 3D.

## 2. Mô hình tính giá EC2 (Pricing Models)
- **On-Demand:** Trả theo giây/phút, linh hoạt tối đa, chi phí cao nhất. Dùng cho việc ngắn hạn, chưa rõ tải.
- **Spot Instances:** Tận dụng công suất dư thừa, rẻ đến 90%. AWS có thể thu hồi bất cứ lúc nào với cảnh báo trước 2 phút. Dùng cho batch processing, stateless, chấp nhận gián đoạn.
- **Savings Plans / Reserved Instances (RI):** Cam kết 1 hoặc 3 năm, giảm giá đến 72%. Dùng cho tải nền ổn định 24/7 (Steady-state).
- **Dedicated Hosts:** Thuê máy chủ vật lý riêng, kiểm soát socket/core. Dùng cho bài toán mang bản quyền phần mềm (BYOL) hoặc tuân thủ pháp lý.

## 3. So sánh 3 dịch vụ lưu trữ cốt lõi: EBS vs EFS vs S3

| Tiêu chí | Amazon EBS | Amazon EFS | Amazon S3 |
|---|---|---|---|
| **Mô hình** | Block Storage | File Storage (NFSv4) | Object Storage (REST API) |
| **Phạm vi** | Gói gọn trong 1 AZ | Đa vùng (Regional - đa AZ) | Đa vùng (Regional - đa AZ) |
| **Gắn kết** | 1 máy EC2 *(trừ io1/io2 Multi-Attach trong 1 AZ)* | Hàng trăm EC2/Container đọc ghi đồng thời | Không gắn; gọi qua HTTP/HTTPS GET/PUT |
| **Hệ điều hành** | Windows & Linux | **Chỉ Linux** *(Windows dùng FSx)* | Độc lập nền tảng |
| **Độ trễ** | Dưới 1ms (io2) đến vài ms (gp3) | Vài mili-giây (single-digit ms) | 100 - 200ms |
| **Độ bền** | Cấp ổ đĩa (Snapshot lưu về S3) | Tự nhân bản đa AZ | 99.999999999% (11 số 9) |

- **Phân loại ổ EBS:**
  - `gp3`: SSD mặc định cân bằng chi phí và hiệu năng.
  - `io2`: SSD hiệu năng cao nhất (> 64,000 IOPS, sub-millisecond) cho DB quan trọng.
  - `st1` (HDD tuần tự/Big Data) & `sc1` (Cold HDD): **Không thể dùng làm Boot Volume**.
- **EC2 Instance Store:** Ổ đĩa NVMe gắn vật lý trực tiếp, tốc độ tối đa nhưng là bộ nhớ tạm thời (**Ephemeral** - mất sạch dữ liệu khi Stop máy).
