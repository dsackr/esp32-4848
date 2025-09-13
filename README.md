# esp32-4848

This project configures a 480x480 openHASP panel (ESP32‑S3) as a Home Assistant UI with MQTT.

Only the dynamic package is supported on the GPT branch. Legacy builder/router files were removed to simplify setup.

**What you get**
- A single package that creates one MQTT device in HA: “Dash480”.
- Built‑in MQTT Discovery for three onboard relays under that device.
- Device‑level configuration in HA (no YAML edits): Node Name, Theme, Number of Pages, and 6 slots per page (enter entity IDs).

## Prerequisites

- Home Assistant with MQTT configured and working.
- In `configuration.yaml`, enable packages:
  - `homeassistant: packages: !include_dir_named packages`
- openHASP firmware on the panel: flash “Guition ESP32‑S3‑4848S040” from https://nightly.openhasp.com and connect Wi‑Fi.
- In the panel’s web UI, configure outputs if you use the onboard relays:
  - GPIO 1 → output1 → group 1
  - GPIO 2 → output2 → group 2
  - GPIO 40 → output40 → group 3

## Install

1) Copy `openhasp_package.yaml` into your Home Assistant `packages` folder.

2) Restart Home Assistant (recommended) so discovery and automations load cleanly.

3) Configure the device in HA:
- Go to Settings → Devices & Services → MQTT → Devices → Dash480.
- Set “Node Name” to the panel’s Device name from the openHASP web UI (topics will be `hasp/<node>/...`).
- Set “Theme” to dark or light.
- Set “Number of Pages” (1–6).
- For each page/slot you want on the screen, set “P{n} Slot {m} Entity” to a HA entity ID (e.g., `light.kitchen`, `switch.lamp`, `fan.living_room`).

4) Build the UI on the panel:
- Either power‑cycle the panel (auto‑rebuild on boot), or
- Run the service `script.panel_build_dynamic_ui` from Developer Tools → Services.

## Relays in Home Assistant

The package publishes MQTT Discovery for three switches attached to the Dash480 device. These control the panel’s outputs:

- `switch.dash480_relay1` → `output1` (GPIO 1)
- `switch.dash480_relay2` → `output2` (GPIO 2)
- `switch.dash480_relay3` → `output40` (GPIO 40)

## Configure Pages In HA (No YAML)

- Use the Dash480 device controls created by discovery:
  - “Number of Pages” sets how many pages are built (1–6).
  - For each page 1..N, set the “P{n} Slot {m} Entity” text to a valid entity ID.
  - The package infers type from the entity domain (`switch`, `light`, `fan`).
- Defaults: If no slots are configured, Page 1 shows Relay 1/2/3 automatically.
- Behavior: Lights get dimmer presets and color chips when supported. Fans use presets or Off/Low/Med/High depending on capabilities.

## Troubleshooting

- Nothing shows on screen:
  - Ensure the Dash480 “Node Name” exactly matches the panel’s Device name.
  - Verify MQTT connectivity and that topics like `hasp/<node>/state/statusupdate` appear.
  - Run `script.panel_build_dynamic_ui` again.
- Touch does nothing: Confirm the entity exists in HA and you set that entity ID in a P{n} Slot {m} field.
- Header title wrong: Ensure the panel publishes `hasp/<node>/state/page`; the router updates the header on page change.

## Files

- `openhasp_package.yaml`: dynamic package that creates the Dash480 device and builds the UI.
- `openhasp_painter.yaml`: reserved for future drawing/styling utilities.
