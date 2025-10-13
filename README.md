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

If new blueprints are added in the future, each will be stored in its own subdirectory under `blueprints/automation/`.

## 🗂 Blueprint catalog

| Blueprint | Version | File | Description |
| --- | --- | --- | --- |
| Valve temperature offset calibration | 0.1.0 | `blueprints/automation/valve_temperature_calibration/auto_offset_calibration.yaml` | Keeps the valve calibration aligned with an external temperature sensor so that the reported room temperature remains accurate. |

## 📥 Installing blueprints in Home Assistant

The steps below show how to import any blueprint from this repository into Home Assistant. Repeat the process for every blueprint you want to enable.

1. **Download the repository files**
   - Option A: Click the green `Code` button on GitHub and choose `Download ZIP`. Unpack the archive locally.
   - Option B: Use Git to clone the repository directly on the machine that hosts Home Assistant: `git clone https://github.com/<your_fork>/hass-blueprints.git`.
2. **Locate the blueprint file**
   - Inside the repository, open the `blueprints/automation/` directory.
   - Each blueprint has its own folder (for example, `valve_temperature_calibration/`). The YAML file inside that folder (such as `auto_offset_calibration.yaml`) is the blueprint definition.
3. **Copy the blueprint to Home Assistant**
   - If you manage Home Assistant via the file system (Home Assistant Operating System, Container, or Core), copy the selected blueprint folder into `<config>/blueprints/automation/` on your Home Assistant instance. Create missing directories if necessary, keeping the same folder structure as in this repository.
   - When using the Home Assistant UI file editor, upload the YAML file to `config/blueprints/automation/<folder_name>/<blueprint_file>.yaml`.
4. **Reload blueprints**
   - In Home Assistant, open `Settings` → `Automations & Scenes`, click the three-dot menu in the upper right corner, and choose `Reload Blueprints`.
5. **Create an automation from the blueprint**
   - Still on the `Automations` tab, click `+ Create automation` → `Start with a blueprint`.
   - Pick the imported blueprint from the list and fill in the required inputs.
6. **Repeat for additional blueprints**
   - If you want to use more than one blueprint from this repository, repeat steps 2–5 for each blueprint directory you copy into Home Assistant.

> **Tip:** To stay up to date, periodically pull the latest changes from the repository or replace the blueprint files with newer versions before reloading blueprints in Home Assistant.

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
