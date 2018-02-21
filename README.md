
This repository contains documentation and examples on how to communicate via Bosch IoT Hub to Bosch IoT Things.

The Bosch IoT Suite is built on several services, two of them having the role of connecting devices and providing APIs
(utilizing the digital twin pattern) in order to interact with devices.

* the [Bosch IoT Hub](http://docs.bosch-iot-hub.com) provides means to *connect devices* through various protocols to 
  applications in the Internet of Things in a secure, reliable and elastic scaling manner.
* [Bosch IoT Things](https://things.s-apps.de1.bosch-iot-cloud.com/dokuwiki/doku.php?id=001_learn_about_cr:001_learn_about_cr)
  enables applications to *manage digital twins* of their IoT device assets in a simple, convenient, robust, and secure way.

Please follow the links above for detailed documentation of both of the services.

For an overview of relevant tutorials for BCX18, head over to the [BCX18 tutorials](https://preview.bosch-iot-suite.com/tutorials/).


## Hub - Things integration

We're at Bosch are team players and also our Bosch IoT Suite services play together in order to add even more value than
when using them alone. 

![Bosch Iot Hub/Things integration](hub-things-overview.png)

The picture shows that `south` of the Bosch IoT Hub real world devices are connected via various IoT protocols for which
the Hub provides protocol adapters.

`North` of the Hub the Bosch IoT Things service connects via the Hub's AMQP 1.0 interface in order to consume telemetry
data from the Hub.

`West` and `east` of Bosch IoT Things "solution specific" applications can access the Things APIs 
(HTTP and WebSocket) in order to interact with the **digital twins** of the connected devices. Via a WebSocket or 
Server-Sent-Events your applications can for example be notified about changes of connected devices.


## Connect your own devices

You can connect your own devices in this setup by connecting to one of the Bosch IoT Hub protocol adapters.

Following steps are necessary for that:
1. (optional) create a [Thing Type](https://preview.bosch-iot-suite.com/tutorials/dx_create_thingtype/) via 
   [Eclipse Vorto](https://eclipse.org/vorto) for your device describing its data and capabilities in order to have
   type information for your device
2. register your device in the Bosch IoT Suite via the [Developer console](https://console-bcx.bosch-iot-suite.com): 
   [tutorial](https://preview.bosch-iot-suite.com/tutorials/dx_register_device/)
   * if this step fails and you want to connect an XDK, [use the manual way](#manual-way-for-connecting-xdk)
3. choose a protocol (MQTT or HTTP) your device is capable to speak
    1. alternatively, if your device is not powerful enough to do HTTP or MQTT directly, use a gateway (e.g. a RaspberryPi)
       as an intermediate connecting your device
4. configure the endpoint to use
    1. for HTTP: `https://rest.bosch-iot-hub.com/telemetry/BCX18/<your-device-id>`
    2. for MQTT: `mqtt.bosch-iot-hub.com` - topic: `telemetry/BCX18/<your-device-id>`
5. whenever data changes on your device, send a message to either the HTTP endpoint or via the MQTT topic in 
   [Ditto Protocol](https://www.eclipse.org/ditto/protocol-specification.html) - [here you can find](https://www.eclipse.org/ditto/protocol-examples.html)
   some example Ditto Protocol messages
   
You can also let the [Developer console](https://console-bcx.bosch-iot-suite.com) generate an example (e.g. Arduino)
as starting point, have a look at the [tutorials](https://preview.bosch-iot-suite.com/tutorials/) for that.



### HTTP example

Here a simple bash/cURL example on how to send data in [Ditto Protocol](https://www.eclipse.org/ditto/protocol-specification.html)
via the Bosch IoT Hub HTTP adapter:

```bash
DEVICE_ID='the-device-id'
AUTH_INFO=$(echo -n "$DEVICE_ID@BCX18:the-password" | base64) 
curl -v -X PUT https://rest.bosch-iot-hub.com/telemetry/BCX18/$DEVICE_ID \
    -H "Content-Type: application/json" \
    -H "Authorization: Basic $AUTH_INFO" \
    -d "{
          'topic': 'BCX18/$DEVICE_ID/things/twin/commands/modify',
          'headers': { 'response-required': false },
          'path': '/features/location/properties/status',
          'value': {
            'latitude': 52.5250840,
            'longitude': 13.3694020
          }
        }"
```

Use the password you provided when you created the device in the [Developer console](https://console-bcx.bosch-iot-suite.com).


## MQTT example

And a simple MQTT example (via `mosquitto_pub` which you can install on Ubuntu by `sudo apt install mosquitto-clients`)
on how to send data in [Ditto Protocol](https://www.eclipse.org/ditto/protocol-specification.html)
via the Bosch IoT Hub MQTT adapter:

```bash
curl -o iothub.crt http://docs.bosch-iot-hub.com/cert/iothub.crt

DEVICE_ID='the-device-id'
mosquitto_pub -h mqtt.bosch-iot-hub.com -p 8883 --cafile iothub.crt \
    -u $DEVICE_ID@BCX18 -P the-password -i $DEVICE_ID \
    -t telemetry/BCX18/$DEVICE_ID -d \
    -m "{
          'topic': 'BCX18/$DEVICE_ID/things/twin/commands/modify',
          'headers': { 'response-required': false },
          'path': '/features/location/properties/status',
          'value': {
            'latitude': 52.5250840,
            'longitude': 13.3694020
          }
        }"
```


Further documentation on the protocol adapters can be found in the 
[getting started of Bosch IoT Hub](http://docs.bosch-iot-hub.com/userguide/gettingstarted.html).



## Manual way for connecting XDK

The Developer Console autmatically does all of the following steps. If you need more control of the creation process, 
feel free to do it the manual way.

The manual way of connecting an XDK to both Bosch IoT Things and Bosch IoT Hub is the following:

```bash
DEVICE_ID='the-device-id'

# Register device in Bosch IoT Things
AUTH_INFO=$(echo -n "bcx18:bcx18!Open2" | base64) 
curl -X PUT https://things.s-apps.de1.bosch-iot-cloud.com/api/2/things/BCX18:$DEVICE_ID \
    -H "authorization: Basic $AUTH_INFO" \
    -H 'x-cr-api-token: db7f4e0cca344d32be72914311f1055f' \ 
    -d "{
          'thingId': 'BCX18:$DEVICE_ID',
          'attributes': {
            'schema': {
              'Accelerometer_0': 'com.ipso.smartobjects.Accelerometer:0.0.2',
              'Barometer_0': 'com.ipso.smartobjects.Barometer:0.0.2',
              'Button_0': 'com.ipso.smartobjects.Push_button:0.0.2',
              'AlertNotification_0': 'com.bosch.demo.xdk.fb.AlertNotification:1.0.0',
              'Illuminance_0': 'com.ipso.smartobjects.Illuminance:0.0.2',
              'Temperature_0': 'com.ipso.smartobjects.Temperature:0.0.2',
              'Gyrometer_0': 'com.ipso.smartobjects.Gyrometer:0.0.2',
              'FirmwareUpdate_0': 'org.oma.lwm2m.Firmware_Update:0.0.2',
              'Humidity_0': 'com.ipso.smartobjects.Humidity:0.0.2',
              'LightControl_0': 'com.ipso.smartobjects.Light_Control:0.0.2',
              'LightControl_1': 'com.ipso.smartobjects.Light_Control:0.0.2',
              'LightControl_2': 'com.ipso.smartobjects.Light_Control:0.0.2',
              'Magnetometer_0': 'com.ipso.smartobjects.Magnetometer:0.0.2'
            },
            '_modelId': 'com.bosch.devices.XDK:2.0.0',
            'thingName': 'MyXDK',
            'createdOn': '2018-02-21 10:06:31+0000',
            'deviceId': '$DEVICE_ID'
          },
          'features': {}
        }"


# Allow XDK connector to update the just created Thing in Bosch IoT Things:
curl -X PUT https://things.s-apps.de1.bosch-iot-cloud.com/api/2/policies/BCX18:$DEVICE_ID/entries/hubbridge \
    -H "authorization: Basic $AUTH_INFO" \ 
    -H 'x-cr-api-token: db7f4e0cca344d32be72914311f1055f' \ 
    -d '{
          "subjects": {
            "iot-things:461b0695-3ae8-4e50-a2db-e2d66e6fcbd2:hub": {
              "type": "iot-things-clientid"
            }
          },
          "resources": {
            "thing:/features": {
              "grant": ["READ","WRITE"],
              "revoke": []
            }
          }
        }'

# Register device in Bosch IoT Hub:
curl -X POST https://device-registry.bosch-iot-hub.com/registration/BCX18 \
    -H "Content-Type: application/json" \
    -d "{'device-id': '$DEVICE_ID','thingId':'BCX18:$DEVICE_ID','modelId':'com.bosch.devices.XDK:2.0.0'}"


# Add credential of device in Bosch IoT Hub:
PWD_HASH=$(echo -n "" | openssl dgst -binary -sha512 | base64 | tr -d '\n') 
curl -X POST https://device-registry.bosch-iot-hub.com/credentials/BCX18 \
    -H "Accept: application/json" \
    -d "{
          'device-id': '$DEVICE_ID',
          'auth-id': null,
          'type': 'hashed-password',
          'secrets': [
            {
              'hash-function': 'sha-512',
              'pwd-hash': '$PWD_HASH'
            }
          ]
        }"
```
