# brMesh scene command capture

Captured from the Android brMesh app over ADB on 2026-11-22. The mesh key and
key-derived advertising frames are intentionally omitted.

The commands are listed in activation order using their brMesh scene names.

| Order | Scene | Inner payload |
|---:|---|---|
| 1 | Bills Colors | `3b0000310000000000000000` |
| 2 | Spooky 1 | `3b0000600000000000000000` |
| 3 | Thanksgiving | `3b0000fa0000000000000000` |
| 4 | Warm White | `3b0000c10000000000000000` |

## Initial observation

All four commands use the same 12-byte layout. Only byte 4 changes:

| Scene | Variable byte |
|---|---|
| Bills Colors | `0x31` |
| Spooky 1 | `0x60` |
| Thanksgiving | `0xFA` |
| Warm White | `0xC1` |

This appears to be a scene-selection command family (`0x3B`) rather than the
multi-step `fastcon_scene` definition format already documented by the component.
More captures are needed to establish whether byte 4 is a stored scene ID, a
bitfield, or another encoded parameter.
