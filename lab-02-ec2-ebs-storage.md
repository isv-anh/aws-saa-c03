# Lab 02: Khởi Tạo Amazon EC2, Gắn Ổ Đĩa Amazon EBS & Kiểm Chứng Tính Năng Phân Vùng

## 1. Mục tiêu bài lab
- Khởi chạy máy chủ ảo Amazon EC2 chạy hệ điều hành Amazon Linux 2023.
- Tạo một ổ đĩa ngoài Amazon EBS gp3 độc lập và thực hiện thao tác gắn kết (Attach) vào EC2.
- Kết nối vào hệ điều hành qua EC2 Instance Connect, định dạng phân vùng (format filesystem) và gắn thư mục (mount).
- Thực nghiệm kiểm chứng tính chất **AZ-Locked** (ổ EBS bị khóa cứng trong 1 Availability Zone).
- Thực hiện quy trình sao lưu và chuyển vùng ổ đĩa bằng **EBS Snapshot**.
- Dọn dẹp tài nguyên đúng quy chuẩn để tránh phát sinh chi phí ngoài ý muốn.

---

## 2. Thông số thiết lập môi trường
- **AWS Region:** Singapore (`ap-southeast-1`)
- **Tài khoản thực hiện:** IAM Admin User (`cloud-admin`)
- **Mạng:** Default VPC
- **Loại máy chủ:** `t3.micro` hoặc `t2.micro` (tùy thuộc gói Free Tier của tài khoản)
- **Hệ điều hành:** Amazon Linux 2023 AMI (64-bit x86)

---

## 3. Các bước thực hiện chi tiết

### Bước 1: Khởi chạy máy chủ EC2 (Compute Tier)
1. Truy cập **AWS Management Console** $\rightarrow$ chọn dịch vụ **EC2**.
2. Chọn **Instances** ở thanh điều hướng bên trái $\rightarrow$ nhấn nút màu cam **Launch instances**.
3. Điền các trường thông số:
   - **Name:** `lab03-web-server`
   - **Application and OS Images:** Chọn **Amazon Linux** $\rightarrow$ **Amazon Linux 2023 AMI**.
   - **Instance type:** Chọn `t3.micro` (hoặc `t2.micro`).
   - **Key pair (login):** Chọn **Proceed without a key pair (Not recommended)**.
4. Cấu hình **Network settings** (Nhấn nút **Edit** bên góc phải):
   - **VPC:** Chọn **Default VPC**.
   - **Subnet:** Chọn Subnet có Availability Zone cụ thể và **GHI NHỚ LẠI MÃ VÙNG NÀY** (ví dụ: `ap-southeast-1a`).
   - **Auto-assign public IP:** Chọn **Enable**.
   - **Firewall (security groups):** Chọn *Create security group* $\rightarrow$ Đặt tên `lab03-ec2-sg` $\rightarrow$ Thêm Inbound Rule cho phép **SSH (Port 22)** từ nguồn **Anywhere-IPv4 (`0.0.0.0/0`)**.
5. Cấu hình **Configure storage:** Giữ nguyên ổ đĩa mặc định (1x 8 GiB gp3 Root volume).
6. Nhấn nút màu cam **Launch instance**.

> **Kiểm tra thành công:**  
> Vào bảng danh sách **Instances**, tìm máy chủ `lab03-web-server`. Đợi 1 – 2 phút cho đến khi cột **Instance state** chuyển sang màu xanh lá **Running** và cột **Status check** báo **2/2 checks passed**. Ghi nhận chính xác cột **Availability Zone** (ví dụ: `ap-southeast-1a`).

---

### Bước 2: Tạo thêm một ổ đĩa dữ liệu Amazon EBS độc lập
1. Ở thanh menu bên trái mục **Elastic Block Store**, chọn **Volumes**.
2. Nhấn nút **Create volume** ở góc trên bên phải.
3. Cấu hình thông số ổ đĩa:
   - **Volume type:** **General Purpose SSD (gp3)**.
   - **Size (GiB):** Điền `1` GiB.
   - **IOPS:** Giữ nguyên mặc định `3000`.
   - **Throughput:** Giữ nguyên mặc định `125` MiB/s.
   - **Availability Zone:** **BẮT BUỘC CHỌN TRÙNG VỚI AZ CỦA MÁY EC2 Ở BƯỚC 1** (ví dụ: `ap-southeast-1a`).
   - **Tags:** Nhấn *Add tag* $\rightarrow$ Key: `Name` | Value: `lab03-ebs-data`.
4. Nhấn **Create volume**.

> **Kiểm tra thành công:**  
> Tại danh sách **Volumes**, ổ đĩa `lab03-ebs-data` xuất hiện với cột **Volume state** là **Available** (màu xanh dương).

---

### Bước 3: Gắn (Attach) ổ EBS vào máy chủ EC2
1. Tích chọn vào ổ đĩa `lab03-ebs-data` vừa tạo.
2. Nhấn nút **Actions** $\rightarrow$ chọn **Attach volume**.
3. Cấu hình liên kết:
   - **Instance:** Nhấp vào ô tìm kiếm và chọn `lab03-web-server`.
   - **Device name:** Giữ nguyên tên tự động gán (ví dụ: `/dev/xvdf` hoặc `/dev/sdf`).
4. Nhấn nút **Attach volume**.

> **Kiểm tra thành công:**  
> Cột **Volume state** của `lab03-ebs-data` chuyển từ *Available* sang **In-use** (màu xanh lá). Vào chi tiết máy `lab03-web-server`, mở tab **Storage** sẽ thấy xuất hiện cả 2 ổ đĩa: Root volume (8 GiB) và Block device phụ (1 GiB).

---

### Bước 4: Định dạng phân vùng và gắn ổ đĩa trên hệ điều hành Linux
1. Vào menu **Instances** $\rightarrow$ tích chọn `lab03-web-server` $\rightarrow$ nhấn nút **Connect** ở thanh công cụ trên cùng.
2. Tại tab **EC2 Instance Connect**, giữ nguyên User name mặc định (`ec2-user`) $\rightarrow$ nhấn nút **Connect** màu cam để mở cửa sổ dòng lệnh Terminal trực tiếp trên trình duyệt.
3. Kiểm tra danh sách ổ đĩa vật lý được nhận diện:
   ```bash
   lsblk
   ```
   *Kết quả:* Xuất hiện ổ đĩa chính `xvda` (8G) và một ổ đĩa thô mới `xvdf` (1G) chưa có phân vùng.
4. Kiểm tra xem ổ đĩa đã có định dạng dữ liệu (filesystem) hay chưa:
   ```bash
   sudo file -s /dev/xvdf
   ```
   *Kết quả:* Trả về `/dev/xvdf: data` (chứng tỏ ổ đĩa hoàn toàn trắng).
5. Tạo hệ thống tệp tin chuẩn **XFS**:
   ```bash
   sudo mkfs -t xfs /dev/xvdf
   ```
6. Tạo thư mục đích và gắn (mount) ổ đĩa vào thư mục:
   ```bash
   sudo mkdir /data
   sudo mount /dev/xvdf /data
   ```
7. Kiểm tra trạng thái gắn kết:
   ```bash
   df -h /data
   ```
   *Kết quả:* Hiển thị thư mục `/data` có dung lượng ~1.0G với đường dẫn thiết bị `/dev/xvdf`.
8. Tạo một file thử nghiệm để kiểm chứng khả năng lưu trữ:
   ```bash
   echo "Du lieu quan trong duoc luu tren EBS Volume" | sudo tee /data/test.txt
   cat /data/test.txt
   ```

> **Kiểm tra thành công:**  
> Lệnh `cat /data/test.txt` in ra đúng chuỗi văn bản vừa ghi mà không xuất hiện bất kỳ cảnh báo lỗi quyền truy cập nào.

---

### Bước 5: Kiểm chứng tính chất "AZ-Locked" của Amazon EBS
1. Quay lại AWS Console $\rightarrow$ vào mục **Volumes** $\rightarrow$ nhấn **Create volume**.
2. Cấu hình ổ đĩa thử nghiệm:
   - **Size:** `1` GiB.
   - **Availability Zone:** Chọn một AZ **KHÁC** với AZ của máy chủ EC2 (ví dụ: chọn `ap-southeast-1b`).
   - **Tag Name:** `lab03-ebs-wrong-az`.
3. Nhấn **Create volume**.
4. Khi ổ đĩa chuyển sang trạng thái *Available*, chọn nó $\rightarrow$ nhấn **Actions** $\rightarrow$ chọn **Attach volume**.
5. Nhấp vào ô tìm kiếm **Instance**:
   - **Hiện tượng:** Màn hình hiển thị thông báo **No matching instances found**.
   - **Bản chất kiến trúc:** EBS Volume gắn với máy chủ qua hạ tầng lưu trữ cục bộ của từng Availability Zone. Không thể gắn trực tiếp một ổ EBS sang máy chủ nằm ngoài Availability Zone của nó.

---

### Bước 6: Di chuyển dữ liệu sang AZ khác bằng EBS Snapshot
Khi cần chuyển dữ liệu của ổ `lab03-ebs-data` từ vùng `1a` sang vùng `1b`, quy trình chuẩn của kỳ thi SAA-C03 là: **Volume $\rightarrow$ Snapshot $\rightarrow$ Volume mới**.

1. Tại danh sách **Volumes**, chọn `lab03-ebs-data` $\rightarrow$ nhấn **Actions** $\rightarrow$ chọn **Create snapshot**.
2. Điền Description: `Snapshot backup for lab03` $\rightarrow$ nhấn **Create snapshot**.
3. Ở menu bên trái, vào mục **Snapshots**: Chờ trạng thái của snapshot chuyển sang **Completed** (100%).
4. Tích chọn snapshot vừa tạo $\rightarrow$ nhấn **Actions** $\rightarrow$ chọn **Create volume from snapshot**.
5. Cấu hình ổ đĩa mới:
   - **Availability Zone:** Chọn vùng đích mong muốn (ví dụ: `ap-southeast-1b`).
   - **Tag Name:** `lab03-ebs-restored-in-1b`.
6. Nhấn **Create volume**.

> **Kiểm tra thành công:**  
> Vào mục **Volumes**, ổ đĩa `lab03-ebs-restored-in-1b` xuất hiện tại vùng `ap-southeast-1b` ở trạng thái **Available**, sẵn sàng gắn vào bất kỳ máy EC2 nào thuộc vùng `1b`.

---

## 4. Quy trình dọn dẹp tài nguyên (Resource Cleanup)

Thực hiện tuần tự các bước sau để giải phóng toàn bộ tài nguyên, tránh phát sinh chi phí:

1. **Gỡ gắn kết trên hệ điều hành và tách ổ EBS:**
   - Trên cửa sổ Terminal EC2, chạy lệnh ngắt kết nối:
     ```bash
     sudo umount /data
     ```
   - Tại AWS Console $\rightarrow$ mục **Volumes** $\rightarrow$ chọn `lab03-ebs-data` $\rightarrow$ nhấn **Actions** $\rightarrow$ chọn **Detach volume** $\rightarrow$ Xác nhận **Detach**.
2. **Xóa các ổ đĩa EBS độc lập:**
   - Chọn lần lượt 3 ổ: `lab03-ebs-data`, `lab03-ebs-wrong-az`, và `lab03-ebs-restored-in-1b`.
   - Nhấn **Actions** $\rightarrow$ **Delete volume** $\rightarrow$ Xác nhận xóa.
3. **Xóa bản sao lưu Snapshot:**
   - Vào mục **Snapshots** $\rightarrow$ chọn snapshot vừa tạo $\rightarrow$ **Actions** $\rightarrow$ **Delete snapshot**.
4. **Xóa máy chủ EC2:**
   - Vào mục **Instances** $\rightarrow$ chọn `lab03-web-server` $\rightarrow$ **Instance state** $\rightarrow$ chọn **Terminate instance** $\rightarrow$ Xác nhận **Terminate**.

---

## 5. Bảng tổng hợp kiến thức trọng tâm cho kỳ thi SAA-C03

| Khái niệm | Đặc tính kỹ thuật cốt lõi | Dấu hiệu nhận biết trong đề thi SAA |
|---|---|---|
| **EBS Scope** | Khóa theo Availability Zone (**AZ-locked**) | Đề bài yêu cầu dùng chung giữa 2 AZ $\rightarrow$ Loại EBS, chọn **EFS**. |
| **Cross-AZ Migration** | Phải thông qua quy trình: **Volume $\rightarrow$ Snapshot $\rightarrow$ Restore Volume at new AZ** | Đề bài yêu cầu di chuyển volume sang AZ khác hoặc Region khác. |
| **Delete on Termination** | Mặc định: Root Volume = **Yes**, Attached Secondary Volumes = **No** | Đề bài hỏi tại sao sau khi xóa EC2 chi phí lưu trữ EBS vẫn phát sinh hàng tháng. |
| **Snapshot Storage** | Dữ liệu Snapshot được lưu ngầm trên **Amazon S3** theo cơ chế vi sai lũy kế (**Incremental**) | Đề bài hỏi nơi lưu trữ an toàn và cơ chế tính dung lượng khi sao lưu EBS. |
