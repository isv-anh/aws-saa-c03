# Lab 1: Thiết Lập IAM Admin User & Khóa Tài Khoản Root

## 1. Mục tiêu bài thực hành
- Áp dụng nguyên tắc an ninh tối quan trọng của AWS: **Không sử dụng AWS Root Account cho các tác vụ quản trị hàng ngày**.
- Tạo IAM User Group quản trị (`Administrators`) và gán chính sách `AdministratorAccess`.
- Tạo IAM User cá nhân gán vào nhóm và cấp quyền truy cập Web Console.
- Kích hoạt xác thực đa yếu tố (MFA - Multi-Factor Authentication).
- Đăng nhập kiểm tra và tách biệt hoàn toàn khỏi tài khoản Root.

---

## 2. Các bước thực hiện chi tiết

### Bước 1: Tạo IAM User Group có quyền Administrator
1. Đăng nhập vào **AWS Management Console** bằng tài khoản **Root**.
2. Trên thanh tìm kiếm trên cùng, gõ **IAM** và nhấp chọn dịch vụ. *(IAM là dịch vụ Global nên vùng hiển thị sẽ là Global).*
<img width="1908" height="948" alt="image" src="https://github.com/user-attachments/assets/0fcad2b4-1d8b-41b2-ac46-87bbf51759c3" />

3. Tại thanh menu bên trái, chọn **IAM user groups** nhấn nút **Create group**.
<img width="1929" height="945" alt="image" src="https://github.com/user-attachments/assets/e1344d3d-5780-4d40-81a7-aeedbbbb14c1" />


4. Cấu hình thông tin nhóm:
   - **User group name:** Điền `Administrators`
5. Kéo xuống phần **Attach permissions policies**:
   - Tại ô tìm kiếm, nhập: `AdministratorAccess`
   - Tích chọn vào ô vuông bên cạnh chính sách **AdministratorAccess** (chính sách AWS managed).
6. Kéo xuống dưới cùng và nhấn **Create user group**.

---

### Bước 2: Tạo IAM User và cấp quyền truy cập Web Console
1. Tại menu bên trái, chọn **Users** nhấn nút **Create user**.
2. **Step 1 - Specify user details:**
   - **User name:** Đặt tên định danh cá nhân (ví dụ: `cloud-admin`).
   - Tích chọn: **Provide user access to the AWS Management Console - optional**.
   - Chọn mục: **I want to create an IAM user**.
   - **Console password:** Chọn **Custom password** và nhập mật khẩu an toàn.
   - Bỏ tích chọn mục *Users must create a new password at next sign-in* nếu không muốn đổi mật khẩu ở lần đăng nhập tiếp theo.
   - Nhấn **Next**.
3. **Step 2 - Set permissions:**
   - Chọn mục **Add user to group**.
   - Tích chọn nhóm `Administrators` đã tạo ở Bước 1.
   - Nhấn **Next**.
4. **Step 3 - Review and create:**
   - Xem lại cấu hình và nhấn **Create user**.
5. **Step 4 - Retrieve credentials:**
   - Sao chép và lưu lại dòng **Console sign-in URL** (định dạng: `https://<account-id>.signin.aws.amazon.com/console`).
---

### Bước 3: Cấu hình MFA (Xác thực 2 yếu tố) cho IAM User
1. Trong danh sách **Users**, nhấp trực tiếp vào tên user vừa tạo (`cloud-admin`).
2. Chuyển sang tab **Security credentials**.
3. Tại mục **Multi-factor authentication (MFA)**, nhấn **Assign MFA device**.
4. Điền thông tin thiết bị:
   - **Device name:** Điền `Phone-MFA`
   - **MFA device:** Chọn **Authenticator app** (Google Authenticator, Microsoft Authenticator, Authy).
   - Nhấn **Next**.
5. Mở ứng dụng trên điện thoại, bấm **Show QR code** trên màn hình máy tính và quét mã.
6. Nhập 2 mã OTP liên tiếp:
   - **MFA code 1:** Nhập mã 6 số hiện tại.
   - **MFA code 2:** Đợi sau 30 giây để app sinh mã 6 số mới và nhập tiếp vào ô này.
7. Nhấn **Add MFA**.
---

### Bước 4: Đăng xuất Root và kiểm tra đăng nhập bằng IAM User
1. Nhấp vào tên tài khoản ở góc trên bên phải màn hình chọn **Sign out**.
2. Mở cửa sổ ẩn danh (Incognito) hoặc tab mới, dán đường dẫn **Console sign-in URL** đã lưu ở Bước 2.
3. Màn hình đăng nhập sẽ tự động điền sẵn **Account ID (12 chữ số)**:
   - **IAM user name:** Điền tên user (ví dụ: `cloud-admin`).
   - **Password:** Nhập mật khẩu đã đặt.
   - Nhấn **Sign in**.
4. Nhập mã OTP 6 số từ ứng dụng điện thoại để xác thực.


---

## 3. Kiến thức trọng tâm cho kỳ thi SAA-C03
- **Quy tắc Least Privilege:** Không bao giờ dùng tài khoản Root cho công việc thông thường; Root User chỉ dùng khi có sự cố khẩn cấp (break-glass).
- **Anti-Pattern:** Gán quyền trực tiếp cho User (Inline Policy). Quy chuẩn chuẩn xác luôn là: `Gán Policy vào Group` `Thêm User vào Group`.
- **Thông tin đăng nhập:** IAM User muốn đăng nhập vào Console luôn cần 3 thông tin: **Account ID (hoặc Alias)** + **Username** + **Password (+ MFA)**.
