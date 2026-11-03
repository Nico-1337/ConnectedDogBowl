# AGENTS.md — ConnectedDogBowl

## Purpose

A bowl that weighs itself: load cell + HX711 read by a NodeMCU (ESP8266) running ESPHome, reading sent in
litres to Home Assistant over the native API. A 2020-2021 project, a single configuration file, no hand-compiled code.

## Files

| Path | Role |
|---|---|
| `dogbowl.yaml` | complete ESPHome configuration of the node (the only "code" file) |
| `README.md` | human tutorial in English |
| `README.fr.md` | the same tutorial in French |
| `readmeImgSrc/` | README images (`image5.webp` photo of the build, `image2.jpg` the dog, `image3.jpg` dashboard, `image4.png` wiring, `image1.jpg` / `image6.jpg` / `image7.jpg` component photos from the sellers) |
| `LICENSE` | MIT, Nicolas Holder |

## Versions

- ESPHome syntax from 2020-2021: `platform: ESP8266` and `board: nodemcuv2` under `esphome:`, `api.password`,
  `ota.password` without `platform`. Recent ESPHome versions expect an `esp8266:` block, an
  `api.encryption.key` key and `ota: - platform: esphome`. Only migrate on request, keeping the same behavior.
- Node name `dog_bowl` (recent ESPHome versions discourage the underscore; renaming it changes the entity on the Home Assistant side).

## Structure of `dogbowl.yaml`

- `esphome:` name and board (`nodemcuv2`).
- `wifi:` main network + fallback hotspot `ap:`; `captive_portal:` to reconfigure Wi-Fi.
- `logger:` logs (used to read the raw value during calibration).
- `api:` native connection to Home Assistant; `ota:` over-the-air updates.
- `sensor:` a single `platform: hx711` entry:
  - `dout_pin: D0`, `clk_pin: D1`, `gain: 128`, `update_interval: 10s`;
  - filters in order: `filter_out: nan` → `calibrate_linear` → `lambda` that clamps any negative value to 0;
  - `unit_of_measurement: litres`, `accuracy_decimals: 2`, `name: "dog bowl"`.

## Fields to replace before flashing

| Field | Value in the repository | Set to |
|---|---|---|
| `wifi.ssid` | `your_ssid` | network name |
| `wifi.password` | `your_ssid_password` | Wi-Fi password |
| `wifi.ap.password` | `change_me_fallback` | fallback hotspot password |
| `api.password` | `your_HA_API_password` | API password (or encryption key on recent ESPHome) |
| `ota.password` | `your_password` | OTA password |
| `sensor[0].dout_pin`, `clk_pin` | `D0`, `D1` | only if the wiring differs |
| `calibrate_linear` | see below | points measured on your own scale |

Prefer `!secret` (ESPHome `secrets.yaml` file) over plain values.

## Calibration

```yaml
- calibrate_linear:
  - 164800 -> 0
  - 555000 -> 1
  - 943700 -> 2
```

Form "raw HX711 value -> returned value". The points in the repository correspond to 0, 1 and 2 litres on the
author's scale (about 390,000 raw points per litre). They do not carry over: read the raw value in the logs with
the empty bowl, then with 1 L and 2 L, and replace the three lines.

## Wiring (documented invariant)

HX711 VCC → 3.3V, GND → GND, DAT → D0, CLK → D1. If a pin changes in the YAML, update the table in
both READMEs and do not silently modify `readmeImgSrc/image4.png`.

## Home Assistant side

The node is discovered automatically. The author's alerts (0.25 L threshold: IFTTT SMS, TTS, browser_mod popup)
are not part of this repository.

## Invariants

- One node, one sensor; keep the `nan` filter and the floor at 0.
- Keep `README.md` (English) and `README.fr.md` (French) in sync (the repository has readers abroad).
- Do not invent anything about the hardware: the prices and build time in the README are the author's.

## Never commit

- Real SSIDs and Wi-Fi passwords, API / OTA / hotspot passwords, encryption keys, tokens.
- `secrets.yaml`, the `.esphome/` folder, compiled binaries.
- IP addresses, GPS coordinates, street address, phone numbers.
- Real first names (only Nicolas may be named; the dog is called Roméo).
