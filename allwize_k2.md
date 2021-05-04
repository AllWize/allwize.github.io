The AllWize K2 is an Arduino(TM) MKR compatible board based on the SAMD21G microchip and a RadioCrafts RC1701HP module.

[[File:20190504_231200s.jpg|800x600px|none|left|AllWize K2 RevA]]


== Reference links ==

* [https://www.allwize.io/product-page/the-allwize-k2 AllWize K2 product page]
* [https://github.com/AllWize/AllWizeK2-hardware AllWize K2 Github repository]
* [https://github.com/AllWize/AllWizeK2-hardware/raw/master/RevPA/AllWize%20K2%20-%20RevPA.pdf AllWize K2 Schematic (PDF)]

== Schematic ==

[[File:Allwize_k2_schematic.png|800x600px|none|left|AllWize K2 RevPA Schematic]]

== Features ==

* '''Arduino MKR form factor''', pin-compatible
* '''RadioCrafts RC1701HP-XXX''' radio module supporting Wize protocol
* '''iPEX antenna connector''' with the option to add an iPEX connector for an external antenna

== Description ==

The AllWize K2 is a prototyping board with the same form factor as the MKR series by Arduino. It is based on a SAMD21G microcontroller by Atmel/Microchip which is much more powerful than the 8-bits microcontrollers like the ATMega328 in the Uno or Leonardo.

ATSAMD21G128A Features:

* ARM Cortex M0+
* 32-bit architecture
* 48MHz processor speed
* 256Kb flash memory (program space)
* 32Kb SRAM memory
* 32Kb emulated EEPROM
* 3.3V operating voltage
* 38 GPIOs
* 14 12-bits ADC
* 1 DAC
* 12 channels DMA
* USB controller

The radio module, as with the AllWize K1, is a Radiocrafts RC1701HP with the WIZE firmware loaded (version 1.0 as of today).

== Installation ==

=== Arduino IDE ===

Starting with 1.6.4, Arduino allows installation of third-party platform packages using Boards Manager.

* Install the current upstream Arduino IDE at the 1.8.7 level or later. The current version is on the Arduino website.
* Start Arduino and open the Preferences window.
* Enter https://raw.githubusercontent.com/AllWize/allwize-boards/master/package_allwize_boards_index.json into the Additional Board Manager URLs field. You can add multiple URLs, separating them with commas.


[[File:Arduino-ide-preferences.png|800x600px|none|left|Arduino IDE Preferences]]


* Open Boards Manager from Tools > Board menu and install "'''Allwize SAMD Boards (32-bits ARM Cortex-M0+)'''"


[[File:Arduino-ide-board-manager.png|800x600px|none|left|Arduino IDE Board Manager]]


* Don't forget to select your AllWize board from Tools > Board menu after installation.


[[File:Arduino-ide-board-selection.png|800x600px|none|left|Arduino IDE Board Selection]]


* Then you will need to install the '''AllWize library''' from the Arduino IDE Library Manager.


[[File:Arduino-ide-library-manager.png|800x600px|none|left|Arduino IDE Library Manager]]


=== Platform IO ===

PlatformIO compatibility is on the way (see https://github.com/platformio/platform-atmelsam/issues/71). 

In the meantime you have two options to use the AllWize K2 from PlatformIO:

==== Manually add support to it ====

You can modify your local copy of PlatformIO to include the required files. To do so, follow the next steps (here `<version>` is the latest folder, `0.1.1` as of August 2019):

* checkout or download the [https://github.com/AllWize/allwize-boards allwize-boards repository]
* copy the `<version>/boards.txt` file to the Platformio `framework-arduinosam` package folder as `boards_allwize.txt`
* copy the `<version>/variants/allwizek2` folder to the Platformio `framework-arduinosam` package `variants` folder
* copy the `allwizek2.json` file to the Platformio `atmelsam` platform `boards` folder

If using Linux, you have a script in the repository that does all these steps for you, just call it from the repo root folder with the version you want to install:

<pre>
$ ./pio-install.sh 0.1.1
</pre>

==== Configure the connections in your sketch ====

You can also configure the connections to the radio module manually from your script and build the code for the MKRZERO board like this:

<pre>
#include "wiring_private.h"               // pinPeripheral() function
#define RX_PIN                  (29ul)
#define TX_PIN                  (26ul)
#define RESET_PIN               (30ul)
Uart SerialWize(&sercom4, RX_PIN, TX_PIN, SERCOM_RX_PAD_3, UART_TX_PAD_0);
void SERCOM4_Handler() { 
    SerialWize.IrqHandler(); 
}
#define MODULE_SERIAL           SerialWize
#define DEBUG_SERIAL            SerialUSB
</pre>

And then initialize de GPIOs in your setup code:

<pre>
pinPeripheral(RX_PIN, PIO_SERCOM_ALT);
pinPeripheral(TX_PIN, PIO_SERCOM_ALT);
</pre>

==== (future) Simple PlatformIO ====

Once it is included by the PlatformIO team you will be able to setup the board like this:

<pre>
pio init -b allwizek2
pio lib install AllWize
</pre>

REMEMBER: This option is not yet supported!!

== Example code ==

=== Slave ===

<pre>
/*

AllWize - Simple Slave Example - AllWize K2

Simple slave that sends an auto-increment number every 5 seconds.

Copyright (C) 2018-2019 by AllWize <github@allwize.io>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

*/

#include "AllWize.h"

#if not defined(ARDUINO_ALLWIZE_K2)
    #error "This example is meant to run on an AllWize K2 board!"
#endif

// -----------------------------------------------------------------------------
// Configuration
// -----------------------------------------------------------------------------

#define WIZE_CHANNEL            CHANNEL_04
#define WIZE_POWER              POWER_20dBm
#define WIZE_DATARATE           DATARATE_2400bps
#define WIZE_UID                0x20212223

// -----------------------------------------------------------------------------
// AllWize
// -----------------------------------------------------------------------------

AllWize * allwize;

void wizeSetup() {

    SerialUSB.println("Initializing radio module");

    // Create and init AllWize object
    allwize = new AllWize(&SerialWize, PIN_WIZE_RESET);
    allwize->begin();
    if (!allwize->waitForReady()) {
        SerialUSB.println("[WIZE] Error connecting to the module, check your wiring!");
        while (true);
    }

    allwize->slave();
    allwize->setChannel(WIZE_CHANNEL, true);
    allwize->setPower(WIZE_POWER);
    allwize->setDataRate(WIZE_DATARATE);
    allwize->setUID(WIZE_UID);

    //allwize->dump(SerialUSB);

    SerialUSB.println("[WIZE] Ready...");

}

void wizeSend(const char * payload) {

    char buffer[64];
    snprintf(buffer, sizeof(buffer), "[WIZE] Sending '%s'\n", payload);
    SerialUSB.print(buffer);

    if (!allwize->send(payload)) {
        SerialUSB.println("[WIZE] Error sending message");
    }

}

// -----------------------------------------------------------------------------
// Main
// -----------------------------------------------------------------------------

void setup() {

    // Init debug serial
    SerialUSB.begin(115200);
    while (!SerialUSB && millis() < 5000);
    SerialUSB.println();
    SerialUSB.println("[MAIN] Basic slave example");

    // Init radio
    wizeSetup();

}

void loop() {

    // This static variables will hold the number as int and char string
    static uint8_t count = 0;
    static char payload[4];

    // Convert the number to a string
    itoa(count, payload, 10);

    // Send the string as payload
    wizeSend(payload);

    // Increment the number (it will overflow at 255)
    count++;

    // Polling responses for 5 seconds
   delay(5000);

}
</pre>

=== Master ===

<pre>
/*

AllWize - Simple Master Example - AllWize K2

Listens to messages on the same channel and data rate
and prints them out via the serial monitor.

Copyright (C) 2018-2019 by AllWize <github@allwize.io>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

*/

#include "AllWize.h"

#if not defined(ARDUINO_ALLWIZE_K2)
    #error "This example is meant to run on an AllWize K2 board!"
#endif

// -----------------------------------------------------------------------------
// Configuration
// -----------------------------------------------------------------------------

#define WIZE_CHANNEL            CHANNEL_04
#define WIZE_DATARATE           DATARATE_2400bps

// -----------------------------------------------------------------------------
// Wize
// -----------------------------------------------------------------------------

AllWize * allwize;

void wizeSetup() {

    // Create and init AllWize object
    allwize = new AllWize(&SerialWize, PIN_WIZE_RESET);
    allwize->begin();
    if (!allwize->waitForReady()) {
        SerialUSB.println("[WIZE] Error connecting to the module, check your wiring!");
        while (true);
    }

    allwize->master();
    allwize->setChannel(WIZE_CHANNEL, true);
    allwize->setDataRate(WIZE_DATARATE);

    char buffer[64];
    snprintf(buffer, sizeof(buffer), "[WIZE] Listening... CH %d, DR %d\n", allwize->getChannel(), allwize->getDataRate());
    SerialUSB.print(buffer);

}

void wizeDebugMessage(allwize_message_t message) {

    // Code to pretty-print the message
    char buffer[512];
    if (CI_WIZE == message.ci) {
        snprintf(
            buffer, sizeof(buffer),
            "[WIZE] C: 0x%02X, CI: 0x%02X, MAN: %s, ADDR: 0x%02X%02X%02X%02X, CONTROL: %d, OPID: %d, APPID: %d, COUNTER: %d, RSSI: %d, DATA: { ",
            message.c, message.ci, 
            message.man,
            message.address[0], message.address[1],
            message.address[2], message.address[3],
            message.wize_control, message.wize_operator_id, message.wize_application, message.wize_counter,
            (int16_t) message.rssi / -2
        );
    } else {
        snprintf(
            buffer, sizeof(buffer),
            "[WIZE] C: 0x%02X, CI: 0x%02X, MAN: %s, ADDR: 0x%02X%02X%02X%02X, RSSI: %d, DATA: { ",
            message.c, message.ci, 
            message.man,
            message.address[0], message.address[1],
            message.address[2], message.address[3],
            (int16_t) message.rssi / -2
        );
    }
    SerialUSB.print(buffer);

    for (uint8_t i=0; i<message.len; i++) {
        char ch = message.data[i];
        snprintf(buffer, sizeof(buffer), "0x%02X ", ch);
        SerialUSB.print(buffer);
    }
    SerialUSB.print("}, STR: \"");
    SerialUSB.print((char *) message.data);
    SerialUSB.println("\"");

}

void wizeLoop() {

    if (allwize->available()) {

        // Get the message
        allwize_message_t message = allwize->read();

        // Show it to console
        wizeDebugMessage(message);

    }

}

// -----------------------------------------------------------------------------
// Main
// -----------------------------------------------------------------------------

void setup() {

    // Setup debug serial
    SerialUSB.begin(115200);
    while (!SerialUSB && millis() < 5000);
    SerialUSB.println();
    SerialUSB.println("[MAIN] Wize Master Example");
    SerialUSB.println();

    // Init radio
    wizeSetup();

}

void loop() {

    // Listen to messages
    wizeLoop();

}
</pre>