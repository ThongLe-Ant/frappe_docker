# Hướng dẫn cấu hình UTF-8 cho MariaDB trong ERPNext Docker

## Vấn đề
Khi sử dụng ERPNext với tiếng Việt, có thể gặp vấn đề hiển thị các ký tự đặc biệt hoặc dấu tiếng Việt do MariaDB chưa được cấu hình đúng với UTF-8.

## Giải pháp
Để khắc phục, cần cấu hình MariaDB sử dụng UTF-8 đầy đủ (utf8mb4) thay vì utf8mb3 mặc định.

## Các bước thực hiện

### 1. Kiểm tra cấu hình hiện tại
```bash
docker compose exec db bash -c "mysql -u root -p<password> -e \"SHOW VARIABLES LIKE '%char%'; SHOW VARIABLES LIKE '%collation%';\""
```

### 2. Tạo file cấu hình UTF-8 cho MariaDB
Tạo file `my-custom.cnf` với nội dung:
```
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
```

### 3. Thêm cấu hình vào container
Có hai cách:

#### Cách 1: Thêm volume trong docker-compose.override.yml
```yaml
services:
  db:
    volumes:
      - ./my-custom.cnf:/etc/mysql/conf.d/my-custom.cnf:ro
```

#### Cách 2: Tạo file cấu hình trực tiếp trong container
```bash
docker compose exec db bash -c "echo '[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect=\"SET NAMES utf8mb4\"' > /etc/mysql/conf.d/utf8mb4.cnf"
```

### 4. Khởi động lại container MariaDB
```bash
docker compose restart db
```

### 5. Kiểm tra cấu hình sau khi áp dụng
```bash
docker compose exec db bash -c "mysql -u root -p<password> -e \"SHOW VARIABLES LIKE '%char%'; SHOW VARIABLES LIKE '%collation%';\""
```

Kết quả mong muốn:
```
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8mb3                    |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8mb4_unicode_ci |
| collation_database   | utf8mb4_unicode_ci |
| collation_server     | utf8mb4_unicode_ci |
+----------------------+--------------------+
```

### 6. Khởi động lại các container khác
```bash
docker compose restart
```

## Lưu ý
- Nếu sử dụng cách 1, cần đảm bảo file cấu hình có quyền đọc cho tất cả người dùng (chmod 644)
- Cấu hình `utf8mb4` hỗ trợ đầy đủ các ký tự Unicode 4 byte, bao gồm emoji và các ký tự đặc biệt
- Sau khi cấu hình, các thông báo lỗi và nội dung trong ERPNext sẽ hiển thị đúng tiếng Việt 
