# SSL

Kombinasi Cloudflare Tunnel + Nginx — SSL udah dihandle Cloudflare. Tapi kalo mode SSL di dashboard Cloudflare diset **Full (strict)**, server lo butuh origin certificate.

## Opsi 1 — Origin Certificate (Cloudflare)

Biar mode Full (strict) aman, pake origin certificate dari Cloudflare.

**Generate di dashboard Cloudflare:**
SSL/TLS → Origin Server → Create Certificate

**Simpan cert & key di server:**

```bash
sudo mkdir -p /etc/ssl/cloudflare
# Copy paste isi certificate ke sini
sudo vim /etc/ssl/cloudflare/origin.pem
# Copy paste isi private key ke sini
sudo vim /etc/ssl/cloudflare/origin-key.pem
```

**Config Nginx pake origin cert:**

```nginx
server {
    listen 443 ssl;
    server_name <SUBDOMAIN.DOMAIN.COM>;

    ssl_certificate /etc/ssl/cloudflare/origin.pem;
    ssl_certificate_key /etc/ssl/cloudflare/origin-key.pem;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Jangan lupa redirect HTTP ke HTTPS:

```nginx
server {
    listen 80;
    server_name <SUBDOMAIN.DOMAIN.COM>;
    return 301 https://$host$request_uri;
}
```

Reload Nginx:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

## Opsi 2 — Let's Encrypt (Certbot)

Kalo app gak pake Cloudflare Tunnel (akses langsung), pake Let's Encrypt.

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d <SUBDOMAIN.DOMAIN.COM>
```

Auto-renewal udah otomatis dari certbot:

```bash
sudo certbot renew --dry-run
```

## Opsi 3 — Self-Signed (Internal/Development)

Buat local testing atau internal network.

```bash
sudo mkdir -p /etc/ssl/selfsigned
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/selfsigned/selfsigned.key \
  -out /etc/ssl/selfsigned/selfsigned.crt
```

Config Nginx:

```nginx
server {
    listen 443 ssl;
    server_name <IP_ATAU_HOSTNAME>;

    ssl_certificate /etc/ssl/selfsigned/selfsigned.crt;
    ssl_certificate_key /etc/ssl/selfsigned/selfsigned.key;

    location / {
        proxy_pass http://127.0.0.1:8000;
    }
}
```

## Recommended Setup (Full Pipeline)

```
Client ---HTTPS---> Cloudflare ---HTTPS---> Nginx (SSL) ---HTTP---> App (localhost)
```

Cloudflare diconfig **Full (strict)**, Nginx pake origin certificate dari Cloudflare.
