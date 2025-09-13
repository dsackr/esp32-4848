# esp32-4848

This project configures a 480x480 openHASP panel (ESP32‑S3) as a Home Assistant UI with MQTT.

Only the dynamic package is supported on the GPT branch. Legacy builder/router files were removed to simplify setup.

**What you get**
- A configurable Home Assistant package that builds and maintains the panel UI over MQTT.
- Helpers for device name and theme selection.
- Built‑in MQTT Discovery for the three onboard relays (auto‑published on HA start and when the node changes).
 - Optional UI configuration (no YAML edits) to define up to 6 tiles.

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

2) Reload or restart so HA creates helpers and loads automations/scripts:
- Recommended: Restart Home Assistant (ensures everything loads in one step).
- Or use Developer Tools → YAML and click:
  - Reload Input Texts (creates `input_text.hasp_node`)
  - Reload Input Selects (creates `input_select.hasp_theme`)
  - Reload Booleans/Numbers/Selects (creates the UI layout helpers)
  - Reload Automations
  - Reload Scripts

3) Set the device name (this is where you do it):
- Go to Settings → Devices & Services → Helpers.
- Find “openHASP Node” (`input_text.hasp_node`) and set its value to the panel’s Device name (from the panel’s web UI). MQTT topics follow `hasp/<node>/...`.

4) Choose a theme:
- In the same Helpers list, set “openHASP Theme” (`input_select.hasp_theme`) to `dark` or `light`.

5) Build the UI on the panel:
- Either power‑cycle the panel (the package auto‑rebuilds on boot), or
- Run the service `script.panel_build_dynamic_ui` from Developer Tools → Services.

## Relays in Home Assistant

The package already includes a `script.hasp_publish_discovery` and an automation that publishes MQTT Discovery for 3 relays at startup and whenever `input_text.hasp_node` changes. Default names are "Relay 1", "Relay 2", and "Relay 3".

- Resulting entities (by default):
  - `switch.<node>_relay1`
  - `switch.<node>_relay2`
  - `switch.<node>_relay3`
- Manual publish (optional): run `script.hasp_publish_discovery` and pass `node` if you want to override the helper, or custom `relay*_name` values.

## Configure Pages In The UI (No YAML)

You can define up to 6 tiles via Helpers. Turn on the UI layout switch and fill the slot helpers. Then rebuild the UI.

- Enable: set `input_boolean.hasp_use_ui_layout` to On.
- For each slot (1–6):
  - `input_boolean.hasp_slotX_enabled`: turn on to include the tile
  - `input_select.hasp_slotX_type`: choose `switch`, `light`, or `fan`
  - `input_text.hasp_slotX_entity`: enter the entity ID (e.g., `light.kitchen`)
  - `input_text.hasp_slotX_label`: the on‑screen label (optional)
  - `input_number.hasp_slotX_page`: page number (1–9)
- Apply: run `script.panel_build_dynamic_ui`.

Notes
- If `hasp_use_ui_layout` is Off, the default page shows 3 relays (`switch.<node>_relay1/2/3`).
- Lights get dimmer presets and color chips when supported. Fans use presets or Off/Low/Med/High.

## Troubleshooting

- Nothing shows on screen:
  - Ensure `input_text.hasp_node` exactly matches the panel’s Device name.
  - Verify MQTT connectivity and that topics like `hasp/<node>/state/statusupdate` appear.
  - Run `script.panel_build_dynamic_ui` again.
- Touch does nothing: Confirm the entity exists in HA and is defined in the `devices` list.
- Header title wrong: Ensure the panel publishes `hasp/<node>/state/page`; the router updates the header on page change.

## Files

- `openhasp_package.yaml`: dynamic, configurable package (supported).
- `openhasp_discovery.yaml`: optional MQTT discovery for three relays.
- `openhasp_painter.yaml`: reserved for future drawing/styling utilities.
