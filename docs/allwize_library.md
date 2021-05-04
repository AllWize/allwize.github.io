---
title: "AllWize Library for RC1701HP"
---

The AllWize Library is an Arduino compatible library to interface the RadioCrafts RC1701HP-XXX radio module.

The library is available in the Arduino IDE Library Manager:

![Arduino IDE Library Manager](images/arduino_ide_library_manager.png)

And also in the PlatformIO Library Manager:

```
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
```

## Code examples

The library provides several examples to learn the different possibilities it offers:

* **[factoryReset](https://github.com/AllWize/allwize/blob/master/examples/factoryReset/factoryReset.ino)**: shows how to factory reset the radio module
* **[lorawan_gateway](https://github.com/AllWize/allwize/tree/master/examples/lorawan/lorawan_gateway/src)**: code to use with an ESP8266 to receive a Wize encapsulated LoRaWAN frame and send it to TTN using Semtech Legacy Protocol, uses CayenneLPP to build the payload
* **[lorawan_node](https://github.com/AllWize/allwize/blob/master/examples/lorawan/lorawan_node/lorawan_node.ino)**: code to send a LoRaWAN frame via Wize
* **[lowpower](https://github.com/AllWize/allwize/blob/master/examples/lowpower/lowpower.ino)**: example on how to lowpower the radio an microcontroller (AVR, SAMD)
* **[master_allwize_k1_esp8266](https://github.com/AllWize/allwize/blob/master/examples/master/master_allwize_k1_esp8266/master_allwize_k1_esp8266.ino)**: ESP8266 master
* **[master_allwize_k1_leonardo](https://github.com/AllWize/allwize/blob/master/examples/master/master_allwize_k1_leonardo/master_allwize_k1_leonardo.ino)**: Arduino Leonardo master
* **[master_allwize_k2](https://github.com/AllWize/allwize/blob/master/examples/master/master_allwize_k2/master_allwize_k2.ino)**: AllWize K2 master
* **[moduleInfo](https://github.com/AllWize/allwize/blob/master/examples/moduleInfo/moduleInfo.ino)**: shows radio module configuration and non-volatile memory dump
* **[slave_allwize_k1_esp32](https://github.com/AllWize/allwize/tree/master/examples/slave/slave_allwize_k1_esp32/slave_allwize_k1_esp32.ino)**: ESP32-based slave
* **[slave_allwize_k1_esp8266](https://github.com/AllWize/allwize/tree/master/examples/slave/slave_allwize_k1_esp8266/slave_allwize_k1_esp8266.ino)**: ESP8266 slave
* **[slave_allwize_k1_leonardo](https://github.com/AllWize/allwize/blob/master/examples/slave/slave_allwize_k1_leonardo/slave_allwize_k1_leonardo.ino)**: Arduino Leonardo slave
* **[slave_allwize_k1_uno](https://github.com/AllWize/allwize/blob/master/examples/slave/slave_allwize_k1_uno/slave_allwize_k1_uno.ino)**: Arduino Uno slave
* **[slave_allwize_k1_zero](https://github.com/AllWize/allwize/blob/master/examples/slave/slave_allwize_k1_zero/slave_allwize_k1_zero.ino)**: Arduino Zero / Zero Pro slave
* **[slave_allwize_k2](https://github.com/AllWize/allwize/blob/master/examples/slave/slave_allwize_k2/slave_allwize_k2.ino)**: AllWize K2 simple slave
* **[slave_allwize_k2_lpp](https://github.com/AllWize/allwize/blob/dev/examples/slave/slave_allwize_k2_lpp/slave_allwize_k2_lpp.ino)**: AllWize K2 slave using CayenneLPP to build the app payload
* **[slave_allwize_k2_mbus](https://github.com/AllWize/allwize/blob/dev/examples/slave/slave_allwize_k2_mbus/slave_allwize_k2_mbus.ino)**: AllWize K2 slave using MBUSPayload to build the app payload
* **sensor-bme280**: BME280 temperature, humidity and pressure sensor
* **sensor-hcsr04**: HC-SR04 ultrasound sensor
* **sensor-mcp9701**: MCP9701 analog temperature sensor, the one included in the AllWize K1 shield
* **sensor-mhz16**: MH-Z16 CO2 sensor
* **sensor-mics4514**: MICS4514 CO & NO sensor
* **sensor-si7021**: SI7021 temperature and humidity sensor
* **uc-vineryards**: BME280 and a soil humidity sensor for agriculture monitoring
* **uc-wcpaper**: toilet paper sensor :)
* **[wize2mqtt](https://github.com/AllWize/allwize/blob/dev/examples/wize2mqtt/wize2mqtt.ino)**: Wize to MQTT bridge that supports CSV, CayenneLPP and MBUS payload formats. Meant to run on an ESP8266-based board.
* **[wize2serial](https://github.com/AllWize/allwize/blob/dev/examples/wize2serial/wize2serial.ino)**: Wize to serial bridge that supports CSV, CayenneLPP and MBUS payload formats. 

## Reference links

* [Documentation](https://allwize.github.io/allwize/classAllWize.html AllWize Library)
* [Repository](https://github.com/AllWize/allwize AllWize Library Github)