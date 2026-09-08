# Hướng dẫn cấu hình Backup Tự động cho Nginx Proxy Manager (NPM)

Tài liệu này hướng dẫn cách thiết lập một kịch bản (Bash Script) và hẹn giờ (Cronjob) để tự động sao lưu dữ liệu của hệ thống Nginx Proxy Manager vào **2:00 sáng mỗi ngày**. Hệ thống cũng sẽ tự động xóa các bản backup cũ hơn 7 ngày để tránh đầy ổ cứng Server.

---

## Bước 1: Tạo thư mục chứa file Backup. 
Tạo một thư mục riêng biệt chuyên dùng để chứa các file nén backup. Mở Terminal trên Server và chạy lệnh sau:

```bash
sudo mkdir -p /opt/docker/npm/backups
```
## Bước 2: Viết kịch bản tự động (Bash Script). Tạo một file script bằng trình soạn thảo nano:
```bash
sudo nano /opt/docker/npm/auto_backup.sh
```

Dán toàn bộ đoạn mã dưới đây vào file:

```bash
#!/bin/bash
# Khai báo PATH để Cronjob nhận diện được lệnh docker
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# ==========================================
# CẤU HÌNH THÔNG SỐ
# ==========================================
NPM_DIR="/opt/docker/npm"
BACKUP_DIR="/opt/docker/npm/backups"
TIMESTAMP=$(date +%Y-%m-%d_%H-%M)
BACKUP_FILE="npm_backup_$TIMESTAMP.tar.gz"

echo "=== Bắt đầu backup NPM lúc $(date) ==="

# Chuyển vào thư mục NPM
cd $NPM_DIR || exit

# 1. Xuất dữ liệu Database ra file SQL tạm
echo "Đang dump database..."
docker compose exec -T db sh -c 'exec mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" npm' > backup_db_temp.sql

# 2. Nén tất cả thành 1 file duy nhất (vào thư mục backups)
# Lưu ý: Bỏ qua thư mục /mysql nguyên gốc để tránh lỗi hỏng file
echo "Đang nén dữ liệu..."
tar -czvf $BACKUP_DIR/$BACKUP_FILE docker-compose.yml .env data letsencrypt backup_db_temp.sql

# 3. Xóa file SQL tạm sau khi nén xong để dọn dẹp
rm backup_db_temp.sql

# 4. Dọn dẹp: Chỉ giữ lại các bản backup trong 7 ngày gần nhất
echo "Đang dọn dẹp các bản backup cũ hơn 7 ngày..."
find $BACKUP_DIR -type f -name "npm_backup_*.tar.gz" -mtime +7 -exec rm {} \;

echo "=== Backup thành công: $BACKUP_FILE ==="
echo "----------------------------------------"
```

Lưu file bằng cách nhấn Ctrl+O, Enter, sau đó thoát bằng Ctrl+X.

## Bước 3: Cấp quyền thực thi cho Script
Để hệ điều hành cho phép script này chạy, bạn cần cấp quyền thực thi (executable):

```bash
sudo chmod +x /opt/docker/npm/auto_backup.sh
```

(Mẹo: Bạn có thể chạy thử lệnh sudo /opt/docker/npm/auto_backup.sh để kiểm tra ngay xem hệ thống nén file thành công chưa).

## Bước 4: Hẹn giờ chạy tự động bằng Cronjob
Tiến hành lập lịch để hệ thống tự động gọi Script trên vào lúc 2h00 sáng mỗi ngày. Mở cấu hình Cronjob:

```bash
sudo crontab -e
```

(Nếu hệ thống hỏi, hãy chọn số tương ứng với trình soạn thảo nano). Di chuyển con trỏ xuống dòng cuối cùng của file và thêm dòng sau:

```bash
0 2 * * * /opt/docker/npm/auto_backup.sh >> /var/log/npm_backup.log 2>&1
```
Lưu lại và thoát ra.

Giải thích cấu hình Cronjob:

0 2 * * *: Thiết lập thời gian chạy là Phút 0, Giờ 2 (2:00 AM) mỗi ngày.

>> /var/log/npm_backup.log 2>&1: Nhật ký (thành công/thất bại) sẽ được ghi lại âm thầm vào file /var/log/npm_backup.log. Bạn có thể đọc file này bằng lệnh cat /var/log/npm_backup.log để kiểm tra lịch sử chạy nếu cần.


Để chạy thử kịch bản backup ngay lập tức (không cần chờ đến 2h sáng), bạn chỉ cần gọi trực tiếp đường dẫn của file script bằng quyền sudo. Bạn hãy gõ lệnh sau vào Terminal:
```bash
sudo /opt/docker/npm/auto_backup.sh
```

dùng lệnh ls kết hợp tham số -lh để xem danh sách file trong thư mục backup kèm theo dung lượng thực tế của chúng:
```bash
ls -lh /opt/docker/npm/backups
```
