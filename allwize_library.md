The AllWize Library is an Arduino compatible library to interface the RadioCrafts RC1701HP-XXX radio module.

The library is available in the Arduino IDE Library Manager:

[[File:Arduino-ide-library-manager.png|800x600px|none|left|Arduino IDE Library Manager]]

And also in the PlatformIO Library Manager:

<pre>
$ pio lib search allwize
Found 1 libraries:

AllWize
=======
#ID: 5804
Arduino-compatible library to interface RC1701HP-OSP/WIZE radio modules

Keywords: wize, radio, radiocrafts, wmbus4
Compatible frameworks: Arduino
Compatible platforms: Atmel AVR, Atmel SAM, Espressif 32, Espressif 8266
Authors: AllWize.io, Xose PÃ©rez
</pre>

== Code examples ==

The library provides several examples to learn the different possibilities it offers:

* '''[https://github.com/AllWize/allwize/blob/master/examples/factoryReset/factoryReset.ino factoryReset]''': shows how to factory reset the radio module
* lorawan: example on how to use Wize to encapsulate a LoRaWAN frame and send it to The Things Network
** '''[https://github.com/AllWize/allwize/tree/master/examples/lorawan/lorawan_gateway/src lorawan_gateway]''': code to use with an ESP8266 to receive a Wize encapsulated LoRaWAN frame and send it to TTN using Semtech Legacy Protocol, uses CayenneLPP to build the payload
** '''[https://github.com/AllWize/allwize/blob/master/examples/lorawan/lorawan_node/lorawan_node.ino lorawan_node]''': code to send a LoRaWAN frame via Wize
* '''[https://github.com/AllWize/allwize/blob/master/examples/lowpower/lowpower.ino lowpower]''': example on how to lowpower the radio an microcontroller (AVR, SAMD)
* master: different examples on how to create a master (single channel gateway) with the library
** '''[https://github.com/AllWize/allwize/blob/master/examples/master/master_allwize_k1_esp8266/master_allwize_k1_esp8266.ino master_allwize_k1_esp8266]''': ESP8266 master
** '''[https://github.com/AllWize/allwize/blob/master/examples/master/master_allwize_k1_leonardo/master_allwize_k1_leonardo.ino master_allwize_k1_leonardo]''': Arduino Leonardo master
** '''[https://github.com/AllWize/allwize/blob/master/examples/master/master_allwize_k2/master_allwize_k2.ino master_allwize_k2]''': AllWize K2 master
* '''[https://github.com/AllWize/allwize/blob/master/examples/moduleInfo/moduleInfo.ino moduleInfo]''': shows radio module configuration and non-volatile memory dump
* slave: different examples on how to create a slave node with the library
** '''[https://github.com/AllWize/allwize/tree/master/examples/slave/slave_allwize_k1_esp32/slave_allwize_k1_esp32.ino slave_allwize_k1_esp32]''': ESP32-based slave
** '''[https://github.com/AllWize/allwize/tree/master/examples/slave/slave_allwize_k1_esp8266/slave_allwize_k1_esp8266.ino slave_allwize_k1_esp8266]''': ESP8266 slave
** '''[https://github.com/AllWize/allwize/blob/master/examples/slave/slave_allwize_k1_leonardo/slave_allwize_k1_leonardo.ino slave_allwize_k1_leonardo]''': Arduino Leonardo slave
** '''[https://github.com/AllWize/allwize/blob/master/examples/slave/slave_allwize_k1_uno/slave_allwize_k1_uno.ino slave_allwize_k1_uno]''': Arduino Uno slave
** '''[https://github.com/AllWize/allwize/blob/master/examples/slave/slave_allwize_k1_zero/slave_allwize_k1_zero.ino slave_allwize_k1_zero]''': Arduino Zero / Zero Pro slave
** '''[https://github.com/AllWize/allwize/blob/master/examples/slave/slave_allwize_k2/slave_allwize_k2.ino slave_allwize_k2]''': AllWize K2 simple slave
** '''[https://github.com/AllWize/allwize/blob/dev/examples/slave/slave_allwize_k2_lpp/slave_allwize_k2_lpp.ino slave_allwize_k2_lpp]''': AllWize K2 slave using CayenneLPP to build the app payload
** '''[https://github.com/AllWize/allwize/blob/dev/examples/slave/slave_allwize_k2_mbus/slave_allwize_k2_mbus.ino slave_allwize_k2_mbus]''': AllWize K2 slave using MBUSPayload to build the app payload
* use_cases: different slaves using specific sensors for specific use cases
** '''sensor-bme280''': BME280 temperature, humidity and pressure sensor
** '''sensor-hcsr04''': HC-SR04 ultrasound sensor
** '''sensor-mcp9701''': MCP9701 analog temperature sensor, the one included in the AllWize K1 shield
** '''sensor-mhz16''': MH-Z16 CO2 sensor
** '''sensor-mics4514''': MICS4514 CO & NO sensor
** '''sensor-si7021''': SI7021 temperature and humidity sensor
** '''uc-vineryards''': BME280 and a soil humidity sensor for agriculture monitoring
** '''uc-wcpaper''': toilet paper sensor :)
* '''[https://github.com/AllWize/allwize/blob/dev/examples/wize2mqtt/wize2mqtt.ino wize2mqtt]''': Wize to MQTT bridge that supports CSV, CayenneLPP and MBUS payload formats. Meant to run on an ESP8266-based board.
* '''[https://github.com/AllWize/allwize/blob/dev/examples/wize2serial/wize2serial.ino wize2serial]''': Wize to serial bridge that supports CSV, CayenneLPP and MBUS payload formats. 
* '''[https://github.com/AllWize/allwize/blob/dev/examples/wize2thethingsio/wize2thethingsio.ino wize2thethingsio]''': Wize to TheThingsIO bridge that supports CSV, CayenneLPP and MBUS payload formats. Meant to run on an ESP8266-based board.

== Reference links ==

* [https://allwize.github.io/allwize/classAllWize.html AllWize Library Documentation]
* [https://github.com/AllWize/allwize AllWize Library Github Repository]