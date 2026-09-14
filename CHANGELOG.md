# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-09-14

### Added

- Lights can now be selected by area, device, floor or label as well as by
  individual entity. Existing configurations keep working unchanged.
- `allow_negative` on the CO2 offset selector, so the negative offset the
  blueprint depends on is actually expressible in the UI
- `homeassistant.min_version`, declaring the Home Assistant 2024.10 requirement
  that the current syntax implies

### Changed

- Use `has_value()` for availability checks instead of comparing states against
  a hand-written unavailable/unknown list
- An unavailable start time helper now aborts the run via a condition, rather
  than falling back to 08:00 and driving the tank on the wrong schedule
- Removed the light availability trigger, which cannot accept a target. Lights
  coming back online are picked up by the next 60-second update instead.

## [1.1.0] - 2026-09-14

### Fixed

- Automation editor crashed with `Error in describing condition: can't access
  property "includes", e is undefined` because the repeat sequence used a Jinja
  template as the value of the `condition:` key instead of a condition type
- An unavailable or unknown start time helper made `today_at()` raise, aborting
  every run of the automation once a minute with no way to recover; the start
  time now falls back to `08:00:00`
- A transition longer than half the total duration stretched the cycle past the
  configured total duration; the transition is now clamped to half the total
- One failing or unresponsive light aborted the repeat loop and skipped every
  light after it; entity calls now continue on error
- Lights snapped off at the end of the fade-out instead of transitioning
- Availability triggers also fired on `unknown` -> `unavailable`

### Changed

- Fade-out now ends with a transition to off, matching the rest of the ramp
- `mode: single` no longer logs a warning when a run overlaps the next update
- Migrated to the current `triggers:`/`conditions:`/`actions:` and `action:`
  syntax, which requires Home Assistant 2024.10 or newer
- Added `source_url` so the blueprint can be updated in place after import

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
