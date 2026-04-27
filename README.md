# Smart Safety IoT System

## Real Hardware Diagram

> Sơ đồ bên dưới dùng ảnh thật linh kiện trong project.

![System Diagram](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/diagram%201.jpg)

---

## Components Used

### 1. ESP32 WROOM 32 Type C 38Pin

![ESP32](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/ESP32%20WROOM%2032%20Type%20C%2038Pin.webp)

### 2. ESP32-CAM

![ESP32-CAM](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/esp32-cam.webp)

### 3. Buck Converter (Step Down Module)

![Buck Converter](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/module%20giam%20ap.webp)

### 4. USB TTL Programmer

![USB TTL](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/usb.webp)

### 5. Adapter Power Supply

![Adapter](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/adapter.webp)

---

# System Overview

## ESP32 WROOM 32 Type C 38Pin

- Read temperature from MLX90614
- Send temperature data to ESP32-CAM through UART

## ESP32-CAM

- Receive temperature via UART
- Run camera webserver
- Show realtime temperature on web
- Alert when temperature > 37.5°C

👉 Communication between both boards uses **UART only**

---

# Architecture

```text
MLX90614 -> ESP32 ----UART----> ESP32-CAM ----WiFi----> Web Browser

# Wiring Diagram

## 1. MLX90614 → ESP32

| MLX90614 | ESP32  |
| -------- | ------ |
| VIN      | 3.3V   |
| GND      | GND    |
| SDA      | GPIO21 |
| SCL      | GPIO22 |

---

## 2. ESP32 ↔ ESP32-CAM via UART

| ESP32        | ESP32-CAM |
| ------------ | --------- |
| GPIO17 (TX2) | U0R       |
| GPIO16 (RX2) | U0T       |
| GND          | GND       |

> TX → RX, RX → TX

---

## 3. ESP32-CAM Power

| ESP32-CAM | Power |
| --------- | ----- |
| 5V        | 5V    |
| GND       | GND   |

---

# Important Notes

When running real system:

* Remove USB TTL from ESP32-CAM
* UART0 port will be used for receiving data from ESP32

---

# ESP32 Code (Read MLX90614 + Send UART)

```cpp
#include <Wire.h>
#include <Adafruit_MLX90614.h>

Adafruit_MLX90614 mlx;
HardwareSerial camSerial(2);

void setup() {
  Serial.begin(115200);
  Wire.begin(21,22);
  mlx.begin();

  camSerial.begin(115200, SERIAL_8N1, 16, 17);
}

void loop() {
  float temp = mlx.readObjectTempC();

  Serial.print("Send Temp: ");
  Serial.println(temp);

  camSerial.println(temp);

  delay(1000);
}
```

---

# ESP32-CAM Code (UART + Camera Webserver)

```cpp
#include "esp_camera.h"
#include <WiFi.h>
#include <WebServer.h>

const char* ssid = "YOUR_WIFI";
const char* password = "YOUR_PASS";

WebServer server(80);
String tempValue = "--";

void setup() {
  Serial.begin(115200);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) delay(500);

  server.begin();
}

void loop() {
  server.handleClient();

  while (Serial.available()) {
    tempValue = Serial.readStringUntil('\n');
    tempValue.trim();
  }
}
```

---

# How To Use

## Step 1

Upload ESP32 code.

## Step 2

Upload ESP32-CAM code using USB TTL.

## Step 3

Connect UART:

| ESP32  | ESP32-CAM |
| ------ | --------- |
| GPIO17 | U0R       |
| GPIO16 | U0T       |
| GND    | GND       |

## Step 4

Power on both boards.

## Step 5

Open browser using ESP32-CAM IP address.

Example:

```text
http://192.168.1.150
```

---

# Common Problems

## Garbage Characters

* Wrong baud rate
* USB TTL still connected

## No Temperature Data

* TX RX reversed
* Missing shared GND

## Camera Fail

* Weak power supply
* Wrong board selected

## Continuous Reset

* Weak USB power

Use **5V 2A adapter**

## Cannot Open Webpage

* Wrong WiFi password
* Different network

---

# Project Result

* Realtime temperature monitoring
* Live camera view
* High temperature warning
* UART communication system
* Smart IoT safety solution

```
```
