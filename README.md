ESP32-S3 + W5500 Ethernet (ESP-IDF)
====================

Firmware ESP-IDF untuk ESP32-S3 dengan modul Ethernet W5500 (SPI), lengkap dengan web UI untuk konfigurasi WiFi dan Ethernet (Static IP / DHCP) tanpa perlu re-flash firmware.

## Fitur

- **Ethernet W5500 via SPI** — koneksi kabel dengan dukungan **Static IP** maupun **DHCP**, otomatis fallback ke Static IP jika DHCP timeout.
- **WiFi Station + Access Point (AP)** — mode AP untuk provisioning awal, mode STA untuk konek ke jaringan WiFi yang ada.
- **Web-based configuration** — halaman web tertanam (embedded) untuk mengatur kredensial WiFi dan konfigurasi IP Ethernet secara live dari browser.
- **Simpan konfigurasi ke NVS** — kredensial WiFi dan konfigurasi IP Ethernet tersimpan permanen (persist setelah reboot).
- **OTA update** — update firmware lewat HTTP endpoint.
- **SNTP time sync** — sinkronisasi waktu otomatis setelah WiFi/Ethernet tersambung.
- **RGB LED status indicator** — memanfaatkan LED RGB onboard ESP32-S3 (GPIO 48) sebagai indikator status koneksi.
- **Tombol reset WiFi** — hapus kredensial WiFi tersimpan via tombol fisik.

## Hardware

| Komponen | Keterangan |
|---|---|
| MCU | ESP32-S3 |
| Ethernet PHY/MAC | WIZnet W5500 (interface SPI) |
| LED status | RGB LED onboard, GPIO 48 |

### Wiring W5500 (SPI2)

| Sinyal W5500 | GPIO ESP32-S3 (default) |
|---|---|
| MISO | GPIO 13 |
| MOSI | GPIO 11 |
| SCLK | GPIO 12 |
| CS | GPIO 10 |
| INT | GPIO 4 |
| RST | Tidak terhubung (-1) |

Pin-pin ini bisa disesuaikan di `main/ethernet_app.h` (`ETH_SPI_*_GPIO`) sesuai wiring board masing-masing.

## Struktur Project

```
.
├── main/
│   ├── main.c                 # Entry point aplikasi
│   ├── ethernet_app.c/h       # Driver & task Ethernet W5500 (Static IP/DHCP)
│   ├── wifi_app.c/h           # Task WiFi STA + AP
│   ├── http_server.c/h        # Web server + REST endpoint konfigurasi
│   ├── app_nvs.c/h            # Penyimpanan konfigurasi WiFi/Ethernet ke NVS
│   ├── sntp_time_sync.c/h     # Sinkronisasi waktu NTP
│   ├── rgb_led.c/h            # Kontrol LED RGB status
│   ├── wifi_reset_button.c/h  # Handler tombol reset WiFi
│   ├── tasks_common.h         # Konfigurasi stack size, priority, core untuk tiap task
│   ├── Kconfig.projbuild      # Opsi konfigurasi default (SSID/password) via menuconfig
│   └── webpage/                # Halaman web (HTML/CSS/JS) yang di-embed ke firmware
├── CMakeLists.txt
├── sdkconfig
└── dependencies.lock
```

## Prasyarat

- [ESP-IDF](https://github.com/espressif/esp-idf) v5.3.1 (lihat `dependencies.lock`)
- Target chip: `esp32s3`

## Build & Flash

```bash
# set target (sekali saja per project)
idf.py set-target esp32s3

# (opsional) atur SSID/password WiFi default lewat menuconfig
idf.py menuconfig
# -> Example Configuration -> WiFi SSID / WiFi Password

# build
idf.py build

# flash + monitor
idf.py -p <PORT> flash monitor
```

## Konfigurasi Jaringan

### Ethernet

Default konfigurasi didefinisikan di `main/ethernet_app.h`:

| Parameter | Default |
|---|---|
| Mode | DHCP (fallback ke Static jika timeout 15 detik) |
| Static IP | `192.168.0.101` |
| Gateway | `192.168.0.1` |
| Netmask | `255.255.255.0` |
| DNS | `8.8.8.8` |

Konfigurasi ini bisa diubah lewat web UI (disimpan ke NVS) atau diubah langsung di source sebelum build.

### WiFi Access Point

| Parameter | Default |
|---|---|
| SSID | `ESP32_AP` |
| Password | `password` |
| IP AP | `192.168.0.1` |
| Channel | 6 |

Setelah flashing, hubungkan ke AP `ESP32_AP` lalu buka `192.168.0.1` di browser untuk melakukan konfigurasi WiFi STA maupun Ethernet.

## Web Endpoint

Firmware mengekspos beberapa endpoint HTTP untuk keperluan web UI:

| Endpoint | Fungsi |
|---|---|
| `/wifiConnect.json` | Kirim kredensial WiFi untuk konek |
| `/wifiConnectStatus` | Status koneksi WiFi |
| `/wifiConnectInfo.json` | Info IP/SSID WiFi terkini |
| `/wifiDisconnect.json` | Putuskan koneksi WiFi |
| `/ethConnect.json` | Kirim/ubah konfigurasi koneksi Ethernet |
| `/ethConnectStatus` | Status koneksi Ethernet |
| `/ethConnectInfo.json` | Info IP Ethernet terkini |
| `/ethConfig.json` | Ambil/atur konfigurasi IP Ethernet (Static/DHCP) |
| `/ethDisconnect.json` | Putuskan Ethernet |
| `/localTime.json` | Waktu lokal hasil SNTP sync |
| `/OTAupdate`, `/OTAstatus` | Upload firmware baru & cek status OTA |

## Lisensi

This is a template application to be used with [Espressif IoT Development Framework](https://github.com/espressif/esp-idf).

Please check [ESP-IDF docs](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html) for getting started instructions.

*Code in this repository is in the Public Domain (or CC0 licensed, at your option.)
Unless required by applicable law or agreed to in writing, this
software is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
CONDITIONS OF ANY KIND, either express or implied.*


[MIT License](LICENSE) (c) 2025 Muhammad Jumi'at Mokhtar
