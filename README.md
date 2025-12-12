# 🛡️ SafeKid: IoT Child Safety System for Vehicles

**SafeKid** is a dual-core IoT system designed to prevent vehicular heatstroke by detecting children left unattended in cars. It combines Computer Vision (Edge AI) with environmental monitoring to trigger automatic safety measures and real-time remote alerts via Telegram.

## 📋 System Overview

The system operates using a **Master-Slave architecture**:

1.  **The Eye (Slave - ESP32-CAM):** Runs a TinyML model (Edge Impulse) to visually detect humans in the backseat.
2.  **The Brain (Master - ESP32):** Monitors temperature, controls vehicle relays (Blower/Windows), and handles WiFi/Telegram communication.

### Key Features
* **AI Human Detection:** Uses an ESP32-CAM running a quantized neural network to detect humans.
* **Temperature Monitoring:** DHT22 sensor tracks cabin temperature in real-time.
* **Automated Response:**
    * If **Human Detected** + **Temp > 35°C**: Activates Blower fan and pulses the Power Window.
* **Remote Alerts:** Sends immediate Telegram notifications to parents with temperature data and system status.
* **Power Management:** Features a "Logical Sleep" mode after 15 minutes of inactivity to preserve vehicle battery.

---

## 🛠️ Hardware Requirements

| Component | Function |
| :--- | :--- |
| **ESP32 Dev Module** | Master Controller (WiFi, Logic, Relays) |
| **ESP32-CAM (AI-Thinker)** | Slave Unit (Vision Processing) |
| **DHT22 / AM2302** | Temperature & Humidity Sensor |
| **2-Channel Relay Module** | Controls Blower Fan and Power Window |
| **USB-TTL Adapter** | For programming the ESP32-CAM |
| **Power Supply** | 5V DC (Stable source required) |

---

## 🔌 Pin Configuration

### 1. Master (ESP32 Dev Module)
*Based on `masterbaruuu.ino`* 

| Pin | Connection | Notes |
| :--- | :--- | :--- |
| **GPIO 13** | DHT22 Data | Temperature Sensor |
| **GPIO 25** | Relay 1 (IN) | **Blower Fan** (Active LOW) |
| **GPIO 26** | Relay 2 (IN) | **Power Window** (Active LOW, 0.4s Pulse) |
| **GPIO 16 (RX2)** | Slave TX | Connect to ESP32-CAM GPIO 1 |
| **GPIO 17 (TX2)** | Slave RX | Connect to ESP32-CAM GPIO 3 |

### 2. Slave (ESP32-CAM)
*Based on `slavecikfad.ino`*

| Pin | Connection | Notes |
| :--- | :--- | :--- |
| **GPIO 1 (U0TXD)** | Master RX | UART Transmit |
| **GPIO 3 (U0RXD)** | Master TX | UART Receive |
| **5V / GND** | Power Input | **Warning:** Do not power via 3.3V pin |

> **⚠️ Wiring Note:** Common Ground (GND) is required between the Master and Slave for UART communication to work.

---

## 💻 Software & Libraries

### Prerequisites
* Arduino IDE
* **Master Libraries:**
    * `WiFi.h` & `WiFiClientSecure.h` 
    * `UniversalTelegramBot` by Brian Lough 
    * `DHT sensor library` by Adafruit 
    * `Adafruit Unified Sensor` 
* **Slave Libraries:**
    * `SafeKid_inferencing.h` (Exported Edge Impulse C++ Library)
    * `esp_camera.h`

### Configuration
1.  **WiFi & Telegram:** Open `masterbaruuu.ino` and update the following lines:
    ```cpp
    const char* ssid = "Your_SSID";          // 
    const char* password = "Your_Password";  // 
    const char* BOTtoken = "Your_Bot_Token"; // [cite: 2]
    const char* CHAT_IDS[] = {"Your_Chat_ID"}; // [cite: 2]
    ```
2.  **Thresholds:** You can adjust the temperature trigger in `masterbaruuu.ino`:
    ```cpp
    const float TEMP_THRESHOLD = 35.0; // Trigger temp in Celsius [cite: 9]
    ```

---

## 🔄 Logic Flow

1.  **Startup:** System waits **1 minute** upon boot for sensors to stabilize (Warmup Phase).
2.  **Detection:** The ESP32-CAM continuously scans for humans.
3.  **Trigger:**
    * If **Human Detected** → Slave sends serial command `HUMAN_DETECTED` to Master.
    * Master checks DHT22 sensor.
    * **IF Temp ≥ 35°C:**
        * Relay 1 (Blower) Latches **ON**.
        * Relay 2 (Window) Pulses **ON** for 0.4 seconds (Safety ventilation).
        * Telegram Alert sent: *"Human Detected! Temp: X°C. Action: Blower ON"*.
    * **IF Temp < 35°C:**
        * Telegram Alert sent: *"Human Detected! Temp Normal."*
4.  **Timeout (Sleep):**
    * If no human is detected for **15 minutes** after startup, or **5 minutes** after the last detection, the Master enters `SYSTEM_SLEEP` mode to save power.

---

## ⚠️ Disclaimer & Notes

* **Prototype Status:** This project is a prototype. Do not rely on it as the sole safety mechanism for children.
* **Hardware Setup:** The ESP32-CAM uses the main UART pins (1 & 3) for communication. You must **disconnect the Master** from these pins when uploading code to the ESP32-CAM to avoid conflicts.
* **Relay Logic:** Relays are configured as **Active-LOW** (LOW signal turns them ON).

---

## 📜 Credits

* **Edge Impulse:** For the Machine Learning model training and export.
* **UniversalTelegramBot:** For the ESP32 Telegram interface.
