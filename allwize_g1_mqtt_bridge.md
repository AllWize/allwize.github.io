MQTT is a communication protocol specially designed for sensors and telemetry. It has become a ''de-facto'' standard and a lot of different IoT service providers support it as a way to ingest data from sensors. To use MQTT you need an MQTT broker. The broker is the responsible for receiving and transmitting messages. Think of it as a post office service. You send a message (payload) to a mailbox (topic) and anyone subscribed to that mailbox will receive the message. There are a few configuration options (keep-alive time, quality of service, last will and testament, retain flag...) you can read about them in the [http://mosquitto.org/man/mqtt-7.html Mosquitto MAN page]. Mosquitto is a popular open-source MQTT broker.

Turning your AllWize G1 into a "Wize to MQTT Bridge" is very simple. There is even a full [https://github.com/AllWize/allwize/tree/dev/examples/wize2mqtt wize2mqtt example] that comes with the AllWize library but we will summarize it here.

== Configuration ==

The configuration.h file holds the basic configuration for the bridge. The options you have to configure are:
* the WiFi settings to connect the device to your router
* the Wize radio settings, the gateway must be listening in the same channel and datarate the sensors are using to report data
* the MQTT broker settings

You can also 

<pre>
/*

Configuration file

*/

#pragma once

//------------------------------------------------------------------------------
// Payload data format
// You have to uncomment just one of these defines
//------------------------------------------------------------------------------

#define PAYLOAD_CSV
//#define PAYLOAD_MBUS
//#define PAYLOAD_LPP

//------------------------------------------------------------------------------
// General configuration
//------------------------------------------------------------------------------

#define DEBUG_SERIAL            Serial

//------------------------------------------------------------------------------
// WiFi credentials
//------------------------------------------------------------------------------

#define WIFI_SSID               "..."
#define WIFI_PASSWORD           "..."

//------------------------------------------------------------------------------
// Radio module connections
//------------------------------------------------------------------------------

#define RESET_PIN               14
#define RX_PIN                  5
#define TX_PIN                  4

//------------------------------------------------------------------------------
// Wize configuration
//------------------------------------------------------------------------------

#define WIZE_CHANNEL            CHANNEL_04
#define WIZE_POWER              POWER_20dBm
#define WIZE_DATARATE           DATARATE_2400bps

//------------------------------------------------------------------------------
// MQTT configuration
//------------------------------------------------------------------------------

#define MQTT_HOST               "192.168.2.2"
#define MQTT_PORT               1883
#define MQTT_USER               ""
#define MQTT_PASS               ""
#define MQTT_QOS                2
#define MQTT_RETAIN             0
</pre>

== Code ==

The main code takes care of the WiFi and Wize connections and perform the parsing of the messages to MQTT topics. You can see the [https://github.com/AllWize/allwize/blob/dev/examples/wize2mqtt/wize2mqtt.ino full sketch for the wize2mqtt bridge] but the main part here is the mqtt parsing method:

<pre>
void wizeMQTTParse(allwize_message_t message) {

    char uid[10];
    snprintf(
        uid, sizeof(uid), "%02X%02X%02X%02X",
        message.address[0], message.address[1],
        message.address[2], message.address[3]
    );

    // RSSI
    char topic[32];
    snprintf(topic, sizeof(topic), "/device/%s/rssi", uid);
    mqttSend(topic, String(message.rssi / -2).c_str());

    #ifdef PAYLOAD_CSV

        // Parse a comma-separated payload
        uint8_t field = 1;
        char * payload;
        payload = strtok((char *) message.data, ",");

        // While there is a field
        while (NULL != payload) {

            // Publish message
            snprintf(topic, sizeof(topic), "/device/%s/field_%d", uid, field);
            mqttSend(topic, payload);

            // Get new token and update field counter
            payload = strtok (NULL, ",");
            field++;

        }

    #endif // PAYLOAD_CSV

    #ifdef PAYLOAD_MBUS

        // Parse payload
        MBUSPayload payload;
        DynamicJsonDocument jsonBuffer(512);
        JsonArray root = jsonBuffer.createNestedArray();
        payload.decode(message.data, message.len, root);

        // Walk JsonArray
        for (uint8_t i = 0; i<root.size(); i++) {
            
            snprintf(topic, sizeof(topic), "/device/%s/%06X", uid, (uint32_t) root[i]["vif"]);
            
            char value[12];
            uint8_t decimals = ((int32_t) root[i]["scalar"] > 0) ? 0 : - ((int32_t) root[i]["scalar"]);
            if (decimals == 0) {
                snprintf(value, sizeof(value), "%u", (uint32_t) root[i]["value_scaled"]);
                mqttSend(topic, value);
            } else {
                dtostrf(root[i]["value_scaled"], sizeof(value)-1, decimals, value);
                uint8_t start = 0;
                while (value[start] == ' ') start++;
                mqttSend(topic, &value[start]);
            }


        }

    #endif // PAYLOAD_MBUS

    #ifdef PAYLOAD_LPP

        // Parse payload
        CayenneLPP payload(1);
        DynamicJsonDocument jsonBuffer(512);
        JsonArray root = jsonBuffer.createNestedArray();
        payload.decode(message.data, message.len, root);

        char value[16];
        
        // Walk JsonArray
        for (JsonObject element: root) {
            JsonVariant v = element["value"];
            if (v.is<JsonObject>()) {
                for (JsonPair kv : v.as<JsonObject>()) {

                    const char * name = kv.key().c_str();
                    dtostrf(kv.value().as<float>(), sizeof(value)-1, 4, value);
                    uint8_t start = 0; while (value[start] == ' ') start++;
                    snprintf(topic, sizeof(topic), "/device/%s/%s", uid, name);
                    mqttSend(topic, &value[start]);

                }
            } else {

                const char * name = element["name"].as<char*>();
                dtostrf(element["value"].as<float>(), sizeof(value)-1, 2, value);
                uint8_t start = 0; while (value[start] == ' ') start++;
                snprintf(topic, sizeof(topic), "/device/%s/%s", uid, name);
                mqttSend(topic, &value[start]);

            }
        }

    #endif // PAYLOAD_LPP

}
</pre>

== Parsers ==

As you can see there are actually 3 different parsers there: CSV, Cayenne LPP and MBUS.

=== CSV Parser ===

The '''CSV parser''' expects a raw string with sensor values in a comma-separated fashion, like this: "23.5,34,1003.50". It just parses the string and sends one MQTT message for each field plus an RSSI message, so the previous payload would translate into (where 20212223 is the sender UID):

<pre>
/devices/20212223/rssi -45
/devices/20212223/field_1 23.5
/devices/20212223/field_2 34
/devices/20212223/field_3 1003.5
</pre>

=== Cayenne LPP Parser ===

The '''Cayenne LPP Parser''' uses the [https://developers.mydevices.com/cayenne/docs/lora/#lora-cayenne-low-power-payload Cayenne Low Power Payload] frame format description to decode the message. The sketch uses the [https://github.com/ElectronicCats/CayenneLPP ElectronicCat CayenneLPP library] that supports the standard field types plus a set of new ones. You can use the very same library to encode your messages too (check the [https://github.com/AllWize/allwize/blob/dev/examples/slave/slave_allwize_k2_lpp/slave_allwize_k2_lpp.ino slave_allwize_k2_lpp example] in the AllWize library).

The protocol is designed to encode not only the value but also the magnitude so the gateway can be agnostic to the data format it receives. An example payload is (hex-encoded): "0164000000000267010B03688E047326E905880651F70053340003E8" which might look long but it gets translated into the following messages:

<pre>
/devices/20212223/rssi -42
/devices/20212223/generic 0
/devices/20212223/temperature 26.7
/devices/20212223/humidity 71
/devices/20212223/pressure 996.1
/devices/20212223/latitude 41.42
/devices/20212223/longitude 2.13
/devices/20212223/altitude 10
</pre>

=== MBUS Payload Parser ===

The '''MBUS Payload Parser''' uses the [http://www.m-bus.com/mbusdoc/md8.php MBUS application payload protocol] to decode the message. The sketch uses the [https://github.com/AllWize/mbus-payload AllWize MBUSPayload library] that supports a good subset of the field types defined in the standard. You can use the very same library to encode your messages too (check the [https://github.com/AllWize/allwize/blob/dev/examples/slave/slave_allwize_k2_mbus/slave_allwize_k2_mbus.ino slave_allwize_k2_mbus example] in the AllWize library).

The protocol is designed to encode not only the value but also the magnitude and the precission of the representation so the gateway can be agnostic to the data format it receives. An example payload is (hex-encoded): "01FD08670166E1022BB404011339" which gets translated into the following messages:

<pre>
/devices/20212223/rssi -43
/devices/20212223/00FD08 103    // access number
/devices/20212223/000066 22.5   // external temperature (C)
/devices/20212223/00002B 1204   // power (W)
/devices/20212223/000013 0.057  // volume (m3)
</pre>

=== Encoding comparison ===

Each of the previous encoding formats have its strong and weak points. This table might help you choose (of course you can always use a different one or a custom binary format to suit your needs, whatever your choice just remember to use the same format in the encoding and decoding :).

In the examples below we are pretending to have deployed a set of home comfort and energy consumption sensors. The data reporting at a given time is: temperature 22.5C, humidity 57% and power 1204W.

{| class="wikitable"
|-
! Format !! Binary Payload !! Size !! Comments
|-
| CSV || 32322E352 C3 5372 C3 1323034 || 12 || 
* Only values, the receiver has to know what they mean!
|-
| Cayenne LPP || 016700E1 028004B4 036872 || 11 || 
* Field types are reported along with values. 
* Field size is fixed and depends on the field. 
* Each field is labelled with a "channel".
* Example, the first field is channel 0x01, type 0x67 (temperature) and temperature is always encoded in 0.1C steps using 2 bytes (0x00E1 == 225)
|-
| MBUS Payload || 0166E1 022BB404 01FD3C39 || 11 || 
* Field types are reported along with values. 
* Field size is adjusted to be the minimum every time (temperature is automatically encoded as a single byte: 255).
* Humidity is encoded as generic sensor since there is no specific type for it.
* Example, the first field value is encoded as one single byte 0x01, type 0x66 (temperature in 0.1C steps) and value is 0xE1, 255 in decimal.

|}