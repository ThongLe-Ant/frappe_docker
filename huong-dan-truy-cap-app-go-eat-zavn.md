# Hướng dẫn truy cập app Go Eat ZAVN

## 1. Truy cập ERPNext

Sau khi cài đặt app Go Eat ZAVN, bạn có thể truy cập ERPNext qua địa chỉ:

```
http://erp.goeat.com:8080
```

Đăng nhập với tài khoản Administrator và mật khẩu đã tạo.

## 2. Kiểm tra app đã cài đặt

Để kiểm tra app Go Eat ZAVN đã được cài đặt thành công, bạn có thể:

- Vào "Modules Def" trong phần "Developer" để xem danh sách các module
- Kiểm tra danh sách apps thông qua terminal:

```bash
bench --site erp.goeat.com list-apps
```

## 3. Khắc phục lỗi không hiển thị app trên giao diện

Nếu app Go Eat ZAVN không hiển thị trên giao diện, bạn có thể thực hiện các bước sau:

### 3.1. Kiểm tra file desktop.py

Đảm bảo file desktop.py trong thư mục config của app có nội dung đúng:

```python
from frappe import _

def get_data():
        return [
                {
                        "module_name": "Go Eat ZAVN",
                        "color": "grey",
                        "icon": "octicon octicon-home",
                        "type": "module",
                        "label": _("Go Eat ZAVN")
                }
        ]
```

### 3.2. Bật developer mode

```bash
bench --site erp.goeat.com set-config developer_mode 1
```

### 3.3. Xóa cache và build lại assets

```bash
bench --site erp.goeat.com clear-cache
bench --site erp.goeat.com build
```

### 3.4. Khởi động lại các container

```bash
docker compose restart
```

### 3.5. Kiểm tra modules.txt

Đảm bảo file modules.txt trong thư mục app có nội dung:

```
go_eat_zavn
```

## 4. Tạo DocType mới

Sau khi app hiển thị trên giao diện, bạn có thể tạo các DocType mới:

1. Vào module Go Eat ZAVN
2. Chọn "DocType" > "New"
3. Điền thông tin cho DocType mới và lưu

## 5. Phát triển app

Để phát triển app Go Eat ZAVN, bạn có thể:

1. Tạo các DocType mới
2. Tạo Page và Report
3. Viết các script tùy chỉnh
4. Tùy chỉnh giao diện

## 6. Cập nhật app

Khi có thay đổi trong code của app, bạn cần:

```bash
bench --site erp.goeat.com migrate
bench --site erp.goeat.com build
docker compose restart
```

## 7. Tài liệu tham khảo

- [Frappe Framework Documentation](https://frappeframework.com/docs)
- [ERPNext Documentation](https://docs.erpnext.com)
- [Frappe App Development Guide](https://frappeframework.com/docs/user/en/guides/app-development) 
