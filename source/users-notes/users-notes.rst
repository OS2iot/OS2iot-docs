Users' notes
============

This page is meant to serve as useful information for users of the system.

Default passwords
-----------------

These passwords are the default, and should be changed in the respective systems.

========================================== ======================= =====================
System                                     Username                Password
========================================== ======================= =====================
OS2iot                                     global-admin@os2iot.dk  hunter2
OS2iot-postgres                            os2iot                  toi2so
Chirpstack                                 admin                   admin
Chirpstack-postgres (Network server)       chirpstack_ns           chirpstack_ns
Chirpstack-postgres (Application server)   chirpstack_as           chirpstack_as
========================================== ======================= =====================


How to change chirpstacks admin password
----------------------------------------

Step-by-step:
############

1. Log in to the ChirpStack Application Server web interface using the default credentials
2. Change the password of the admin user under: 'All users' -> 'admin' -> 'Change Password'

In previous versions of ChirpStack Application Server it wasn't possible to change the admin user password through the UI, but since v.3.13.0 this as worked as expected.

Alternatively the Admin user can removed and replaced by another user entirely. This requires changing the user name that OS2iot-backend uses:
https://github.com/OS2iot/OS2IoT-backend/blob/master/src/services/chirpstack/jwt-token.ts#L17


