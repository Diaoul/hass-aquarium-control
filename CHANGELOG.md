# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-01-05

Initial release of Aquarium Control Blueprint for Home Assistant.

### Features

- Sunrise/sunset simulation with configurable fade-in/fade-out transitions
- Dashboard-configurable schedule via input helpers (start time, duration, brightness)
- Configurable light transition duration (0-20s)
- Maintenance mode with full brightness override
- CO2 injection control synchronized with lighting schedule
- Configurable CO2 time offset (default: 60 min before lights)
- Automatic state recovery after Home Assistant restarts
- Support for multiple synchronized light entities
- Device availability detection and automatic recovery
