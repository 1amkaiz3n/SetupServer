# Deploy Aplikasi dengan Cloudflare Tunnel & Systemd

## BUAT TUNNEL

```bash
cloudflared tunnel create <NAMA_TUNNEL>
```

## SET DNS TUNNEL

```bash
cloudflared tunnel route dns <NAMA_TUNNEL> <SUBDOMAIN.DOMAIN.COM>
```

## SET CREDENTIALS FILE

```bash
sudo vi ~/.cloudflared/<NAMA_APLIKASI>.yml
```

```yaml
# ~/.cloudflared/<NAMA_APLIKASI>.yml
tunnel: <NAMA_TUNNEL>
credentials-file: /home/<USER>/.cloudflared/<UUID_TUNNEL>.json

ingress:
  - hostname: <SUBDOMAIN.DOMAIN.COM>
    service: http://localhost:<PORT_APLIKASI>

  - service: http_status:404
```

## BUAT SERVICE CLOUDFLARED

```bash
sudo vi /etc/systemd/system/cloudflared-<NAMA_APLIKASI>.service
```

```ini
## /etc/systemd/system/cloudflared-<NAMA_APLIKASI>.service
[Unit]
Description=Cloudflare Tunnel: <NAMA_TUNNEL>
After=network.target

[Service]
Type=simple
User=<USER>
ExecStart=/usr/bin/cloudflared tunnel --config /home/<USER>/.cloudflared/<NAMA_APLIKASI>.yml run
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

## RELOAD SYSTEMD

```bash
sudo systemctl daemon-reload
sudo systemctl enable cloudflared-<NAMA_APLIKASI>
sudo systemctl start cloudflared-<NAMA_APLIKASI>
```

## BUAT SERVICE APLIKASI

```bash
sudo vim /etc/systemd/system/<NAMA_APLIKASI>.service
```

```ini
# /etc/systemd/system/<NAMA_APLIKASI>.service
[Unit]
Description=<NAMA_APLIKASI>
After=network.target

[Service]
User=<USER>
Group=<USER>
WorkingDirectory=/home/<USER>/<PATH_PROYEK>
Environment="PATH=/home/<USER>/<PATH_PROYEK>/<NAMA_VENV>/bin"
ExecStart=/home/<USER>/<PATH_PROYEK>/<NAMA_VENV>/bin/gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker --bind 0.0.0.0:<PORT_APLIKASI>
Restart=always
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

## Logs

```bash
sudo systemctl daemon-reload
sudo systemctl enable <NAMA_APLIKASI>
sudo systemctl start <NAMA_APLIKASI>
sudo journalctl -u <NAMA_APLIKASI>.service -f
```
