# 🧠 Home Assistant – Valve Temperature Offset Blueprint

A simple Home Assistant automation blueprint that updates the calibration (offset) of a selected thermostatic valve (`climate` entity) based on the reading from an external temperature sensor. By continuously aligning the valve's measurement with the actual room temperature, your thermostat reflects real conditions more accurately.

## 🚀 Features

- automatically calculates the offset value from the difference between the external sensor and the valve's built-in sensor,
- applies an optional manual correction (bias) before rounding so you can fine-tune how aggressively the valve heats,
- rounds the offset to the increment supported by the device (for example `0.5` °C or `1.0` °C),
- writes the calculated offset to the corresponding `number` entity (`*_local_temperature_offset`) of the device,
- includes a basic time lock to limit how often the value is updated (minimum every 5 minutes by default).

## 📁 Repository structure

```
blueprints/
└── automation/
    └── valve_temperature_calibration/
        └── auto_offset_calibration.yaml
```

If new blueprints are added in the future, each will be stored in its own subdirectory under `blueprints/automation/`.

## 🗂 Blueprint catalog

| Blueprint | Version | File | Description |
| --- | --- | --- | --- |
| Valve temperature offset calibration | 0.2.0 | `auto_offset_calibration.yaml` | Keeps the valve calibration aligned with an external temperature sensor, allows an optional bias, and rounds the offset to the valve's supported step. |

## 📥 Installing blueprints in Home Assistant

Follow the steps below to add any blueprint from this repository directly through the Home Assistant interface.

1. **Copy the import link**
   - In this repository, open the blueprint file you want to use (for example, [`blueprints/automation/valve_temperature_calibration/auto_offset_calibration.yaml`](blueprints/automation/valve_temperature_calibration/auto_offset_calibration.yaml) or directly at <https://github.com/trandus/hass-blueprints/blob/main/blueprints/automation/valve_temperature_calibration/auto_offset_calibration.yaml>).
   - Click the `Raw` button in the GitHub toolbar and copy the URL from your browser's address bar.
2. **Start the import in Home Assistant**
   - In Home Assistant, go to `Settings` → `Automations & Scenes` → `Blueprints`.
   - Click `Import Blueprint` in the upper-right corner.
3. **Paste the GitHub URL**
   - Paste the copied `raw` GitHub URL into the `URL` field.
   - Confirm by pressing `Preview`. Home Assistant loads the blueprint metadata and shows a summary.
4. **Finish the import**
   - Review the details and click `Import`. Home Assistant saves the blueprint under `Settings` → `Automations & Scenes` → `Blueprints`.
5. **Create an automation**
   - Click `Create Automation` on the confirmation dialog or return later to `Settings` → `Automations & Scenes` → `Automations` and choose `Create automation` → `Start with a blueprint`.
   - Select the newly imported blueprint and configure the required inputs.

> **Tip:** When an update is available, go to `Settings` → `Automations & Scenes` → `Blueprints`, open the blueprint, and choose `Update blueprint` from the menu to re-download it from GitHub. Manual file management is also possible if you prefer, but it is not covered here.

## 🔧 Blueprint configuration

The blueprint requires the following inputs:

1. **Thermostatic valve** – select the `climate` entity that represents the valve you want to calibrate. You can find it under `Settings` → `Devices & Services`, open the device for your valve, and copy the entity ID displayed next to the climate entity (for example, `climate.living_room_valve`).
2. **External temperature sensor** – choose a `sensor` entity that reports the actual room temperature. Look it up on the `Settings` → `Devices & Services` page or via the `Settings` → `Devices` list, then copy the entity ID (for example, `sensor.living_room_temperature`). Make sure the sensor updates regularly and uses the `temperature` device class.
3. **Offset entity** – select the `number` entity that stores the offset value for the valve. On the valve's device page, look for an entity with a name similar to `Local temperature offset`. If there are multiple offset entities, choose the one linked to the climate entity you are calibrating. You can confirm the entity ID (for example, `number.living_room_valve_local_temperature_offset`) in the `Developer Tools` → `States` view.
4. **Minimum interval between updates** – set the number of seconds that must pass before a new offset can be written (defaults to 300 seconds). This prevents rapid changes that could spam the device. If you are unsure, keep the default value or increase it if your device reacts slowly to adjustments. The automation will ignore triggers that occur more frequently than this interval.
5. **Rounding step** – define the increment that the calculated offset should be rounded to (default `1.0`). Open the offset entity in the UI and check the `Step` attribute to see what increments the device supports. Many valves only allow full degrees (`1.0`), while others support finer steps such as `0.5` or `0.1`. Enter the same value here to avoid rejected updates.
6. **Manual correction (bias)** – optionally add a positive or negative bias (default `0.0`). Positive values (for example `+0.3`) make the valve assume the room is warmer and therefore reduce heating sooner. Negative values (for example `-0.3`) make the valve assume the room is cooler and therefore prolong heating. Adjust in small increments and observe the results for a few heating cycles.

> **Tip:** After saving the automation, you can monitor the offset entity in `Developer Tools` → `States` to verify that the value is being updated according to the configured interval and step.

## ℹ️ How it works

The automation performs a straightforward calculation:

```
raw_offset = external_sensor_temperature - (valve_temperature - current_offset)
adjusted_offset = raw_offset + manual_correction
new_offset = round_to_step(adjusted_offset, rounding_step)
```

The result is stored in the offset entity. The automation waits for the specified interval before attempting the next update to limit the number of write operations.
