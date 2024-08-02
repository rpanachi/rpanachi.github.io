---
layout: post
title: "Monitoring my swimming pool temperature with a cheap BLE sensor and ESPHome"
date: 2024-08-05 00:00:00
categories: [home assistant, home automation, esphome, 3dprinting, diy]
excerpt: How I used an ESP32 board to improve features of a cheap temperature sensor
disqus: true
archive: true
---

> One accurate measurement is worth a thousand expert opinions.<br/>
> – Grace Hopper

Maintaining the right temperature for my swimming pool is crucial for enjoying a good swim. Instead of frequently checking a thermometer or relying on a mobile app, I wanted to get real-time temperature updates remotely. Here's how I achieved this using an affordable BLE sensor and ESPHome.

## BLE Sensor

I found a suitable [BLE sensor on AliExpress](https://s.click.aliexpress.com/e/_mNXCHZQ) that fit my requirements perfectly:
- Power efficiency: the sensor runs on two AAA batteries, lasting about six months.
- Precision: it has good precision with an error of just 1° C approximately.
- Data collecting: it works well with its dedicated mobile app, but doesn't integrate with third-party apps.

![BLE Pool Sensor](/assets/images/Hb00c297cf3e249018102d7f62cedd77bE.png)

Despite its limitations, the sensor was perfect for my project. However, I needed a way to access the temperature data in real-time without using the mobile app or being near the pool.

## ESPHome

To bridge this gap, I used an ESP32-WROOM-32U, which I also [bought from AliExpress](https://s.click.aliexpress.com/e/_msnaRdA). The external antenna on this model ensures a reliable connection over a longer range.

![ESP-WROOM-32U](/assets/images/H718a967ecf3b48429b23ade45b4a9543n.png)

The ESP32 will work just as a BLE <> Wi-Fi bridge, connecting to the sensor on each hour, reading the temperature and sending the data to Home Assistant sensors.

Here’s how I set it up on ESPHome config:

```yaml
esphome:
  name: "esphome-pool-monitor"
  friendly_name: Pool Monitor

esp32:
  board: esp32dev
  framework:
    type: arduino

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "wmIdKCoFredactedNGeGZHeQI="

ota:

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

time:
  - platform: homeassistant
    id: homeassistant_time
    on_time:
      - seconds: 0
        minutes: 30
        then:
          - switch.turn_on: ibs_p01b_switch
          - delay: 5s
          - while:
              condition:
                and:
                  - switch.is_on: ibs_p01b_switch
                  - binary_sensor.is_off: ibs_p01b_connected
              then:
                - logger.log: "Trying connection to IBS-P01/B"
                - esp32_ble_tracker.stop_scan:
                - delay: 5s
                - esp32_ble_tracker.start_scan:
                - delay: 1min

esp32_ble_tracker:
  scan_parameters: 
    active: false
    continuous: false
    duration: 1min
  on_ble_advertise:
    - mac_address: "XX:XX:XX:XX:XX:XX"
      then:
        - lambda: |-
            ESP_LOGD("ble_adv", "IBS-P01/B device found");

button:
  - platform: template
    name: "BLE Start Scan"
    on_press:
      - esp32_ble_tracker.start_scan:
  
  - platform: template
    name: "BLE Stop Scan"
    on_press:
      - esp32_ble_tracker.stop_scan:

  - platform: restart
    name: "ESP32 Restart"

binary_sensor:
- platform: status
  name: "ESP32 Status"

- platform: template
  id: ibs_p01b_connected
  icon: mdi:bluetooth-connect
  name: "IBS-P01/B Connected"

ble_client: 
  - mac_address: XX:XX:XX:XX:XX:XX
    id: ibs_p01b
    on_connect:
      then:
        - wait_until:   
            lambda: |-
              esphome::ble_client::BLEClient* client = id(ibs_p01b);
              esphome::ble_client::BLECharacteristic* chr = client->get_characteristic(0xFFF0, 0xFFF2);

              return chr != nullptr;              
        - lambda: |- 
            ESP_LOGD("ble_client_lambda", "Connected to IBS-P01/B");
            id(ibs_p01b_connected).publish_state(true);

    on_disconnect:
      then:
        - lambda: |-
            ESP_LOGD("ble_client", "Disconnected from IBS-P01/B");
            id(ibs_p01b_connected).publish_state(false);

switch:
  - platform: ble_client
    id: ibs_p01b_switch
    ble_client_id: ibs_p01b
    name: "IBS-P01/B Enabled"
    restore_mode: ALWAYS_OFF

sensor:
  - platform: uptime
    name: "ESP32 Uptime"

  - platform: template
    device_class: timestamp
    name: "IBS-P01/B Last Measurement"
    id: "ibs_p01b_last_measurement"

  - platform: template
    name: "IBS-P01/B Temperature"
    id: ibs_p01b_temperature
    unit_of_measurement: "°C"
    accuracy_decimals: 1
    state_class: measurement
    device_class: temperature

  - platform: ble_client
    id: ibs_p01b_sensor
    type: characteristic
    ble_client_id: ibs_p01b
    service_uuid: 'fff0'
    characteristic_uuid: 'fff2'
    notify: true
    internal: true
    update_interval: 30s
    lambda: |-
      if (x.size() == 0) return NAN;

      // https://community.home-assistant.io/t/hex-string-to-dec-value/458945/2
      auto temp = (float)((int16_t)(x[1]<< 8) + x[0])/100;

      if (temp != NAN) {
        id(ibs_p01b_temperature).publish_state(temp);

        // https://community.home-assistant.io/t/publish-timestamp-into-text-sensor/122531/11
        id(ibs_p01b_last_measurement).publish_state(id(homeassistant_time).now().timestamp);

        id(ibs_p01b_switch).turn_off();
      }

      return 0.0;
```

Notes:
* For obvious reasons, I redacted the mac address of my device.
* You can find the mac address and inspect the characteristics for your sensor with apps like [BT Inspector](https://apps.apple.com/us/app/bluetooth-inspector/id1509085044) or
  [LightBlue](https://play.google.com/store/apps/details?id=com.punchthrough.lightblueexplorer&hl=en).
* `IBS-P01/B` is the name/model of my sensor. You can use other devices for this
  purpose since they have bluetooth support.
* I'll not cover the details for setting up ESPHome and devices, but you can
  watch a [detailed tutorial from Everything Smart Home Youtube
  channel](https://www.youtube.com/watch?v=iufph4dF3YU).

After uploading the firmware to ESP32 and with a little luck, you will see something like that on the logs:

```
[16:24:04][D][switch:012]: 'IBS-P01/B Enabled' Turning ON.
[16:24:04][D][switch:055]: 'IBS-P01/B Enabled': Sending state ON
[16:24:05][D][button:010]: 'BLE Start Scan' Pressed.
[16:24:05][D][esp32_ble_tracker:266]: Starting scan...
[16:24:11][D][ble_adv:069]: IBS-P01/B device found
[16:24:11][D][esp32_ble_client:110]: [0] [XX:XX:XX:XX:XX:XX] Found device
[16:24:11][D][esp32_ble_tracker:665]: Found device XX:XX:XX:XX:XX:XX RSSI=-82
[16:24:11][D][esp32_ble_tracker:686]:   Address Type: PUBLIC
[16:24:11][D][esp32_ble_tracker:215]: Pausing scan to make connection...
[16:24:11][D][esp32_ble_tracker:303]: End of scan.
[16:24:11][I][esp32_ble_client:067]: [0] [XX:XX:XX:XX:XX:XX] 0x00 Attempting BLE connection
[16:24:24][I][ble_sensor:031]: [ibs_p01b_sensor] Connected successfully!
[16:24:26][I][esp32_ble_client:227]: [0] [XX:XX:XX:XX:XX:XX] Connected
[16:24:26][D][ble_client_lambda:115]: Connected to IBS-P01/B
[16:24:26][D][binary_sensor:036]: 'IBS-P01/B Connected': Sending state ON
[16:24:32][D][sensor:094]: 'IBS-P01/B Temperature': Sending state 22.77000 °C with 1 decimals of accuracy
[16:24:32][D][sensor:094]: 'IBS-P01/B Last Measurement': Sending state 1722626688.00000  with 1 decimals of accuracy
[16:24:32][D][sensor:094]: 'ibs_p01b_sensor': Sending state 0.00000  with 0 decimals of accuracy
[16:25:02][D][sensor:094]: 'IBS-P01/B Temperature': Sending state 22.77000 °C with 1 decimals of accuracy
[16:25:02][D][sensor:094]: 'IBS-P01/B Last Measurement': Sending state 1722626688.00000  with 1 decimals of accuracy
[16:25:02][D][sensor:094]: 'ibs_p01b_sensor': Sending state 0.00000  with 0 decimals of accuracy
[16:25:43][D][ble_client:121]: Disconnected from IBS-P01/B
[16:25:43][D][binary_sensor:036]: 'IBS-P01/B Connected': Sending state OFF
[16:25:43][W][ble_sensor:037]: [ibs_p01b_sensor] Disconnected!
```

And, of course, the data in Home Assistant device:

![Home Assistant sensor](/assets/images/yXf3chgQ.png)

## Wrapping Up

It's working but it's not finished. I need to keep the ESP32 turned on and near the pool, running continuously.

I used a standard USB 5V DC adapter plugged into a wall outlet near the pool. To protect the hardware from the elements, I 3D printed a box to accommodate both the ESP32 and the DC adapter:

![ESP32 BLE Pool Monitor](/assets/images/esp32-pool-monitor.jpg)

If you want to print the same box for your project, [download the model from my Printables profile](https://www.printables.com/model/962422-esp32-poll-monitor-box).

This setup has been running flawlessly for the past 10 months, providing me with accurate and (almost) real-time water temperature, making pool maintenance much easier.

Enjoy o/

## References

* <https://www.home-assistant.io/integrations/esphome>
* <https://esphome.io/guides/getting_started_hassio.html>
* <https://esphome.io/components/ble_client.html>
