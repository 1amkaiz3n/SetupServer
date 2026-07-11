# Cloudflare Tunnel + Nginx

Kenapa pake Nginx di depan? Biar lebih fleksibel — misal lu punya beberapa aplikasi di server yang sama, tinggal atur reverse proxy di Nginx. Tunnel cukup pointing ke Nginx aja, urusan routing domain ke app mana itu urusan Nginx.

## 1. Install Nginx

```bash
sudo apt update && sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

## 2. Buat Konfigurasi Nginx

Bikin file config buat tiap domain:

```bash
sudo vim /etc/nginx/sites-available/<SUBDOMAIN.DOMAIN.COM>
```

**Contoh — routing ke app di port 8000:**

```nginx
server {
    listen 80;
    server_name <SUBDOMAIN.DOMAIN.COM>;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Contoh — routing static file:**

```nginx
server {
    listen 80;
    server_name <STATIC.DOMAIN.COM>;

    root /var/www/<PROYEK>;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Contoh — routing ke beberapa app beda path:**

```nginx
server {
    listen 80;
    server_name <APP.DOMAIN.COM>;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
    }

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
    }
}
```

## 3. Aktifkan Config

```bash
sudo ln -s /etc/nginx/sites-available/<SUBDOMAIN.DOMAIN.COM> /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## 4. Config Tunnel — Pointing ke Nginx

Ubah config tunnel (`~/.cloudflared/<NAMA_APLIKASI>.yml`) biar ngarah ke Nginx di port 80, bukan langsung ke app:

```yaml
tunnel: <NAMA_TUNNEL>
credentials-file: /home/<USER>/.cloudflared/<UUID_TUNNEL>.json

ingress:
  - hostname: <SUBDOMAIN.DOMAIN.COM>
    service: http://localhost:80

  - service: http_status:404
```

Restart tunnel:

```bash
sudo systemctl restart cloudflared-<NAMA_APLIKASI>
```

## 5. Service Aplikasi (Opsional)

Kalo app lu pake systemd, service-nya tetap jalan sendiri kayak biasa. Nginx tinggal proxy ke port localhost yang udah ditentuin.

## Arsitektur Akhir

```
Client --> Cloudflare --> Cloudflare Tunnel --> Nginx (port 80) --> App 1 (port 8000)
                                                                  --> App 2 (port 3000)
                                                                  --> Static files (/var/www/)
```

## Keuntungan

- **Multi-app** — 1 tunnel bisa serve banyak aplikasi beda port/path
- **Static file** — Nginx lebih cepat serving static dibanding app framework
- **Security** — App gak perlu exposed ke public, cukup localhost
- **SSL** — Cloudflare udah handle SSL, Nginx cuma terima HTTP biasa
- **Load balancing** — Bisa tambah upstream multiple instance kalo perlu
