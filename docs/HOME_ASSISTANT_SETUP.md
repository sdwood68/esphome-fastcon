# Home Assistant setup

This repository can be used as both the Fastcon component source and the ESPHome
device project for the BRMesh bridge.

## Private local files

`brmesh.yaml`, `secrets.yaml`, and `BRMESH_BACKUP*.json` are ignored because they
contain device credentials or the BRMesh mesh key. Keep a private backup of them.

## Configure and build

1. Copy `examples/brmesh-home-assistant.yaml` to `brmesh.yaml`.
2. Copy `examples/secrets.example.yaml` to `secrets.yaml` and replace every value.
3. Adjust the board, light IDs, names, and capability flags for the actual bulbs.
4. Create a local environment with `py -m venv .venv`.
5. Install ESPHome with `.venv\\Scripts\\python -m pip install "esphome>=2026.3.0"`.
6. Validate with `.venv\\Scripts\\esphome config brmesh.yaml`.
7. Connect the ESP32 by USB and install with `.venv\\Scripts\\esphome run brmesh.yaml`.

Python 3.11 through 3.15 and ESPHome 2026.3.0 or newer are required. Older
ESPHome versions cannot load the action-registration API used by this release.

The example uses `components/` from this checkout. That is intentional: component
changes can be built and tested locally before they are proposed upstream.

## Add it to Home Assistant

After the ESP32 joins Wi-Fi, ESPHome normally triggers Home Assistant discovery.
In Home Assistant, open **Settings > Devices & services**, select the discovered
ESPHome device, and enter the API encryption key from `secrets.yaml` if prompted.

If discovery does not appear, add the ESPHome integration manually using the
device hostname (`esp-brmesh-bridge.local`) or its IP address. The Fastcon lights
then appear as native Home Assistant light entities.

## Operational notes

- Do not enable `esp32_ble_tracker` unless another component requires scanning.
- Fastcon bulbs do not report state, so Home Assistant shows the last state sent.
- Use `default_transition_length: 0s` to avoid bursts of intermediate commands.
- Group `0` controls every bulb without assigning a numbered group.
