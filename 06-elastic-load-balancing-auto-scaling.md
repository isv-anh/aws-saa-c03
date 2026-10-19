# 06. Elastic Load Balancing (ELB) & Auto Scaling Group (ASG)

## 1. Phân loại Load Balancer
- **Application Load Balancer - ALB (Layer 7):**
  - Đọc nội dung gói tin HTTP/HTTPS, định tuyến theo URL path (`/api`), hostname, headers.
  - Không có IP cố định (chỉ cung cấp DNS Name tĩnh).
- **Network Load Balancer - NLB (Layer 4):**
  - Truyền dẫn TCP/UDP/TLS, độ trễ cực thấp (micro-seconds), xử lý hàng triệu request/giây.
  - **Hỗ trợ gán Static IP / Elastic IP** cho từng AZ (phục vụ khách hàng cần IP whitelist trên firewall).
- **Gateway Load Balancer - GWLB (Layer 3/4 Gateway + LB):**
  - Phân tải cho thiết bị mạng bên thứ ba (**third-party virtual appliances: Firewall Palo Alto/Fortinet, IDS/IPS**).
  - Dùng giao thức **GENEVE (cổng 6081)** đóng gói gói tin, giữ nguyên IP nguồn/đích trong suốt.
  - Loại bỏ lỗi lệch luồng định tuyến (Asymmetric Routing) bằng cơ chế Flow Stickiness.

## 2. Auto Scaling Group (ASG)
- **3 thông số quy mô:** Minimum Size (Tối thiểu) <= Desired Capacity (Mong muốn) <= Maximum Size (Tối đa).
- **Các chính sách co giãn (Scaling Policies):**
  - *Target Tracking:* Đặt mục tiêu cố định (vd: duy trì bình quân CPU ở mức 50%), ASG tự tính toán thêm/bớt máy.
  - *Step / Simple Scaling:* Tăng giảm theo ngưỡng CloudWatch Alarm định sẵn.
  - *Scheduled Scaling:* Bật tắt máy đón đầu theo khung giờ cố định biết trước.
  - *Predictive Scaling:* Dùng Machine Learning dự đoán lưu lượng tải trước 24h.
- **Mixed Instances Policy:**
  - Chạy công suất nền (Baseline 24/7) bằng On-Demand (kết hợp Savings Plans/RI).
  - Chạy công suất đỉnh biến thiên bằng Spot Instances (giảm giá 90%).
- **Health Check Replacement:**
  - Mặc định ASG dùng `EC2 Health Check` (chỉ phát hiện lỗi phần cứng, ứng dụng lỗi HTTP 500 máy vẫn báo xanh).
  - Muốn ASG tự xóa máy và bật máy mới khi web server bị treo/lỗi 500: **Đổi Health Check Type của ASG sang `ELB`**.
