Software Architecture (O0500)
=====================================

Introduction
------------

Architectural goals and constraints
-----------------------------------

Use case perspective
--------------------

TBA

Logical perspective
-------------------

This section describes the logical architecture of OS2iot.

Overview
~~~~~~~~

The solution is divided into a number of logical, loosely coupled
components. This ensures that the solution is extensible with regard to
both supported IoT devices and data targets, and it makes replacing and
maintaining each component simpler. Figure 1 shows an overview of the
different solution components. Orange components are 3\ :sup:`rd` party
solution dependencies that are not developed or maintained in this
project, but are necessary for OS2iot to function

Figure 1 - Solution overview

Figure 2 - Layered architecture

Logging
~~~~~~~

OS2iot uses the following log types:

-  **System log**: Contains log messages related to the solution such as
   information or errors from internal processes or internal/external
   integrations, views etc. This is stored in the database table
   "SystemLog".

-  **Audit Log**: When a data entity is changed (e.g. an IoT device is
   created or updated) OS2iot logs which user made the change and when
   it was made. This is stored in the database table "ChangeLog".

In addition, the 3\ :sup:`rd` party components used (Chirpstack, MQTT
brokers etc.) contain their own logs according to their documentation.
Refer to section for a specific list of 3\ :sup:`rd` party components.

Integrations
~~~~~~~~~~~~

|image1|

Process perspective
-------------------

Receive IoT device data
~~~~~~~~~~~~~~~~~~~~~~~

|image2|

As the message broker, Kafka is used. Furthermore ZooKeeper is used to
manage it. We deploy both as separate docker containers as part of our
docker-compose for development and to Kubernetes (via. Helm). All the
processing will be done by stateless functions, the data we pass on the
Kafka event stream will never contain copies of the entities stored in
the database. And only contain their ids, or another unique reference.
This way we avoid using stale data, or other inaccuracies. Instead we
opt to query the database more often, but since the queries will mostly
be very simple, i.e. fetching a single row, or a row with a joined
table, the cost is deemed low.

Raise alarm
~~~~~~~~~~~

Implementation perspective
--------------------------

OS2iot is implemented using Node.js for the backend and Angular for the
frontend.

3\ :sup:`rd` party components
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

OS2iot uses the following 3\ :sup:`rd` party components as dependencies:

============ ============================================================================== =========================== ====================
Component    Use                                                                            Reference                   License
============ ============================================================================== =========================== ====================
Chirpstack   LoRaWAN device Integration                                                     https://chirpstack.io       MIT License
Mosquitto    MQTT broker for LoRaWAN and data target integrations                           https://mosquitto.org/      EPL/EDL License
Apache Kafka Internal message broker used in the OS2iot backed for device data integrations https://kafka.apache.org/   Apache License 2.0
PostgreSQL   Persistent data storage                                                        https://www.postgresql.org/ PostgreSQL License
Redis        In-memory data store                                                           https://redis.io/           BSD License
Docker       Virtualization software                                                        https://www.docker.com/     Apache License 2.0
============ ============================================================================== =========================== ====================

Backend
~~~~~~~

Technology stack
^^^^^^^^^^^^^^^^

In the backend we use the following technologies:

========== ========================================================================== =============================== ==================
Technology Purpose                                                                    URL                             License
========== ========================================================================== =============================== ==================
Node.js    Server-side Javascript runtime                                             https://nodejs.org/en/          MIT License
Typescript Typesafety and other improvements upon Javascript (compiles to Javascript) https://www.typescriptlang.org/ Apache License 2.0
Jest       Testing framework                                                          https://jestjs.io/              MIT License
TypeORM    Object Relatrional Mapper to our database (persistence)                    https://typeorm.io/             MIT License
Nest.js    Web framework                                                              https://nestjs.com/             MIT License
========== ========================================================================== =============================== ==================

Solution architecture
^^^^^^^^^^^^^^^^^^^^^

We use a classical three-layer model consisting of the following three
layers:

-  Controller layer

   -  Exposes the API endpoints:

      -  To be used by the front-end

      -  To be used by device integrations

      -  To be used by data targets

-  Service layer

   -  Holds business logic

-  Data access layer

   -  Controls all access to the database

Each layer is only capable of accessing the adjacent layers, so the
controller cannot access the data access layer and vice versa.

Frontend
~~~~~~~~

Dennis udfylder dette

.. _technology-stack-1:

Technology stack
^^^^^^^^^^^^^^^^

========== ============= ================== ===========
Technology Purpose       URL                License
========== ============= ================== ===========
Angular    Web framework http://angular.io/ MIT License
========== ============= ================== ===========

.. _solution-architecture-1:

Solution architecture
^^^^^^^^^^^^^^^^^^^^^

Deployment perspective
----------------------

The solution is deployed as a number of Docker containers.

-  OS2iot Frontend

-  OS2iot Backend

-  Data target MQTT Broker

-  Chirpstack

-  LoRaWAN MQTT Broker

-  Postgres

-  Apache Kafka

Docker Compose is used to ease deployment of the solution. For
scalability and increased robustness, the solution can be deployed to a
cluster such as Kubernetes, or OpenShift.

Of these container only the OS2iot frontend and OS2iot backend
containers are made in the OS2iot project, the remaining is made by
3\ :sup:`rd` parties and used as part of the solution.

Data perspective
----------------

OS2iot contains the following types of data:

-  Device payloads (only the latest payload from a device is stored)

-  Metadata about device payloads (timestamps etc of the latest N
   transmissions or all within a small timeframe)

-  System parameters and configuration

-  User data (usernames, passwords and permissions)

-  Audit logs

-  System logs

-  Application data (applications, devices, alarms, gateways, device
   metadata etc.)

This data is by default stored by the backend in PostgreSQL. The logs
are stored in the filesystem, with the future possibility of ingesting
it into an ELK stack or similar.

Data temporality
~~~~~~~~~~~~~~~~

Data in OS2iot is non-temporal. Each entity has "createdAt" and
"updatedAt" attributes which contains the date and time an object was
created and last modified, respectively. If an object has been created
but not modified, "createdAt" and "updatedAt" contain the same values.

Each entity also has "createdBy" and "modifiedBy" attributes, which
contain the username of the user that created the object and the last
person to modify an object. If an object has been created but not
modified, "createdBy" and "modifiedBy" contain the same values.

Security perspective
--------------------

|image3|

User login and permissions
~~~~~~~~~~~~~~~~~~~~~~~~~~

In OS2iot, user authentication is done by either en external system or
by OS2iot. Authorization is handled by OS2iot by assigning users to
either organizations or applications with a given permission level.

This does not comply with "Den fælleskommunale rammearkitektur", which
states that authorization should happen in KOMBIT Adgangstyring if
possible. The reasons for authorization being done in OS2iot instead of
KOMBIT Adgangstyring are:

-  To make user management in OS2iot uniform regardless of where the
   user logs in from.

-  To support separate permissions to organizations and applications,
   along with dynamically created applications.

Authentication
^^^^^^^^^^^^^^

User authentication is handled in one of two systems:

-  KOMBIT Adgangsstyring

-  OS2iot

Authorization
^^^^^^^^^^^^^

By default, a user does not have access to data in OS2iot. A global
admin or Organization admin must manually give the user permissions to
organizations or applications.

User permissions
^^^^^^^^^^^^^^^^

================== =========== =======================================================
User role          System name Permissions
================== =========== =======================================================
Read access        Read        Read all data within an application.
Write              Write       Create, modify and delete objects within an application
Organization admin Orgadmin   
Global admin       Globaladmin
================== =========== =======================================================

Web application security
~~~~~~~~~~~~~~~~~~~~~~~~

This section describes the security measues taken to ensure
conficentiality an integrity of the part of OS2iot that is the web
application. This includes both the frontend and backend of the
solution, but not IoT device integrations or data target integrations.

Device security
~~~~~~~~~~~~~~~

LoRaWAN
^^^^^^^

Data fra LoRaWAN devices are end-to-end encrypted and protected against
replay attacks
(https://lora-alliance.org/sites/default/files/2019-05/lorawan_security_whitepaper.pdf).
There is a theoretical possibility of packet forging and DoS attacks
(https://backend.orbit.dtu.dk/ws/portalfiles/portal/200458018/PID5885861.pdf,
https://ieeexplore.ieee.org/document/8766430/).

Once device data is received by Chirpstack it is sent to OS2iot using
MQTT and TLS.

NB-IoT
^^^^^^

Security for NB-IoT is enforced by Telia as the network operator or by
the IoT-device administrators. Data sent to OS2iot from the devices are
encrypted using TLS.

Sigfox
^^^^^^

Data from Sigfox devices are sent to OS2iot using callbacks from the
Sigfox core network. These are encrypted using TLS.

| Sigfox security is described in detail here:
| https://www.sigfox.com/sites/default/files/1701-SIGFOX-White_Paper_Security.pdf

According to an article from DTU published in Proceedings of 3rd Global
IoT Summit
(https://backend.orbit.dtu.dk/ws/portalfiles/portal/200458018/PID5885861.pdf,
https://ieeexplore.ieee.org/document/8766430/), Sigfox should not be
used for critical applications due to poor protection from replay
attacks.

.. |image0| image:: ./media/image4.emf
   :width: 1.51111in
   :height: 0.23194in
.. |image1| image:: ./media/image7.png
   :width: 6.56806in
   :height: 4.89861in
.. |image2| image:: ./media/image8.png
   :width: 6.56806in
   :height: 3.78125in
.. |image3| image:: ./media/image9.png
   :width: 6.56806in
   :height: 3in