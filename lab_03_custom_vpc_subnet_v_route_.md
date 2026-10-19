# Lab 03: Xây Dựng Mạng Riêng Ảo (Custom VPC), Phân Vùng Public/Private Subnet & Thiết Lập Route Table

## 1. Mục tiêu bài lab

* Tự tay tạo một Custom VPC hoàn chỉnh với dải địa chỉ CIDR chuẩn doanh nghiệp thay vì sử dụng Default VPC.
* Khởi tạo kiến trúc đa vùng (Multi-AZ) gồm 4 Subnet: 2 Public Subnets và 2 Private Subnets phân bổ trên 2 Availability Zones.
* Tạo và gắn kết Internet Gateway (IGW) vào Custom VPC.
* Phân tách và cấu hình 2 bảng định tuyến (Route Tables) độc lập: **Public Route Table** và **Private Route Table**.
* Hiểu rõ cơ chế liên kết tường minh (Explicit Subnet Association) và quy tắc 5 IP dự phòng của AWS.
* Khởi chạy EC2 kiểm chứng khả năng kết nối mạng thực tế và thực hiện dọn dẹp tài nguyên với chi phí $0 USD.

---

## 2. Thông số thiết lập môi trường

* **AWS Region:** Singapore (`ap-southeast-1`)
* **Tài khoản thực hiện:** IAM Admin User (`cloud-admin`)
* **VPC CIDR:** `10.0.0.0/16` ($65,536$ địa chỉ IP)
* **Bảng phân bổ Subnet:**

| Tên Subnet | Loại Subnet | Availability Zone | Dải CIDR | Số IP khả dụng | Mục đích sử dụng |
|---|---|---|---|---|---|
| `lab04-public-1a` | Public | `ap-southeast-1a` | `10.0.1.0/24` | $251$ | Load Balancer, Bastion Host |
| `lab04-public-1b` | Public | `ap-southeast-1b` | `10.0.2.0/24` | $251$ | Load Balancer (Multi-AZ) |
| `lab04-private-1a` | Private | `ap-southeast-1a` | `10.0.11.0/24` | $251$ | Backend API, EC2 App |
| `lab04-private-1b` | Private | `ap-southeast-1b` | `10.0.12.0/24` | $251$ | Cơ sở dữ liệu RDS / Aurora |

---

## 3. Các bước thực hiện chi tiết

### Bước 1: Khởi tạo Custom VPC độc lập
1. Mở **AWS Management Console** $\rightarrow$ gõ tìm kiếm **VPC** và truy cập vào giao diện quản lý.
2. Tại menu bên trái, chọn **Your VPCs** $\rightarrow$ nhấn nút màu cam **Create VPC**.
3. Tại mục cấu hình:
   * Chọn tùy chọn: **VPC only** *(tự dựng từng thành phần thủ công để hiểu bản chất thay vì dùng wizard tự động)*.
   * **Name tag:** Điền `lab04-custom-vpc`.
   * **IPv4 CIDR block:** Chọn *IPv4 CIDR manual input* $\rightarrow$ nhập chính xác: `10.0.0.0/16`.
   * **IPv6 CIDR block:** Chọn *No IPv6 CIDR block*.
   * **Tenancy:** Chọn *Default*.
4. Nhấn nút **Create VPC**.

> **Kiểm tra thành công:**  
> Trong danh sách *Your VPCs*, xuất hiện `lab04-custom-vpc` với trạng thái **Available**. Ghi lại mã định danh VPC (ví dụ: `vpc-0123456789abcdef0`).

---

### Bước 2: Tạo và gắn kết Internet Gateway (IGW)
Internet Gateway là cổng kết nối hai chiều giữa VPC và Internet công cộng. Một VPC chỉ có thể gắn tối đa duy nhất 1 Internet Gateway.

1. Tại thanh menu bên trái, chọn **Internet gateways** $\rightarrow$ nhấn **Create internet gateway**.
2. Cấu hình:
   * **Name tag:** `lab04-igw`.
3. Nhấn **Create internet gateway**.
4. Màn hình hiển thị trạng thái của `lab04-igw` là **Detached** (chưa gắn vào đâu):
   * Nhấn nút **Actions** ở góc trên bên phải $\rightarrow$ chọn **Attach to VPC**.
   * Tại ô *Available VPCs*, chọn `lab04-custom-vpc`.
   * Nhấn nút màu cam **Attach internet gateway**.

> **Kiểm tra thành công:**  
> Trạng thái (State) của `lab04-igw` chuyển sang màu xanh lá: **Attached**.

---

### Bước 3: Khởi tạo 4 Subnets đa vùng (Multi-AZ)
Mỗi Subnet bị giới hạn vật lý trong đúng 1 Availability Zone. Chúng ta sẽ tạo lần lượt 4 Subnet:

1. Tại menu bên trái, chọn **Subnets** $\rightarrow$ nhấn **Create subnet**.
2. Ô **VPC ID**: Chọn `lab04-custom-vpc`.
3. **Cấu hình Subnet 1 (Public 1a):**
   * **Subnet name:** `lab04-public-1a`
   * **Availability Zone:** Chọn vùng có đuôi `ap-southeast-1a`.
   * **IPv4 subnet CIDR block:** `10.0.1.0/24`.
4. Nhấn nút **Add new subnet** phía dưới để tạo tiếp **Subnet 2 (Public 1b):**
   * **Subnet name:** `lab04-public-1b`
   * **Availability Zone:** Chọn vùng có đuôi `ap-southeast-1b`.
   * **IPv4 subnet CIDR block:** `10.0.2.0/24`.
5. Nhấn nút **Add new subnet** để tạo tiếp **Subnet 3 (Private 1a):**
   * **Subnet name:** `lab04-private-1a`
   * **Availability Zone:** Chọn vùng có đuôi `ap-southeast-1a`.
   * **IPv4 subnet CIDR block:** `10.0.11.0/24`.
6. Nhấn nút **Add new subnet** để tạo tiếp **Subnet 4 (Private 1b):**
   * **Subnet name:** `lab04-private-1b`
   * **Availability Zone:** Chọn vùng có đuôi `ap-southeast-1b`.
   * **IPv4 subnet CIDR block:** `10.0.12.0/24`.
7. Kéo xuống dưới cùng và nhấn **Create subnet**.

> **Kiểm tra thành công & Quan sát bẫy đề thi:**  
> Tại bảng danh sách Subnet, nhìn vào cột **Available IPv4 addresses** của cả 4 Subnet vừa tạo: Bạn sẽ thấy số hiển thị là **251** chứ không phải 256. AWS đã tự động giữ lại 5 IP nội bộ (`.0`, `.1`, `.2`, `.3`, `.255`).

---

### Bước 4: Kích hoạt cấp phát Public IP tự động cho Public Subnets
Mặc định trong Custom VPC, các máy chủ khởi chạy sẽ không tự có địa chỉ Public IPv4. Ta cần kích hoạt cờ tự động cho 2 Public Subnet:

1. Tại danh sách **Subnets**, tích chọn `lab04-public-1a`.
2. Nhấn **Actions** $\rightarrow$ chọn **Edit subnet settings**.
3. Tại phần *Auto-assign IP settings*, tích chọn: **Enable auto-assign public IPv4 address**.
4. Nhấn **Save**.
5. Lặp lại thao tác này cho Subnet `lab04-public-1b`.

*(Lưu ý: Tuyệt đối không bật tính năng này cho 2 Private Subnet).*

---

### Bước 5: Cấu hình bảng định tuyến (Route Tables)

Điều gì biến một Subnet thành "Public" hay "Private"? Đó chính là Route Table mà Subnet đó liên kết tới.

#### 1. Tạo Public Route Table
1. Ở menu bên trái, chọn **Route tables** $\rightarrow$ nhấn **Create route table**.
2. Cấu hình:
   * **Name:** `lab04-public-rt`
   * **VPC:** Chọn `lab04-custom-vpc`
3. Nhấn **Create route table**.
4. Thêm tuyến đường đi ra ngoài Internet:
   * Chuyển sang tab **Routes** $\rightarrow$ nhấn **Edit routes** $\rightarrow$ nhấn **Add route**.
   * **Destination:** Nhập `0.0.0.0/0` (đại diện cho mọi luồng dữ liệu bên ngoài).
   * **Target:** Bấm chọn **Internet Gateway** $\rightarrow$ chọn `lab04-igw`.
   * Nhấn **Save changes**.
5. Gán tường minh 2 Public Subnet vào Route Table này:
   * Chuyển sang tab **Subnet associations** $\rightarrow$ nhấn **Edit subnet associations**.
   * Tích chọn 2 Subnet: `lab04-public-1a` và `lab04-public-1b`.
   * Nhấn **Save associations**.

#### 2. Tạo Private Route Table
1. Tại mục **Route tables**, nhấn **Create route table**.
2. Cấu hình:
   * **Name:** `lab04-private-rt`
   * **VPC:** Chọn `lab04-custom-vpc`
3. Nhấn **Create route table**.
4. Kiểm tra tab **Routes**:
   * Chỉ có duy nhất 1 dòng mặc định: `10.0.0.0/16` $\rightarrow$ `local`. Tuyệt đối **không thêm** đường `0.0.0.0/0 -> igw` vào đây.
5. Gán 2 Private Subnet vào Route Table này:
   * Chuyển sang tab **Subnet associations** $\rightarrow$ nhấn **Edit subnet associations**.
   * Tích chọn 2 Subnet: `lab04-private-1a` và `lab04-private-1b`.
   * Nhấn **Save associations**.

> **Kiểm tra thành công:**  
> Vào chi tiết VPC `lab04-custom-vpc`, chuyển sang tab **Resource map**: Bạn sẽ thấy sơ đồ trực quan gồm VPC phân nhánh ra 4 Subnet; 2 Public Subnet nối vào `lab04-public-rt` và trỏ thẳng tới `lab04-igw`, trong khi 2 Private Subnet nối vào `lab04-private-rt` và dừng lại ở ranh giới mạng nội bộ.

---

### Bước 6: Kiểm chứng thực tế sự khác biệt giữa Public và Private

Để chứng minh kiến trúc hoạt động chính xác theo lý thuyết:

1. **Khởi tạo máy chủ thử nghiệm trên Public Subnet:**
   * Vào **EC2** $\rightarrow$ **Instances** $\rightarrow$ **Launch instances**.
   * Tên: `test-public-ec2` | OS: Amazon Linux 2023 | Type: `t3.micro` hoặc `t2.micro`.
   * Network settings: Chọn `lab04-custom-vpc` $\rightarrow$ Subnet: `lab04-public-1a`.
   * Thấy dòng *Auto-assign public IP* tự động hiển thị: **Enable**.
   * Nhấn **Launch instance**.
   * Sau khi máy chạy, chọn máy $\rightarrow$ nhấn **Connect** qua **EC2 Instance Connect** $\rightarrow$ Kết nối mở thành công và ping ra ngoài Internet bình thường:
     ```bash
     ping -c 4 8.8.8.8
     ```

2. **Khởi tạo máy chủ thử nghiệm trên Private Subnet:**
   * Nhấn **Launch instances** thêm một máy khác.
   * Tên: `test-private-ec2` | OS: Amazon Linux 2023.
   * Network settings: Chọn `lab04-custom-vpc` $\rightarrow$ Subnet: `lab04-private-1a`.
   * Thấy dòng *Auto-assign public IP* hiển thị: **Disable**.
   * Nhấn **Launch instance**.
   * Sau khi máy chạy: Bạn sẽ thấy máy này **hoàn toàn không có Public IPv4 address**. Thử bấm nút **Connect** qua EC2 Instance Connect $\rightarrow$ Trình duyệt báo lỗi kết nối thất bại vì không có đường truyền Internet công cộng trực tiếp.

---

## 4. Quy trình dọn dẹp tài nguyên (Resource Cleanup)

Do toàn bộ thành phần VPC, Subnet, Route Table và IGW đều **hoàn toàn miễn phí ($0)**, chi phí duy nhất chỉ phát sinh từ 2 máy ảo EC2 test ($0.01/giờ). Hãy dọn dẹp theo thứ tự sau:

1. **Xóa máy chủ EC2:**
   * Vào **EC2** $\rightarrow$ **Instances** $\rightarrow$ chọn cả `test-public-ec2` và `test-private-ec2`.
   * Chọn **Instance state** $\rightarrow$ **Terminate instance** $\rightarrow$ Xác nhận Terminate.
   * Chờ cả 2 máy chuyển sang trạng thái **Terminated**.
2. **Xóa Custom VPC:**
   * Vào dịch vụ **VPC** $\rightarrow$ chọn **Your VPCs**.
   * Tích chọn `lab04-custom-vpc` $\rightarrow$ nhấn **Actions** $\rightarrow$ chọn **Delete VPC**.
   * Điền chữ `delete` vào ô xác nhận $\rightarrow$ Nhấn **Delete**.
   * *(Cơ chế tiện ích của AWS: Khi bạn xóa VPC, AWS sẽ tự động gỡ và xóa toàn bộ các Subnet, Route Table tự tạo, Security Group nội bộ và tách Internet Gateway đi kèm mà bạn không cần phải xóa thủ công từng cái).*

---

## 5. Bảng tổng hợp kiến thức thi SAA-C03 từ Lab 04

| Khái niệm kiến trúc | Đặc tính kỹ thuật cốt lõi | Bẫy câu hỏi trong bài thi SAA |
|---|---|---|
| **5 Reserved IPs** | Mỗi Subnet luôn mất 5 IP: `.0` (Network), `.1` (Router), `.2` (DNS), `.3` (Future use), `.255` (Broadcast). | Đề bài cho dải mạng `/28` ($16$ IP) và hỏi có chạy được 12 máy EC2 không? $\rightarrow$ Không, vì $16 - 5 = 11$ IP khả dụng. |
| **Bản chất Public Subnet** | Subnet được coi là Public khi và chỉ khi Route Table của nó có tuyến `0.0.0.0/0` trỏ tới **Internet Gateway (`igw-xxxx`)**. | Đề bài hỏi máy chủ có Public IP nhưng không vào được web $\rightarrow$ Kiểm tra Route Table đã có tuyến trỏ ra `igw` chưa. |
| **Main Route Table** | Route Table mặc định sinh ra cùng VPC. Mọi Subnet mới không gán tường minh sẽ tự rơi vào Main Route Table. | Best Practice: Giữ Main Route Table làm Private; tạo Route Table mới riêng biệt cho Public Subnet. |
| **Internet Gateway Scope** | Gắn ở cấp độ VPC (Regional), tự động co giãn băng thông, không bị lỗi nghẽn cổ chai vật lý. | Mỗi VPC chỉ gắn được **duy nhất 1 Internet Gateway**. |
