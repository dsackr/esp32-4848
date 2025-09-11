# esp32-4848
I bought this panel from ali - https://www.aliexpress.us/item/3256806149272881.html

It comes with a kind of demo program that allows you to connect to your wifi and controls the 3 relays (I assume 1 relay if you get that version). You couldn't configure the screen in anyway. I flashed it with a yaml file I found online and the screen went blank. I spent a lot of hours trying to get this to work at all and after a lot of failures:

https://nightly.openhasp.com/

flash it with Guition ESP32-S3-4848S040 from openhasp.com's nightly builds - then configure it for your wifi. Connect to the device over web and configure the gpio outputs for pins 1, 2, and 40 as power relays and put them in groups 1, 2, and 3 respectively. 

then go to file editor and update it with the code from pages.jsonl.

You now have a working screen with 3 relays (wire your lights, fan, whatever) to those relays and you can power them on and off. BUT you also have the ability to now use MQTT with Home Assistant to build new screens and do cool things. There are 4 files - relays.yaml to create the 3 relays in Home Assistant as switches, one builder yaml, one router yaml, and a painter yaml. 

I put all 4 yaml files in a packages directory and my configuration.yaml file has this includes line to pull in packages:

# Enable packages (add this line)
homeassistant:
  packages: !include_dir_named packages

**Node Name (hasp_node)**
- Default: `plate`. Update to your device name from the openHASP web UI (Device name). MQTT topics will be `hasp/<node>/...`.
- Update in files:
  - `openhasp_builder.yaml:7` — `script.panel_build_full_ui` variables
  - `openhasp_builder.yaml:140` — `automation` variables for bootstrap
  - `openhasp_router.yaml:9` — `automation` variables
- All MQTT publishes in the builder and router now use `hasp_node`, so changing it in those spots is sufficient.

Note: The discovery file is `openhasp_discovery.yaml` (this is what the README previously called `relays.yaml`) and it publishes three relays to Home Assistant.

## Development

- Branch `GPT`: working branch for automated/Codex-driven changes. This note ensures a minimal diff so a PR can be opened.
  - Current PR: opened from branch `GPT` to `main`.

## Dynamic Package (New)

- File: `openhasp_package.yaml`
- Install: place in your Home Assistant `packages` folder (with `homeassistant: packages: !include_dir_named packages` in `configuration.yaml`).
- Configure:
  - Device name: set `input_text.hasp_node` to your device’s node (e.g., `dash`).
  - Theme: select `input_select.hasp_theme` (dark or light). Changing it rebuilds the UI automatically.
  - Pages: by default, includes a Home page with three relay buttons. Extend the `pages` variable inside the script `script.panel_build_dynamic_ui` by adding more page definitions and JSONL elements. Colors reference the selected theme.
- Boot behavior: on device boot (uptime < 15s), the package rebuilds the UI for the configured node.
- Event routing: `panel_event_router_dynamic` maps button presses to entities via the `controls` list (defaults to 3 relays). Add entries for your added controls.

## Usage Overview

- Install the package file in Home Assistant and enable packages.
- Set your device node in `input_text.hasp_node` (e.g., `dash`).
- Choose a theme in `input_select.hasp_theme` (dark or light).
- Reboot the panel or run `script.panel_build_dynamic_ui` to build the UI.

## Quick Start Steps

- Enable packages in HA (`homeassistant: packages: !include_dir_named packages`).
- Copy `openhasp_package.yaml` into your `packages` folder.
- In HA, set `input_text.hasp_node` to match your openHASP device name.
- Select a theme in `input_select.hasp_theme`.
- Run `script.panel_build_dynamic_ui` once to build immediately, or power‑cycle the device.

## Configure Pages And Routing

- Pages: edit `script.panel_build_dynamic_ui` → `pages` list to add elements (each line is a JSONL object published to the device over MQTT).
- Routing: edit `automation.panel_event_router_dynamic` → `controls` list to map UI control IDs (e.g., `p1b101`) to HA entities and a domain (`switch`, `light`).
- Add more controls by extending both the `pages` list (to place the control on screen) and the `controls` list (to define what it controls).

## Devices (Templatized Widgets)

- Define your devices once and let the package generate the correct UI + routing based on capabilities.
- Edit `devices` under `script.panel_build_dynamic_ui` (and matching `devices` under the dynamic router).

Example:
- devices:
  - { page: 1, type: switch, entity: switch.larry, label: Larry }
  - { page: 1, type: light,  entity: light.studio_lights, label: Studio }
  - { page: 1, type: fan,    entity: fan.living_room,     label: Fan }

What gets built:
- Switch: a toggle button that calls `homeassistant.turn_on/off`.
- Light: a toggle button; if brightness is supported, a 4-step brightness matrix; if color is supported, color chips (R/G/B/W) that call `light.turn_on` with appropriate parameters.
- Fan: either a preset btnmatrix using `preset_modes`, or a percentage btnmatrix (Off/Low/Med/High) based on capabilities.

### Examples

- Add a light toggle button
  - Page element (add to `pages[0].elements` or your chosen page):
    - `{"page":1,"obj":"btn","id":132,"x":60,"y":268,"w":88,"h":88,"radius":44,"text":"\uE425","text_font":75}`
  - Routing (add to `controls` list):
    - `{ id: p1b132, domain: light, entity: light.living_room }`

- Add a fan speed btnmatrix (Off/Low/Med/High)
  - Page element:
    - `{"page":2,"obj":"btnmatrix","id":22,"x":264,"y":116,"w":176,"h":48,"text_font":16,"options":["Off","Low","Med","High"],"toggle":1,"one_check":1,"val":0,"radius":10}`
  - Router logic (append under the automation’s `choose` list):
    - conditions: `{{ id == 'p2m22' and event in ['changed','up'] }}`
      - choose Off -> `fan.turn_off`; otherwise `fan.set_percentage` with mapping `{0:0,1:50,2:75,3:100}`

- Add a shade control btnmatrix (Open/Stop/Close)
  - Page element:
    - `{"page":3,"obj":"btnmatrix","id":52,"x":48,"y":356,"w":384,"h":44,"text_font":32,"options":["\uE143","\uE4DB","\uE140"],"one_check":0,"radius":10}`
  - Router logic (append under the automation’s `choose` list):
    - conditions: `{{ id == 'p3m52' and event in ['up','changed'] }}`
      - `val == 0` → `cover.open_cover`
      - `val == 1` → `cover.stop_cover`
      - `val == 2` → `cover.close_cover`

## Themes

- `input_select.hasp_theme` controls a dark or light palette applied to the UI.
- Changing the theme triggers a rebuild to apply colors across all elements.

## Files In This Repo

- `openhasp_package.yaml`: dynamic, configurable package (recommended).
- `openhasp_builder.yaml`, `openhasp_router.yaml`, `openhasp_discovery.yaml`: legacy static approach (reference while migrating).
- `openhasp_painter.yaml`: reserved for future drawing/styling utilities.

## Troubleshooting

- Blank screen: verify `input_text.hasp_node`, MQTT connectivity, and run `script.panel_build_dynamic_ui`.
- Touch does nothing: add the control’s ID to the `controls` list and point it at the correct entity.
- Title doesn’t update: confirm the device publishes `hasp/<node>/state/page` events; the router updates the header on page changes.
