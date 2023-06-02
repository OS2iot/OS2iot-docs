IoT Data Management
===================

This chapter describes how data is ingested into OS2iot, how it is processed, and how it is sent to data target.

The following sequence diagram illustrates how data is processed when it is received.

|image2|

Data ingestion
--------------

OS2iot supports four types of devices:

    1. LoRaWAN devices (using Chirpstack)

    2. SigFox devices

    3. Generic HTTP devices
    
    4. MQTT devices

OS2iot ingests data for each of these device types in a slightly different way:

LoRaWAN devices
^^^^^^^^^^^^^^^

OS2iot connects to LoRaWAN devices using Chirpstack, similarly the data from these are ingested through Chirpstack.
In the class: :code:`ChirpstackMQTTListenerService` in OS2IoT-backend, a MQTT listener is setup to listen to the MQTT broker in chirpstack.
It listens to the device data event topic using the pattern: :code:`application/+/device/+/event/up`.
When a message is received, it is parsed into the DTO: :code:`ChirpstackMQTTMessageDto`, if the device EUI is registered in OS2iot, then the message is further processed, otherwise it is discarded.

SigFox devices
^^^^^^^^^^^^^^

When a SigFox device is created in OS2iot, the device type of it in the SigFox backend is updated to include a bi-directional callback to the backend at the endpoint: :code:`/api/v1/sigfox-callback/data/bidir?apiKey={deviceTypeId}`.
Furthermore the body is set to:

 .. code-block:: javascript

    {
        "time": {time},
        "deviceTypeId": "{deviceTypeId}",
        "deviceId": "{device}",
        "data": "{data}",
        "seqNumber": {seqNumber},
        "ack": {ack}
    }

When the callback is called, the :code:`deviceTypeId` is validated to be one that matches what OS2iot knows exists.
The :code:`deviceId` of the body is validated to exist in OS2iot, if it does, then the message is further processed, and the callback is responded with 204 No Content.
Note: If the value of :code:`ack` is :code:`true`, then the device is ready for a downlink message, then OS2iot will see if it has a message queued for the device, and if it does, remove it from the queue and send it back with status 200 OK.

Generic HTTP devices
^^^^^^^^^^^^^^^^^^^^

OS2iot supports generic HTTP(S) devices, that means that each device of this type gets an API-key at the time of creation, and can send their data using that key, both for identification and authorization.
The endpoint :code:`/api/v1/receive-data?apiKey=<guid>` is used for this, where :code:`<guid>` is a GUID like: :code:`2704e33f-b678-4d3a-a3b9-b36454cabf22`.
If the API-key is valid and corresponds to a device in OS2iot, then the response will be 204 No Content, and the message is further processed. 
If the API-key is unknown, then the resposne will be 403 Bad Request with the message: :code:`MESSAGE.DEVICE-DOES-NOT-EXIST`

MQTT
^^^^

OS2IoT supports two kinds of MQTT devices. An MQTT external broker and an MQTT internal broker. 

OS2IoT supports two kinds of authorization for MQTT devices: Username/password and certificate.

MQTT external broker
~~~~~~~~~~~~~~

When an MQTT external broker is created, the credentials for connecting to the internal MQTT-broker are generated. 
These consists of a URL, a port and a topic for the device to send data to. 
If the authorization type is certificate then the required certificates will be generated. If username/password is used these are manually entered.

When a known MQTT external broker sends data to the topic assigned to it, the message will be further processed. 
If a device attempts to send to a different topic the message will be discarded.

MQTT internal broker
~~~~~~~~~~~~~~~

When an MQTT internal broker is created a connection to the external broker is made using the entered credentials. 
When a message is received by the external broker on the topic configured, OS2IoT will also receive the message and further process it.


Payload transformation and storage
----------------------------------

Once that data is ingested from one of the sources described above, it can be transformed before being sent to its intended target(s).
Two examples of `payload decoders are shown here <../payload-decoders/payload-decoders.html>`_ as well as an introduction to how they work in OS2iot. 

At the same time as payload transformation, the latest message payload is saved, and the metadata (timestamp and signal) is saved for the last 20 messages.
This is done for debugging purposes, such that you can test payload decoders on actual and up-to date data.

Sending to data-targets
-----------------------

Once the data is transformed/decoded, it can be sent to the intended data-targets.
OS2iot currently supports HTTP push, FIWARE and MQTT as data-targets. It can be extended with other kinds of data-targets, e.g. PostgreSQL, etc. 

The data-targets are configured as part of an application, and therefore can only include data from the IoT-Devices in said application. 
For each IoT-Device, it is up to the user to select an appropriate payload decoder, to translate the raw payload to JSON.

At the time of writing, there is no retry mechanism in OS2iot over HTTP(s). It uses a "fire-and-forget" / "at most once delivery" pattern.

For more info on the different data-target options, look `here <.../external-interface-design/external-interface-design.html?highlight=data%20target#id2>`_

.. |image2| image:: ./media/image8.png
   
