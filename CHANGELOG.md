# Changelog

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
