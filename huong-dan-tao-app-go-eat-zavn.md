# Hướng dẫn tạo và khắc phục lỗi app Go Eat ZAVN trên ERPNext

Tài liệu này mô tả chi tiết quá trình tạo app tùy chỉnh Go Eat ZAVN trên ERPNext và cách khắc phục các lỗi thường gặp.

## 1. Tạo app tùy chỉnh

### Bước 1: Truy cập container backend

```bash
docker compose exec backend bash
```

### Bước 2: Tạo app mới

```bash
bench new-app go_eat_zavn --description "Go Eat ZAVN"
```

### Bước 3: Cài đặt app vào site

```bash
bench --site erp.goeat.com install-app go_eat_zavn
```

### Bước 4: Kiểm tra app đã tạo

```bash
bench --site erp.goeat.com list-apps
```

Kết quả sẽ hiển thị danh sách các app đã cài đặt, bao gồm:
```
frappe      15.72.0 UNVERSIONED
erpnext     15.66.0 UNVERSIONED
go_eat_zavn 0.0.1   develop
```

## 2. Khắc phục lỗi thường gặp

Sau khi tạo app, có thể gặp một số lỗi làm cho hệ thống không hoạt động. Dưới đây là các lỗi thường gặp và cách khắc phục:

### 2.1. Lỗi ModuleNotFoundError: No module named 'go_eat_zavn'

#### Nguyên nhân
Lỗi này xảy ra khi các container không thể tìm thấy module go_eat_zavn, thường do các file cấu hình không đúng định dạng.

#### Kiểm tra và khắc phục

1. **Kiểm tra file modules.txt**:
   ```bash
   docker compose exec backend bash -c "cat /home/frappe/frappe-bench/apps/go_eat_zavn/go_eat_zavn/modules.txt"
   ```
   
   Nếu thấy ký tự `%` ở cuối file, sửa lại:
   ```bash
   docker compose exec backend bash -c "echo 'Go Eat ZAVN' > /home/frappe/frappe-bench/apps/go_eat_zavn/go_eat_zavn/modules.txt"
   ```

2. **Kiểm tra file site_config.json**:
   ```bash
   docker compose exec backend bash -c "cat /home/frappe/frappe-bench/sites/erp.goeat.com/site_config.json"
   ```
   
   Nếu thấy ký tự `%` ở cuối file, sửa lại:
   ```bash
   docker compose exec backend bash -c "cat > /home/frappe/frappe-bench/sites/erp.goeat.com/site_config.json << 'EOF'
   {
    \"db_host\": \"db\",
    \"db_name\": \"_01b29dcafe1e237a\",
    \"db_password\": \"07FSFukRycZURvdo\",
    \"db_type\": \"mariadb\",
    \"maintenance_mode\": 0,
    \"redis_cache\": \"redis://redis-cache:6379\",
    \"redis_queue\": \"redis://redis-queue:6379\",
    \"redis_socketio\": \"redis://redis-socketio:6379\",
    \"setup_complete\": 1,
    \"socketio_port\": \"9000\"
   }
   EOF"
   ```

3. **Kiểm tra file common_site_config.json**:
   ```bash
   docker compose exec backend bash -c "cat /home/frappe/frappe-bench/sites/common_site_config.json"
   ```
   
   Nếu thấy ký tự `%` ở cuối file hoặc URL Redis không đúng, sửa lại:
   ```bash
   docker compose exec backend bash -c "cat > /home/frappe/frappe-bench/sites/common_site_config.json << 'EOF'
   {
    \"db_host\": \"db\",
    \"db_port\": 3306,
    \"redis_cache\": \"redis://redis-cache:6379\",
    \"redis_queue\": \"redis://redis-queue:6379\",
    \"redis_socketio\": \"redis://redis-socketio:6379\",
    \"socketio_port\": 9000
   }
   EOF"
   ```

4. **Kiểm tra file apps.json**:
   ```bash
   docker compose exec backend bash -c "cat /home/frappe/frappe-bench/sites/apps.json"
   ```
   
   Nếu thấy ký tự `%` ở cuối file, sửa lại:
   ```bash
   docker compose exec backend bash -c "cat > /home/frappe/frappe-bench/sites/apps.json << 'EOF'
   {
       \"frappe\": {
           \"resolution\": {
               \"commit_hash\": null,
               \"branch\": null
           },
           \"required\": [],
           \"idx\": 1,
           \"version\": \"15.72.0\"
       },
       \"erpnext\": {
           \"is_repo\": true,
           \"resolution\": {
               \"commit_hash\": \"de03618b0916f39905ea10c85f2c087b53deb2da\",
               \"branch\": \"v15.66.0\"
           },
           \"required\": [],
           \"idx\": 2,
           \"version\": \"15.66.0\"
       },
       \"go_eat_zavn\": {
           \"is_repo\": true,
           \"resolution\": {
               \"commit_hash\": \"b19eb65c115c313f043dc0453f5e0e4f2a881bd8\",
               \"branch\": \"develop\"
           },
           \"required\": [],
           \"idx\": 3,
           \"version\": \"0.0.1\"
       }
   }
   EOF"
   ```

5. **Tạo file apps.txt trong thư mục site**:
   ```bash
   docker compose exec backend bash -c "echo -e 'frappe\nerpnext\ngo_eat_zavn' > /home/frappe/frappe-bench/sites/erp.goeat.com/apps.txt"
   ```

6. **Restart các container để áp dụng thay đổi**:
   ```bash
   docker compose restart
   ```

### 2.2. Lỗi 404 khi truy cập site

#### Nguyên nhân
Lỗi này xảy ra khi truy cập site qua localhost:8080 thay vì hostname đã cấu hình trong Nginx.

#### Kiểm tra và khắc phục

1. **Kiểm tra cấu hình Nginx**:
   ```bash
   docker compose exec frontend bash -c "cat /etc/nginx/conf.d/frappe.conf"
   ```

2. **Kiểm tra file hosts**:
   ```bash
   cat /etc/hosts
   ```
   
   Đảm bảo có dòng sau:
   ```
   127.0.0.1 erp.goeat.com
   ```

3. **Truy cập site qua hostname**:
   ```
   http://erp.goeat.com:8080
   ```

## 3. Cấu trúc app đã tạo

App `go_eat_zavn` sẽ có cấu trúc thư mục như sau:
```
go_eat_zavn/
├── go_eat_zavn/
│   ├── __init__.py
│   ├── config/
│   │   ├── __init__.py
│   │   ├── desktop.py
│   │   └── docs.py
│   ├── hooks.py
│   ├── modules.txt
│   ├── patches.txt
│   └── templates/
├── setup.py
└── README.md
```

## 4. Các bước tiếp theo

Sau khi tạo app thành công, bạn có thể:

1. **Tạo Custom DocTypes**:
   ```bash
   docker compose exec backend bash -c "bench make-doctype 'Custom Item' --module 'Go Eat ZAVN'"
   ```

2. **Tạo Custom Pages**:
   ```bash
   docker compose exec backend bash -c "bench new-page 'Custom Dashboard' --module 'Go Eat ZAVN'"
   ```

3. **Tùy chỉnh Theme và Logo**:
   - Tạo thư mục public trong app
   - Thêm file CSS/JS tùy chỉnh

4. **Tạo Custom Reports**:
   - Sử dụng Report Builder hoặc tạo custom reports

## 5. Lưu ý quan trọng

1. Luôn kiểm tra định dạng các file JSON và cấu hình để tránh lỗi.
2. Khi gặp lỗi, kiểm tra logs của các container để xác định nguyên nhân.
3. Sử dụng hostname đã cấu hình (erp.goeat.com:8080) thay vì localhost:8080 để truy cập site.
4. Backup thường xuyên để tránh mất dữ liệu.

## 6. Tham khảo

- [Frappe Framework Documentation](https://frappeframework.com/docs)
- [ERPNext Documentation](https://docs.erpnext.com)
- [Frappe Docker Repository](https://github.com/frappe/frappe_docker) 
