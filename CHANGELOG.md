# Changelog

All notable changes to this project are documented in this file.

## Unreleased

### Added

- Added a Home Assistant example configuration and setup guide.
- Added `fastcon.recall_scene` for replaying scenes stored by the brMesh app.
- Added captured scene and color-command documentation.
- Added configurable advertising transmit power and command repeat count.
- Added group lights, group membership assignment, built-in effects, and state mirroring between groups and member entities.
- Added capability flags for RGB, white, and color-temperature controls.

### Changed

- Changed the default advertising duration from 50 ms to 3000 ms to match the brMesh app's global transmit window. `adv_duration` remains configurable in YAML for installations that can reliably use a shorter window.
- Changed single-light ON and OFF commands to match the app's short `0x80` and `0x00` payloads when color and brightness do not need to change.
- Changed BLE advertisements to use the connectable advertising type and flags emitted by the brMesh app.
- Changed command queuing so the newest state for each light supersedes stale queued commands.

### Fixed

- Persisted and reserved command sequence numbers across bridge restarts so bulbs do not reject reused values as replayed commands.
- Fixed the BLE manufacturer-data length, which previously omitted the final Fastcon RF byte from the advertised field.
- Fixed brightness-only changes so they send a complete command when required.
- Fixed plain ON, white mode, group control, and effect handling to preserve the bulb's expected color state.
- Marked the mesh key as sensitive in supported ESPHome versions.

## 1.1.1

- Last upstream release before the changes maintained in this fork.
