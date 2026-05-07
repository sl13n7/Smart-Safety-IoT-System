# Smart Safety IoT System

## Real Hardware Diagram

> The diagram below uses real hardware assembled for this project.

![System Diagram](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/diagram%201.jpg)

---

## Illustration Wiring Diagram

> Simplified connection diagram for easier understanding.

![Illustration Diagram](https://github.com/sl13n7/Smart-Safety-IoT-System/blob/main/picture/Screenshot%20from%202026-04-30%2008-51-16.png)

---

# Components Used

## 1. ESP32 WROOM 32 Type-C 38 Pin
![ESP32](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/ESP32%20WROOM%2032%20Type%20C%2038Pin.webp)

## 2. ESP32-CAM
![ESP32-CAM](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/esp32-cam.webp)

## 3. Buck Converter (Step-Down Module)
![Buck Converter](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/module%20giam%20ap.webp)

## 4. USB TTL Programmer
![USB TTL](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/usb.webp)

## 5. Adapter Power Supply
![Adapter](https://github.com/sl13n7/Smart-Safety-IoT-System/raw/main/picture/adapter.webp)

## 6. MLX90614 Infrared Temperature Sensor
Contactless infrared temperature sensor used for realtime measurement.

---

# System Overview

## ESP32 WROOM 32
- Read temperature from MLX90614 sensor  
- Apply average filter for stable values  
- Send temperature data to ESP32-CAM using UART  

## ESP32-CAM
- Receive UART temperature data  
- Connect to WiFi network  
- Host web dashboard  
- Show realtime temperature  
- Trigger warning when threshold is exceeded  

👉 Communication between both boards uses **UART only**

---

# System Architecture

```text
MLX90614 Sensor
      |
      v
ESP32 WROOM -------- UART --------> ESP32-CAM -------- WiFi --------> Browser / Mobile Phone
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

## 3. ESP32-CAM Power Supply

| ESP32-CAM | Power |
| --------- | ----- |
| 5V        | 5V    |
| GND       | GND   |

---

# Important Notes

## Upload Mode

Use USB TTL to upload code to ESP32-CAM.

## Run Mode

Disconnect USB TTL after upload.
UART0 will be used for communication with ESP32.

---

# ESP32 Main Code

## Read MLX90614 + Average Filter + UART Send


#include <Wire.h>
#include <Adafruit_MLX90614.h>

Adafruit_MLX90614 mlx;
HardwareSerial camSerial(2);

const int SDA_PIN = 21;
const int SCL_PIN = 22;
const int RX2_PIN = 16;
const int TX2_PIN = 17;

float readAverageTemp() {
  float sum = 0;

  for (int i = 0; i < 5; i++) {
    sum += mlx.readObjectTempC();
    delay(50);
  }

  return sum / 5.0;
}

void setup() {
  Serial.begin(115200);

  Wire.begin(SDA_PIN, SCL_PIN);

  if (!mlx.begin()) {
    Serial.println("MLX90614 NOT FOUND");
    while (1);
  }

  camSerial.begin(115200, SERIAL_8N1, RX2_PIN, TX2_PIN);

  Serial.println("ESP32 READY");
}

void loop() {
  float temp = readAverageTemp();

  Serial.print("Temperature: ");
  Serial.println(temp, 2);

  camSerial.println(temp, 2);

  delay(1000);
}
```

---

# ESP32-CAM Code

## UART + Webserver + Temperature Dashboard


#include "esp_camera.h"
#include <WiFi.h>
#include <WebServer.h>

const char* ssid = "YOUR_WIFI";
const char* password = "YOUR_PASSWORD";

WebServer server(80);

String tempValue = "--";
float alertTemp = 37.5;

void handleRoot() {
  String page = "<html><head>";
  page += "<meta http-equiv='refresh' content='2'>";
  page += "</head><body style='text-align:center;font-family:Arial;'>";

  page += "<h1>Smart Safety IoT System</h1>";
  page += "<h2>Realtime Temperature</h2>";
  page += "<h1>" + tempValue + " C</h1>";

  if (tempValue.toFloat() >= alertTemp) {
    page += "<h2 style='color:red;'>WARNING: HIGH TEMPERATURE</h2>";
  } else {
    page += "<h2 style='color:green;'>NORMAL STATUS</h2>";
  }

  page += "</body></html>";

  server.send(200, "text/html", page);
}

void setup() {
  Serial.begin(115200);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }

  server.on("/", handleRoot);
  server.begin();
}

void loop() {
  server.handleClient();

  while (Serial.available()) {
    String data = Serial.readStringUntil('\n');
    data.trim();

    if (data.length() > 0) {
      tempValue = data;
    }
  }
}
```

---

# How To Use

1. Upload ESP32 main code
2. Upload ESP32-CAM code using USB TTL
3. Connect UART (TX ↔ RX)
4. Power on both boards
5. Open ESP32-CAM IP in browser

Example:

```
http://192.168.1.150
```

---

# Future Development

## RF Emergency Power Cut System

* Detect fire / high temperature
* Send RF signal
* Cut AC power via relay
* Stop dangerous devices

---

# Common Problems

## Garbage Characters

* Wrong baud rate
* USB TTL still connected

## No Temperature Data

* TX RX reversed
* Missing GND

## Cannot Open Webpage

* Wrong WiFi
* Different network

## Continuous Reset

* Weak power supply

👉 Use **5V 2A adapter**

---

# Final Result

* Realtime temperature monitoring
* UART communication
* WiFi dashboard
* Warning system
* Expandable RF shutdown

---

# Conclusion

A practical and scalable Smart Safety IoT System using ESP32 + ESP32-CAM.

````
