# esp32-4848

This project configures a 480x480 openHASP panel (ESP32‑S3) as a Home Assistant UI with MQTT.

Only the dynamic package is supported on the GPT branch. Legacy builder/router files were removed to simplify setup.

**What you get**
- A configurable Home Assistant package that builds and maintains the panel UI over MQTT.
- Helpers for device name and theme selection.
- Optional MQTT discovery for three onboard relays.

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

## Optional: Expose 3 Relays in Home Assistant

If you wired the onboard relays and want them as HA switches, copy `openhasp_discovery.yaml` into your `packages` folder and reload scripts. Then call `script.hasp_publish_discovery` with:
- `node`: your panel’s device name (must match `input_text.hasp_node`)
- `relay1_name`, `relay2_name`, `relay3_name`: optional friendly names

## Customize Pages & Devices

Edit the `devices` list in `openhasp_package.yaml` under `script.panel_build_dynamic_ui`. Add items like:
- `{ page: 1, type: switch, entity: switch.living_room_lamp, label: Lamp }`
- `{ page: 1, type: light,  entity: light.studio_lights,     label: Studio }`
- `{ page: 1, type: fan,    entity: fan.living_room,         label: Fan }`

The router in the same file handles events for these items automatically (toggles, brightness presets, color chips, and fan modes/percentages when supported by the entity).

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
