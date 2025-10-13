# 🧠 Home Assistant – Valve Temperature Offset Blueprint

A simple Home Assistant automation blueprint that updates the calibration (offset) of a selected thermostatic valve (`climate` entity) based on the reading from an external temperature sensor. By continuously aligning the valve's measurement with the actual room temperature, your thermostat reflects real conditions more accurately.

## 🚀 Features

- automatically calculates the offset value from the difference between the external sensor and the valve's built-in sensor,
- writes the calculated offset to the corresponding `number` entity (`*_local_temperature_offset`) of the device,
- includes a basic time lock to limit how often the value is updated (minimum every 5 minutes by default).

## 📁 Repository structure

```
blueprints/
└── automation/
    └── valve_temperature_calibration/
        └── auto_offset_calibration.yaml
```

Copy the `blueprints/automation` directory to the `config/blueprints/automation` folder in your Home Assistant installation, then reload blueprints in the UI (`Settings` → `Automations & Scenes` → three-dot menu → `Reload Blueprints`).

## 🔧 Blueprint configuration

The blueprint requires the following inputs:

1. **Thermostatic valve** – select the `climate` entity that represents the valve you want to calibrate. You can find it under `Settings` → `Devices & Services`, open the device for your valve, and copy the entity ID displayed next to the climate entity (for example, `climate.living_room_valve`).
2. **External temperature sensor** – choose a `sensor` entity that reports the actual room temperature. Look it up on the `Settings` → `Devices & Services` page or via the `Settings` → `Devices` list, then copy the entity ID (for example, `sensor.living_room_temperature`). Make sure the sensor updates regularly and uses the `temperature` device class.
3. **Offset entity** – select the `number` entity that stores the offset value for the valve. On the valve's device page, look for an entity with a name similar to `Local temperature offset`. If there are multiple offset entities, choose the one linked to the climate entity you are calibrating. You can confirm the entity ID (for example, `number.living_room_valve_local_temperature_offset`) in the `Developer Tools` → `States` view.
4. **Minimum interval between updates** – set the number of seconds that must pass before a new offset can be written (defaults to 300 seconds). This prevents rapid changes that could spam the device. If you are unsure, keep the default value or increase it if your device reacts slowly to adjustments.
5. **Rounding step** – define the increment that the calculated offset should be rounded to (default `1.0`). Open the offset entity in the UI and check the `Step` attribute to see what increments the device supports. Many valves only allow full degrees (`1.0`), while others support finer steps such as `0.5` or `0.1`. Enter the same value here to avoid rejected updates.

> **Tip:** After saving the automation, you can monitor the offset entity in `Developer Tools` → `States` to verify that the value is being updated according to the configured interval and step.

## ℹ️ How it works

The automation performs a straightforward calculation:

```
new_offset = external_sensor_temperature - (valve_temperature - current_offset)
```

The result is rounded to the configured step (for example, every 0.5 °C) and stored in the offset entity. The automation waits for the specified interval before attempting the next update to limit the number of write operations.

## 🧪 Testing

The blueprint was created from a working user configuration and does not add extra safety checks or hysteresis—its behaviour matches previously shared YAML automations.
