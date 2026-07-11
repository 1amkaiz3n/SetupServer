# Setup Cloudflare Tunnel

1. Login ke Cloudflare

```bash
cloudflared tunnel login
```

2. Buat tunnel

```bash
cloudflared tunnel create <nama tunnel>
```


3. Buat file konfigurasi

```bash
mkdir -p ~/.cloudflared

vim ~/.cloudflared/config.yml
```

**Contoh :**

```yaml
tunnel: UUID_TUNNEL
credentials-file: /home/USERNAME/.cloudflared/UUID_TUNNEL.json

ingress:
  - hostname: sub.domain.com
    service: http://localhost:8000

  - service: http_status:404
```

4. Hubungkan DNS

```bash
cloudflared tunnel route dns <nama tunnel> sub.domain.com
```

5. Jalankan tunnel

```bash
cloudflared tunnel run <nama tunnel>
```