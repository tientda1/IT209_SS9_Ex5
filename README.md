# Bài 5: Khởi chạy dịch vụ đơn giản với Docker Compose

Khởi chạy một dịch vụ web Nginx bằng Docker Compose CLI v2 thay vì gõ lệnh `docker run`.

## Cấu trúc

```
ex5/
├── docker-compose.yml   # Định nghĩa dịch vụ web
└── README.md
```

## Nội dung tệp

**docker-compose.yml**

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8082:80"
```

## Giải thích từng phần (và lý do)

- `services:` — khối gốc liệt kê các dịch vụ (container) mà Compose sẽ quản lý. Không dùng từ khoá `version:` vì từ Compose spec (CLI v2) đã bỏ, thêm vào sẽ bị cảnh báo `obsolete`.
- `web:` — tên dịch vụ do ta đặt. Compose dùng tên này để đặt tên container (mặc định `<thư-mục>_web_1` hoặc `<project>-web-1`) và để tham chiếu giữa các dịch vụ. Không cần khai báo `container_name` vì Compose đã tự sinh tên ổn định.
- `image: nginx:alpine` — dùng lại image có sẵn thay vì `build`, phù hợp vì bài này chỉ cần chạy Nginx không kèm nội dung tùy biến. Bản `alpine` giúp image nhỏ, nhẹ.
- `ports: - "8082:80"` — map cổng `8082` trên máy host vào cổng `80` của container (định dạng `host:container`), cùng ý nghĩa với cờ `-p 8082:80` trong `docker run`.
  - **Vì sao để trong ngoặc kép?** YAML có thể hiểu nhầm `8082:80` là giá trị dạng số/giờ (`sexagesimal`); đặt trong dấu `"..."` buộc nó là chuỗi, tránh lỗi map sai cổng.

**Vì sao dùng Compose thay `docker run`?**

- Khai báo cấu hình trong file thay vì gõ lệnh dài, dễ đọc và version-control cùng mã nguồn.
- Quản lý vòng đời cả cụm bằng một lệnh: `up` / `ps` / `down` — thay vì nhớ `docker run`, `docker ps`, `docker stop`, `docker rm`.
- YAML dùng **2 space** để thụt lùi, **không dùng Tab** (Tab làm YAML báo lỗi cú pháp).

## Các bước khởi chạy và kiểm tra

```bash
# 1. Khởi chạy dịch vụ ở chế độ nền
docker compose up -d

# 2. Xem trạng thái dịch vụ
docker compose ps

# 3. Kiểm tra nội dung trang Nginx
curl http://localhost:8082

# 4. Hạ dịch vụ và dọn tài nguyên
docker compose down
```

## Kết quả mong đợi

`docker compose ps` hiển thị service `web` đang chạy (trạng thái `running`/`Up`) và cổng `8082`:

```
NAME           IMAGE          COMMAND                  SERVICE   CREATED          STATUS          PORTS
ex5-web-1      nginx:alpine   "/docker-entrypoint.…"   web       5 seconds ago    Up 4 seconds    0.0.0.0:8082->80/tcp
```

`curl http://localhost:8082` trả về trang chào mừng mặc định của Nginx (`Welcome to nginx!`).
