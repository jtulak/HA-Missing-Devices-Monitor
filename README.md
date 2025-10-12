# HomeAssistant Missing Devices Monitor (Script Blueprint)

Monitor your Home Assistant installation for devices that disappear, and receive a tidy, grouped notification whenever something goes missing. [HA forum thread](https://community.home-assistant.io/t/missing-devices-monitor-script/939695)

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjtulak%2FHA-Missing-Devices-Monitor%2Fblob%2Fmain%2Fmissing_devices.yaml)

## Features

- **Persistent notification** summarising all devices whose representative entity stays `unavailable` or `unknown` longer than allowed.
- **Area-aware grouping** so you instantly see where outages are happening.
- **Split thresholds**: fast vs. slow alert buffers, so if you have devices that report once a day or only when something changes, they can use different grace periods.
- **Automatic opt-outs** for devices/entities that are disabled in HA.
- **Startup guard** that skips the scan until HA has been up long enough (optional).

## Report example
The links point to the offending device in an actual report. The one [slow] example device is configured to use the longer buffer delay and the lines limit is set to 4.

> **Missing devices: 10**<br>
> Balcony<br>
> • [IKEA switch Christmass lights Switch](http://) — unavailable since *2h 46m 32s*
> 
> Workshop<br>
> • [OctoPrint Current File Size](http://)  — unavailable since *2h 46m 26s*
> 
> Bedroom<br>
> • [Window sensor Battery](http://) — unavailable since *2h 46m 32s* [slow]<br>
> • [Small lamp next to bed Light](http://) — unavailable since *2h 46m 32s*
>
> … 6 more not shown


## Optional Dependency: Uptime Integration

The blueprint uses the built-in `sensor.uptime` entity to skip its first run while Home Assistant is still starting.  
If the [Uptime integration](https://www.home-assistant.io/integrations/uptime) is unavailable, the script simply runs immediately—no failures, just no startup grace.

## Setup

1. Install via link to this repo, or place `missing_devices_blueprint.yaml` under: `config/blueprints/script/<namespace>/missing_devices.yaml`
    1. If installing manually, reload blueprints (Settings ▸ Automations & Scenes ▸ Blueprints ▸ “Reload blueprints”) or restart HA.
2. Create a new script from **Missing Devices Monitor**:
    1. Pick alert buffers, grace window, and domains.
    2. Choose the notification ID and line limit.
    3. Use the device selector to ignore or flag slow talkers, and/or paste `device_id: note` entries (`device_id` is part of the URL when you look at individual devices).
3. Save the script and call it manually or via automation (e.g. on a time pattern). If using Uptime and your HA starts up, it will just keep aborting until enough time (that you set up) elapsed to avoid flagging slowly-initializig devices.

## How It Works

- The blueprint scans the configured domains of entities and picks a single entity per device.
- It filters entities that are currently `unavailable`/`unknown`, skips disabled or ignored devices, and calculates how long each has been in that state.
- If the duration exceeds the chosen buffer (slow talkers use the slow buffer, everyone else the fast one), the device is flagged.
- The notification message groups flagged devices by area and renders each line using the entity’s friendly name (or the device name if the entity lacks one), so the alert shows the name you see in the UI, not the raw entity id.
- When the script finds no missing devices, it dismisses the notification; otherwise, it updates/creates one with the grouped report.

I have over a thousand of entities and the runtime of this blueprint on a low-end Odroid board is about 0.2 second. If you have significantly more entities and you are not sure how it will be handled performance wise, you can first limit the number of entities processed and then increase it in steps. Also, I suggest running this script manually first, before actually creating an automation, because while I took care to avoid memory-hogging, it is not pleasant to be locked out the moment your automations initialize after startup and trigger OOM. Ask me how I know. :-) 


