# Introduction

My home was equipped with a ComfoFan S mechanical ventilator that pulls air from across various rooms in the house. The fan had both a wired and an RF remote (Zehnder) to control its operation, allowing for three predefined speeds. The reason I wanted to connect the fan to Home Assistant (HA) was because I had some automations in mind to control its functioning. The easiest solution for this would be to directly connect a 0-10 V control to the fan itself, but I had some concerns about this:

- Warranty on the service contract for the fan
- Insurance liabilities
- It would render the existing RF remote useless and I didn't want to substitute the remote so that the process would be 'invisible' to my girlfriend

So I went for the slightly more complicated route to build a HA-connected remote that could exist in parallel to the existing remote. For this, I leaned on the work done by [@golles](https://github.com/golles/ESPHome-Config/blob/main/docs/MECHANISCHE_VENTILATIE.md), [@Sanderhuisman](https://github.com/Sanderhuisman/ESPHome-Zehnder-RF) and [JBSweb](https://www.jbswebcom.nl/knutselen/index.php?view=article&id=32:zehnder-comfofan-silent-aansturen-met-home-assistant-via-wemos-d1-met-nrf905-en-esphome&catid=2), which made the job pretty easy.

# Preparation

The first thing you need to build this is the components and proper setup in HA. I used the following:

## Hardware
- ESP32-WROOM-32 board
- NRF905 transceiver
- A big pack of jumper wires (female-to-female), because I went no-solder
- A USB-C charger and cable
- A junction box

## Software

- HA installation
- ESPHome add-on in HA

# Building

## Wiring up the boards
Using the jumper wires, I connected the ESP32 board to the NRF905 module via the following wiring:

| nRF905 pin | ESP32 pin | ESP32 GPIO |
| :--------: | :-------: | :--------: |
|    Vcc     |   3.3V    |    3.3V    |
|    Gnd     |    Gnd    |    Gnd     |
|     AM     |    D32    |  GPIO 32   |
|     CD     |    D33    |  GPIO 33   |
|     CE     |    D27    |  GPIO 27   |
|     DR     |    D35    |  GPIO 35   |
|    PWR     |    D26    |  GPIO 26   |
|   TX_EN    |    D25    |  GPIO 25   |
|    MOSI    |    D13    |  GPIO 13   |
|    MISO    |    D12    |  GPIO 12   |
| SCK |    D14    |  GPIO 14   |
| CSN  |    D15    |  GPIO 15   |

## Installing ESPHome
I connected the ESP32 to my PC. Via the ESPHhome add-on, I created a binary (Modern), which I downloaded and flashed to the board using ESPHome Web. This allowed the board to connect to WiFi, so after that I used it as a standalone device. 

## Configuring the device
Following the approach of @golles, I then used the ESPHome add-on to add the following to my ```secrets.yaml```:

``` yaml
wifi_ssid: ["name"]
wifi_password: ["password"]


gateway: [ip]
subnet: 255.255.255.0
ap_password: "ap_pw"
mechanische_ventilatie_ota_password: ["password"]
mechanische_ventilatie_static_ip: [192.168.1.132]
mechanische_ventilatie_encryption_key: ["key"]
```
Then for my device I added '''mechfan-remote.yaml''' with the following content, again very much on the basis of the work of @golles. Note that I include external components from his Github repository, as he has been maintaining the code that allows the ESP32 and NRF905 to communicate properly with the fan. One thing I did not get right in the beginning was to properly set the board type. For the ESP32-WROOM-32 that is dinky32. Please also be aware the configuration requires keys/passwords that you should have obtained while setting up ESPHome.

``` yaml
esp32:
  board: denky32
  framework:
    type: arduino

substitutions:
  name: mechanische_ventilatie
  friendly_name: Mechanische ventilatie
  static_ip: !secret mechanische_ventilatie_static_ip
  version: v0.5

esphome:
  name: ${name}
  friendly_name: ${friendly_name}
  project:
    name: tijms.${friendly_name}
    version: ${version}


external_components:
  - source:
      type: git
      url: https://github.com/golles/ESPHome-Config
      ref: main
    components: all
      

api:
  encryption:
    key: "[insert key]"
  services:
    - service: set_speed_timer
      variables:
        speed: int
        timer: int
      then:
        - lambda: |-
            id(${name}_fan).setSpeed(speed, timer);

spi:
  clk_pin: GPIO14
  mosi_pin: GPIO13
  miso_pin: GPIO12

nrf905:
  id: nrf905_rf
  cs_pin: GPIO15
  am_pin: GPIO32
  cd_pin: GPIO33
  ce_pin: GPIO27
  dr_pin: GPIO35
  pwr_pin: GPIO26
  txen_pin: GPIO25

fan:
  - platform: zehnder
    id: ${name}_fan
    name: Ventilatie
    nrf905: nrf905_rf
    update_interval: 10s

button:
  - platform: template
    id: ${name}_high_15
    name: 15 minuten hoog
    on_press:
      then:
        lambda: |-
          id(${name}_fan).setSpeed(4, 15);

  - platform: template
    id: ${name}_high_30
    name: 30 minuten hoog
    on_press:
      then:
        lambda: |-
          id(${name}_fan).setSpeed(4, 30);

  - platform: template
    id: ${name}_high_60
    name: 60 minuten hoog
    on_press:
      then:
        lambda: |-
          id(${name}_fan).setSpeed(4, 60);



# Enable logging
logger:

ota:
  - platform: esphome
    password: "[insert password]"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Mechfan-Remote Fallback Hotspot"
    password: "[insert password]"

```
Flashing this to the ESP32 using the Update button in the ESPHome add-on will make the board ready to pair.

# Pairing
To put the ComfoFan S in pairing mode, you will need to power it off first. Once you restart the device, it will spend some time looking for devices that wish to pair to it. It may be necessary to simultaneously reboot the ESP32 for succesful pairing. The '''mech-remote.yaml''' above creates some buttons you can use to test whether you've succesfully paired the ESP32/NRF905 with the mechanical fan.
