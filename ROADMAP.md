# Future Work

This file records larger features that need protocol validation or coordinated ESPHome and Home Assistant work before implementation.

## Target architecture

Home Assistant should provide the brMesh-app-like setup and configuration experience. The ESP device should implement only the physical BLE transport.

Home Assistant owns:

- Mesh-key generation, import, protected storage, backup, and recovery.
- Fastcon frame construction, encryption, whitening, CRC, discovery decoding, binding, and heartbeat validation.
- The light/device database, address allocation, capabilities, names, groups, scenes, effects, and commissioning state machine.
- Dynamic Home Assistant entities and the complete setup, group, and scene user interfaces.

The ESP radio owns:

- Starting and stopping BLE scans and forwarding raw manufacturer advertisements.
- Advertising an arbitrary raw payload for a requested duration and interval with specified power, flags, and advertising type.
- Cancelling or replacing queued transmissions and reporting accepted, started, completed, cancelled, and failed states.
- A small bounded queue for temporary network jitter plus radio, Wi-Fi, and firmware diagnostics.

The radio API must be protocol-neutral. A transmit request contains an already encoded payload and radio parameters; the ESP should not need a mesh key, light address, group, scene, or Fastcon packet encoder. Communication must be encrypted and authenticated. Prototype an ESPHome native-API extension first to retain ESPHome provisioning, OTA, logs, and health reporting; use a small dedicated encrypted API only if binary scan streaming and acknowledgements cannot be implemented cleanly through ESPHome.

### Repository direction: fork or continue

Evaluate three choices with a short radio-API proof of concept before committing to a migration:

#### Fork BRMesh Hub

Advantages:

- Working discovery, binding, heartbeat parsing, transmit scheduling, and protocol self-tests.
- Direct NimBLE radio control and tested ESP32-C6 hardware support.
- MIT license permits reuse with attribution.

Disadvantages:

- Its mesh database, Fastcon codec, web UI, REST API, MQTT discovery, and application state live on the ESP.
- Turning it into a protocol-neutral radio would remove or relocate most of its current product architecture.
- It is standalone PlatformIO firmware rather than ESPHome, so the current provisioning, OTA, API encryption, and diagnostics path would change.

#### Continue esphome-fastcon

Advantages:

- The current ESP32 board and radio path are already hardware-tested with this installation.
- ESPHome already supplies Wi-Fi provisioning, encrypted native API, OTA, logging, safe mode, and Home Assistant device diagnostics.
- Existing packet construction and light entities can remain available as a legacy mode during migration.

Disadvantages:

- The component currently puts Fastcon protocol and light semantics on the ESP, so a new raw-radio mode and binary API are required.
- ESPHome's normal YAML entities are static and cannot provide the desired dynamic device model by themselves.
- Discovery and binding still need implementation and validation.

#### Companion project with selective reuse

Create a Home Assistant custom integration plus a protocol-neutral radio component/firmware, while keeping this repository's existing light component supported. Port or adapt only the validated, MIT-licensed Fastcon codec, discovery, bind, heartbeat, and reference-vector tests from BRMesh Hub, with attribution, into the Home Assistant side or a shared protocol library.

### Current recommendation

Do not fork all of BRMesh Hub as the first step. Continue the current codebase long enough to prototype the raw radio API, and create the Home Assistant integration as a companion project. Use BRMesh Hub as the primary validated protocol and test-vector reference and selectively reuse its MIT-licensed implementation where appropriate.

Reconsider a BRMesh Hub firmware fork if the proof of concept shows that ESPHome cannot reliably provide continuous scan events, binary payload transport, transmission acknowledgements, and command cancellation. The decision gate is a working end-to-end test in which Home Assistant receives a discovery frame, submits one raw 24-byte frame, and receives an unambiguous completion result without any Fastcon knowledge on the ESP.

This separation allows the existing ESPHome light component to remain stable for current users while the new integration develops without forcing a premature firmware rewrite.

## Home Assistant scene and group editor

### Goal

Provide a Home Assistant interface for creating Fastcon groups and native bulb scenes without editing ESPHome YAML.

### Proposed stages

1. Add typed API actions for configuring a group, clearing a group, previewing a scene, saving a scene, deleting a scene, and recalling a scene.
2. Add progress and result responses for operations that transmit several commands.
3. Build a dashboard-based MVP with Home Assistant helpers, scripts, and the existing scene editor.
4. Build a custom Home Assistant integration and frontend editor if the MVP confirms the data model.

The visual editor should support group membership, target group, flash/fade mode, speed, ordered color/white steps, preview, duplicate, save, delete, and activate. Friendly configuration should live in Home Assistant; the ESP32 should retain only the protocol state needed to operate when Home Assistant is unavailable.

Home Assistant Light Group helpers are not a replacement for Fastcon groups: they fan a command out to every entity, while a Fastcon group reaches all member bulbs with one advertisement. With a long reliability window, individual fan-out is especially slow.

Bulbs do not report group membership or current light state, so the UI must treat the saved configuration as the source of truth and clearly report partial or unconfirmed writes.

## Phone-free discovery and commissioning

### Goal

Replicate the brMesh app's ability to create a mesh, discover reset or newly powered lights, assign addresses, and bind them to the bridge's mesh key.

### Reference assessment

The `tofuweasel/ha-esp-ble-bridge` project is a useful reference for its Home Assistant add-on, web UI, MQTT discovery, configuration storage, and ESPHome generation architecture. Its current native-pairing path is not a complete protocol implementation: discovery parses ESPHome logs, device IDs may be temporary, the GATT fallback guesses readable values, and its ESPHome configuration still contains a TODO for parsing and address assignment. It should be treated as a UI and orchestration reference rather than proof of the pairing packet format.

The protocol implementation should instead be validated against captured brMesh traffic and working Fastcon implementations. The expected commissioning flow is:

1. Put a bulb into its short pairing window by power-cycling or factory-resetting it.
2. Broadcast a Fastcon scan command and scan concurrently with duplicate filtering disabled.
3. Parse the bulb's discovery response: six-byte device ID, device type/capabilities, current address, and its per-device provisioning key.
4. Allocate an unused mesh address in the range 1-255.
5. Send a type-2 bind command encrypted with the bulb's provisioning key. The bind data is expected to contain the six-byte device ID, new address, group/default byte, and four-byte mesh key.
6. Repeat the bind during the pairing window and wait for a type-3 heartbeat from the new address as confirmation.
7. Persist the device ID, assigned address, device type/capabilities, friendly name, and confirmation state.
8. Test ON and OFF before declaring commissioning complete.

No GATT connection should be required for Fastcon lights; discovery, binding, confirmation, and control use BLE manufacturer advertisements.

### Mesh-key generation

When creating a new mesh, generate the key on the bridge with a cryptographically seeded random source and persist it before binding the first bulb. Four ASCII digits are known to be compatible with the brMesh ecosystem. Store their four ASCII bytes as eight hexadecimal characters for ESPHome configuration; for example, ASCII `7411` is represented as `37343131`.

The UI should offer three explicit choices:

- Create a new random mesh key. This requires binding every bulb to the new mesh.
- Import an existing eight-character hexadecimal mesh key to retain brMesh-app compatibility.
- Reveal or export the active key only after an explicit confirmation.

The key must be marked sensitive, omitted from normal logs and diagnostics, and backed up with the paired-light database. Changing the configured key must not silently claim that existing bulbs were re-keyed; each bulb needs a confirmed bind.

### Home Assistant entity strategy

New lights are dynamic, while ordinary ESPHome light entities are normally generated from YAML. Evaluate these approaches in order:

1. Save a newly commissioned light, reboot the bridge, and dynamically register saved light outputs before the ESPHome API enumerates entities.
2. Preallocate a configurable number of disabled light slots and activate/rename them as bulbs are commissioned.
3. Use a Home Assistant custom integration or add-on with MQTT discovery, as the reference project does, if ESPHome cannot safely add entities from persisted records.

The first option preserves the native ESPHome experience but needs a proof of concept. The third option offers the best fully dynamic UI but creates a second integration and communication layer.

### Safety and recovery

- Commission only during an explicit, time-limited pairing session.
- Show every discovered device ID, signal strength, proposed address, and detected type before binding unless auto-add is deliberately enabled.
- Prevent duplicate addresses and reserve address zero for broadcasts.
- Keep a pending-bind record so a restart can resume confirmation safely.
- Provide remove-from-Home-Assistant and rebind operations separately; deleting a database record does not reset a bulb.
- Provide encrypted configuration export/import and a recovery path for restoring the mesh key and address map.
- Do not enable automatic bulk binding by default.

### Validation plan

Before exposing pairing in Home Assistant, capture and test:

- Scan request and discovery response from an unpaired bulb.
- Bind packet generated by the brMesh app.
- Successful heartbeat confirmation.
- Rebinding an already paired bulb to a new mesh key.
- Duplicate-address rejection and address exhaustion.
- Power loss during bind and recovery on the next boot.
- Multiple bulbs in pairing mode at once.
- Compatibility across more than one Fastcon light model.
