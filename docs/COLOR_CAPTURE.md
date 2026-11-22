# brMesh single-device color capture

Captured from the Android brMesh app over ADB on 2026-11-22. The mesh key and
key-derived advertising frames are intentionally omitted. The target was light
ID `1`.

## Payload layout observed

Full color commands use:

`72 <light-id> <brightness> <blue> <red> <green> <warm> <cold> 00 00 00 00`

Short state/brightness commands use:

`22 <light-id> <value> 00 00 00 00 00 00 00 00 00`

## Captured sequence

| Order | Inner payload | Decoded observation |
|---:|---|---|
| 1 | `89001a0b16000c2503000000` | Non-light-ID-1 command; retained as capture context |
| 2 | `220180000000000000000000` | Short value `0x80` |
| 3 | `220100000000000000000000` | Off / short value `0x00` |
| 4 | `7201ff00ff04000000000000` | RGB `#FF0400`, brightness `0xFF` |
| 5 | `7201ff00ff08000000000000` | RGB `#FF0800`, brightness `0xFF` |
| 6 | `7201ff00ff0c000000000000` | RGB `#FF0C00`, brightness `0xFF` |
| 7 | `7201ff00ff10000000000000` | RGB `#FF1000`, brightness `0xFF` |
| 8 | `7201ff00ff14000000000000` | RGB `#FF1400`, brightness `0xFF` |
| 9 | `7201ff00ffa5000000000000` | RGB `#FFA500` (orange), brightness `0xFF` |
| 10 | `7201ff808000000000000000` | RGB `#800080` (purple), brightness `0xFF` |
| 11 | `7201ffff00ff000000000000` | RGB `#00FFFF` (cyan), brightness `0xFF` |
| 12 | `7201ffffff00000000000000` | RGB `#FF00FF` (magenta), brightness `0xFF` |
| 13 | `7201ff0000007f7f00000000` | White mode: warm `0x7F`, cold `0x7F` |
| 14 | `7201ff00ff00000000000000` | RGB `#FF0000` (red), brightness `0xFF` |
| 15 | `7201ff0000ff000000000000` | RGB `#00FF00` (green), brightness `0xFF` |
| 16 | `7201ffff0000000000000000` | RGB `#0000FF` (blue), brightness `0xFF` |
| 17 | `22013e000000000000000000` | Short brightness value `0x3E` |
| 18 | `220101000000000000000000` | Short brightness value `0x01` |
| 19 | `220116000000000000000000` | Short brightness value `0x16` |
| 20 | `22012d000000000000000000` | Short brightness value `0x2D` |
| 21 | `220144000000000000000000` | Short brightness value `0x44` |
| 22 | `22015b000000000000000000` | Short brightness value `0x5B` |
| 23 | `22017f000000000000000000` | Short brightness value `0x7F` |
| 24 | `7201ffff007b000000000000` | RGB `#007BFF`, brightness `0xFF` |
| 25 | `7201ffffff7b000000000000` | RGB `#FF7BFF`, brightness `0xFF` |
| 26 | `7201ff0000007f7f00000000` | White mode: warm `0x7F`, cold `0x7F` |
| 27 | `22013c000000000000000000` | Short brightness value `0x3C` |
| 28 | `220101000000000000000000` | Short brightness value `0x01` |
| 29 | `220142000000000000000000` | Short brightness value `0x42` |

The consecutive near-red payloads in rows 4-8 show that dragging the color
selector emits intermediate commands rather than only the final selection.
