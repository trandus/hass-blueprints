# 🧠 Home Assistant Blueprints

Collection of Home Assistant automation blueprints focused on practical home automations, including thermostat calibration and periodic low-battery reporting.

## 🚀 Features

- valve temperature offset calibration based on an external sensor,
- optional manual correction and configurable rounding step for valve offsets,
- periodic low-battery reporting (daily / weekly / monthly),
- per-sensor battery threshold configuration (0..20%, default <= 1%),
- notifications to phone(s) and/or Home Assistant persistent notifications.

## 📁 Repository structure

```
blueprints/
└── automation/
    ├── battery_low_level_report/
    │   └── periodic_low_battery_report.yaml
    └── valve_temperature_calibration/
        └── auto_offset_calibration.yaml
```

If new blueprints are added in the future, each will be stored in its own subdirectory under `blueprints/automation/`.

## 🗂 Blueprint catalog

| Blueprint | Version | File | Description |
| --- | --- | --- | --- |
| Periodic low battery report | 1.0.0 | `periodic_low_battery_report.yaml` | Sends a daily/weekly/monthly low-battery report with per-sensor thresholds and optional phone/persistent notifications. |
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

### Periodic low battery report

The blueprint requires the following inputs:

1. **Report frequency** – choose `Daily`, `Weekly`, or `Monthly`.
2. **Report time** – set the time when the report should be generated.
3. **Weekday for weekly report** – used only when frequency is set to weekly.
4. **Day of month for monthly report** – used only when frequency is set to monthly.
5. **Monitored batteries with individual thresholds** – provide a YAML list of battery sensors, each with its own optional threshold.
   - `battery_sensor` – sensor entity with battery level.
   - `threshold` – value from `0` to `20` (%), step `1`; if not provided, default threshold `1` is used (`<= 1%`).
6. **Phone notification action(s)** – optional phone notification action(s), usually `notify.mobile_app_*`.
7. **Send Home Assistant persistent notification** – if enabled, creates a persistent notification inside Home Assistant.

Example configuration for monitored batteries:

```yaml
- battery_sensor: sensor.kitchen_remote_battery
  threshold: 5
- battery_sensor: sensor.window_sensor_battery
  threshold: 1
```

The report message includes either:
- a list of sensors with battery levels below or equal to each sensor threshold,
- or a single line stating that no low battery levels were detected.

### Valve temperature offset calibration

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
