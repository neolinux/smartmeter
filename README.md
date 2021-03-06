# Send smartmeter data via LoRa/TTN to your own webserver

This little gadget collects data packages from a smartmeter via the IR-interface and transmits them every minute via LoraWan/The Things Network to a private webserver (or any other integration like Cayenne etc.). It's battery powered with standby consumption of 50-100µA.

This is the basic setup
![Layout](img/layout.png?raw=true)

The software is quite quick and dirty and the hardware amateur level. The project was mainly meant to be a prove of concept. I wanted to check if LoRa can transmit smart meter readings from within a metal container in the basement behind at least four walls of massive ferro-concrete. And hey, it works :-)


## The hardware

Hardware costs should be in the range of 20 €, so it's quite cheap. It consists of 
- an Arduino Pro Mini, a RFM95 board for Lora Transmission, 
- a RFM95 for LoRa transmission
- an inverter plus Optoschmitt detector to receive the infrared-data from the smartmeter
- a voltage divider to monitor the battery

![This is how it looks](img/device2.jpg?raw=true)
This is the source I used most.

[Mini-Lora](https://things4u.github.io/HardwareGuide/Arduino/Mini-Sensor-HTU21/mini-lora.html)

As the inverter and the Schmitt-Trigger draw quite some current (>10mA), they are switched off during standby. The LED is also desoldered of course. I guess the challenge for battery life will be the consumption during the wake phase, not the idle power.

## The software

The arduino sketch uses the LMIC library to transmit the data. 
Parsing of the collected IR-data, which is in [SML](https://de.wikipedia.org/wiki/Smart_Message_Language), is done in a small smartmeter.cpp library.
Depending on your meter you might need to adjust some numbers here. A full implementation of the SML stack would have taken too much memory, so he reads a package of ~300 bytes and searches for relevant Hex-Codes.
Best thing is to just read out some SML packages and see if parameters need to be adjusted. I have a EHZ EDL Smartmeter. 

The smartmeter data is encoded into the TTN payload:
- 8 bytes energy (total) in kWh/10000
- 4 bytes actual power consumption in W 
- 2 bytes battery voltage in mV

This is how it should look like in the TTN console:
![TTN Console](img/TTN_Console.png?raw=true)

In TTN I use the HTTP integration to send the data to my webserver (it's plain hosting, so no access to MQTT).
![TTN Integration](img/TTN_http.png?raw=true)

On the webserver, the [Volkszaehler](https://www.volkszaehler.org/) is installed. A little script (which needs some additional security features) sends the data to the Volkszaehler middleware.

Et voilá :-)
![Webinterface](img/webinterface.png?raw=true)



