# Bài Tập 03 - Sử Dụng WordPress Để Tạo Website

**Sinh viên:** Nguyễn Tiến Đức  
**MSSV:** K225480106081  
**Lớp:** K58KTPM  
**Môn:** Phát triển ứng dụng với mã nguồn mở - TEE0421  

---

##  Link Website
 https://wordpress.nguyentienduc04.io.vn

---

##  Hướng dẫn triển khai từng bước

### Bước 1: SSH vào Ubuntu Server

```bash
ssh tienduc@172.28.197.231
```
<img width="1480" height="758" alt="image" src="https://github.com/user-attachments/assets/245487ff-ff89-43ae-9947-0351047f0d96" />


---

### Bước 2: Cập nhật hệ thống

```bash
sudo apt update && sudo apt upgrade -y
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/7a6f22d3-a3f5-47bd-b8d9-34954f8b0472" />


---

### Bước 3: Kiểm tra Docker

```bash
docker --version && docker compose version
```

<img width="1478" height="627" alt="image" src="https://github.com/user-attachments/assets/5a711173-6d8d-4ae5-9105-0456317669d1" />

---

### Bước 4: Tạo file docker-compose.yml

```bash
mkdir ~/wordpress && cd ~/wordpress
cat > docker-compose.yml << 'EOF'
services:
  mariadb:
    image: mariadb:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: wp_password
    volumes:
      - db_data:/var/lib/mysql
  phpmyadmin:
    image: phpmyadmin:latest
    restart: always
    ports:
      - "8181:80"
    environment:
      PMA_HOST: mariadb
      MYSQL_ROOT_PASSWORD: rootpassword
    depends_on:
      - mariadb
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "9000:80"
    environment:
      WORDPRESS_DB_HOST: mariadb:3306
      WORDPRESS_DB_NAME: wordpress_db
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wp_password
    depends_on:
      - mariadb
    volumes:
      - wp_data:/var/www/html
volumes:
  db_data:
  wp_data:
EOF
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/e7996574-d1ce-4270-885a-df8c98126593" />


---

### Bước 5: Chạy Docker Compose

```bash
docker compose up -d
```

<img width="1481" height="630" alt="image" src="https://github.com/user-attachments/assets/0a94a7ae-f09b-49f2-a02d-c5bde3735cef" />


>  **Lỗi gặp phải:** Port 8080 và 8000 đã bị chiếm bởi các container khác.  
> **Cách xử lý:** Đổi port phpMyAdmin sang `8181` và WordPress sang `9000`.

<img width="1486" height="628" alt="image" src="https://github.com/user-attachments/assets/1f0e4e79-5bb7-4008-b92d-068ceb39fd7f" />

---

### Bước 6: Kiểm tra container đang chạy

```bash
docker ps
```

<img width="1487" height="628" alt="image" src="https://github.com/user-attachments/assets/889efbab-412f-4f81-9293-5845cabfc4c6" />


---

### Bước 7: Truy cập WordPress và cài đặt

Mở trình duyệt: `http://172.28.197.231:9000`

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/54804978-f500-47d5-86b9-b91d1513f4e5" />


---

### Bước 8: Đăng nhập phpMyAdmin

Mở trình duyệt: `http://172.28.197.231:8181`  
- Tài khoản: `root`  
- Mật khẩu: `rootpassword`

<img width="1918" height="1073" alt="image" src="https://github.com/user-attachments/assets/f6737d0c-6484-448e-a413-ad0fcfc9bab9" />


---

### Bước 9: Cài đặt Cloudflare Tunnel

```bash
# Thêm repo Cloudflare
curl -L https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared any main" | sudo tee /etc/apt/sources.list.d/cloudflared.list
sudo apt update && sudo apt install cloudflared -y
```

>  **Lỗi gặp phải:** Không tải được file từ GitHub do bị chặn kết nối.  
> **Cách xử lý:** Dùng repo chính thức của Cloudflare thay vì GitHub.

<img width="1907" height="1078" alt="image" src="https://github.com/user-attachments/assets/dcd66b46-709c-488c-a4ef-8e44c25a82c1" />

---

### Bước 10: Tạo tunnel và cấu hình DNS

```bash
# Tạo tunnel
cloudflared tunnel create wordpress-tunnel

# Tạo file config
cat > ~/.cloudflared/config.yml << 'EOF'
tunnel: b06d0772-bcb3-4680-a578-b4396e2b1afc
credentials-file: /home/tienduc/.cloudflared/b06d0772-bcb3-4680-a578-b4396e2b1afc.json
ingress:
  - hostname: wordpress.nguyentienduc04.io.vn
    service: http://localhost:9000
  - service: http_status:404
EOF

# Tạo DNS record
cloudflared tunnel route dns wordpress-tunnel wordpress.nguyentienduc04.io.vn
```

<img width="1477" height="636" alt="image" src="https://github.com/user-attachments/assets/d2a2435a-84ef-4d90-819c-caf9808e8941" />


---

### Bước 11: Cài service tự động khởi động

```bash
sudo cp ~/.cloudflared/config.yml /etc/cloudflared/config.yml
sudo cloudflared service install
sudo systemctl enable --now cloudflared
sudo systemctl status cloudflared
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/5ad3434f-5cdd-425e-9301-14f9e74db088" />


---

### Bước 12: Cập nhật URL WordPress

```bash
docker exec wordpress-mariadb-1 mariadb -u wp_user -pwp_password wordpress_db -e \
"UPDATE wp_options SET option_value='https://wordpress.nguyentienduc04.io.vn' WHERE option_name='siteurl' OR option_name='home';"
```

<img width="1477" height="628" alt="image" src="https://github.com/user-attachments/assets/b96ae9e1-69e2-4a6f-9065-4e16c39df97e" />

---

### Bước 13: Website hoạt động

Truy cập: https://wordpress.nguyentienduc04.io.vn

<img width="1910" height="1078" alt="image" src="https://github.com/user-attachments/assets/20bd5879-0e51-476d-aa28-482dba86656c" />


---

##  Các bài viết trên WordPress

### Bài 1: Giới thiệu bản thân
![Bài viết 1](images/14_post1.png)

### Bài 2: Ngành KTPM tại TNUT
<img width="1453" height="810" alt="image" src="https://github.com/user-attachments/assets/3f61920e-4cb7-4914-bdd6-511e09761ae5" />


### Bài 3: Nhận xét WordPress
<img width="1456" height="820" alt="image" src="https://github.com/user-attachments/assets/edf5f8cc-b6f7-46d4-8558-0d5756c1dfdf" />


---

##  Tổng hợp lỗi gặp phải

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|-------------|------------|
| Port 8080 đã bị chiếm | Container khác dùng trước | Đổi sang port 8181, 9000 |
| Không tải được cloudflared từ GitHub | Mạng bị chặn | Dùng repo pkg.cloudflare.com |
| Website timeout khi truy cập domain | Tunnel chạy foreground, SSH ngắt thì tắt | Chạy background bằng `&`, cài systemd service |
| URL WordPress vẫn là IP nội bộ | WordPress lưu URL cũ trong database | Cập nhật bảng wp_options trong MariaDB |

---

##  Hướng dẫn thêm ảnh vào repo

1. Vào repo GitHub → nhấn **"Add file"** → **"Upload files"**
2. Tạo thư mục `images/` và upload các ảnh chụp màn hình
3. Đặt tên ảnh theo đúng tên trong README (01_ssh.png, 02_update.png,...)
