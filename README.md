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
