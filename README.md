# IoT-Based Automatic Plant Watering System

## Project Overview

The **IoT-Based Automatic Plant Watering System** is a smart solution designed to automate the process of watering plants based on real-time soil moisture data. The system uses a **NodeMCU (ESP8266)** microcontroller, a **Soil Moisture Sensor**, a **Relay Module**, and a **Water Pump**. The system ensures that plants are watered when the soil moisture level falls below a certain threshold. It also integrates IoT functionality via **ThingSpeak**, allowing for remote monitoring of the soil moisture level.

## Features

- **Automated Irrigation**: The system waters plants automatically when the soil moisture falls below a set threshold.
- **Real-Time Monitoring**: Soil moisture levels are monitored and displayed on ThingSpeak, a cloud platform for IoT applications.
- **Remote Access**: The system provides real-time data access through ThingSpeak’s dashboard, which can be viewed remotely on a web browser or mobile device.
- **Email Alerts (Optional)**: The system can be enhanced to send email alerts when soil moisture reaches a critical level.

## Components Required

- **NodeMCU (ESP8266)** - 1
- **Soil Moisture Sensor** - 1
- **Relay Module** - 1
- **Water Pump** - 1
- **Jumper Wires**
- **Power Supply (5V)**
- **ThingSpeak Account**

## System Architecture

1. **Soil Moisture Sensor** reads the soil moisture levels.
2. The **NodeMCU (ESP8266)** processes the data and checks if the moisture level is below a preset threshold.
3. If the moisture level is low, the **Relay Module** is activated to power on the **Water Pump**.
4. The system sends the moisture data to **ThingSpeak**, providing real-time updates.

## Circuit Diagram

![Circuit Diagram](https://www.example.com/circuit-diagram.png)  *(Replace this URL with your actual circuit diagram)*

## Wiring Connections

- **Soil Moisture Sensor**:
  - VCC to 3V3 on NodeMCU
  - GND to GND on NodeMCU
  - A0 (Analog output) to A0 pin on NodeMCU

- **Relay Module**:
  - VCC to 5V on NodeMCU
  - GND to GND on NodeMCU
  - IN to D2 (GPIO4) on NodeMCU

- **Water Pump**:
  - The water pump is controlled by the relay module to activate when needed.

## Setup Instructions

### 1. Install Required Libraries

Before uploading the code, install the required libraries:

1. Open **Arduino IDE**.
2. Go to **Sketch > Include Library > Manage Libraries**.
3. Search for and install the following libraries:
   - **ESP8266WiFi** (for connecting to Wi-Fi).
   - **ThingSpeak** (for sending data to ThingSpeak).

### 2. Create a ThingSpeak Account

1. Sign up for a free account on [ThingSpeak](https://thingspeak.com/).
2. Create a new channel to store the soil moisture data.
3. Copy your **Channel ID** and **Write API Key** from the channel settings.

### 3. Update the Code

Replace the placeholders in the code with your actual values:

- **SSID**: Your Wi-Fi network name.
- **Password**: Your Wi-Fi password.
- **Channel ID**: Your ThingSpeak Channel ID.
- **Write API Key**: Your ThingSpeak Write API Key.

### 4. Upload the Code to NodeMCU

1. Connect your **NodeMCU (ESP8266)** to your computer via USB.
2. Select the correct **board** (NodeMCU 1.0) and **port** in Arduino IDE.
3. Upload the code by clicking the **Upload** button.

### 5. Monitor the System

- Open the **Serial Monitor** in Arduino IDE to view real-time soil moisture readings.
- Visit your ThingSpeak channel to monitor the soil moisture remotely.

## Code Example

```cpp
#include <ESP8266WiFi.h>
#include <ThingSpeak.h>

const char *ssid = "YourWiFiName";  // Your Wi-Fi SSID
const char *password = "YourWiFiPassword";  // Your Wi-Fi Password
unsigned long channelID = YourChannelID;  // Your ThingSpeak Channel ID
const char *writeAPIKey = "YourWriteAPIKey";  // Your ThingSpeak Write API Key

const int moisturePin = A0;  // Soil moisture sensor connected to A0 (analog)
const int relayPin = D2;     // Relay module connected to D2 (GPIO4)

const int moistureThreshold = 400;  // Below this value, the pump will activate

WiFiClient client;

void setup() {
  Serial.begin(115200); // Start the serial monitor

  // Connect to WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  // Connect to ThingSpeak
  ThingSpeak.begin(client);

  // Initialize relay pin
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, LOW); // Ensure the relay is off initially
}

void loop() {
  // Read soil moisture level
  int moistureLevel = analogRead(moisturePin);

  // Print soil moisture level to serial monitor for debugging
  Serial.print("Soil Moisture Level: ");
  Serial.println(moistureLevel);

  // If moisture is below threshold, activate the water pump
  if (moistureLevel < moistureThreshold) {
    digitalWrite(relayPin, HIGH);  // Turn on the water pump
    Serial.println("Soil is dry. Watering the plant...");
  } else {
    digitalWrite(relayPin, LOW);  // Turn off the water pump
    Serial.println("Soil moisture is sufficient.");
  }

  // Upload moisture level to ThingSpeak
  ThingSpeak.setField(1, moistureLevel);  // Field 1 for moisture level
  ThingSpeak.writeFields(channelID, writeAPIKey); // Send data to ThingSpeak
  
  // Wait for 15 seconds before next reading
  delay(15000);
}
