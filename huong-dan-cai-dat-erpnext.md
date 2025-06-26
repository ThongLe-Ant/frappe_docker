# ERPNext Docker Setup

Dự án này chứa các tệp cấu hình và hướng dẫn để cài đặt và nâng cấp ERPNext sử dụng Docker.

## Tệp tin chính

- `huong-dan-cai-dat-erpnext.md`: Hướng dẫn cài đặt ERPNext từ đầu
- `docker-compose.override.yml`: File cấu hình Docker Compose để chạy ERPNext

## Cấu trúc thư mục

- `frappe_docker_config/`: Chứa các tệp cấu hình và sao lưu từ cài đặt ERPNext

## Yêu cầu hệ thống

- Docker và Docker Compose
- Ít nhất 4GB RAM
- Ít nhất 10GB dung lượng ổ đĩa trống

## Cài đặt

Vui lòng xem tệp `huong-dan-cai-dat-erpnext.md` để biết chi tiết về cách cài đặt.


Tài liệu này hướng dẫn cài đặt ERPNext/Frappe sử dụng Docker và cách khắc phục các lỗi thường gặp.

## Yêu cầu hệ thống

- Docker và Docker Compose đã được cài đặt
- Ít nhất 4GB RAM
- Ít nhất 10GB dung lượng ổ đĩa trống

## Các bước cài đặt

### 1. Tải mã nguồn

```bash
git clone https://github.com/ThongLe-Ant/frappe_docker.git
cd frappe_docker
```

### 2. Tạo file .env

Tạo file `.env` trong thư mục gốc với nội dung sau:

```
FRAPPE_VERSION=version-15
ERPNEXT_VERSION=version-15
SITE_NAME=erp.goeat.com
DB_ROOT_PASSWORD=admin
```

### 3. Tạo file docker-compose.override.yml

Tạo file `docker-compose.override.yml` với nội dung sau để thêm các dịch vụ Redis và cấu hình kết nối:

```yaml
services:
  configurator:
    environment:
      REDIS_CACHE: redis-cache:6379
      REDIS_QUEUE: redis-queue:6379
      REDIS_SOCKETIO: redis-socketio:6379
    depends_on:
      - redis-cache
      - redis-queue
      - redis-socketio

  redis-cache:
    image: redis:6.2-alpine
    restart: unless-stopped

  redis-queue:
    image: redis:6.2-alpine
    restart: unless-stopped
    volumes:
      - redis-queue-data:/data

  redis-socketio:
    image: redis:6.2-alpine
    restart: unless-stopped

  websocket:
    command: ["bash", "-c", "cd /home/frappe/frappe-bench && node apps/frappe/socketio.js"]
    environment:
      - REDIS_CACHE=redis://redis-cache:6379
      - REDIS_QUEUE=redis://redis-queue:6379
      - REDIS_SOCKETIO=redis://redis-socketio:6379
    ports:
      - "9000:9000"
  
  backend:
    environment:
      - REDIS_CACHE=redis://redis-cache:6379
      - REDIS_QUEUE=redis://redis-queue:6379
      - REDIS_SOCKETIO=redis://redis-socketio:6379
    ports:
      - "8000:8000"

  queue-short:
    environment:
      - REDIS_CACHE=redis://redis-cache:6379
      - REDIS_QUEUE=redis://redis-queue:6379
      - REDIS_SOCKETIO=redis://redis-socketio:6379

  queue-long:
    environment:
      - REDIS_CACHE=redis://redis-cache:6379
      - REDIS_QUEUE=redis://redis-queue:6379
      - REDIS_SOCKETIO=redis://redis-socketio:6379

  scheduler:
    environment:
      - REDIS_CACHE=redis://redis-cache:6379
      - REDIS_QUEUE=redis://redis-queue:6379
      - REDIS_SOCKETIO=redis://redis-socketio:6379

  frontend:
    environment:
      - BACKEND=backend:8000
      - SOCKETIO=websocket:9000
    ports:
      - "8080:8080"

volumes:
  redis-queue-data:
  redis-cache-data:
  redis-socketio-data:
  mariadb-data:
```

### 4. Khởi động các container

```bash
docker compose up -d
```

### 5. Tạo site mới

```bash
docker compose exec backend bench new-site erp.goeat.com --mariadb-root-password admin --admin-password admin
```

### 6. Cài đặt ERPNext

```bash
docker compose exec backend bench --site erp.goeat.com install-app erpnext
```

### 7. Cấu hình Redis trong common_site_config.json

```bash
docker compose exec backend bash -c 'echo "{\"db_host\": \"db\", \"db_port\": 3306, \"redis_cache\": \"redis://redis-cache:6379\", \"redis_queue\": \"redis://redis-queue:6379\", \"redis_socketio\": \"redis://redis-socketio:6379\", \"socketio_port\": 9000}" > sites/common_site_config.json'
```

### 8. Cấu hình file hosts

Thêm dòng sau vào file `/etc/hosts` của máy host:

```
127.0.0.1 erp.goeat.com
```

## Khắc phục lỗi thường gặp

### 1. Lỗi "Bad Gateway" khi truy cập website

Nguyên nhân: Kết nối giữa frontend và backend bị lỗi.

Cách khắc phục:
- Kiểm tra log của frontend và backend:
  ```bash
  docker compose logs frontend --tail 20
  docker compose logs backend --tail 20
  ```

- Nếu thấy lỗi kết nối đến database, tạo người dùng database và cấp quyền:
  ```bash
  # Đăng nhập vào MariaDB
  docker compose exec db mysql -uroot -p
  
  # Tạo người dùng và cấp quyền (thay thế DB_NAME và DB_PASSWORD)
  CREATE USER 'DB_NAME'@'%' IDENTIFIED BY 'DB_PASSWORD';
  GRANT ALL PRIVILEGES ON DB_NAME.* TO 'DB_NAME'@'%';
  FLUSH PRIVILEGES;
  ```

- Khởi động lại các container:
  ```bash
  docker compose restart backend frontend
  ```

### 2. Lỗi kết nối đến Redis

Nguyên nhân: Cấu hình Redis không đúng hoặc dịch vụ Redis chưa được khởi động.

Cách khắc phục:
- Kiểm tra các container Redis đã chạy chưa:
  ```bash
  docker compose ps redis-cache redis-queue redis-socketio
  ```

- Cập nhật cấu hình Redis trong common_site_config.json:
  ```bash
  docker compose exec backend bash -c 'echo "{\"redis_cache\": \"redis://redis-cache:6379\", \"redis_queue\": \"redis://redis-queue:6379\", \"redis_socketio\": \"redis://redis-socketio:6379\", \"socketio_port\": 9000}" > sites/common_site_config.json'
  ```

- Khởi động lại các container:
  ```bash
  docker compose restart backend websocket queue-short queue-long scheduler
  ```

### 3. Lỗi "Websocket disabled for stability" hoặc lỗi kết nối websocket

Nguyên nhân: Cấu hình websocket không đúng hoặc lệnh khởi động không chính xác.

Cách khắc phục:
- Kiểm tra log của websocket:
  ```bash
  docker compose logs websocket
  ```

- Nếu thấy thông báo "Websocket disabled for stability", cần sửa lại cấu hình trong docker-compose.override.yml:
  ```yaml
  websocket:
    command: ["bash", "-c", "cd /home/frappe/frappe-bench && node apps/frappe/socketio.js"]
    environment:
      - REDIS_CACHE=redis://redis-cache:6379
      - REDIS_QUEUE=redis://redis-queue:6379
      - REDIS_SOCKETIO=redis://redis-socketio:6379
    ports:
      - "9000:9000"
  ```

- Sau đó khởi động lại container websocket:
  ```bash
  docker compose restart websocket
  ```

- Nếu vẫn gặp lỗi kết nối, hãy kiểm tra cấu hình trong frontend để đảm bảo nó trỏ đến đúng địa chỉ websocket:
  ```yaml
  frontend:
    environment:
      - BACKEND=backend:8000
      - SOCKETIO=websocket:9000
  ```

### 4. Lỗi kết nối cơ sở dữ liệu

Nguyên nhân: Người dùng cơ sở dữ liệu không tồn tại hoặc không có quyền truy cập.

Cách khắc phục:
- Xác định tên database và người dùng:
  ```bash
  docker compose exec backend cat sites/erp.goeat.com/site_config.json
  ```

- Tạo người dùng và cấp quyền:
  ```bash
  # Đăng nhập vào MariaDB
  docker compose exec db mysql -uroot -p
  
  # Tạo người dùng và cấp quyền
  CREATE USER 'DB_NAME'@'%' IDENTIFIED BY 'DB_PASSWORD';
  GRANT ALL PRIVILEGES ON DB_NAME.* TO 'DB_NAME'@'%';
  FLUSH PRIVILEGES;
  ```

### 5. Lỗi "Could not start up: Không thể thiết lập mặc định"

Nguyên nhân: Có vấn đề trong quá trình thiết lập ban đầu, thường liên quan đến cấu hình tài khoản hoặc công ty.

Cách khắc phục:
- Xóa cache và chạy migrate để cập nhật database:
  ```bash
  docker compose exec backend bench --site erp.goeat.com clear-cache
  docker compose exec backend bench --site erp.goeat.com migrate
  ```

- Đặt cấu hình setup_complete thành 1 để bỏ qua quá trình thiết lập ban đầu:
  ```bash
  docker compose exec backend bench --site erp.goeat.com set-config setup_complete 1
  ```

- Khởi động lại tất cả các container:
  ```bash
  docker compose restart
  ```

## Truy cập hệ thống

Sau khi cài đặt thành công, bạn có thể truy cập ERPNext qua địa chỉ:
- http://erp.goeat.com:8080

Đăng nhập với tài khoản:
- Username: Administrator
- Password: admin (hoặc mật khẩu đã đặt khi tạo site)

## Lưu ý quan trọng

1. Đảm bảo các cổng 8080, 8000 và 9000 không bị sử dụng bởi ứng dụng khác.
2. Nếu sử dụng trong môi trường production, cần cấu hình HTTPS và các biện pháp bảo mật khác.
3. Định kỳ sao lưu dữ liệu để tránh mất mát. 