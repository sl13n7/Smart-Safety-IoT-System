# 🔥 AI-Powered Fire & Safety Monitoring System

<p align="center">
  <img src="assets/banner.png" width="700"/>
</p>

<p align="center">
  Smart IoT system for real-time hazard detection and automatic safety response  
  <br><b>ESP32 • Computer Vision • Embedded Systems</b>
</p>

---

## 🚀 Overview

This project is a **real-world IoT safety system** that detects dangerous conditions such as overheating and fire risks.

It combines:

* 🌡️ Temperature sensing
* 📷 Human presence detection
* ⚡ Automatic power control

The system **automatically cuts off power** when unsafe conditions are detected.

---

## 🧩 System Architecture

<p align="center">
  <img src="docs/system_architecture.png" width="600"/>
</p>

---

## ⚙️ Hardware

* ESP32 DevKit V1
* ESP32-CAM (AI Thinker)
* MLX90614 Temperature Sensor
* Relay Module (5V)
* USB to TTL (CP2102)

---

## 🧠 Core Logic

```text
IF temperature > threshold AND no human detected
    → Cut off power (Relay ON)

IF rapid temperature increase
    → Trigger warning

IF human detected
    → Ignore shutdown
```

---

## 📸 Demo

<p align="center">
  <img src="hardware/wiring_diagram.png" width="500"/>
</p>

🎬 Demo video: `media/demo_video.mp4`

---

## 🛠️ Setup Guide

👉 See: `docs/setup.md`

---

## 📦 Project Structure

```bash
firmware/   → Embedded code  
hardware/   → Schematics & PCB  
docs/       → Documentation  
media/      → Demo files  
```

---

## 🚀 Future Improvements

* Web dashboard
* Telegram alerts
* AI detection (real implementation)
* RF communication module

---

## 👨‍💻 Author

sl13n7

---

## ⭐ Highlights

* Real hardware project
* Full system integration
* Practical IoT application
* Portfolio-ready project

---
