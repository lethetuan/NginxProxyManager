# Hướng dẫn cài đặt và cấu hình Nginx Proxy Manager trên Ubuntu Server

Tài liệu này hướng dẫn chi tiết các bước để thiết lập địa chỉ IP tĩnh, cấu hình bảo mật SSH (đăng nhập bằng SSH Key) để chạy Nginx Proxy Manager trên Ubuntu Server.

---

## Phần 1: Cấu hình IP tĩnh trên Ubuntu Server

### Bước 1: Di chuyển đến thư mục chứa cấu hình mạng và kiểm tra tên tập tin
Mở terminal và gõ các lệnh sau để truy cập thư mục cấu hình Netplan và xem danh sách file:

```bash
cd /etc/netplan
ls -l
```

### Bước 2: Chỉnh sửa tập tin cấu hình mạng
Sau khi dùng lệnh `ls -l`, bạn sẽ thấy một tập tin có định dạng `.yaml` (ví dụ: `00-installer-config.yaml` hoặc `50-cloud-init.yaml`). Tập tin này chứa các thông tin cấu hình mạng.

Dùng trình soạn thảo `nano` để chỉnh sửa (thay `file_name.yaml` bằng tên file thực tế của bạn):

```bash
sudo nano file_name.yaml
```
*(Nhập mật khẩu đăng nhập để xác thực và tiến hành chỉnh sửa).*

![Cấu hình IP 1](https://github.com/user-attachments/assets/5537ef14-aadf-4ac8-8a85-da83f56a804e)

Chỉnh sửa các thông số mạng theo nhu cầu mạng của bạn như: địa chỉ IP, địa chỉ DNS server, địa chỉ IP Default Gateway (thông qua mục routes).

![Cấu hình IP 2](https://github.com/user-attachments/assets/2602bed3-8524-4ea8-97ed-e0b661c8c42d)

**Lưu file và thoát khỏi nano:**
Sử dụng tổ hợp phím sau để lưu và thoát đúng cách:
1. Bấm `Ctrl + O` (để lưu).
2. Nhấn `Enter` (để xác nhận tên file).
3. Bấm `Ctrl + X` (để thoát).

*(Hoặc cách nhanh hơn: Bấm `Ctrl + X` -> Nhấn phím `Y` -> Nhấn `Enter`).*

### Bước 3: Lưu và áp dụng cấu hình
Dùng lệnh sau để áp dụng các thông số đã chỉnh sửa:

```bash
sudo netplan apply
```

![Apply Netplan](https://github.com/user-attachments/assets/a5041f73-bc57-481b-8d5a-f25b15907336)

Kiểm tra trạng thái kết nối mạng bằng lệnh ping:

```bash
ping google.com
```

![Ping Test](https://github.com/user-attachments/assets/fd0f30f6-5dbf-4270-964e-72fce0516cc5)

---

## Phần 2: Chi tiết cấu hình SSH trên Ubuntu Server

### Bước 1: Cập nhật hệ thống
Luôn đảm bảo hệ thống được cập nhật trước khi cài đặt phần mềm mới:

```bash
sudo apt update
sudo apt upgrade -y
```

### Bước 2: Cài đặt OpenSSH Server

```bash
sudo apt install openssh-server -y
```

### Bước 3: Kích hoạt SSH tự động khởi động cùng hệ thống

```bash
sudo systemctl enable --now ssh
```

Kiểm tra trạng thái SSH xem đã hoạt động (running) chưa:

```bash
sudo systemctl status ssh
```

![SSH Status](https://github.com/user-attachments/assets/01b9616c-1d23-4f2a-a4ef-8424708f01e8)

### Bước 4: Cấu hình Tường lửa (UFW)
Mở cổng trên Tường lửa (UFW) cho phép lưu lượng truy cập SSH qua cổng mặc định (cổng 22):

```bash
sudo ufw allow ssh
sudo ufw enable
```

![UFW Config](https://github.com/user-attachments/assets/989e0911-bcc9-478f-a80c-56949a6a5694)

### Bước 5: Tạo và sao chép SSH Key (Thực hiện trên máy tính Admin - Windows)

**1. Tạo SSH Key trên máy tính Windows của Admin:**
Mở Command Prompt (cmd) hoặc PowerShell và chạy lệnh:

```bash
ssh-keygen -t ed25519 -C "ten_may_tinh_cua_ban"
```
*(Ghi chú: `-C "ten_may_tinh_cua_ban"` chỉ để note lại tên máy tính giúp bạn dễ quản lý).*

**2. Sao chép Public Key lên Server Linux:**
Di chuyển đến thư mục chứa key:

```cmd
cd C:\Users\username\.ssh
```
*(Thay thế `username` bằng username máy tính của bạn).*

Chạy lệnh sau để đẩy key lên server *(Khuyên dùng Command Prompt - cmd để tránh lỗi encoding của PowerShell)*:

```cmd
type id_ed25519.pub | ssh tuan@10.1.1.30 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

*Lưu ý: Nếu bạn sử dụng PowerShell, lệnh pipe `|` đôi khi sẽ đổi bảng mã sang UTF-16 gây lỗi định dạng key trên Linux. Hãy dùng **Command Prompt (cmd)** cho bước này để an toàn nhất, hoặc dùng lệnh `ssh-copy-id` nếu máy bạn có hỗ trợ.*

**Minh họa lệnh dùng Command Prompt (cmd):**
![CMD Push Key](https://github.com/user-attachments/assets/1afa38a1-0718-42ba-9a74-2386943e5d7c)

**Minh họa lệnh dùng PowerShell:**
![PowerShell Push Key](https://github.com/user-attachments/assets/9bf99a63-7ce6-4190-bf15-152fc3ec9001)

![Key generation process](https://github.com/user-attachments/assets/20b354c0-ce4c-4a06-8af3-a4de525d47bd)

### Bước 6: Đăng nhập SSH bằng Key
Bây giờ bạn có thể đăng nhập vào server mà không cần nhập mật khẩu thông qua lệnh:

```bash
ssh username@dia_chi_ip_server
```

![SSH Login](https://github.com/user-attachments/assets/3ba4fa4d-03ab-4c9a-a6ef-2f9bd40b09c2)

Lưu trữ lại key đăng nhập thành công:

![SSH Key Saved](https://github.com/user-attachments/assets/b2cd2fe5-9e1b-42c7-ad8e-c1cda56baaf0)

### Bước 7: (Tùy chọn/Nâng cao) Vô hiệu hóa đăng nhập bằng mật khẩu
Sau khi bạn đã chắc chắn có thể đăng nhập thành công bằng SSH Key, bạn nên tắt tính năng đăng nhập bằng mật khẩu để ngăn chặn hoàn toàn các cuộc tấn công dò quét mật khẩu (brute-force):

1. Mở file cấu hình sshd:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
2. Tìm dòng `#PasswordAuthentication yes` hoặc `PasswordAuthentication yes` và sửa lại thành:
   ```text
   PasswordAuthentication no
   ```
3. Lưu file (`Ctrl + O`, `Enter`, `Ctrl + X`) và khởi động lại dịch vụ SSH:
   ```bash
   sudo systemctl restart ssh
   ```
