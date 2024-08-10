# DIY-Zigbee-RainGauge
This is a customization from the already verywell crafted raingauge from SmartSolutionsforHome(https://www.youtube.com/@SmartSolutionsForHome)

One flaw in the homeassistent code is that when ever it restarts it adds 0,5 to the measurement, and also, requests for daily reports has been made. 
Thous have i made some code for this. 

## 1. step

You need to add the doorsensor to homeassistent. Personally i use a Sonoff Dongle E, Zigbee2MQTT and then the mqtt broker in homeassistent to add devices. All in a docker setup with Traefik (More on this)

So add the door sensor to HA

## 2. step

(I have mine i my native language, so it may differ from yours)

Add a Count helper. i have done that via the GUI. 

Go into Settings->Devices->Helpers

Then press the + button, find the counter and give it a name and press create

## 3. step
Noe its time to add som automation.
In a yaml i looks like this:

```yaml
alias: Regn Incrementer
description: ""
trigger:
  - type: opened
    platform: device
    device_id: 20a945159b3b04c6484bef0b37ff0a95
    entity_id: 3776e78b740e019c8b975ea764ba2a49
    domain: binary_sensor
  - type: not_opened
    platform: device
    device_id: 20a945159b3b04c6484bef0b37ff0a95
    entity_id: 3776e78b740e019c8b975ea764ba2a49
    domain: binary_sensor
condition: []
action:
  - service: counter.increment
    target:
      entity_id: counter.regn_taeller
    data: {}
mode: single

```

I have just created an automation in the GUI

Goto settings->Automations and then press the + button

you choose the doorsensor you just added and then choose when its open

Then duplicate it and change it to then its closed

you should now have 2, one open and one closed

in the do part choose call a service and incremnt counter +1


## 4. step
Now we need to create a template

first i have created a templates.yaml file and included this in the HA config file

```yaml
# Other config

sensor: !include templates.yaml

# Other Config
```

the templates file look like this:

```yaml
- sensor:
  - name: Rainfall [mm]
    state_class: measurement
    unique_id: rainfall_mm
    unit_of_measurement: mm
    icon: mdi:weather-pouring
    state: >-
      {% set count_rain = states('counter.regn_taeller') | int(0) %}
      {% set m_m = count_rain * 0.52615 %}
      {% if m_m | float(0) >= 0 %}
        {{ m_m | round(1, 'floor') }}
      {% else %}
         0
      {% endif %}
    availability: >-
      {{ states('counter.regn_taeller') != 'unknown' }}
```

You need to change counter.regn_taeller to the name of your counter! 

## 5. step
Now we need to create some "meters"

For this purpose i have created a new file i the same folder as homeassitant
and called it rainfall_meters.yaml . It should look like this:


```yaml
rainfall_hourly:
  source: sensor.rainfall_mm
  cycle: hourly
  unique_id: rainfall_hourly
rainfall_daily:
  source: sensor.rainfall_mm
  cycle: daily
  unique_id: rainfall_daily
rainfall_weekly:
  source: sensor.rainfall_mm
  cycle: weekly
  unique_id: rainfall_weekly
rainfall_monthly:
  source: sensor.rainfall_mm
  cycle: monthly
  unique_id: rainfall_monthly
rainfall_yearly:
  source: sensor.rainfall_mm
  cycle: yearly
  unique_id: rainfall_yearly
```

Then i have included it in the configuration file:

```yaml
# Other config

sensor: !include templates.yaml
rainfall_meters: !include rainfall_meters.yaml
# Other Config
```

## 6. step
Now add it to your dashboard. 
i have added it as a sensor but you could step it up with the mini-graph-card or something else

Enjoy! 

Feel free to reach out! 

