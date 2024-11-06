# esp8266_p1meter_VenusOS

This script came up as an idea but still needs to be tested!!!

Software for the ESP2866 that sends P1 smart meter data to an mqtt broker in a JSON format that is used by ![mr-manuel/venus-os_dbus-mqtt-grid](https://github.com/mr-manuel/venus-os_dbus-mqtt-grid). The script emulates a physical Grid/Genset/AC Load Meter in Venus OS. This makes it possible to use a you P1 smart meter in stead of, for example, VM-3P75CT. With this JSON structure there is no need for an extra step like home assistant.

## about this fork
This fork (tries) to add support for the `Landys and Gyr E360` smartmeter (DSMR5.0)

The ![original source](https://github.com/fliphess/esp8266_p1meter) has issues with DSMR5.0 meters who like to send telegrams every 1 second at a high 115200 baud rate. 
This causes the used SoftwareSerial to struggle to keep up and thus only receives corrupted messages. This fork switches to using the main Hardware serial port (RX) for communication with the meter.

# Getting started
This setup requires:
- An esp8266 (nodeMcu and Wemos d1 mini have been tested)
- A 10k ohm resistor
- A 4 pin RJ11 or [6 pin RJ12 cable](https://www.tinytronics.nl/shop/nl/kabels/adapters/rj12-naar-6-pins-dupont-jumper-adapter) Both cables work great, but a 6 pin cable can also power the esp8266 on most DSMR5+ meters.

Compiling up using Arduino IDE:
- Ensure you have selected the right board
- Using the Tools->Manage Libraries... install `PubSubClient` and `WifiManager`
- In the file `Settings.h` change `OTA_PASSWORD` to a safe secret value
- Flash the software

Compiling up using PlatformIO:
- Ensure the correct board type is selected in project configuration
- In the file `Settings.h` change `OTA_PASSWORD` to a safe secret value
- Upload the software.

Finishing off:
- You should now see a new wifi network `ESP******` connect to this wifi network, a popup should appear, else manually navigate to `192.168.4.1`
- Configure your wifi and Mqtt settings
- To check if everything is up and running you can listen to the MQTT topic `hass/status`, on startup a single message is sent.

## Connecting to the P1 meter
Connect the esp8266 to an RJ11 cable/connector following the diagram.

**Note: when using a 4-pin RJ11 connector (instead of a 6-pin connector), pin 1 and 6 are the pins that are not present, so the first pin is pin 2 and the last pin is pin 5**

| P1 pin   | ESP8266 Pin |
| ----     | ---- |
| 2 - RTS  | 3.3v |
| 3 - GND  | GND  |
| 4 -      |      |
| 5 - RXD (data) | RX (gpio3) |

On most Landys and Gyr models a 10K resistor should be used between the ESP's 3.3v and the p1's DATA (RXD) pin. Many howto's mention RTS requires 5V (VIN) to activate the P1 port, but for me 3V3 suffices.

![Wiring](https://raw.githubusercontent.com/daniel-jong/esp8266_p1meter/master/assets/esp8266_p1meter_bb.png)

### Optional: Powering the esp8266 using your DSMR5+ meter 
<details><summary>Expand to see wiring description</summary>
<p>
  
When using a 6 pin cable you can use the power source provided by the meter.
  
| P1 pin   | ESP8266 Pin |
| ----     | ---- |
| 1 - 5v out | 5v or Vin |
| 2 - RTS  | 3.3v |
| 3 - GND  | GND  |
| 4 -      |      |
| 5 - RXD (data) | RX (gpio3) |
| 6 - GND  | GND  |

![Wiring powered by meter](https://raw.githubusercontent.com/daniel-jong/esp8266_p1meter/master/assets/esp8266_p1meter_bb_PoweredByMeter.png)

</p>
</details>

## Data Sent

The nessasery metrics are put in a JSON Structure.
The software sends out to the a topic what can be configured in the settings.h file

```
{
    "grid": {
        "power": 0.0,
        "voltage": 0.0,
        "current": 50.0
        "L1": {
            "power": 0.0,
            "voltage": 0.0,
            "current": 0.0,
            "frequency": 50.0
        },
        "L2": {
            "power": 0.0,
            "voltage": 0.0,
            "current": 0.0,
            "frequency": 50.0
        },
        "L3": {
            "power": 0.0,
            "voltage": 0.0,
            "current": 0.0,
            "frequency": 50.0
        }
    }
}
```

## Thanks to

This sketch is mostly copied and pasted from several other projects.
Standing on the heads of giants, big thanks and great respect to the writers and/or creators of:

- https://github.com/jantenhove/P1-Meter-ESP8266
- https://github.com/neographikal/P1-Meter-ESP8266-MQTT
- http://gejanssen.com/howto/Slimme-meter-uitlezen/
- https://github.com/rroethof/p1reader/
- http://romix.macuser.nl/software.html
- http://blog.regout.info/category/slimmeter/
- http://domoticx.com/p1-poort-slimme-meter-hardware/
