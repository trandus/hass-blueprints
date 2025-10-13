# 🧠 Home Assistant – Valve Temperature Calibration Blueprint

Automatically calibrate the **temperature offset** of your smart radiator valve (TRV) using an external temperature sensor.  
This blueprint keeps your thermostat’s readings accurate — even if the valve is located near a radiator, curtain, or in a warm corner.

---

## 🌡️ Features

- 🏠 Works with any **`climate` entity** (e.g. Zigbee, Hama, Tuya, Danfoss, Aqara, etc.)
- 📏 Calculates the **true valve sensor temperature** (without offset)
- 🔁 Automatically updates the **local temperature offset** entity (`number.xxx_local_temperature_offset`)
- ⏱️ Built-in **time throttle** (prevents too frequent changes)
- 🎚️ Supports **hysteresis** and **rounding** for stability
- 🚫 Optional **min/max limits** for offset values
- ⚙️ No coding required — configurable entirely via the Home Assistant UI

---

## 📦 Installation

1. Copy the blueprint file into your Home Assistant configuration:
