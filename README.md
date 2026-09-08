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

Di chuyển đến thư mục chứa key trên máy tính Window của Admin:

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


## Phần 3. Cài đặt Docker và các thành phần cốt lõi của Docker

### Bước 1: Cập nhật index các gói phần mềm của hệ thống Server
```bash
sudo apt update && sudo apt upgrade -y
```

### Bước 2: Cài đặt các dependencies cần thiết
```bash
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    apt-transport-https
```

### Bước 3: Cấu hình repository của Docker
Tạo thư mục chứa keyrings nếu chưa có:
```bash
sudo mkdir -p /etc/apt/keyrings
sudo chmod 0755 /etc/apt/keyrings
```

Tải và lưu GPG key của Docker an toàn:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Thêm Docker repository vào APT sources:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Bước 4: Cài đặt Docker
Cập nhật lại apt cache để nhận repo mới và tiến hành cài đặt:
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Bước 5: Cấu hình bảo mật Docker Daemon (Quan trọng)
Mặc định Docker daemon khá "thoải mái". Nếu đang cài Docker mới hoàn toàn, bạn cần thực hiện bước này để siết chặt lại.
```bash
sudo mkdir -p /etc/docker
sudo nano /etc/docker/daemon.json
```
Dán nội dung dưới vào file `daemon.json` và lưu lại:
```json
{
  "icc": false,
  "no-new-privileges": true,
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "3"
  },
  "live-restore": true,
  "userland-proxy": false
}
```
> **Giải thích các thông số bảo mật:**
> - `"icc": false`: Ngăn chặn các container trong cùng default bridge network tự do nói chuyện với nhau. Phải link explicitly qua custom network.
> - `"no-new-privileges": true`: Ngăn chặn tiến trình trong container tự ý leo thang đặc quyền (ví dụ dùng `su` hay `sudo`).
> - `"log-opts"`: Ngăn tình trạng log của container phình to làm tràn ổ cứng (chỉ giữ tối đa 3 file, mỗi file 50MB).
> - `"live-restore": true`: Cho phép container tiếp tục chạy khi Docker daemon tạm thời bị restart/mất kết nối, trong các điều kiện được Docker hỗ trợ
> - `"userland-proxy": false`: Tắt proxy không cần thiết, giảm bề mặt tấn công. Sử dụng iptables thuần túy để route port.

### Bước 6: Khởi động lại và phân quyền
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
Cấu hình tự động khởi động Docker và containerd cùng hệ điều hành:
```bash
sudo systemctl enable docker
sudo systemctl enable containerd
```

### Bước 7: Kiểm tra hệ thống
Xác minh phiên bản và trạng thái hoạt động:
```bash
# Kiểm tra version Docker Engine
sudo docker version

# Kiểm tra version Docker Compose Plugin
sudo docker compose version



# Chạy thử container an toàn để test
sudo docker run --rm hello-world
```

---



## Phần 4. Cài đặt Nginx Proxy Manager
### Bước 1. Chuẩn bị môi trường và Cấu trúc thư mục. 
Để các container giao tiếp an toàn và tách biệt, chúng ta sẽ tạo một Docker Network riêng (ví dụ tên là proxy-tier). Bất kỳ dịch vụ nào sau này bạn muốn chạy qua tên miền đều sẽ được gắn vào mạng này, thay vì mở port trực tiếp ra ngoài. Tạo Docker Network:
```bash
sudo docker network create proxy-tier
```
sau đó tạo cấu trúc thư mục lưu trữ Nginx Proxy Manager:
```bash
sudo mkdir -p /opt/docker/npm
cd /opt/docker/npm
```
### Bước 2. Quản lý Secret với file .env. 
Tuyệt đối không lưu mật khẩu database dưới dạng "clear text" trong file cấu hình chính. Chúng ta sẽ dùng file .env để quản lý. Đứng ngay tại thư mục /opt/docker/npm Tạo file .env bằng lệnh: 
```bash
sudo nano .env
```
sau đó dán nội dung dưới vào vào file .env 
```bash
# Database Passwords
DB_ROOT_PASSWORD=Thay_Bang_Mat_Khau_Root_Sieu_Kho
DB_PASSWORD=Thay_Bang_Mat_Khau_User_Sieu_Kho
```
Lưu file bằng cách nhấn Ctrl+O, Enter và Ctrl+X. 

Để bảo mật, phân quyền lại file .env chỉ cho phép tài khoản root mới có thể đọc file bằng lệnh:

```bash
sudo chmod 600 .env
```

### Bước 3. Cấu hình docker-compose.yml chuẩn. 
Mặc định, NPM dùng SQLite (khá yếu và dễ lỗi khi có nhiều luồng truy cập). Chúng ta sẽ dùng mySQL làm cơ sở dữ liệu để đảm bảo hiệu suất. Đứng ngay tại thư mục Tạo file docker-compose.yml:
```bash
cd /opt/docker/npm
sudo nano docker-compose.yml
```
sau đó dán nội dung dưới vào trong file docker-compose.yml
```bash
services:
  app:
    image: 'jc21/nginx-proxy-manager:2.15.1'
    container_name: npm_app
    restart: unless-stopped

    ports:
      - '80:80'
      - '443:443'
      # Chỉ cho phép truy cập Admin Panel qua localhost bằng ssh Tunnel
      - '127.0.0.1:81:81'

    environment:
      TZ: "Asia/Ho_Chi_Minh"

      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "npm"
      DB_MYSQL_PASSWORD: "${DB_PASSWORD}"
      DB_MYSQL_NAME: "npm"

    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt

    depends_on:
      db:
        condition: service_healthy

    networks:
      - proxy-tier

  db:
    image: 'mysql:8.0'
    container_name: npm_db
    restart: unless-stopped

    environment:
      TZ: "Asia/Ho_Chi_Minh"
      MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD}"
      MYSQL_DATABASE: "npm"
      MYSQL_USER: "npm"
      MYSQL_PASSWORD: "${DB_PASSWORD}"

    volumes:
      - ./mysql:/var/lib/mysql

    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${DB_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s

    networks:
      - proxy-tier

networks:
  proxy-tier:
    external: true
```

Lưu ý: Để file docker-compose.yml này chạy được, bạn nhớ phải có file .env nằm cùng thư mục (chứa 2 biến mật khẩu DB_ROOT_PASSWORD và DB_PASSWORD) như chúng ta đã cấu hình ở bước trước nhé.
<img width="747" height="143" alt="image" src="https://github.com/user-attachments/assets/a16e8779-05f5-4033-9228-e23851def3ef" />

Ngay tại thư mục /opt/docker/npm chứa file docker-compose.yml, chạy lệnh sau để Docker kéo các image về và chạy ngầm (chữ -d viết tắt của detached - chạy ngầm):
```bash
sudo docker compose up -d
```
Lần đầu tiên chạy sẽ mất một chút thời gian để hệ thống tải jc21/nginx-proxy-manager và mysql từ Internet về server.

Kiểm tra log (Rất quan trọng). Sau khi lệnh trên chạy xong, cần kiểm tra xem NPM đã kết nối thành công với Database chưa (để chắc chắn mật khẩu trong file .env đã được nhận diện đúng).

```bash
sudo docker compose logs -f app
```
Nếu bạn thấy thông báo kiểu như: [Nginx] › ℹ  info      Reloading Nginx và không có dòng chữ báo lỗi nào (màu đỏ) về Database. Bạn nhấn Ctrl + C để thoát khỏi màn hình xem log.

### Bước 4: Tạo SSH Tunnel để truy cập an toàn
Vì chúng ta đã thiết lập tính năng bảo mật cao nhất (không mở cổng 81 ra Internet), nên bạn không thể gõ trực tiếp http://IP_Server:81 trên trình duyệt được. Mỗi lần muốn vào Web GUI Nginx Proxy Manager, bạn đều phải mở cửa sổ Terminal và chạy lệnh SSH Tunnel.

Mở một cửa sổ Terminal/Command Prompt mới trên máy tính cá nhân của bạn (Laptop/PC đang dùng) và gõ lệnh sau:
```bash
ssh -L 8081:127.0.0.1:81 user_cua_ban@IP_Server_Cua_Ban
```

Nếu hệ thống của bạn chỉ sử dụng file Private Key để đăng nhập ssh server hãy dùng lệnh

```bash
ssh -i /duongdan/file_Privatekey_cua_ban.pem -L 8081:127.0.0.1:81 user_cua_ban@IP_Server_Cua_Ban
```

(Thay user_cua_ban và IP_Server_Cua_Ban bằng tài khoản đăng nhập SSH vào server của bạn). Lệnh này tạo một "đường hầm" an toàn, nối cổng 8081 trên máy tính của bạn với cổng 81 trên Server. Cứ treo cửa sổ này ở đó, đừng tắt đi trong quá trình chạy này chúng ta sẽ đăng nhập được Web GUI.

### Bước 5: Đăng nhập Web GUI và thiết lập bảo mật lần đầu
Mở trình duyệt web trên máy tính của bạn (Chrome, Edge, Safari...).

Truy cập vào địa chỉ: http://localhost:8081 sau đó đăng nhập bằng tài khoản mặc định của hệ thống:

Email: admin@example.com

Password: changeme

Ngay sau khi đăng nhập thành công, hệ thống sẽ yêu cầu bạn đổi thành Tên, Email thật của bạn và thiết lập Mật khẩu mới an toàn hơn.

