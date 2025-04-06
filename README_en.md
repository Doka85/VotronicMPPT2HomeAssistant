# Votronic MPPT Solar Charger Integration via ESPHome

This project documents the integration of a Votronic MPPT solar charge controller with an ESPHome-based Home Assistant system. The goal is to reliably read key data (battery voltage, solar panel voltage, current, calculated power, and energy yield) via UART and make it available to Home Assistant.

---

## 🛠️ Hardware Used

- **ESP32-S2 Mini**
- **Votronic MPP Solar Charge Controller** with serial output (UART, 1000 baud)
- UART connection via GPIO17 (RX)
- Custom 3D-printed case with hand-soldered cable connection

---

## 📡 Communication Protocol

The Votronic MPPT sends 16-byte data packets via UART. The first byte is always `0xAA` (start byte). Checksum validation:

> The checksum is calculated by XORing all bytes from index 1 to 14 (excluding the start byte 0xAA). This result must match byte 15.

**Example (valid):**
```
[0xAA] [0x1A] [0x3B] [0x05] [0xC8] [0x05] ... [0x00] [0xXX]
```

```cpp
uint8_t checksum = 0;
for (int i = 1; i < 15; i++) checksum ^= data[i];
```

---

## ⚙️ ESPHome Configuration (excerpt)

> Note: Passwords and secrets are removed.

```yaml
esphome:
  name: votronic-mppt

esp32:
  board: esp32-s2-saola-1
  framework:
    type: arduino

logger:
  level: INFO

api:
  encryption:
    key: "<removed>"

ota:
  - platform: esphome
    password: "<removed>"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

uart:
  id: uart_votronic
  rx_pin: GPIO17
  baud_rate: 1000

sensor:
  - platform: template
    name: "Battery Voltage"
    id: batt_voltage
    unit_of_measurement: "V"
    accuracy_decimals: 2

  - platform: template
    name: "Solar Voltage"
    id: solar_voltage
    unit_of_measurement: "V"
    accuracy_decimals: 2

  - platform: template
    name: "Solar Current"
    id: solar_current
    unit_of_measurement: "A"
    accuracy_decimals: 1

  - platform: template
    name: "Solar Power"
    id: solar_power_calc
    unit_of_measurement: "W"
    accuracy_decimals: 1
    lambda: |-
      return id(solar_voltage).state * id(solar_current).state;
    update_interval: 10s
    device_class: power
    state_class: measurement

  - platform: integration
    name: "Solar Yield Ah"
    sensor: solar_current
    time_unit: h
    unit_of_measurement: "Ah"
    accuracy_decimals: 2
    restore: true

  - platform: integration
    name: "Solar Yield Wh"
    sensor: solar_power_calc
    time_unit: h
    unit_of_measurement: "Wh"
    accuracy_decimals: 1
    restore: true

  - platform: total_daily_energy
    name: "Daily Solar Energy"
    power_id: solar_power_calc
    unit_of_measurement: "Wh"
    accuracy_decimals: 1
    restore: true
    filters:
      - multiply: 1.0
    state_class: total_increasing
    device_class: energy
```

---

## 💡 Usage Notes

- If no sunlight is available (e.g., winter storage), values will show 0.0 W / 0.0 V.
- Energy values (Ah, Wh) are preserved with `restore: true`.
- The charge controller only sends data when the solar voltage exceeds approx. 14 V.
- You may see messages like:   `[E][uart:015]: Reading from UART timed out at byte 0!`

---

## 📌 Project Status

- Current status: ✅ **Stable**
- Long-term test running
- Fully integrated into Home Assistant via API
- Future expansion: GitHub documentation, PCB adapter, graphical dashboards