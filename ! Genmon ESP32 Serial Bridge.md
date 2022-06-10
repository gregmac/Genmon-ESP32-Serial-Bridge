I bought a house with a Generac generator installed, and so of course I wanted to get it wired into my network (and ultimately connected to [Home Assistant](https://www.home-assistant.io/)). The [Genmon](https://github.com/jgyates/genmon) project is perfect for this.

## ESP32

Normally Genmon is installed on a Pi that's connected to the generator, but I decided to take a different approach and use a simple, cheap [ESP32](https://www.espressif.com/en/products/socs/esp32) as an RS232-wifi bridge while running Genmon on a VM inside the house.

1. Though the Pi can apparently work fine below its rated operating temperature of 0°C, the ESP32 is rated for –40°C to +125°C. 
2. The Pi is supposedly suspectiple to electromagnetic interference from the generator operating; I assume the ESP32 being much simpler is less susceptible 
3. The ESP32 uses much less power (can run from the 5V the generator's AUX port provides)
4. The ESP32 is significantly smaller and along with all the above, is much easier to mount and connect

I specifically went with the ESP32-WROOM-32U devkit board -- which has an external antenna -- to get a decent signal from the all-metal generator enclosure.

> I did initially prototype using a regular ESP32 with a PCB antenna; it had a Wifi signal around -85 dBm and around 25% packet loss. Switching to an external antenna raised the signal to -62 dBm and 0% packet loss.

## Hardware

* ESP32-WROOM-32U devkit board
* External screw-on antenna
* 8-pin ATX EPS (CPU power) extension cable -- only need male end
* MAX3232 RS232<>TTL converter module
* Enclosure

### Optional

Temperature sensors:

* 2x DS18B20 one-wire temperature sensor
* 1x 4.7k resistor

### Wiring

I did all the wiring inline, which was OK before I added the one-wire temperature sensors and ended up changing the ESP32. If I did it over again I'd do the connections on a small perfboard with sockets to plug in the ESP32.

EPS connector:

* Pin 1: +5V ↔ ESP32 VCC in
* Pin 2: Ground
* Pin 7: RS232 TX
* Pin 8: RS232 RX

MAX3232:

* RS232 Out ↔ EPS Pin 7
* RS232 In ↔ EPS Pin 8
* TTL In ↔ ESP32 TX (GPIO1)
* TTL Out ↔ ESP32 GPIO22
* Power + ↔ ESP32 3V3
* Power - ↔ Ground

> Note: I used GPIO22 instead of RX because I was having issues with receiving data which was a problem in my test setup, but I had changed the pin already. RX should work fine.

One-wire temperature:

* Each sensor:
  * VCC ↔ ESP32 3V3
  * Data ↔ ESP32 GPIO4
* 4.7k resistor between VCC and Data (only one)

All grounds need to be tied together.

## ESP32 Software

I use [ESPHome](https://esphome.io/) to configure this, mainly because I have several other ESP32's.

This uses:
* [UART Bus](https://esphome.io/components/uart.html)
* [Stream server for ESPHome
](https://github.com/oxan/esphome-stream-server)
* [Dallas Temperature Sensor
](https://esphome.io/components/sensor/dallas.html)

See `esphome.yaml` for the code.

## Genmon

### Install on Alpine Linux

I run genmon in an LXC container under [Proxmox](https://www.proxmox.com/), on Alpine 3.11

1. `apk add bash python3 sudo make gcc build-base libressl-dev python3-dev rust py3-pip`
2. `pip install -U pip` (needs v22+ for the python `cryptography` package)
3. `cd /opt && git clone https://github.com/jgyates/genmon.git && cd /opt/genmon`
4. `./genmonmaint.sh -i`
 
### Configuration

Start genmon, and in the UI, set:

* **Serial Server TCP/IP Address** = IP of the ESP32
* **Serial Server TCP/IP port** = `6638` (or whatever port the ESP32 stream server uses)

_This can also be done via `/etc/genmon/genmon.conf`_

Addons:

* Enable **MQTT** and configure server address
* **Blacklist Filter** = `Run Time,Monitor Time,Generator Time,Platform Stats,Communication Stats`
* **Flush Interval** = `60`
* **JSON for Numerics** = on

## Home Assistant

Add `homeassistant-generator.yaml` to `packages` folder and restart Home Assistant

