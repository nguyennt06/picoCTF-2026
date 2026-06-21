<img width="911" height="620" alt="image" src="https://github.com/user-attachments/assets/4267d0ca-37ce-4a3f-9636-b5cfd06f33f2" />

\- Mở đầu bài CTF, có vẻ như ta cần ở một địa điểm nhất định để có thể lấy flag

\- Tải và xem file `nginx.conf` được cho sẵn:

```
load_module /usr/lib/nginx/modules/ngx_http_geoip2_module.so;

worker_processes 1;
events { worker_connections 1024; }

http {
    include       mime.types;
    default_type  application/octet-stream;

    geoip2 /etc/nginx/GeoLite2-Country.mmdb {
        auto_reload 5m;
        $geoip2_data_country_code default=ZZ country iso_code;
    }

    upstream north {
        server 127.0.0.1:8000;
    }

    upstream south {
        server 127.0.0.1:9000;
    }

    server {
        listen 80;

        location / {
            if ($geoip2_data_country_code = IS) {
                proxy_pass http://south;
            }

            proxy_pass http://north;
        }
    }
}

```

\- Tìm hiểu về `$geoip2_data_country_code = IS`, biết được đây là một biến trong cấu hình Nginx nhằm xác định mã quốc gia của người dùng truy cập là Iceland

$\implies$ Gần như ta cần fake IP sang Iceland để truy cập được vào route `http://south`

\- Nhận thấy ProtonVPN, EonVPN, 1.1.1.1 và các ứng dụng cho phép đổi IP khác đều không có Iceland, Tor Browser hiện lên như một lựa chọn duy nhất
- Ta hoàn toàn có thể sử dụng Tor qua CLI Linux: `sudo apt install tor`
- Nhưng tiện mình đã tải sẵn Tor Browser nên bài này sẽ khai thác theo hướng sử dụng trình duyệt

\- Truy cập file `C:\Users\xxxx\Desktop\Tor Browser\Browser\TorBrowser\Data\Tor\torrc` (Tor Run Config - file cấu hình chính của mạng lưới Tor), thêm vào cuối file:

```
ExitNodes {is}
StrictNodes 1
```
- ExitNodes {is}: là "Node Điểm Cuối" (tức là trạm dừng cuối cùng của mạng Tor trước khi kết nối đến trang web đích)
  - Địa chỉ IP của ExitNode chính là IP mà máy chủ web sẽ nhìn thấy.
- StrictNodes 1 (True - Bật) - Tor sẽ bắt buộc phải sử dụng Exit Node ở Iceland

<img width="1920" height="1022" alt="image" src="https://github.com/user-attachments/assets/2cdfb572-03e7-458d-9f25-edb37515778d" />


<img width="689" height="223" alt="image" src="https://github.com/user-attachments/assets/9a34c2a5-c765-4d35-a8e1-8e111a65c5c6" />
