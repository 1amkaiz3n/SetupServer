# Setup Raspberry pi

## Wifi

```bash
sudo vi /etc/wpa_supplicant/wpa_supplicant.conf
```

```bash
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=ID

network={
    ssid="<SSID>"
    psk="<PASSWORD>"
    scan_ssid=1
}

```
