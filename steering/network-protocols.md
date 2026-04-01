# Network Protocols for Embedded Development

Best practices for TCP, UDP, and LoRa/LoRaWAN networking on embedded devices. Load this when working with networked firmware.

## TCP on Embedded Devices

### Best Practices
- **Use non-blocking sockets** or async I/O. Blocking calls stall the main loop and can cause watchdog resets on ESP32/ESP8266.
- **Set reasonable timeouts.** Default TCP timeouts can be minutes long — far too long for embedded. Set connect timeout to 5-10 seconds, read timeout to 5 seconds.
- **Limit concurrent connections.** Most embedded TCP stacks support 4-8 simultaneous connections. ESP32 lwIP default is 10.
- **Reuse connections** (HTTP keep-alive, persistent MQTT) rather than opening/closing per message. TCP handshake overhead is significant on constrained devices.
- **Handle disconnections gracefully.** WiFi drops are common. Implement reconnection logic with exponential backoff.
- **Buffer management:** Allocate fixed-size buffers. Avoid dynamic allocation in network callbacks — heap fragmentation kills embedded systems.
- **TLS/SSL** is expensive on constrained MCUs. ESP32 handles it well; AVR cannot. Use hardware crypto acceleration where available. BearSSL (ESP8266) and mbedTLS (ESP32) are the common stacks.

### Common Application Protocols over TCP

#### MQTT
The standard for IoT messaging. Lightweight publish/subscribe over TCP.

```ini
# platformio.ini — common MQTT libraries
lib_deps =
    knolleary/PubSubClient@^2.8      ; Arduino MQTT (simple, widely used)
    ; OR
    marvinroger/AsyncMqttClient@^0.9  ; Async MQTT for ESP (non-blocking)
```

Best practices:
- Use QoS 0 for telemetry (fire-and-forget), QoS 1 for commands (at-least-once).
- Keep client ID unique per device. Use MAC address or chip ID.
- Set keepalive interval (15-60 seconds) and implement last-will-and-testament (LWT) for disconnect detection.
- Payload size: keep under 1 KB for reliability. Fragment larger payloads.
- Topic hierarchy: `project/device-id/sensor/temperature` — structured, filterable.

#### HTTP/HTTPS
For REST APIs and webhooks.

```ini
lib_deps =
    ; ESP32/ESP8266
    ; Use built-in WiFiClient + HTTPClient (no extra lib needed)
```

Best practices:
- Prefer HTTPS in production. Pin certificates or use root CA bundles.
- Use chunked transfer encoding for large responses to avoid buffering entire body.
- Parse JSON responses with ArduinoJson in streaming mode for memory efficiency.
- Implement retry with backoff for transient failures (HTTP 5xx, timeouts).

### WiFi Connection Management (ESP32/ESP8266)

```cpp
// Robust WiFi connection pattern
WiFi.mode(WIFI_STA);
WiFi.begin(ssid, password);

uint32_t timeout = millis() + 10000;
while (WiFi.status() != WL_CONNECTED && millis() < timeout) {
    delay(100);
}

if (WiFi.status() != WL_CONNECTED) {
    // Handle failure: retry, fallback, or deep sleep
}
```

- Store WiFi credentials in NVS/EEPROM, not hardcoded.
- Use `WiFi.setAutoReconnect(true)` on ESP32 for automatic reconnection.
- Monitor `WiFi.RSSI()` for signal quality diagnostics.
- For battery-powered devices, use `WiFi.disconnect(true); WiFi.mode(WIFI_OFF);` before deep sleep.

## UDP on Embedded Devices

### Best Practices
- UDP is connectionless — no handshake, no guaranteed delivery. Ideal for telemetry, time-critical data, and local network discovery.
- **Implement your own acknowledgment** if delivery matters. Simple sequence numbers + ACK packets work well.
- **Maximum payload:** 1472 bytes for standard Ethernet MTU (1500 - 20 IP header - 8 UDP header). Stay under 512 bytes for maximum compatibility across networks.
- **Broadcast/multicast** is useful for device discovery on local networks. ESP32 supports both.
- **NTP** uses UDP. Use built-in `configTime()` on ESP32 or NTPClient library for time synchronization.
- **DNS** uses UDP. Ensure DNS resolution works before attempting TCP connections.

### Common UDP Use Cases
- **mDNS/DNS-SD:** Local network service discovery. ESP32 has built-in mDNS support.
- **CoAP:** Constrained Application Protocol — REST-like semantics over UDP for IoT. Lighter than HTTP.
- **Syslog:** Remote logging over UDP (port 514). Simple and effective for fleet monitoring.
- **OSC:** Open Sound Control — real-time control messages. Common in art/music installations.

## LoRa / LoRaWAN

Long-range, low-power radio communication. Range: 2-15+ km depending on conditions.

### LoRa (Physical Layer)

Point-to-point or star topology using Semtech SX127x (SX1276, SX1278) or SX126x (SX1262, SX1268) chipsets.

```ini
# platformio.ini — LoRa libraries
lib_deps =
    sandeepmistry/LoRa@^0.8.0           ; Simple SX127x library
    ; OR
    jgromes/RadioLib@^6.0.0              ; Multi-radio library (SX126x, SX127x, etc.)
```

### Configuration Parameters
- **Frequency:** Region-dependent. 433 MHz (Asia), 868 MHz (EU), 915 MHz (US/AU). **Using the wrong frequency is illegal.**
- **Spreading Factor (SF):** 7-12. Higher SF = longer range, slower data rate, more airtime.
  | SF | Bit Rate | Sensitivity | Use Case |
  |----|----------|-------------|----------|
  | 7 | 5.47 kbps | -123 dBm | Short range, high throughput |
  | 9 | 1.76 kbps | -129 dBm | Medium range (default) |
  | 12 | 0.29 kbps | -137 dBm | Maximum range, minimum throughput |
- **Bandwidth:** 125 kHz (standard), 250 kHz, 500 kHz. Narrower = better sensitivity, slower.
- **Coding Rate:** 4/5, 4/6, 4/7, 4/8. Higher = more error correction, more airtime.
- **TX Power:** Up to +20 dBm (100 mW) on SX1276. Respect regional regulations.

### Best Practices
- **Duty cycle compliance** is legally required in most regions. EU868: 1% duty cycle = max 36 seconds of airtime per hour per channel.
- **Keep payloads small.** LoRa is not for bulk data. Typical max: 51-222 bytes depending on SF and region.
- **Use the lowest SF that achieves reliable communication.** SF12 has 32x the airtime of SF7.
- **Antenna matters enormously.** A proper tuned antenna can add 6+ dB of gain (equivalent to 4x range). Never transmit without an antenna connected — it can damage the radio.
- **SPI connection to radio:** LoRa modules (SX1276/SX1262) communicate via SPI. Ensure correct SPI mode (Mode 0), CS pin, and interrupt pin configuration.
- **DIO pins:** SX127x uses DIO0 for TX/RX done interrupts. SX126x uses DIO1. Map these correctly.
- **RSSI and SNR** from received packets are essential for link quality assessment. Log them.
- **Frequency hopping** improves reliability and regulatory compliance. RadioLib supports this.

### LoRaWAN (Network Layer)

Managed network protocol over LoRa. Requires a gateway and network server (TTN, Chirpstack, Helium).

```ini
lib_deps =
    mcci-catena/MCCI LoRaWAN LMIC library@^4.1.1  ; Full LoRaWAN stack
    ; OR
    thesolarnomad/LoRa Serialization@^3.2.1        ; Payload encoding helpers
```

Best practices:
- **OTAA (Over-The-Air Activation)** is preferred over ABP for security. OTAA negotiates session keys dynamically.
- **Adaptive Data Rate (ADR)** lets the network optimize SF per device. Enable it for stationary devices.
- **Confirmed vs unconfirmed messages:** Use unconfirmed for telemetry, confirmed only when delivery acknowledgment is critical (uses more airtime).
- **Payload encoding:** Use binary encoding (CayenneLPP or custom) instead of JSON/text. Every byte counts.
- **Class A** (default): Device initiates all communication, lowest power. **Class C**: Always listening, highest power. Choose based on use case.
- **Fair access policy:** The Things Network limits to 30 seconds of airtime and 10 downlinks per day per device.

### SX1276 vs SX1262 Comparison
| Feature | SX1276 | SX1262 |
|---------|--------|--------|
| Max TX Power | +20 dBm | +22 dBm |
| RX Current | 10.3 mA | 4.6 mA |
| Sleep Current | 0.2 µA | 0.16 µA |
| Interface | SPI + 6 DIO pins | SPI + 2 DIO pins |
| Library Support | Excellent (mature) | Good (newer) |
| Recommended for | Existing designs | New designs |

## General Networking Best Practices for Embedded

1. **Never hardcode credentials.** Use NVS, EEPROM, or provisioning protocols (WiFi Manager, BLE provisioning).
2. **Implement watchdog timers.** Network operations can hang. Hardware WDT ensures recovery.
3. **Log connection metrics** (RSSI, reconnect count, packet loss) for field diagnostics.
4. **Power management:** Disable radios when not in use. WiFi consumes 80-160 mA; LoRa TX consumes 20-120 mA; sleep modes drop to µA.
5. **OTA updates:** Plan for firmware updates over the network from day one. ESP32 has built-in OTA support. For LoRa, consider FUOTA (Firmware Update Over The Air) via LoRaWAN.
6. **Time synchronization:** Use NTP (WiFi) or network time (LoRaWAN) to timestamp sensor data. Unsynchronized timestamps make data useless.
7. **Security:** Use TLS for TCP, encrypt LoRaWAN payloads at the application layer if needed, rotate keys periodically.
