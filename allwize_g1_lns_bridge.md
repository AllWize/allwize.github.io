A LoRaWAN Network Server (LNS) is the piece of the LoRaWAN stack responsible for de-duplicating messages and route them to the Application Server where the user can configure the appropriate handlers to send the data to depending on the device address of the source node. Most of the servers available, including the The Things Network LNS and the LoRaServer open-source LNS support the Semtech Legacy Protocol. This protocol is a simple way to send data from a LoRaWAN Gateway to the LNS using UDP.

== Theory == 

=== LoRaWAN Frame Format ===

The Semtech Legacy Protocol is open-spec and fairy simple so we can actually use it to send data to a LNS from wherever we want, the only requirement is that the data must be a LoRaWAN-compatible frame so the LNS can understand it. This is what a LoRaWAN frame looks like:

[[File:lorawan-frame.png|800x600px|none|left|LoRaWAN Frame]]

The first row is the raw LoRa frame we don't have to worry about, the PHY Payload is the MAC part of the frame. In order:

* MAC Header (type of message, version of the protocol,... for instance: 0x40 means unconfirmed uplink)
* Device Address (the source device address)
* Frame Control (LoRaWAN specific control bits: ACK, ADR, device class,...)
* Frame Counter (a 16-bit autoincrement number)
* Frame Options (LoRaWAN MAC options)
* Frame Port (used to tell the frame payload format)
* Frame Payload (the data, often LLP since the Application Server already have a decoder)
* MIC (Message Integrity Code, used by the LNS to check the message is valid)


=== Wize Frame Format ===

A Wize frame looks like this. Encryption is optional unlike in LoRaWAN.

[[File:wize-frame.png|800x600px|none|left|LoRaWAN Frame]]

=== Sending a LoRaWAN message over Wize ===

Of course the simpler option would be to encapsulate a LoRaWAN message inside the Wize Payload, but the total payload would have a lot of duplicated or fixed data:

{| class="wikitable"
|-
! LoRaWAN !! Wize !! Size !! Notes
|-
| MAC Header || C-Field || 1 || Encoding original MAC Header
|-
| Device Address || A-Field || 4 || 
|-
| Frame Control || - || 1 || Always 0x00 in our case
|-
| Frame Counter || L6-Cpt || 2 || 
|-
| Frame Port || L6-App || 1 || 
|}

As you can see we can actually map up to 8 bytes of the LoRAWAN frame to Wize specific frame bytes or fixed values. Then just add the Frame Payload (encrypted using the Application Session Key) and the MIC (encrypted using the Network Session Key).

And this is what the [https://github.com/AllWize/allwize/blob/dev/src/AllWize_LoRaWAN.cpp AllWize_LoRaWAN class] does:

* Encrypts the payload using the Application Session Key
* Generates the Message Integrity Code (MIC) using the Network Session Key
* Configures the radio module to use a specific C-field (0x80)
* Maps the LoRaWAN fields to Wize values
* Sends only the required bytes

The same class can the be used in the receiver to rebuild the LoRaWAN Frame and forward the contents to a LoRaWAN Network Server using Semtech Legacy Protocol.

== Code ==

The code examples here are part of the LoRaWAN examples you can find with the AllWize library: [https://github.com/AllWize/allwize/tree/dev/examples/lorawan/lorawan_gateway/src lorawan_gateway] and [https://github.com/AllWize/allwize/blob/dev/examples/lorawan/lorawan_node/lorawan_node.ino lorawan_node]. You will first need to create an account on a LoRaWAN Network Server (if you don't already have one). An easy and free one is [https://console.thethingsnetwork.org/ The Things Network].

=== Node ===

Using the AllWize_LoRaWAN class is very simple. First we add it to our sketch and instantiate an object, this code will work on an AllWize K2 board:

<pre>
#include "AllWize_LoRaWAN.h"
AllWize_LoRaWAN allwize(&SerialWize, PIN_WIZE_RESET);
</pre>

Next you will need to initialize the Wize radio, the same way we are doing in all the examples:

<pre>
allwize.begin();
if (!allwize.waitForReady()) {
    SerialUSB.println("[WIZE] Error connecting to the module, check your wiring!");
    while (true);
}
allwize.slave();
allwize.setChannel(CHANNEL_04, true);
allwize.setPower(POWER_20dBm);
allwize.setDataRate(DATARATE_2400bps);
</pre>

Now we are going to configure the LoRaWAN part, this is done very similar to other LoRaWAN libraries out there. You will need to create a device in your LoRAWAN application (TTN, for instance), configure it for ABP activation and copy the credentials like this:

<pre>
uint8_t DEVADDR[4] = { 0x26, 0x01, 0x13, 0x3D };
uint8_t NWKSKEY[16] = { 0xF6, 0x59, 0x13, 0x16, 0x99, 0x35, 0x0E, 0xBF, 0x41, 0xE5, 0xEA, 0xA6, 0x95, 0xFE, 0x64, 0x27 };
uint8_t APPSKEY[16] = { 0xA2, 0x40, 0xE9, 0xF6, 0xB1, 0xB1, 0x3B, 0x53, 0xB1, 0xE2, 0x0A, 0x0D, 0x3D, 0x50, 0x74, 0xCB };
allwize.joinABP(DEVADDR, APPSKEY, NWKSKEY);
</pre>

Finally we can start sending data. Here we are using CayenneLPP to encode the payload since most LoRaWAN Application Server already have a decoder for it. We will need to include the library first:

<pre>
#include "CayenneLPP.h"
</pre>

And the this code can go into your main loop:

<pre>
CayenneLPP lpp(32);
lpp.reset();
lpp.addTemperature(1, getTemperature());
lpp.addRelativeHumidity(2, getHumidity());
lpp.addBarometricPressure(3, getPressure());
if (!allwize.send(lpp.getBuffer(), lpp.getSize(), 1)) {
    SerialUSB.println("[WIZE] Error sending message");
}
</pre>

=== Gateway ===

The Gateway part is a little more involved since it has to:

* Connect to the Wize radio module using the AllWize_LoRAWAN class to get correct LoRaWAN frames
* Connect to a local WiFi router to gain Internet access
* Query a NTP server to get current time
* Encode the messages using Semtech's Legacy Protocol and send them to a LoRaWAN Network Server

You can check the full example in the [https://github.com/AllWize/allwize/tree/dev/examples/lorawan/lorawan_gateway/src lorawa_gateway] folder in the repository, but here you have some important things to remember:

* Use the AllWize_LoRaWAN class, that will rebuild and return the original LoRaWAN frame 
* Use the same channel and datarate in the Wize configuration as your nodes, otherwise the gateway won't receive the messages
* You will need to know the IP and port (usually the 1700) of your LoRaWAN Network Server to send the UDP packets to. If you are using TTN these are <pre>router.eu.thethings.network:1700</pre> in Europe.