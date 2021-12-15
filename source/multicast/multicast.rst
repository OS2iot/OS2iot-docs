Multicast
======================

This chapter describes the multicast-groups.

What can multicast do?
---------------------------------------------
Multicast-groups makes it possible to send a single downlink payload to a group of devices.

A multicast consist of some different properties like multicast-address, session-keys and frame counter. All these properties makes it possible to send only one downlink payload to a group
of devices instead of sending many downlink payloads to devices one by one.

Creation in OS2iot
-------------------
If you want to make a multicast in OS2iot you have to go into a specific application and then create the multicast from there.

You have to fill out all the required forms, and in the end you can add some devices. It's required that the devices share the same service profile, which is set when you create a LoRaWAN device. Otherwise, the multicast-group can't be created.

When you have added the devices that have the same service profile, and have filled out the other forms, then a multicast-group will be created in the database and in Chirpstack.

Sending downlink payload
-------------------------

When the multicast-group is created it's possible to send a downlink payload to the group of devices. This is done by navigating to the details about the multicast-group, and from here it's possible to send choose a port and a payload that you wish to send to the devices.

The downlink payload is send using Chirpstack which also has the multicast-group with the devices.  

------------

