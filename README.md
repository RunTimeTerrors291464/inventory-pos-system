# Hướng dẫn chạy (bench start) các dự án trong container

Repo này chứa nhiều Frappe Bench (mỗi thư mục là một bench riêng):

- `firstsite/` → site chính: `library.localhost`
- `frappe-bench/` → các site: `d-code.localhost`, `development.localhost`
- `inventory-pos/` → site chính: `pos.localhost`

Bạn có thể chạy từng bench độc lập bằng lệnh `bench start` trong từng thư mục.

> Lưu ý: Tránh chạy nhiều bench cùng lúc trên cùng máy nếu chưa cấu hình cổng (ports) khác nhau, vì mặc định tất cả dùng port 8000/9000. Nếu cần chạy song song, hãy đổi `webserver_port` trong `sites/common_site_config.json` và cập nhật Procfile cho từng bench.

---

## Yêu cầu tối thiểu

- Python 3.10+ và Node.js (đã có sẵn trong dev container này)
- Redis Server (được `bench start` quản lý cho từng bench)
- Frappe Bench CLI (nếu bạn chạy ngoài container):

```bash
pip install frappe-bench
```

---

## Cấu hình hosts (bắt buộc)
Thêm các domain dev vào file hosts (Linux/macOS: `/etc/hosts`, Windows: `C:\\Windows\\System32\\drivers\\etc\\hosts`).

```text
127.0.0.1 library.localhost
127.0.0.1 d-code.localhost
127.0.0.1 development.localhost
127.0.0.1 pos.localhost
```

---

## Chạy từng bench

### 1) firstsite

```bash
cd firstsite
bench start
```

- Truy cập: http://library.localhost:8000
- Build assets (nếu giao diện không tải đúng):

```bash
bench build
```

- Migrate (khi cập nhật app/code):

```bash
bench --site library.localhost migrate
```

### 2) frappe-bench

```bash
cd frappe-bench
bench start
```

- Truy cập: http://d-code.localhost:8000 hoặc http://development.localhost:8000
- Chạy migrate cho từng site:

```bash
bench --site d-code.localhost migrate
bench --site development.localhost migrate
```

### 3) inventory-pos

```bash
cd inventory-pos
bench start
```

- Truy cập: http://pos.localhost:8000

---

## Thao tác thường dùng

- Kiểm tra tình trạng bench:

```bash
bench doctor
```

- Xoá cache, rebuild nhanh khi phát triển:

```bash
bench clear-cache
bench build
```

- Chạy chỉ web (tuỳ chọn):

```bash
bench serve
```

---

## Ghi chú bảo mật

- Đừng commit các file chứa thông tin nhạy cảm như `sites/*/site_config.json`, `sites/common_site_config.json`, file Redis ACL/CONF, logs, backup,… `.gitignore` đã được cấu hình để bỏ qua phần lớn các file này.

---

## Đẩy toàn bộ mã nguồn lên GitHub

1) Khởi tạo git (nếu chưa có):

```bash
git init
```

2) Kiểm tra `.gitignore` đã loại trừ secrets/logs. Thêm và commit:

```bash
git add .
git commit -m "chore: initial commit (multi-bench)"
```

3) Tạo repo trống trên GitHub rồi thêm remote và push:

- Qua SSH:

```bash
git remote add origin git@github.com:<your-username>/<your-repo>.git
git branch -M main
git push -u origin main
```

- Hoặc HTTPS:

```bash
git remote add origin https://github.com/<your-username>/<your-repo>.git
git branch -M main
git push -u origin main
```

> Mẹo: Dùng SSH để tránh phải nhập token thường xuyên. Cấu hình khoá SSH bằng `ssh-keygen` và thêm khoá public vào GitHub.

---

## Xử lý sự cố nhanh

- Port 8000 đã bận: chạy một bench tại một thời điểm, hoặc đổi `webserver_port` trong `sites/common_site_config.json` và cập nhật Procfile.
- Lỗi Redis/Quyền truy cập: xoá pids và logs trước khi start lại:

```bash
# ví dụ trong một bench
rm -rf pids/*
rm -f logs/*.log
```

- Quyền file khi clone trên Linux: đảm bảo user hiện tại có quyền đọc/ghi; nếu cần `chmod -R u+rwX,g+rwX` trong thư mục bench.

---

Chúc bạn dev vui vẻ với Frappe/ERPNext!
