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