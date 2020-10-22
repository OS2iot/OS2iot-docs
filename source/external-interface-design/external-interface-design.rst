External Interface Design
====================================

This document describes the external interface design of OS2iot. This
includes an overview of the interfaces, the internal and external
integration patterns, and the details of both incoming and outgoing
integrations (IoT device and data target, respectively).

Interfaces
-------------------------------------------------------------------

Figure 1 shows an overview of the external interfaces of OS2iot.

|image1|

Figure - OS2iot external interfaces

Integration patterns
-------------------------------------------------------------------

Internal systems
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Internally, OS2iot consists of five components shown in Figure 2.

|image2|

Figure 2 - Internal interfaces

Two different patterns are used between the internal components:

-  | **Request-response**
   | Used between the frontend, backend and database.

-  | **Publish-subscribe**
   | Used in communication between the backend and MQTT brokers.

External systems
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

External system integrations for OS2iot are divided into three groups:

-  IoT devices

-  Data targets

-  Authentication

IoT devices
~~~~~~~~~~~

|image3|

Figure 3 - IoT device integration overview

All IoT device integrations except LoRaWAN use callbacks to send data to
OS2iot using HTTPS in a request-response manner. Device management for 
Sigfox (i.e. adding and modifying IoT devices) is done by
sending requests to Sigfox's Cloud backend.

Data from Chirpstack devices are sent to OS2iot through Chirpstack using
MQTT, where Chirpstack is the publisher and OS2iot is the subscriber.
Device management is done from OS2iot by sending gRPC requests to
Chirpstack.

Data targets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

|image4|

Figure 4 - Data target integrations

OS2iot supports integrations to:

-  CIM Context Broker based on NGSI-LD

-  Opendata.dk

-  Any external system supporting one of the following

   -  CoAP

   -  HTTPS

   -  WebSocket

   -  MQTT

All use request-response except MQTT, which uses publish-subscribe.

Authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

|image5|

Figure 5 - External authentication systems

OS2iot integrates to two external authentication systems:

-  KOMBIT Adgangsstyring

-  Active Directory

These are described in detail in section 6 and 7, respectively.

IoT Device Integrations
-------------------------------------------------------------------

*This section describes the IoT device integrations. This covers both
upstream and downstream messages, registering new devices etc. Each
subsection covers a specific technology.*

Generic IoT Device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Several of the protocols shown in *Figure 3 - IoT device integration
overview*, use a generic HTTP endpoint for receiving data. Each device
is identified by an API-key which is a UUID, this is passed as a URL
parameter. This endpoint validates that the incoming data is JSON and
that the device exists, by validating the API-key, in the database
before the data is passed added to Kafka to be further processed.

Swagger:
`/api/v1/receive-data <https://test-os2iot-backend.os2iot.dk/api/v1/docs/#/Receive%20Data/ReceiveDataController_receive>`__

NB-IoT (Telia)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

NB-IoT devices are supported in OS2iot using the endpoint for receiving data from generic IoT devices as described in section 4.1. There are several reasons for this:

1.	NB-IoT devices often come bundled with a proprietary backend that devices are hardcoded to send data to. From this backend, data can be sent to other systems such as OS2iot. Other devices might be configurable to send data to a specific endpoint. This heterogenous setup makes it very difficult to make general assumptions for how NB-IoT devices send data to OS2iot.
2.	NB-IoT devices are typically managed either by the provider of the NB-IoT network or by another third party. This means that while it is possible to build NB-IoT device management functionality in OS2iot using the Ericsson DCP Platform, this will likely be redundant functionality in almost all cases.


LoRaWAN (Chirpstack)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Chirpstack (https://chirpstack.io) is used to communicate with LoRaWAN
IoT devices. This means that OS2iot communicates with Chirpstack, which
in turn communicates with gateways and devices.

This in turn means, that data must be synchronized between Chirpstack
and OS2iot, such that LoRaWAN devices created in OS2iot are also created
in Chirpstack.

Data from IoT devices is received by Chirpstack and sent to OS2iot by
publishing to a MQTT broker which has OS2iot as a subscriber.

Data sent to IoT devices is sent from OS2iot to Chirpstack using a
RESTful gRPC interface. Communication between the IoT devices, gateways
and Chirpstack is out of scope of this project.

|image6|

The Chirpstack is comprised of the following 5 modules.

1. `Appplication
   Server <https://www.chirpstack.io/application-server/use/fuota/>`__

2. `Network Server <https://www.chirpstack.io/network-server/>`__

3. `Gateway bridge <https://www.chirpstack.io/gateway-bridge/>`__

4. `Geolocation
   server <https://www.chirpstack.io/geolocation-server/>`__ (only for
   locating edge devices)

5. A postgresql database

Data synchronization
~~~~~~~~~~~~~~~~~~~~

Some data must be always be synchronized between OS2iot and Chirpstack.
This includes:

-  Applications

-  Gateways

-  IoT devices

-  Service profiles

-  Device profiles

Changes to these entities must always happen in OS2iot, which is then
synchronized to Chirpstack. It is not supported to change data directly
in Chirpstack. All manipulation of settings in the Chirpstack is thus
done via the Chirpstack API. (http://localhost:8080/api#!)

Security
~~~~~~~~

OS2iot and Chirpstack have separate security models and do not share
users. Instead, all communication between OS2iot and Chirpstack is done
using a service account with administrator permissions in Chirpstack.

For communicating with the chirpstack api, it is necessary to set up a
JWT token, this can be done via the Chirpstack UI
(http://localhost:8080/#/api-keys) or by using the jwt.js tool found in
OS2iot\OS2IoT-backend\jwt.js

node .\jwt.js

The important part of creating the token is as follows.

| In the claim object, the username and the subject has to be set to a
  current user of the Chirpstack.
| In the jwt.create() method set the secret value to something
  predefined and shared between os2iot and the chirpstack
  implementation.


.. code-block:: javascript

   var uuid = require("uuid");
   var nJwt = require("njwt");
   var claims = {
       iss: "chirpstack-application-server", // issuer of the claim
       aud: "chirpstack-application-server", // audience for which the claim is intended
       nbf: Math.floor(new Date() / 1000), // unix time from which the token is valid
       exp: Math.floor(new Date() / 1000) + 60 * 60 * 24 * 14, // unix time when the token expires
       sub: "user", // subject of the claim (an user)
       username: "admin", // username the client claims to be
   };
   var jwt = nJwt.create(claims, "secret", "HS256"); // set secret
   var token = jwt.compact();
   console.log(token)


-------------------------------------------------------------------

the most important part in relation to security is the header, which
holds the aforementioned token in the 'Grpc-Metadata-Authorization
field. A request would thus be setup as follows:

.. code-block:: javascript

   var request = require('request');
   var headers = {
       'Accept': 'application/json',
       'Grpc-Metadata-Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhcGlfa2V5X2lkIjoiYWExMGVkMmQtODdjZC00YmJlLTljZDktNmM4ODQ0ZTc5OTA2IiwiYXVkIjoiYXMiLCJpc3MiOiJhcyIsIm5iZiI6MTU5NjExMzIwMiwic3ViIjoiYXBpX2tleSJ9.7JfLkDe1xqqrqUtoKuSwHobUo7HGv-RvD0atftsgD_c'
   };

   var options = {
       url: 'http://localhost:8080/api/device-profiles',
       headers: headers
   };

Prerequisites 
""""""""""""""""""""""""""""""

In order to use the Chirpstack surdent things has to be set up, in a
specific order.

-  Network server

   -  The network server has to be added. This can be set up as a
      automatic process for when we set up the docker containers.

-  Gateway

   -  Create gateway profile

   -  Add a minimum of 1 gateway server to the system

-  Devices

   -  Create service profile

   -  Create device profile

   -  Register device

      -  Note that once a Service Profile has been selected, it can't be
         changed.

Error handling
~~~~~~~~~~~~~~

-  What happens if data cannot be sent from OS2iot to Chirpstack?

-  What happens if device data cannot be sent from Chirpstack to OS2iot?

-  Descripe scope of error handling (errors between device and gateway
   is out of scope)

..

   Errors received from chirpstack will be logged and displayed. Simple

Communicating with edge devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

REST API is the easiest way to send payloads to edge devices. Retrieving
data is done via MQTT as illustrated in Fig 2. . Os2Iot LoRaWAN
implementation

**Fig 2.** Os2Iot LoRaWAN implementation

Reading data

-  **Join** - Event published when a device joins the network. Please
   note that this is sent after the first received uplink (data) frame.

-  **Status** - Event for battery and margin status received from
   devices.

-  **Uplink** - Contains the data and meta-data for an uplink
   application payload.

-  **Ack** - Acknowledgements event published on downlink frame.

-  **TxAck** - Event published when a downlink frame has been
   acknowledged by the gateway for transmission

-  **Error** - Event published in case of an error related to payload
   scheduling or handling. E.g. in case when a payload could not be
   scheduled as it exceeds the maximum payload-size.

Register network server
~~~~~~~~~~~~~~~~~~~~~~~

When registering the network server. The server attribute has to be set
according to the docker container followed by port 8000 as shown in the
following example.

**"server":"chirpstack-network-server:8000",**

.. _register-new-device-1:

Register new device
~~~~~~~~~~~~~~~~~~~

Devices can be registered and activated on the network by to different
means. Over-the-Air Activation (OTAA) and Activation by Personalization
(ABP) Over-the-Air Activation (OTAA) is the most secure way to connect
with Network. The network assigns a dynamic DevAddr and negotiate
security keys with the device. In other cases the DevAddr as well as the
security keys is hardcoded in the device. This means activating a device
by personalization (ABP). "Device profile" and "Service profil" has to
be set In order to register a device. The *Device Profile* defines the
boot **parameters** that are needed by ChirpStack Network Server to
“connect” with a edge device. The *Service Profile* defines the features
that are enabled for the devices and the rate of messages that can send
over the network by a device.

.. _update-existing-device-1:

Update existing device
~~~~~~~~~~~~~~~~~~~~~~

Firmware update over the air (sometimes called FUOTA) makes it possible
to push firmware updates to one or multiple devices, making use of
multicast. It it standardized by the following LoRa&reg Alliance
specifications:

lorawan-fota-signing-tool sign-binary -b
example-firmware/xdot-blinky.bin -o xdot-blinky-signed.bin
--output-format bin --override-version

This is an experimental feature, the implementation including the API
might change!

`Link to chirpstack - firmware
update <https://www.chirpstack.io/application-server/use/fuota/>`__

Sigfox
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Sigfox integration is a RESTful API over HTTP. They utilize the HTTP
verbs (POST, GET, DELETE, PUT), and are based on the JSON format. Their
documentation is located at
https://support.sigfox.com/apidocs#section/API-overview.

In Denmark the SigFox network is operated by “IoTDenmark”:
https://iotdk.dk/sigfox-iot/.

In SigFox’es information model they have “DeviceTypes” and “Group”,
these can loosely be translated to “Applikation” and “Brugergruppe” in
the OS2IoT model. Alternatively we could implement the integration using
just one DeviceType and one group for an entire OS2IoT installation.

The API is rate-limited, meaning that if too many requests are sent
within a short timeframe, we will get a 429 too many requests, error
back.

.. _register-new-device-2:

Register new device
~~~~~~~~~~~~~~~~~~~

Before being able to register a new device, it’s required to have a
DeviceType, which in turn requires a Group.

To register a new device, the “createDevice” API endepoint is called
with the POST method. Here we need to provide an id, a name, a
deviceType and a “Porting Access Code” (PAC). The endpoint returns the
unique identifier which the device is identified as by SigFox, this must
be saved for future calls.

SigFox’es documentation for the API is located at:
https://support.sigfox.com/apidocs#operation/createDevice

.. _update-existing-device-2:

Update existing device
~~~~~~~~~~~~~~~~~~~~~~

.. _receive-data-1:

Receive data
~~~~~~~~~~~~

**To receive data SigFox recommends using their callback system (in
other words WebHooks). That is when an IoT device sends a message to the
SigFox network, SigFox will call-back to OS2IoT, with the information
from the IoT device.**

There exists three types of callbacks, DATA, SERVICE **and** ERROR.
Under DATA there is two types, UPLINK and BIDIR (bidirectional), where
UPLINK is probably the most common one for us to use. Under SERVICE,
there is STATUS, ACKNOWLEDGE and DATA_ADVANCED. Here DATA_ADVANCED is
interesting, as it allows us to geolocate and get metadata, but comes at
the cost of a 30second delay, see
https://build.sigfox.com/backend-callbacks-and-api#callbacks for more.

To achive this, we must first create a callback configuration at SigFox
using the API.

The callback is defined for one DeviceType, which means that we will
either have to implement one callback for all devices or one for each
application in OS2IoT. Though they can be registered to the same URL.

They support three channels of callback: URL, BATCH_URL and EMAIL. URL
means that Sigfox will produce one callback for each IoT device
transmission, BATCH_URL means that data is pushed at most once pr.
Second and contains all packages in that timeframe.

We must also define the body of information we wish to receive as the
bodyTemplate parameter.

More information about the custom callback features are located here:
https://support.sigfox.com/docs/custom-callback-creation

SigFox’es documentation for the API is located at:
https://support.sigfox.com/apidocs#operation/createCallback

To receive this information from the callback, we must construct an
endpoint for it to call.

To check if we missed any callbacks we should periodically call the
callback errors API for each device type. It will return all the failed
callbacks:
https://support.sigfox.com/apidocs#operation/getCallbackMessagesErrorListForDeviceType

If downlink communication (from OS2IoT to IoT device) is desired, it can
be achived using the callback or by giving the data to Sigfox for them
to pass along to the device. See
https://support.sigfox.com/docs/downlink-callbacks for more.

.. _restart-device-1:

Restart device
~~~~~~~~~~~~~~

It’s possible to restart multiple devices via their unique identifiers
or to restart all devices of a certain type. Both methods are
asynchronous via the Job concept in Sigfox. To restart multiple device
the API: https://support.sigfox.com/apidocs#operation/devicesBulkRestart
is used. And to restart the devices of a given type the API:
https://support.sigfox.com/apidocs#operation/deviceTypeBulkRestart is
used. At a later point the bulk job status API can be used:
https://support.sigfox.com/apidocs#operation/getBulkJobForDevice.

.. _data-targets-1:

Data Targets
-------------------------------------------------------------------

This section describes the different possible data target types in
OS2IoT, i.e. how incoming data from the IoT devices can be sent to
receiving systems. With the exception of MQTT, where the solution
includes an MQTT broker, it is the responsibility of the receiving
system to ensure availability and responsiveness.

Since, at the time of writing, it is not intended for payload data to be
persisted inside OS2IoT. It will be hard to guarantee the delivery of
this data to a data target. For example if the data target is offline.

HTTP Push
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

*[The points below are the things you need to consider for each
integration regardless of whether the service is a traditional web
service, or a file service.]*

-  Service description (utilisation of the service)

-  Functionality (description of how the service works)

-  Validation (which validations are made, when service is called)

-  Applied integration patterns (how to communicate)

-  Error messages (which error messages can you expect from the service)

MQTT
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MQTT is a messaging standard, which is commonly used within IoT
solutions. It works as a publish-subscribe system, where a message
broker acts as an intermediary. There are many message brokers, each
with their own use-cases. It can be secured with TLS, password and/or
certificates.

The message broker is intended to run outside of the system but as an
integrated part of the OS2IoT package.

MQTT has the concept of Quality of Service (“QoS”), in which there are
three levels:

1. At most once (“Fire and forget”)

2. At least once

3. Exactly once

This is part of the message sent from the client to the MQTT broker. The
broker is then responsible for following the QoS level. We can use this
in OS2IoT, and even expose it to the user. Since we (expect to) have a
stable connection between OS2IoT and the data target, “Fire and forget”
will be sufficient in most cases. Read more about MQTT QoS here:
https://www.hivemq.com/blog/mqtt-essentials-part-6-mqtt-quality-of-service-levels/

CoAP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

*[The points below are the things you need to consider for each
integration regardless of whether the service is a traditional web
service, or a file service.]*

-  Service description (utilisation of the service)

-  Functionality (description of how the service works)

-  Validation (which validations are made, when service is called)

-  Applied integration patterns (how to communicate)

-  Error messages (which error messages can you expect from the service)

WebSocket
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Service description (utilisation of the service)

-  Functionality (description of how the service works)

-  Validation (which validations are made, when service is called)

-  Applied integration patterns (how to communicate)

-  Error messages (which error messages can you expect from the service)

WebHook
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

WebHook is an method of integrating via registering a callback and then
having the capability of receiving it. This is also how the SigFox
integration works. The main advantage is that the initiator of the call
is the party which knows that new information is available, thus
needless pooling is avoided.

To support WebHooks as part of OS2IoT, the user should be allowed to
define the URL on which they’ll receive the callbacks, moreover the
communication should be encrypted (over HTTPS) and using a way of
authentication (HTTP basic Auth; or a secret header/url-parameter
definded by the user, configured in OS2IoT).

Depending on the desired quality of service more functionality should be
implemented. If a “fire-and-forget”-strategy is acceptable, then we
should simply implement the callbacks as described above, and leave it
at that.

If the receiver of the data want a higher level of assurance, then there
are several ways of achieving it. A simple solution is retrying with an
exponential back-off. Or an API exposing all messages which have not
been acknowledged by the receiver, for a short period of time (for
instance 3 days like SigFox).

NGSI-LD
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

OS2iot can update attributes on entities in a CIM Context Broker according to the NGSI-LD specification.
https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.01.01_60/gs_CIM009v010101p.pdf

Prerequisites
~~~~~~~~~~~~~

OS2iot can only update existing attributes and existing entities in the Context Broker. This means that the entites must already be created in the Context Broker before OS2iot can send data to the Context Broker. The responsibility for this lies outside the scope of OS2iot.

Integration pattern
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The integration uses the request-response pattern. The following describes how and when data is sent to the CIM Context Broker from OS2iot:
1.	OS2iot receives data from an IoT device
2.	Once the data has been stored in OS2iot and the data has been transformed, the transformed data is sent to the Context Broker
3.	Data is sent as a HTTP PATCH request

Authorization
~~~~~~~~~~~~~~~~~~~~~~~~~~

The FIWARE reference implementations Orion-LD and Scorpio use no authentication or authorization.

Data and format
~~~~~~~~~~~~~~~~~~~~~~~~~~

Data must use the JSON-LD format. FIWARE Smart Data Models define the available entities and attributes:
https://github.com/smart-data-models

Update entity attribute
~~~~~~~~~~~~~~~~~~~~~~~~~~

The NGSI-LD specification defines how attributes in the Context Broker are updated:
https://github.com/FIWARE/context.Orion-LD/blob/develop/doc/manuals-ld/developer-documentation.md#patch-ngsi-ldv1entitiesentityidattrs
A HTTP PATCH request is sent to "/ngsi-ld/v1/entities/{entityId}/attrs" with a JSON-LD object containing the attribute values. The data transformation in OS2iot is responsible for transforming the IoT device payload and OS2iot device metadata to valid JSON-LD.

Opendata.dk
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

OS2iot can send data from IoT devices to opendata.dk by adding the
received data to a opendata.dk resource after it has been received by
OS2iot.

Data is saved in opendata.dk in a Data Store in a Data Set created under
an Organization.

.. _prerequisites-1:

Prerequisites
~~~~~~~~~~~~~

The integration has the following prerequisites:

-  The necessary opendata.dk credentials has been added to the
   organization in OS2iot. This includes the API key needed for
   authenticating with opendata.dk.

-  The Organization and Data Set must be created in opendata.dk.

-  A payload transformation exists in OS2iot for transforming the device
   payload to valid JSON.

When creating or modifying an IoT device in OS2iot, it is possible to
choose to also send device data to opendata.dk. When doing this, the
user must also chose which data transformation to use and provide the
name of the data store.

Integration pattern
~~~~~~~~~~~~~~~~~~~

The following describes how and when data is sent to opendata.dk from
OS2iot

1. OS2iot receives data from an IoT device

2. Once the data has been stored in OS2iot and the payload has been
   transformed, the data (device metadata plus transformed payload) is
   sent to opendata.dk

3. Data is sent as a HTTP PUSH request to an opendata.dk API.

Authorization
~~~~~~~~~~~~~

Each HTTP request to the opendata.dk must contain a header containing
the api key, i.e.:

Authorization: [GUID]

Data and format
~~~~~~~~~~~~~~~

The data sent to opendata.dk is all properties and metadata for the
entity "IoTDevice" along with the transformed payload. Data is sent as
JSON.

.. code-block:: javascript
   
   {
      "DeviceName": "name",
      "Description": "description",
      [...],
      "Metadata": { "metadata 1": "value 1", "metadata 2": "value 2"},
      "Payload": "payload"
   }

The payload value depends on the associated data transformation, but
must be valid JSON.

|image7|

Figure 6 - IoT device and data target data model

Create/update data store
~~~~~~~~~~~~~~~~~~~~~~~~

Before data can be added to opendata.dk, a data store must be created
which includes the schema of the data store. Creation and updating is
done in the same way, but a field cannot change type once it has been
created.

URL: https://demo.ckan.org/api/action/datastore_create

.. code-block:: javascript

   {
      "resource_id": "[data set id]",
      "fields": [
         {
            "id": "DeviceId",
            "type": "text"
         },
         {
            "id": "DeviceName",
            "type": "text"
         },
         {
            "id": "Description",
            "type": "text"
         },
         {
            "id": "BatteryLevel",
            "type": "int"
         },
         [...],
         {
            "id": "Metadata",
            "type": "json"
         },
         {
            "id": "Payload",
            "type": "json"
         }
      ],
      "primary_key": ["DeviceId"],
      "force": "True"
   }


TODO: Decide when this is done and if it happens automatically.

Supported field types:

https://docs.ckan.org/en/2.8/maintaining/datastore.html#field-types

Insert data into data store
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Data is inserted into the data store by sending JSON using a HTTP PUSH
request to opendata.dk.

URL: https://demo.ckan.org/api/action/datastore_upsert

.. code-block:: javascript

   {
      "resource_id": "data store id",
      "method": "upsert",
      "records": [
         {
            "DeviceId": "device id",
            "DeviceName": "device name",
            [...],
            "payload": { "temp":30, "humidity": "high" }
         }
      ],
      "force": "True"
   }




KOMBIT Adgangstyring
-------------------------------------------------------------------
TODO

Active Directory
-------------------------------------------------------------------
TODO

.. |image0| image:: media/image4.emf
   :width: 1.51111in
   :height: 0.23194in
.. |image1| image:: media/image5.png
   :width: 6.56806in
   :height: 4.60556in
.. |image2| image:: media/image6.png
   :width: 5.048in
   :height: 3.13939in
.. |image3| image:: media/image7.png
   :width: 6.56806in
   :height: 4.27569in
.. |image4| image:: media/image8.png
   :width: 6.56806in
   :height: 3.09306in
.. |image5| image:: media/image9.png
   :width: 6.56806in
   :height: 2.27153in
.. |image6| image:: media/image10.png
   :width: 6.56806in
   :height: 1.33819in
.. |image7| image:: media/image12.png
   :width: 4.58209in
   :height: 3.94017in