# Bài 1: Khảo sát FHS và Phân quyền File/Folder nâng cao

## Mục tiêu

- Thành thạo việc tạo lập thư mục và điều hướng trong cấu trúc cây thư mục Linux FHS.
- Hiểu và áp dụng thành thạo cơ chế phân quyền tệp tin bằng số (octal) và bằng ký tự (symbolic).
- Nắm rõ cách thức thay đổi quyền sở hữu tệp tin và nhóm sở hữu.

---

## Bối cảnh

Triển khai một thư mục dùng chung cho ứng dụng web tại `/var/www/my-app`. Thư mục này bao gồm:

| Thư mục | Mục đích |
|---|---|
| `/var/www/my-app/public` | Chứa mã nguồn static công khai |
| `/var/www/my-app/logs` | Chứa nhật ký hệ thống (logs) bảo mật |

---

## Các lệnh thực hiện

### Bước 1: Tạo user non-root và đảm bảo group `www-data` tồn tại

```bash
# Tạo user non-root webdev (nếu chưa có)
sudo useradd -m -s /bin/bash webdev

# Kiểm tra group www-data (thường có sẵn trên Ubuntu khi cài Apache/Nginx)
getent group www-data
```

**Output:**

```
www-data:x:33:
```

---

### Bước 2: Tạo cấu trúc thư mục

```bash
sudo mkdir -p /var/www/my-app/public
sudo mkdir -p /var/www/my-app/logs
```

**Giải thích:**
- `sudo`: Thực thi với quyền root vì `/var/www/` là thư mục hệ thống.
- `mkdir -p`: Tạo thư mục và toàn bộ thư mục cha trung gian nếu chưa có (`--parents`).

---

### Bước 3: Phân quyền thư mục `public` (octal: 750)

```bash
sudo chmod 750 /var/www/my-app/public
```

**Giải thích quyền `750`:**

| Đối tượng | Octal | Quyền | Ký hiệu |
|---|---|---|---|
| Owner (Chủ sở hữu) | 7 | Đọc + Ghi + Truy cập | `rwx` |
| Group (Nhóm sở hữu) | 5 | Đọc + Truy cập | `r-x` |
| Others (Người khác) | 0 | Không có quyền gì | `---` |

> **Lưu ý:** Với thư mục, bit `x` (execute) nghĩa là quyền **truy cập vào bên trong** (traverse/enter) thư mục đó.

Lệnh tương đương dạng symbolic:

```bash
sudo chmod u=rwx,g=rx,o= /var/www/my-app/public
```

---

### Bước 4: Phân quyền thư mục `logs` (octal: 770)

```bash
sudo chmod 770 /var/www/my-app/logs
```

**Giải thích quyền `770`:**

| Đối tượng | Octal | Quyền | Ký hiệu |
|---|---|---|---|
| Owner (Chủ sở hữu) | 7 | Đọc + Ghi + Truy cập | `rwx` |
| Group (Nhóm sở hữu) | 7 | Đọc + Ghi + Truy cập | `rwx` |
| Others (Người khác) | 0 | Không có quyền gì | `---` |

Lệnh tương đương dạng symbolic:

```bash
sudo chmod u=rwx,g=rwx,o= /var/www/my-app/logs
```

---

### Bước 5: Thay đổi chủ sở hữu và nhóm sở hữu

```bash
sudo chown -R webdev:www-data /var/www/my-app
```

**Giải thích:**
- `chown`: Lệnh thay đổi chủ sở hữu (change owner).
- `-R`: Áp dụng đệ quy cho toàn bộ nội dung bên trong thư mục.
- `webdev`: User non-root bình thường (chủ sở hữu ứng dụng).
- `www-data`: Nhóm mặc định của Apache/Nginx trên Ubuntu, cho phép web server đọc file.

---

### Bước 6: Kiểm tra kết quả

```bash
ls -la /var/www/my-app
```

---

## Kết quả thực tế (`ls -la /var/www/my-app`)

```
total 16
drwxr-xr-x 4 webdev www-data 4096 Oct  6 14:35 .
drwxr-xr-x 3 root   root     4096 Oct  6 14:35 ..
drwxrwx--- 2 webdev www-data 4096 Oct  6 14:35 logs
drwxr-x--- 2 webdev www-data 4096 Oct  6 14:35 public
```

---

## Xác nhận kết quả đạt yêu cầu

| Thư mục | Quyền hiển thị | Octal | Chủ sở hữu | Nhóm | Đạt yêu cầu |
|---|---|---|---|---|---|
| `public` | `drwxr-x---` | `750` | `webdev` | `www-data` | ✅ |
| `logs` | `drwxrwx---` | `770` | `webdev` | `www-data` | ✅ |

---

## Giải thích lý thuyết FHS

**FHS (Filesystem Hierarchy Standard)** là chuẩn định nghĩa cấu trúc thư mục trên Linux:

| Thư mục | Mục đích |
|---|---|
| `/var` | Dữ liệu thay đổi thường xuyên (logs, cache, web data...) |
| `/var/www` | Thư mục gốc mặc định cho web server (Apache, Nginx) |
| `/var/www/my-app` | Thư mục ứng dụng web của dự án |
| `/var/www/my-app/public` | Static files công khai (HTML, CSS, JS, images) |
| `/var/www/my-app/logs` | Log files bảo mật của ứng dụng |

---

## Tổng hợp lệnh (chạy tuần tự)

```bash
# 1. Tạo user non-root và đảm bảo group www-data tồn tại
sudo useradd -m -s /bin/bash webdev
getent group www-data

# 2. Tạo cấu trúc thư mục
sudo mkdir -p /var/www/my-app/public
sudo mkdir -p /var/www/my-app/logs

# 3. Phân quyền cho public (750: rwxr-x---)
sudo chmod 750 /var/www/my-app/public

# 4. Phân quyền cho logs (770: rwxrwx---)
sudo chmod 770 /var/www/my-app/logs

# 5. Gán chủ sở hữu và nhóm
sudo chown -R webdev:www-data /var/www/my-app

# 6. Kiểm tra kết quả
ls -la /var/www/my-app
```
