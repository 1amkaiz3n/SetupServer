# SetupServer

Dokumentasi setup server dengan Cloudflare Tunnel dan Systemd.

## Daftar File

| File | Untuk Apa |
|------|-----------|
| `Setup-Server.md` | Panduan membuat & menjalankan Cloudflare Tunnel (login, buat tunnel, konfigurasi YAML, route DNS, run tunnel) |
| `cloudflare_systemd.md` | Panduan deploy aplikasi menggunakan Cloudflare Tunnel + Systemd (buat tunnel, set DNS, config YAML, service cloudflared, service aplikasi gunicorn, logs) |
| `Cloudflare-Nginx.md` | Panduan kombinasi Cloudflare Tunnel + Nginx (multi-app, static file, routing path, arsitektur akhir) |
| `SSL.md` | Panduan setup SSL — Origin Certificate (Cloudflare Full strict), Let's Encrypt, Self-Signed, rekomendasi pipeline |
