# DevOps Hackathon – Đề 001: Quản lý phòng Lab

## 1. Thông tin sinh viên

| Họ và tên | Mã sinh viên | Lớp | Tài khoản Linux | GitHub | Cổng Nginx |
|-----------|--------------|-----|-----------------|--------|------------|
| Hồ Khánh Linh | B24DTCN359 | CNTT3 | khanhlinh-cntt3 | [hokhanhlinh2006](https://github.com/hokhanhlinh2006) | 8081 |

---

## 2. Môi trường triển khai

- **Hệ điều hành:** Linux Ubuntu 22.04 LTS (x86_64)
- **Môi trường:** Cloud VPS (Host IP: `165.101.46.55`)
- **Web Server:** Nginx 1.18.0
- **Hệ thống quản lý phiên bản:** Git 2.34.1
- **Firewall:** UFW (Uncomplicated Firewall)
- **Trình duyệt & công cụ kiểm thử:** Google Chrome, cURL


## 4. Cấu hình Nginx

File cấu hình Virtual Host được đặt tại `nginx/khanhlinh-cntt3.conf`:

| Tham số trong template | Giá trị đã điền | Giải thích chi tiết |
|:---|:---|:---|
| `<PORT>` | `8081` | Cổng dịch vụ Nginx cá nhân được phân bổ, cấu hình listen cho cả IPv4 và IPv6 (`listen 8081; listen [::]:8081;`) |
| `<SERVER_NAME>` | `165.101.46.55` | Địa chỉ IP của máy chủ VPS dùng để định tuyến request |
| `<WEB_ROOT>` | `/var/www/devops-hackathon-de001-khanhlinh-cntt3/src` | Đường dẫn tuyệt đối trỏ tới thư mục chứa mã nguồn tĩnh `src/` |
| `<INDEX_FILE>` | `index.html` | Tệp tin mặc định được Nginx phục vụ khi người dùng truy cập trang chủ |
| `<TEN_TAI_KHOAN>` | `khanhlinh-cntt3` | Tên tài khoản Linux dùng để định danh file log riêng (`khanhlinh-cntt3.access.log` và `khanhlinh-cntt3.error.log`) |
| `<ALLOW_DIRECTIVE>` | `allow all;` | Chỉ thị cho phép tất cả các nguồn mạng bên ngoài truy cập vào location `/` |

---

## 5. Tường lửa UFW

Cấu hình UFW đảm bảo an toàn cho máy chủ và cho phép truy cập website:

1. **Cho phép cổng SSH (bắt buộc trước khi kích hoạt UFW):**
   ```bash
   sudo ufw allow 22/tcp
   ```
2. **Cho phép cổng dịch vụ website cá nhân:**
   ```bash
   sudo ufw allow 8081/tcp
   ```
3. **Kích hoạt và kiểm tra trạng thái UFW:**
   ```bash
   sudo ufw enable
   sudo ufw status verbose
   ```

**Kết quả mong đợi:**
- Status: `active`
- `22/tcp`: `ALLOW IN Anywhere` (kèm dòng IPv6)
- `8081/tcp`: `ALLOW IN Anywhere` (kèm dòng IPv6)

---

## 6. Các bước triển khai thực tế

### Bước 1: Chuẩn bị môi trường & Tài khoản Linux
```bash
# 1. Tạo tài khoản Linux mới có shell bash và primary group cùng tên
sudo useradd -m -s /bin/bash -U khanhlinh-cntt3

# 2. Thêm vào secondary group sudo
sudo usermod -aG sudo khanhlinh-cntt3

# 3. Đặt mật khẩu
sudo passwd khanhlinh-cntt3

# 4. Đăng nhập sang tài khoản mới
su - khanhlinh-cntt3

# 5. Cấu hình định danh Git
git config --global user.name "hokhanhlinh2006"
git config --global user.email "hokhanhlinh2006@gmail.com"
```

### Bước 2: Clone mã nguồn từ GitHub về Web Server
```bash
# Clone repository về thư mục triển khai chuẩn
sudo git clone https://github.com/hokhanhlinh2006/devops-hackathon-de001-khanhlinh-cntt3.git /var/www/devops-hackathon-de001-khanhlinh-cntt3

# Phân quyền sở hữu cho tài khoản cá nhân
sudo chown -R khanhlinh-cntt3:khanhlinh-cntt3 /var/www/devops-hackathon-de001-khanhlinh-cntt3

# Phân quyền chuẩn bảo mật Web Server (thư mục 755, file 644, tuyệt đối không dùng 777)
sudo find /var/www/devops-hackathon-de001-khanhlinh-cntt3 -type d -exec chmod 755 {} \;
sudo find /var/www/devops-hackathon-de001-khanhlinh-cntt3 -type f -exec chmod 644 {} \;
```

### Bước 3: Cấu hình và kích hoạt Nginx Virtual Host
```bash
# Sao chép cấu hình sang sites-available
sudo cp /var/www/devops-hackathon-de001-khanhlinh-cntt3/nginx/khanhlinh-cntt3.conf /etc/nginx/sites-available/khanhlinh-cntt3.conf

# Tạo symbolic link kích hoạt site
sudo ln -sf /etc/nginx/sites-available/khanhlinh-cntt3.conf /etc/nginx/sites-enabled/

# Gỡ bỏ site default để tránh chiếm dụng tài nguyên và xung đột
sudo rm -f /etc/nginx/sites-enabled/default

# Kiểm tra cú pháp và nạp lại cấu hình Nginx
sudo nginx -t
sudo systemctl reload nginx
```

---

## 7. Kiểm tra & Minh chứng

### 7.1. Tài khoản Linux
Kết quả kiểm tra lệnh `id` và `whoami`:
![Tài khoản Linux](screenshots/01-user.png)

### 7.2. Dịch vụ Nginx
Kiểm tra cú pháp cấu hình `sudo nginx -t` và trạng thái `active (running)` của Nginx:
![Trạng thái Nginx](screenshots/02-nginx.png)

### 7.3. Tường lửa UFW
Trạng thái tường lửa hiển thị cổng SSH `22/tcp` và cổng cá nhân `8081/tcp`:
![Tường lửa UFW](screenshots/03-ufw.png)

### 7.4. Giao diện Website trên trình duyệt
Website hoạt động chính xác tại địa chỉ `http://165.101.46.55:8081`:
![Giao diện Website](screenshots/04-website.png)

### 7.5. Lịch sử commit Git
Lịch sử Git rõ ràng với tối thiểu 4 commit theo chuẩn:
![Git log](screenshots/05-git-log.png)

### 7.6. Minh chứng cập nhật nội dung
Giao diện website sau khi thực hiện quy trình cập nhật lần 2:
![Cập nhật website](screenshots/06-update.png)

---

## 8. Quy trình cập nhật website

Quy trình cập nhật mã nguồn theo chuẩn DevOps không gây gián đoạn dịch vụ:
1. **Trên máy phát triển (Local Machine):**
   - Chỉnh sửa nội dung file `src/index.html` (thêm thông tin phiên bản / cập nhật).
   - Thực hiện commit và push lên GitHub:
     ```bash
     git add src/index.html
     git commit -m "feat: cap nhat noi dung website lan 2"
     git push origin main
     ```
2. **Trên máy chủ (Production Server):**
   - Di chuyển vào thư mục triển khai và kéo mã nguồn mới nhất:
     ```bash
     cd /var/www/devops-hackathon-de001-khanhlinh-cntt3
     git pull origin main
     ```
   - Do Nginx phục vụ file tĩnh trực tiếp từ thư mục `src/`, không cần restart hay reload Nginx.
3. **Kiểm tra kết quả:**
   - Truy cập lại `http://165.101.46.55:8081` để xác nhận nội dung đã thay đổi.

---

## 9. Sự cố gặp phải & Cách khắc phục

1. **Lỗi permission denied khi git pull trên server:**
   - *Nguyên nhân:* Thư mục `/var/www` mặc định thuộc sở hữu của `root`.
   - *Khắc phục:* Phân quyền lại thư mục triển khai về cho user `khanhlinh-cntt3` bằng lệnh `sudo chown -R khanhlinh-cntt3:khanhlinh-cntt3 /var/www/devops-hackathon-de001-khanhlinh-cntt3`.
2. **Lỗi xung đột cấu hình Nginx mặc định:**
   - *Nguyên nhân:* File cấu hình mặc định `/etc/nginx/sites-enabled/default` vẫn đang lắng nghe cổng 80 hoặc giữ cấu hình cũ.
   - *Khắc phục:* Xóa liên kết tượng trưng của file default bằng lệnh `sudo rm -f /etc/nginx/sites-enabled/default` và chạy `sudo systemctl reload nginx`.
