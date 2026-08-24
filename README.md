# Cài đặt Nginx Proxy Manager (NPM) trên Ubuntu Server


Bước 1: Mở cổng tường lửa (UFW):

sudo ufw allow 80/tcp

sudo ufw allow 443/tcp

sudo ufw allow 81/tcp

Bước 2: Tạo thư mục và file compose.yml. Tạo một thư mục riêng biệt để chứa cấu hình và dữ liệu của NPM. Điều này giúp dữ liệu không bị mất khi bạn khởi động lại hoặc cập nhật Docker.

mkdir -p ~/nginx-proxy-manager

cd ~/nginx-proxy-manager

Tạo và mở file compose.yml:

nano compose.yml

Bước 3: Nội dung file compose.yml chuẩn mực

version: '3.8'

services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      # Cổng 80 và 443 dành cho luồng traffic Web (HTTP/HTTPS)
      - '80:80'
      - '443:443'
      # Cổng 81 dành cho giao diện quản trị Admin Web UI
      - '81:81'
    volumes:
      # Lưu trữ dữ liệu cấu hình
      - ./data:/data
      # Lưu trữ chứng chỉ SSL (Let's Encrypt)
      - ./letsencrypt:/etc/letsencrypt
    healthcheck:
      test: ["CMD", "/usr/bin/check-health"]
      interval: 10s
      timeout: 3s


Nhấn Ctrl + O, sau đó nhấn Enter để lưu lại, và nhấn Ctrl + X để thoát nano.

Bước 4: Khởi chạy Nginx Proxy Manager
Ngay trong thư mục ~/nginx-proxy-manager, bạn chạy lệnh sau để kéo (pull) image và chạy ngầm (tham số -d):

docker compose up -d

Hãy chờ khoảng 1-2 phút để hệ thống tải file và khởi tạo cơ sở dữ liệu cho lần chạy đầu tiên.

Bạn có thể kiểm tra xem container đã chạy ổn định (trạng thái healthy) chưa bằng lệnh: docker ps


Bước 5: Đăng nhập lần đầu tiên
Mở trình duyệt web của bạn và truy cập vào giao diện quản trị theo địa chỉ IP của server:

👉 http://<IP_SERVER_CỦA_BẠN>:81

Tài khoản đăng nhập mặc định:

Email: admin@example.com

Password: changeme

(Ngay sau khi đăng nhập thành công, hệ thống sẽ bắt buộc bạn đổi thông tin Tên, Email và Mật khẩu mới. Hãy nhập thông tin của bạn vào để bảo mật).

🎉 Chúc mừng! Bạn đã cài đặt thành công Nginx Proxy Manager
