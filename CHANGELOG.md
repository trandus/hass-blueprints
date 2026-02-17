# Changelog

## [0.4.3] - 2026-02-17
### Changed
- Optimized `Valve Temperature Offset Calibration` blueprint to skip recalculation when source states are unavailable/invalid instead of treating them as `0`.
- Added guard to skip `number.set_value` when the computed offset does not differ meaningfully from the current value.
- Added min/max clamping for computed offset using target `number` entity attributes when available.
- Improved formatting of the `Periodic Low Battery Report` message template block for cleaner YAML/Jinja structure.
- Updated README to reflect the actual battery threshold range (`0..100%`) and documented valve calibration reliability improvements.

## [0.4.2] - 2026-02-16
### Changed
- Renamed Valve Temperature Offset Calibration blueprint file from `auto_offset_calibration.yaml` to `valve_temperature_offset_calibration.yaml`.
- Updated documentation links and paths to use `valve_temperature_offset_calibration.yaml`.
- Kept Periodic Low Battery Report blueprint filename as `periodic_low_battery_report.yaml` and aligned references.

## [0.4.1] - 2026-02-16
### Changed
- Removed duplicated blueprint files from the repository root `blueprints/automation/...` to keep a single canonical blueprint source in `custom_components/trandus_blueprints/blueprints/automation/...`.
- Updated blueprint `source_url` metadata to point to the canonical `custom_components/trandus_blueprints/blueprints/automation/...` paths.
- Updated README catalog download links and repository structure section to reflect the canonical blueprint location.

## [0.4.0] - 2026-02-16
### Added
- Added Home Assistant helper integration skeleton in `custom_components/trandus_blueprints` with `manifest.json` so HACS can validate this repository as Integration.
- Added integration-packaged copies of all blueprints under `custom_components/trandus_blueprints/blueprints/automation/...`.

### Changed
- Updated `hacs.json` to use `content_in_root: false` and declare `trandus_blueprints` domain for HACS integration validation.
- Updated README HACS instructions to use repository type `Integration` and documented restart/import flow.

## [0.3.1] - 2026-02-16
### Changed
- Updated `Periodic Low Battery Report` blueprint to safely skip sensors in `unavailable` state without errors and added per-sensor `include_unavailable` option to include unavailable sensors in report output.
- Updated README documentation for the new unavailable handling behavior and configuration option.

## [0.3.0] - 2026-02-16
### Added
- New `Controller - IKEA E2201 RODRET Dimmer` automation blueprint (`2025.03.20c`, modified fork), based on EPMatt's Awesome HA Blueprints controller with attribution and updated source URL to this repository.

## [0.2.0] - 2026-02-15
### Added
- New `Periodic Low Battery Report` automation blueprint (version `0.1.0`) with daily/weekly/monthly scheduling, per-sensor low-battery thresholds, and optional phone plus persistent notifications.

## [0.1.0] - 2024-05-19
### Added
- Initial release of the Valve Temperature Offset blueprint for automatic calibration of thermostatic valves.
