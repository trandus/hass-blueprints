# 🧠 trandus' Home Assistant Blueprints

Collection of Home Assistant automation blueprints focused on practical home automations, including thermostat calibration and periodic low-battery reporting.

Some blueprints in this repository may be based on and modified from the
[Awesome HA Blueprints](https://github.com/EPMatt/awesome-ha-blueprints)
project by EPMatt. Attribution and original-source links are provided in the
affected blueprint files.

## 🚀 Features

### Valve temperature offset calibration
- supports valves whose `current_temperature` reports the temperature before the offset is applied,
- calculates `external_sensor_temperature - valve_temperature + manual_correction`, independently of the current offset,
- applies an optional manual correction (bias) before rounding so you can fine-tune how aggressively the valve heats,
- rounds to the configured `rounding_step`, then clamps to the offset entity's `min`/`max` without rounding again,
- writes the calculated offset to the corresponding `number` entity (`*_local_temperature_offset`) of the device,
- checks temperature changes and re-evaluates current readings every 30 seconds to retry skipped updates,
- limits changed-offset write attempts independently for each automation (300 seconds by default), including failed writes,
- skips unavailable/invalid readings, invalid entity limits/step, values incompatible with the device step, and changes of at most `0.000001` °C; these skips do not restart the interval.

### Periodic low battery report
- Sends reports on a daily, weekly, or monthly schedule.
- Supports per-sensor battery thresholds in range 0..100% (default <= 1%).
- Includes invalid states (`unavailable`, `unknown`, missing entities, and other non-numeric values) by default.
- Can omit invalid states per sensor while retaining backward compatibility with `include_unavailable`.
- Can notify phone(s), Home Assistant persistent notifications, or both.

## 📁 Repository structure

```
blueprints/
└── automation/
    ├── battery_low_level_report/
    │   └── periodic_low_battery_report.yaml
    ├── controller_ikea_e2201/
    │   └── ikea_e2201_rodret_dimmer.yaml
    └── valve_temperature_calibration/
        └── valve_temperature_offset_calibration.yaml
```

If new blueprints are added in the future, each will be stored in its own subdirectory under `blueprints/automation/`.

## 🗂 Blueprint catalog

| Blueprint | Version | File | Download link | Description |
| --- | --- | --- | --- | --- |
| Controller - IKEA E2201 RODRET Dimmer | 2025.03.20c (modified fork) | `ikea_e2201_rodret_dimmer.yaml` | [Download](https://github.com/trandus/hass-blueprints/raw/main/blueprints/automation/controller_ikea_e2201/ikea_e2201_rodret_dimmer.yaml) | Controller automation for IKEA RODRET (E2201) with short/long/release/double actions, based on and modified from EPMatt's Awesome HA Blueprints. |
| Periodic low battery report | 0.3.0 | `periodic_low_battery_report.yaml` | [Download](https://github.com/trandus/hass-blueprints/raw/main/blueprints/automation/battery_low_level_report/periodic_low_battery_report.yaml) | Sends a daily/weekly/monthly low-battery report with per-sensor thresholds, invalid-state reporting, and optional phone/persistent notifications. |
| Valve temperature offset calibration | 0.2.0 | `valve_temperature_offset_calibration.yaml` | [Download](https://github.com/trandus/hass-blueprints/raw/main/blueprints/automation/valve_temperature_calibration/valve_temperature_offset_calibration.yaml) | Keeps the valve calibration aligned with an external temperature sensor, allows an optional bias, and rounds the offset to the valve's supported step. |

## 📦 HACS installation (repository)

This repository is structured as a standard **HACS `Blueprint` repository**.

1. In Home Assistant open **HACS** → **⋮** → **Custom repositories**.
2. Add `https://github.com/trandus/hass-blueprints`.
3. Select repository type: **Blueprint**.
4. Install the repository from HACS.

## 📥 Installing blueprints in Home Assistant

Follow the steps below to add any blueprint from this repository directly through the Home Assistant interface.

1. **Copy the import link**
   - Use one of the direct download links from the **Blueprint catalog** table above.
   - If you prefer, open the blueprint file in `blueprints/automation/...` and click the `Raw` button in the GitHub toolbar to copy the URL from your browser's address bar.
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
5. **Monitored batteries with individual thresholds** – provide a YAML list of battery sensors, each with its own optional settings.
   - `battery_sensor` – sensor entity with battery level.
   - `threshold` – value from `0` to `100` (%), step `1`; if not provided, default threshold `1` is used (`<= 1%`).
   - `include_invalid_state` – optional boolean (`true`/`false`); defaults to `true` and controls reporting of `unavailable`, `unknown`, missing entities, and other non-numeric states.
   - `include_unavailable` – deprecated compatibility option, used only when `include_invalid_state` is absent.
6. **Phone notification action(s)** – optional phone notification action(s), usually `notify.mobile_app_*`.
7. **Send Home Assistant persistent notification** – if enabled, creates a persistent notification inside Home Assistant.

Example configuration for monitored batteries:

```yaml
- battery_sensor: sensor.kitchen_remote_battery
  threshold: 5
  include_invalid_state: false
- battery_sensor: sensor.window_sensor_battery
  threshold: 1
  include_invalid_state: true
```

The report message includes either:
- a list of sensors with battery levels below or equal to each sensor threshold,
- entries with the actual invalid state when enabled for a sensor,
- or a single line stating that no low battery levels were detected.

### Valve temperature offset calibration

The blueprint requires the following inputs:

1. **Thermostatic valve** – select the `climate` entity that represents the valve you want to calibrate. You can find it under `Settings` → `Devices & Services`, open the device for your valve, and copy the entity ID displayed next to the climate entity (for example, `climate.living_room_valve`).
2. **External temperature sensor** – choose a `sensor` entity that reports the actual room temperature. Look it up on the `Settings` → `Devices & Services` page or via the `Settings` → `Devices` list, then copy the entity ID (for example, `sensor.living_room_temperature`). Make sure the sensor updates regularly and uses the `temperature` device class.
3. **Offset entity** – select the `number` entity that stores the offset value for the valve. On the valve's device page, look for an entity with a name similar to `Local temperature offset`. If there are multiple offset entities, choose the one linked to the climate entity you are calibrating. Open its entity details to confirm the entity ID (for example, `number.living_room_valve_local_temperature_offset`).
4. **Minimum interval between updates** – set the minimum number of seconds between attempts to write a changed offset (defaults to 300 seconds). A new attempt is allowed when at least this interval has elapsed. Failed writes also start the interval. Skipped evaluations, including unchanged values, do not restart it. Changes during the interval are re-evaluated from current readings on the first periodic check after it ends.
5. **Rounding step** – define the increment that the calculated offset should be rounded to (default `1.0`). Check the offset entity's supported increments on the valve's device page or in the device documentation. Choose a rounding step compatible with those increments, such as `1.0` for whole degrees or `0.5` for half degrees. The final value must also fit the device's step after clamping to its limits.
6. **Manual correction (bias)** – optionally add a positive or negative bias (default `0.0`). Positive values (for example `+0.3`) make the valve assume the room is warmer and therefore reduce heating sooner. Negative values (for example `-0.3`) make the valve assume the room is cooler and therefore prolong heating. Adjust in small increments and observe the results for a few heating cycles.

> **Tip:** After saving the automation, monitor the offset entity on the valve's device page. Manual changes to the offset are temporary: the next eligible evaluation restores the calculated value if it differs.

## ℹ️ How it works

Use this blueprint only with valves whose `current_temperature` is the built-in sensor reading before offset correction. It calculates an absolute offset; the current offset is used only to decide whether a write is needed:

```
raw_offset = external_sensor_temperature - valve_temperature + manual_correction
rounded_offset = round_to_step(raw_offset, rounding_step)
new_offset = clamp(rounded_offset, offset_entity_min, offset_entity_max)
```

Clamping happens after rounding, with no second rounding. For example, rounded values of `+3` °C and `-3` °C become `+2.5` °C and `-2.5` °C for an entity limited to that range. Readings and the current offset must be numeric, the entity limits must be valid, and its step must be positive. The final value must lie on the entity's supported step grid; invalid data blocks the write instead of substituting zero.

Temperature changes trigger an evaluation, and a [time pattern trigger](https://www.home-assistant.io/docs/automation/trigger/#time-pattern-trigger) checks every 30 seconds using current entity states. A write requires a difference greater than `0.000001` °C and at least `min_interval` seconds since that automation's previous changed-offset write attempt. These checks run before the actions, so an unchanged offset does not start or extend the interval. A failed write starts the interval because it is still a write attempt.

With the automation enabled, valid data, and no action already running, a correction skipped during the interval is considered again on the first periodic check after the interval ends, even without another temperature change. Periodic checks also resume evaluations after automation reloads or recovery of unavailable entities. Each automation has its own interval, and [`mode: single`](https://www.home-assistant.io/docs/automation/modes/) prevents overlapping action runs within an instance.
