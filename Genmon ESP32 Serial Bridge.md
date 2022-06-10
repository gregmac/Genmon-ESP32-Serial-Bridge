I bought a house with a Generac generator installed, and so of course I wanted to get it wired into my network (and ultimately connected to [Home Assistant](https://www.home-assistant.io/)). The [Genmon](https://github.com/jgyates/genmon) project is perfect for this.

## ESP32

Normally Genmon is installed on a Pi that's connected to the generator, but I decided to take a different approach and use a simple, cheap [ESP32](https://www.espressif.com/en/products/socs/esp32) as an RS232-wifi bridge while running Genmon on a VM inside the house.

1. Though the Pi can apparently work fine below its rated operating temperature of 0°C, the ESP32 is rated for –40°C to +125°C. 
2. The Pi is supposedly suspectiple to electromagnetic interference from the generator operating; I assume the ESP32 being much simpler is less susceptible 
3. The ESP32 uses much less power (can run from the 5V the generator's AUX port provides)
4. The ESP32 is significantly smaller and along with all the above, is much easier to mount and connect
