# Bản chuẩn lại hoàn toàn: ESP32 + MLX90614 + ESP32-CAM giao tiếp **chỉ qua UART**

Bạn yêu cầu đúng hướng: **chỉ dùng UART** để truyền dữ liệu giữa ESP32 và ESP32-CAM.
Mình sẽ sửa lại toàn bộ kiến trúc để **dễ làm, ổn định, đúng thực tế đồ án**.

---

# 🎯 Mục tiêu hệ thống

## ESP32 DevKit

* đọc cảm biến MLX90614
* gửi nhiệt độ qua UART sang ESP32-CAM

## ESP32-CAM

* nhận nhiệt độ qua UART
* chạy camera webserver
* hiển thị nhiệt độ trên web
* cảnh báo nếu > 37.5°C

👉 Toàn bộ giao tiếp giữa 2 board = **UART**

---

# 🧠 Kiến trúc đúng

```text id="gm3uxs"
MLX90614 -> ESP32 ----UART----> ESP32-CAM ----WiFi----> Web Browser
```

---

# 🔌 Nối dây đầy đủ

---

# 1. MLX90614 → ESP32

| MLX90614 | ESP32  |
| -------- | ------ |
| VIN      | 3.3V   |
| GND      | GND    |
| SDA      | GPIO21 |
| SCL      | GPIO22 |

---

# 2. ESP32 ↔ ESP32-CAM bằng UART

| ESP32        | ESP32-CAM |
| ------------ | --------- |
| GPIO17 (TX2) | U0R       |
| GPIO16 (RX2) | U0T       |
| GND          | GND       |

> TX → RX, RX → TX

---

# 3. ESP32-CAM nguồn

| ESP32-CAM | Nguồn |
| --------- | ----- |
| 5V        | 5V    |
| GND       | GND   |

---

# ⚠️ QUAN TRỌNG

Khi chạy thật:

* tháo USB TTL khỏi ESP32-CAM
  vì UART0 của CAM dùng để nhận dữ liệu từ ESP32

---

# 💻 CODE 1: ESP32 đọc nhiệt độ gửi UART

#include <Wire.h>
#include <Adafruit_MLX90614.h>

Adafruit_MLX90614 mlx;
HardwareSerial camSerial(2);

void setup() {
  Serial.begin(115200);

  Wire.begin(21,22);
  mlx.begin();

  camSerial.begin(115200, SERIAL_8N1, 16, 17);

  Serial.println("ESP32 READY");
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

# 💻 CODE 2: ESP32-CAM nhận UART + Web Camera

```cpp id="nt5oif"
#include "esp_camera.h"
#include <WiFi.h>
#include <WebServer.h>

const char* ssid = "YOUR_WIFI";
const char* password = "YOUR_PASS";

WebServer server(80);

String tempValue = "--";

// AI Thinker
#define PWDN_GPIO_NUM 32
#define RESET_GPIO_NUM -1
#define XCLK_GPIO_NUM 0
#define SIOD_GPIO_NUM 26
#define SIOC_GPIO_NUM 27
#define Y9_GPIO_NUM 35
#define Y8_GPIO_NUM 34
#define Y7_GPIO_NUM 39
#define Y6_GPIO_NUM 36
#define Y5_GPIO_NUM 21
#define Y4_GPIO_NUM 19
#define Y3_GPIO_NUM 18
#define Y2_GPIO_NUM 5
#define VSYNC_GPIO_NUM 25
#define HREF_GPIO_NUM 23
#define PCLK_GPIO_NUM 22

void handleRoot() {
  String page =
  "<html><body style='text-align:center;font-family:Arial'>"
  "<h1>ESP32-CAM Monitor</h1>"
  "<h2>Temp: " + tempValue + " C</h2>"
  "<img src='/capture' width='320'><br>";

  if (tempValue.toFloat() > 37.5) {
    page += "<h1 style='color:red'>HIGH TEMP!</h1>";
  }

  page += "</body></html>";

  server.send(200, "text/html", page);
}

void handleCapture() {
  camera_fb_t * fb = esp_camera_fb_get();

  server.send_P(200, "image/jpeg", (char *)fb->buf, fb->len);

  esp_camera_fb_return(fb);
}

void setup() {
  Serial.begin(115200); // UART nhận từ ESP32

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sccb_sda = SIOD_GPIO_NUM;
  config.pin_sccb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;
  config.frame_size = FRAMESIZE_QVGA;
  config.jpeg_quality = 12;
  config.fb_count = 1;

  esp_camera_init(&config);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) delay(500);

  server.on("/", handleRoot);
  server.on("/capture", handleCapture);
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

# 🌐 Cách dùng

Sau khi chạy CAM:

Mở Serial Monitor lúc test sẽ thấy IP:

```text id="xljm3n"
192.168.1.150
```

Mở trình duyệt:

```text id="5r9f2f"
http://192.168.1.150
```

Sẽ thấy:

* camera
* nhiệt độ realtime
* cảnh báo sốt

---

# 🧭 Các bước làm từ đầu tới cuối

---

# Bước 1: Test ESP32 riêng

Nối MLX90614.
Nạp code ESP32.

Nếu thấy:

```text id="thltme"
Send Temp: 36.5
```

OK.

---

# Bước 2: Nạp code ESP32-CAM

### Dùng USB TTL:

| TTL | CAM |
| --- | --- |
| TX  | U0R |
| RX  | U0T |
| 5V  | 5V  |
| GND | GND |

### Khi nạp:

* GPIO0 nối GND

### Upload xong:

* tháo GPIO0
* reset

---

# Bước 3: Chạy thật

Tháo USB TTL khỏi CAM

Nối:

| ESP32  | CAM |
| ------ | --- |
| GPIO17 | U0R |
| GPIO16 | U0T |
| GND    | GND |

---

# Bước 4:

Cấp nguồn cả 2 board.

Mở web xem camera.

---

# ❌ Lỗi thường gặp

---

# 1. Ký tự rác

Nguyên nhân:

* baud sai
* USB TTL còn cắm khi UART đang dùng

Sửa:

* 115200
* rút TTL khi chạy thật

---

# 2. Không nhận nhiệt độ

* TX RX ngược
* chưa nối GND chung

---

# 3. Camera fail

* nguồn yếu
* sai board

---

# 4. Reset liên tục

* dùng USB yếu

Nên dùng 5V 2A.

---

# 5. Không vào web

* sai WiFi
* khác mạng LAN

---



Chỉ cần nói: **làm full bản nộp đồ án**
