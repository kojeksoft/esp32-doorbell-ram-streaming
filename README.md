Copy seluruh isi berikut ke file baru README.md di root repository (bahasa Inggris):
Markdown
# ESP32 Doorbell CCTV - RAM Streaming Version

ESP32-based smart doorbell that captures a JPEG snapshot from an IP CCTV camera when the doorbell is pressed, streams the image directly from the camera through RAM, and uploads it to a remote Photo Server without storing the JPEG on local flash (no SPIFFS). This reduces flash wear and shortens the response path for the bell event.

This repository contains the **RAM streaming version** of the project.

## Architecture

```text
Doorbell
   │
   ▼
ESP32 (monitor ADC) --- triggers ---> stream flow:
   │
   ▼
IP CCTV Camera (Digest auth)
   │
   │ JPEG snapshot (streamed)
   ▼
ESP32 RAM Buffer (no local file)
   │
   │ multipart/form-data (HTTP) streamed to
   ▼
Photo Server
   │
   ▼
WhatsApp Gateway (or other notifier)
   │
   ▼
WhatsApp / Push Notification
How It Works
Doorbell press is detected by the ESP32 ADC monitoring task.
ESP32 requests a JPEG snapshot from the IP CCTV camera (HTTP Digest Authentication supported).
The JPEG is streamed through RAM (no full-file write to SPIFFS).
ESP32 streams the multipart upload to the backend Photo Server while reading the camera stream.
If the server returns HTTP 200, the event is considered successful. If upload fails, no local copy is stored (design tradeoff).
The backend can forward the photo via WhatsApp Gateway or other notification channel.
Hardware
ESP32-WROOM-32 / ESP32-CAM module
Doorbell / push button (ADC or GPIO depending on wiring)
IP CCTV camera that supports snapshot endpoint (e.g. /cgi-bin/snapshot.cgi?channel=1)
Wi‑Fi network
Software
PlatformIO or Arduino Framework
WiFi
HTTPClient
WiFiClientSecure (optional, for HTTPS)
mbedTLS MD5 (for Digest auth)
(Optional) SPIFFS fallback code if you enable offline storage
Project Structure
Text
esp32-doorbell-ram-streaming/
├── src/
│   ├── doorbell_streaming.ino
│   ├── config.h
│   └── secrets.example.h
├── .github/
│   └── workflows/ci.yml
├── .gitignore
└── README.md
Differences vs SPIFFS Version
SPIFFS version: the camera JPEG is saved to SPIFFS temporarily before upload; easier to retry but causes flash writes.
RAM streaming version (this repo): the JPEG is streamed through RAM directly to the server — no SPIFFS usage by default. Faster and reduces flash wear, but if upload fails there is no persisted copy to retry later.
Quick Start
Copy src/secrets.example.h to src/secrets.h and fill your local values (WiFi, camera, backend host, WhatsApp phone number).
Open the repository in VS Code (recommended).
Choose build environment: PlatformIO (recommended) or Arduino CLI/IDE.
Build and flash to your ESP32/ESP32-CAM.
Open serial monitor at 115200 to observe WiFi, NTP and upload diagnostics.
PlatformIO Example
Create a platformio.ini in project root or use the PlatformIO UI:

INI
[env:esp32dev]
platform = espressif32
board = esp32dev         ; or change to specific camera board e.g. esp32cam
framework = arduino
monitor_speed = 115200
build_flags = -D ENABLE_RAM_STREAMING
src_dir = src
Build:

bash
pio run
Upload:

bash
pio run -t upload
Arduino CLI Example
Install Arduino CLI and ESP32 core:

bash
arduino-cli core update-index
arduino-cli core install esp32:esp32
Compile:

bash
arduino-cli compile -b esp32:esp32:esp32dev src/doorbell_streaming.ino
Upload (replace <PORT> with your device port):

bash
arduino-cli upload -p <PORT> -b esp32:esp32:esp32dev src/doorbell_streaming.ino
Configuration
Copy:

Text
src/secrets.example.h
to:

Text
src/secrets.h
and edit values:

C++
#define WIFI_SSID "YOUR_WIFI_SSID"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"

#define CAMERA_HOST "YOUR_CAMERA_IP"
#define CAMERA_USER "YOUR_CAMERA_USER"
#define CAMERA_PASSWORD "YOUR_CAMERA_PASSWORD"

#define BLUE_SERVER_HOST "your-photo-server.example"
#define BLUE_SERVER_PORT 80
#define BLUE_SERVER_PATH "/send_direct"

#define WA_PHONE "62812xxxxxxx"
#define WA_MESSAGE "Doorbell snapshot"
Make sure src/secrets.h is ignored by Git (check .gitignore) and do not commit it.

Notes & Tradeoffs
Pros:
Lower flash wear (no SPIFFS writes).
Lower latency from button press to upload start.
Cons:
If upload fails due to network or server error, no persistent local copy remains for retries.
If camera or server are slow, you must ensure timeouts and buffer sizes are tuned.
Recommendations:
Consider adding a small SPIFFS fallback or minimal retry queue if operating in flaky networks.
Use HTTPS (WiFiClientSecure) for backend if possible.
Tune IO_BUFFER_SIZE and timeouts for your camera and network.
Testing
Verify camera snapshot via browser or curl first (confirm Digest auth behavior).
Run device and watch serial logs. The firmware prints diagnostic lines for camera GET steps and upload flow.
Provide a staging server that accepts multipart/form-data:
fields: phone, message, upfile (file bytes)
Security
Keep src/secrets.h local and out of version control.
Use HTTPS for backend endpoints when possible.
Do not hardcode production keys in code.
Future Development
Optional SPIFFS fallback: write snapshot to flash when streaming/upload fails, retry later.
Add persistent queue metadata (e.g., small JSON queue) to retry uploads across reboots.
Add configurable retries and backoff policy.
Add web UI/dashboard for live status and logs.
Status
v1 - RAM Streaming: Initial implementation (no SPIFFS)
