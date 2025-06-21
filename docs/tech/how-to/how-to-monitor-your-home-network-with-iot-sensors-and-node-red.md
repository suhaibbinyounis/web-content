---
categories:
- Networking
- DevOps
- IoT
comments: true
cover:
  image: https://images.pexels.com/photos/27523128/pexels-photo-27523128.jpeg?auto=compress&cs=tinysrgb&h=650&w=940
date: 2025-06-17 13:05:16.383000
description: Learn to build a robust home network monitoring system using custom IoT
  sensors (ESP32/ESP8266), an MQTT broker, and Node-RED for visualization and alerts.
  Go beyond basic 'online/offline' checks and gain insights into your network's health.
tags:
- IoT
- Node-RED
- MQTT
- Networking
- Home Automation
- ESP32
- ESP8266
- Linux
- Troubleshooting
title: How to Monitor Your Home Network with IoT Sensors and Node-RED
---

![Showcase of various smart home devices controlled via a smartphone, highlighting automation and security.](https://images.pexels.com/photos/27523128/pexels-photo-27523128.jpeg?auto=compress&cs=tinysrgb&h=650&w=940 "Showcase of various smart home devices controlled via a smartphone, highlighting automation and security.")

## How to Monitor Your Home Network with IoT Sensors and Node-RED

Monitoring your home network might sound like overkill, but for anyone who relies heavily on their internet connection, runs local servers, or simply enjoys tinkering, it's incredibly empowering. Knowing when a critical device goes offline, tracking uptime, or even detecting rogue devices can save you headaches and provide peace of mind.

This guide will walk you through building a practical, real-world network monitoring setup using common IoT sensors and Node-RED. We'll focus on simplicity, leveraging standard protocols, and providing actionable insights.

## Why Monitor Your Home Network?

Beyond the "because you can" factor, there are tangible benefits:

*   **Proactive Troubleshooting**: Identify issues before they become critical. Is your media server offline? Is the Wi-Fi camera unreachable? Get notified immediately.
*   **Security Posture**: Detect unexpected devices joining your network or critical security devices (like a firewall or NVR) going offline.
*   **Performance Insight**: While we won't deep dive into bandwidth, basic uptime monitoring contributes to understanding overall network reliability.
*   **Learning & Fun**: It's a fantastic way to deepen your understanding of networking, IoT, and automation.

Our core tools will be:

*   **IoT Sensors (ESP32/ESP8266)**: Small, low-cost microcontrollers perfect for custom network checks.
*   **MQTT Broker**: A lightweight publish/subscribe messaging protocol, ideal for IoT devices to communicate. We'll use Mosquitto.
*   **Node-RED**: A flow-based programming tool for wiring together hardware devices, APIs, and online services. It's excellent for rapid prototyping and visualization.

Let's dive in.

## Setting Up Your MQTT Broker (Mosquitto)

MQTT is the backbone of our IoT communication. It allows our sensors to publish data and Node-RED to subscribe to it. Mosquitto is a popular open-source MQTT broker, easy to install and configure.

You'll need a Linux machine for this. A Raspberry Pi is an excellent choice, but any Linux server or even a Docker container will work. For this guide, we'll assume a Debian/Ubuntu-based system.

### Installation

```bash
sudo apt update
sudo apt install -y mosquitto mosquitto-clients
```

### Basic Configuration

By default, Mosquitto often allows anonymous connections. For a home network, this might be acceptable, but for better security, you should configure authentication. Let's start with a simple, common configuration enabling anonymous access and listening on the default port.

First, stop the Mosquitto service:

```bash
sudo systemctl stop mosquitto
```

Then, create a new configuration file for clarity, or edit `/etc/mosquitto/mosquitto.conf`. We'll create a new one to keep it clean.

```bash
sudo nano /etc/mosquitto/conf.d/network-monitor.conf
```

Add the following content:

```conf
# network-monitor.conf
listener 1883
allow_anonymous true
persistence true
persistence_location /var/lib/mosquitto/
log_dest file /var/log/mosquitto/mosquitto.log
```

*   `listener 1883`: Listens on the standard MQTT port.
*   `allow_anonymous true`: Allows clients to connect without a username/password. **For production, this should be `false` and proper user authentication configured.**
*   `persistence true` and `persistence_location`: Ensures retained messages and subscriptions survive broker restarts.
*   `log_dest`: Specifies where logs are written.

Save the file and restart Mosquitto:

```bash
sudo systemctl start mosquitto
sudo systemctl enable mosquitto # Ensures it starts on boot
```

### Testing Your MQTT Broker

You can test your broker using the `mosquitto_pub` (publisher) and `mosquitto_sub` (subscriber) clients. Open two separate terminal windows.

**Terminal 1 (Subscriber):**

```bash
mosquitto_sub -h localhost -t "home/network/test" -v
```

```output
home/network/test hello from publisher!
```

This command subscribes to the topic `home/network/test` and prints messages (`-v` for verbose, showing topic and payload).

**Terminal 2 (Publisher):**

```bash
mosquitto_pub -h localhost -t "home/network/test" -m "hello from publisher!"
```

If everything is working, you should see `home/network/test hello from publisher!` appear in Terminal 1.

## Setting Up Node-RED

Node-RED is where we'll orchestrate our monitoring. It provides a visual interface to build "flows" that connect inputs, process data, and trigger outputs.


Again, a Linux machine or Docker is ideal. We'll use the recommended script for a Node.js-based install on Linux.

```bash
bash <(curl -sL https://raw.githubusercontent.com/node-red/node-red-installer/master/bin/update-nodejs-and-nodered)
```

This script handles Node.js installation, Node-RED installation, and sets up a `systemd` service for easy management.

After installation, start Node-RED:

```bash
node-red-start
```

You can also enable it to start on boot:

```bash
sudo systemctl enable nodered.service
```

### Accessing Node-RED UI

Open your web browser and navigate to `http://<your_linux_ip>:1880`. You should see the Node-RED flow editor.

### Installing Node-RED Nodes

Node-RED has a vast ecosystem of "nodes" (pre-built blocks) you can install. We'll need a few for network monitoring and MQTT.

1.  Click the **hamburger menu** (top right) -> **Manage palette**.
2.  Go to the **Install** tab.
3.  Search for and install the following nodes:
    *   `node-red-contrib-ping`: For directly pinging devices from Node-RED.
    *   `node-red-contrib-mqtt-broker`: (Already included with Node-RED usually, but good to know) Allows Node-RED to act as an MQTT client.
    *   `node-red-dashboard`: For creating simple web-based dashboards to visualize data.

## Configuring an IoT Sensor (ESP32/ESP8266)

Our IoT sensor will be responsible for periodically checking the online status of a critical network device (e.g., your router, NAS, or a specific server) by pinging it and then publishing the result to our MQTT broker.

We'll use a simple Arduino sketch for an ESP32, which can be easily adapted for an ESP8266.

### Arduino IDE Setup

1.  **Download Arduino IDE**: Get it from the official Arduino website.
2.  **Install ESP32/ESP8266 Boards**:
    *   Go to `File > Preferences`.
    *   In "Additional Boards Manager URLs", add:
        *   For ESP32: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
        *   For ESP8266: `http://arduino.esp8266.com/stable/package_esp8266com_index.json`
    *   Go to `Tools > Board > Boards Manager...`. Search for "ESP32" and "ESP8266" and install them.
3.  **Install Libraries**:
    *   Go to `Sketch > Include Library > Manage Libraries...`.
    *   Search for and install:
        *   `PubSubClient` (by Nick O'Leary): For MQTT communication.
        *   `ESPmDNS` (for ESP32, usually built-in) / `ESP8266mDNS` (for ESP8266): Optional, for mDNS hostname resolution.
        *   `Ping` (by Kevin Darrah): For performing ICMP pings.

### ESP32/ESP8266 Sketch: Device Pinger

This sketch will connect to your Wi-Fi, then every `PING_INTERVAL` seconds, it will ping a target IP address and publish its online/offline status to an MQTT topic.

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <Ping.h> // For ESP32, this library simplifies pinging

// WiFi credentials
const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

// MQTT Broker details
const char* mqtt_server = "YOUR_MQTT_BROKER_IP"; // e.g., 192.168.1.100 (your Raspberry Pi's IP)
const int mqtt_port = 1883;
const char* mqtt_client_id = "esp32-network-pinger"; // Unique ID for your sensor
const char* mqtt_topic_status = "home/network/device/router/status"; // Topic to publish status
const char* mqtt_topic_uptime = "home/network/device/router/uptime"; // Topic to publish uptime

// Device to ping
IPAddress target_ip(192, 168, 1, 1); // Replace with the IP of the device you want to monitor (e.g., your router)
const char* target_name = "Router"; // Name for your device

// Ping interval (in milliseconds)
const long PING_INTERVAL = 10000; // Ping every 10 seconds
unsigned long lastPingTime = 0;

// Reconnect interval for MQTT
const long RECONNECT_INTERVAL = 5000; // Try to reconnect every 5 seconds
unsigned long lastReconnectAttempt = 0;

WiFiClient espClient;
PubSubClient client(espClient);
PingClass pinger; // Ping object

unsigned long bootTime = 0; // To track uptime

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void reconnect_mqtt() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    if (client.connect(mqtt_client_id)) {
      Serial.println("connected");
      // Once connected, publish a "boot" message
      client.publish(mqtt_topic_status, "online");
      bootTime = millis(); // Record boot time for uptime
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(RECONNECT_INTERVAL);
    }
  }
}

void setup() {
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
  reconnect_mqtt(); // Initial connection
}

void loop() {
  if (!client.connected()) {
    if (millis() - lastReconnectAttempt > RECONNECT_INTERVAL) {
      lastReconnectAttempt = millis();
      reconnect_mqtt();
    }
  }
  client.loop(); // Keep MQTT connection alive

  if (millis() - lastPingTime > PING_INTERVAL) {
    lastPingTime = millis();
    
    Serial.print("Pinging ");
    Serial.print(target_name);
    Serial.print(" (");
    Serial.print(target_ip);
    Serial.println(")...");

    // Ping the target
    if (pinger.Ping(target_ip)) {
      Serial.println("Device is online!");
      client.publish(mqtt_topic_status, "online", true); // Retained message
      
      // Calculate and publish uptime (optional, but nice)
      unsigned long currentUptime = (millis() - bootTime) / 1000; // Uptime in seconds since sensor boot
      char uptimeStr[20];
      sprintf(uptimeStr, "%lu", currentUptime);
      client.publish(mqtt_topic_uptime, uptimeStr);

    } else {
      Serial.println("Device is OFFLINE!");
      client.publish(mqtt_topic_status, "offline", true); // Retained message
    }
  }
}
```

**Before Uploading:**

*   Replace `YOUR_WIFI_SSID` and `YOUR_WIFI_PASSWORD` with your actual Wi-Fi credentials.
*   Replace `YOUR_MQTT_BROKER_IP` with the IP address of your Mosquitto server (e.g., your Raspberry Pi's IP).
*   Replace `192, 168, 1, 1` with the IP address of the device you want the ESP to monitor (e.g., your router, NAS, or server).
*   Adjust `mqtt_topic_status` and `mqtt_topic_uptime` if you want different topics.
*   The `client.publish(..., true)` ensures the last known status (`online`/`offline`) is retained on the MQTT broker, meaning new subscribers immediately get the latest status.

**Upload Steps:**

1.  Connect your ESP32/ESP8266 board to your computer via USB.
2.  In Arduino IDE, select `Tools > Board` and choose your specific ESP32/ESP8266 board.
3.  Select `Tools > Port` and choose the serial port connected to your board.
4.  Click the "Upload" button (right arrow icon).

Open the Serial Monitor (Tools > Serial Monitor) at 115200 baud to see the sensor's output and verify it's connecting and pinging.

```output
Connecting to MyHomeWiFi
................
WiFi connected
IP address:
192.168.1.101
Attempting MQTT connection...connected
Pinging Router (192.168.1.1)...
Device is online!
Pinging Router (192.168.1.1)...
Device is online!
Pinging Router (192.168.1.1)...
Device is OFFLINE!
```

## Node-RED Flow Examples for Monitoring

Now that our MQTT broker is running and our sensor is publishing data, let's create Node-RED flows to act on this information.

### 1. Monitoring Device Presence (from MQTT Sensor)

This flow will subscribe to the MQTT topic published by our ESP32 sensor and display its status.

**Flow Components:**

*   **MQTT In node**: Subscribes to the status topic.
*   **Debug node**: To see the raw message.
*   **Text node (Dashboard)**: To display the status on a simple dashboard.

**Steps:**

1.  Drag an `mqtt in` node from the palette onto your workspace.
2.  Double-click the `mqtt in` node:
    *   **Server**: Click the pencil icon to add a new MQTT broker configuration.
        *   **Name**: `My MQTT Broker`
        *   **Server**: `YOUR_MQTT_BROKER_IP` (e.g., `192.168.1.100`)
        *   **Port**: `1883`
        *   Click `Add`.
    *   **Topic**: `home/network/device/router/status` (match your ESP32 sketch)
    *   **QoS**: `2` (ensure message delivery)
    *   **Output**: `a String`
    *   **Name**: `Router Status MQTT`
    *   Click `Done`.
3.  Drag a `debug` node and connect its output to the `mqtt in` node's output.
4.  Drag a `text` node (found under `dashboard` in the palette).
5.  Double-click the `text` node:
    *   **Group**: Click the pencil icon to add a new UI group.
        *   **Tab**: Click the pencil icon to add a new UI tab.
            *   **Name**: `Network Monitor`
            *   Click `Add`.
        *   **Name**: `Device Status`
        *   Click `Add`.
    *   **Label**: `Router Status`
    *   **Value format**: `{{msg.payload}}`
    *   Click `Done`.
6.  Connect the `mqtt in` node's output to the `text` node's input.
7.  Click the **Deploy** button (top right).

Now, open your dashboard by navigating to `http://<your_node-red_ip>:1880/ui`. You should see a new tab "Network Monitor" and the "Router Status" showing "online" or "offline" as your ESP32 reports it.

### 2. Node-RED Pinging Local Network Devices

Instead of a separate sensor, Node-RED itself can ping devices. This is great for devices on the same subnet as your Node-RED instance, or for devices you don't want to dedicate an ESP to.

**Flow Components:**

*   **Inject node**: Triggers the ping periodically.
*   **Ping node**: Performs the ICMP ping.
*   **Switch node**: Checks the ping result.
*   **Change node**: Sets payload to "online" or "offline".
*   **MQTT Out node**: Publishes the status.
*   **Debug node**: For verification.

**Steps:**

1.  Drag an `inject` node onto your workspace.
2.  Double-click the `inject` node:
    *   **Payload**: `timestamp`
    *   **Repeat**: `interval` -> `10 seconds`
    *   **Name**: `Trigger Ping`
    *   Click `Done`.
3.  Drag a `ping` node.
4.  Double-click the `ping` node:
    *   **Target**: `192.168.1.200` (Replace with an IP of another device, e.g., your NAS, a printer, or a desktop PC)
    *   **Name**: `Ping NAS`
    *   Click `Done`.
5.  Connect `inject` to `ping`.
6.  Drag a `switch` node.
7.  Double-click the `switch` node:
    *   **Property**: `msg.payload`
    *   **Rules**:
        *   `is not null` (for online)
        *   `is null` (for offline)
    *   **Name**: `Check Ping Result`
    *   Click `Done`.
8.  Drag two `change` nodes.
    *   **First `change` node (for online):**
        *   **Set `msg.payload` to `online`**
        *   **Name**: `Set Online`
        *   Click `Done`.
    *   **Second `change` node (for offline):**
        *   **Set `msg.payload` to `offline`**
        *   **Name**: `Set Offline`
        *   Click `Done`.
9.  Connect `ping` output to `switch` input.
10. Connect `switch` output 1 (not null) to the `Set Online` node.
11. Connect `switch` output 2 (is null) to the `Set Offline` node.
12. Drag an `mqtt out` node.
13. Double-click the `mqtt out` node:
    *   **Broker**: Select `My MQTT Broker` (the one you configured earlier).
    *   **Topic**: `home/network/device/nas/status` (use a distinct topic)
    *   **QoS**: `2`
    *   **Retain**: `true` (important, so the last status is always available)
    *   **Name**: `Publish NAS Status`
    *   Click `Done`.
14. Connect both `Set Online` and `Set Offline` nodes to the `mqtt out` node.
15. Add `debug` nodes after the `Set Online` and `Set Offline` nodes to see what's being published.
16. **Deploy**.

Now you have two distinct sources publishing device status to MQTT.

### 3. Creating Alerts: Critical Device Offline!

When a critical device goes offline, you want to know immediately. Let's create an alert using a `notification` node (or email, Telegram, etc.). For simplicity, we'll use a `debug` node for the "alert" output, but you can swap it for an email, push notification, or Discord webhook.

**Flow Components:**

*   **MQTT In node**: Subscribes to the critical device's status topic (e.g., from your ESP32).
*   **RBE (Report By Exception) node**: Prevents repeated alerts if status doesn't change.
*   **Switch node**: Triggers only on "offline" status.
*   **Change node**: Formats the alert message.
*   **Debug node**: Represents the alert (can be replaced).

**Steps:**

1.  Drag an `mqtt in` node.
2.  Configure it for your critical device status (e.g., `home/network/device/router/status`).
3.  Drag an `rbe` (Report By Exception) node. This node filters out messages if the payload hasn't changed.
4.  Double-click the `rbe` node:
    *   **Mode**: `block unless value changes`
    *   **Name**: `Filter Duplicate Status`
    *   Click `Done`.
5.  Connect the `mqtt in` node to the `rbe` node.
6.  Drag a `switch` node.
7.  Double-click the `switch` node:
    *   **Property**: `msg.payload`
    *   **Rules**: `is equal to` `string` `offline`
    *   **Name**: `Is Offline?`
    *   Click `Done`.
8.  Connect the `rbe` node to the `switch` node.
9.  Drag a `change` node.
10. Double-click the `change` node:
    *   **Set `msg.payload` to**: `string` `CRITICAL ALERT: Router is OFFLINE!`
    *   **Name**: `Format Alert Message`
    *   Click `Done`.
11. Connect the `switch` node's "offline" output (output 1) to the `change` node.
12. Drag a `debug` node (or your chosen alert node: `email`, `telegram sender`, etc.).
13. Connect the `change` node to the `debug` node.
14. **Deploy**.

Now, if your ESP32 reports the router as "offline", you'll see an alert message in the Node-RED debug sidebar.

### Sample Output from Node-RED Debug Log

When the flows are active, you'd see messages like these in the debug sidebar:

```output
msg.payload : string[6]
"online" (from Router Status MQTT)

msg.payload : string[6]
"online" (from Ping NAS - Set Online)

msg.payload : string[7]
"offline" (from Router Status MQTT)

msg.payload : string[32]
"CRITICAL ALERT: Router is OFFLINE!" (from Format Alert Message)
```

## Next Steps and Advanced Considerations

This setup provides a solid foundation. Here are ways to expand it:

*   **More Sensors**: Deploy more ESP32/ESP8266 devices to monitor more critical devices, or add environmental sensors (temperature, humidity) for server closets.
*   **Power Monitoring**: Integrate smart plugs that support MQTT (like Shelly devices) to monitor the power consumption of key network gear.
*   **Data Persistence & Visualization**:
    *   **InfluxDB**: A time-series database. Use the `influxdb out` node in Node-RED to store historical data (e.g., uptime, ping response times).
    *   **Grafana**: A powerful visualization tool. Connect it to InfluxDB to create rich, interactive dashboards. This is a common pair for monitoring.
*   **Notifications**: Beyond debug messages, explore Node-RED nodes for:
    *   **Email**: `email` node.
    *   **Telegram**: `node-red-contrib-telegrambot`.
    *   **Discord**: Use an `http request` node to send webhooks to Discord.
    *   **Pushover/Pushbullet**: Dedicated notification services.
*   **Security**:
    *   **MQTT**: Set up username/password authentication for Mosquitto and enable TLS/SSL for encrypted communication.
    *   **Node-RED**: Enable admin authentication for the Node-RED UI.
    *   **Firewall**: Restrict access to your Node-RED and MQTT broker ports (`1880` and `1883`) to your local network only.
*   **API Integration**: Integrate with router APIs (if available, e.g., OpenWRT) for more granular data like client lists or traffic statistics. This moves beyond simple IoT sensors but is a powerful extension.

## Conclusion

Building your own home network monitoring system with IoT sensors and Node-RED is a rewarding project. It provides valuable insights into your network's health, helps you troubleshoot proactively, and significantly enhances your home automation capabilities. By leveraging open standards like MQTT and flexible tools like Node-RED, you create a system that's custom, extensible, and completely under your control.

Start simple, learn by doing, and gradually expand your monitoring capabilities. Happy tinkering!