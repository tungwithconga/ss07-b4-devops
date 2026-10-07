# Bài 4: Cấu hình Reverse Proxy Nginx cho ứng dụng Spring Boot

## Mục tiêu
- Cấu hình Nginx làm Reverse Proxy.
- Phục vụ trang tĩnh tại `/`.
- Chuyển tiếp `/api/` đến Spring Boot chạy tại port `8082`.

## Cấu hình Spring Boot

`application.properties`:

```properties
spring.application.name=ss07-b4
server.port=8082
```

API kiểm tra backend:

```java
@RestController
public class HealthController {
    @GetMapping("/health")
    public String health() {
        return "Spring Boot Backend is running!";
    }
}
```

Chạy backend trên Ubuntu WSL:

```bash
./gradlew bootRun
```

## Cấu hình Nginx

File: `/etc/nginx/sites-available/spring-proxy.conf`

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name _;
    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8082/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Kích hoạt cấu hình:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/spring-proxy.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Kết quả `nginx -t`:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

## Kiểm tra

Spring Boot trực tiếp:

```bash
curl -i http://127.0.0.1:8082/health
```

Kết quả: `HTTP/1.1 200` và `Spring Boot Backend is running!`.

Reverse Proxy qua Nginx:

```bash
curl -i http://localhost/api/health
```

Kết quả: `HTTP/1.1 200`, server là Nginx và nội dung trả về từ Spring Boot.

Trang tĩnh:

```bash
curl -I http://localhost/
```

Kết quả: `HTTP/1.1 200 OK`.

## Ảnh kết quả

![Kết quả kiểm tra Reverse Proxy](images/ex4-result.png)

## Kết luận
Nginx đã phục vụ thành công trang tĩnh tại `/` và chuyển tiếp `/api/*` tới Spring Boot tại `127.0.0.1:8082` mà không xảy ra lỗi `502 Bad Gateway`.
