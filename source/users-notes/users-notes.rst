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

1. Create a user (e.g. dummy@os2iot.dk) in Chirpstack with the wanted password.
2. Change the password_hash for ’admin’ in the postgres-database:
    a. SSH into the server
    b. docker exec -it os2iot-docker_postgresql_1 sh
    c. psql -U chirpstack_as
    d. update public.user set password_hash = (select password_hash from public.user where email = 'dummy@os2iot.dk') where id=1;
    e. Now the user has been updated
3. Delete the user created in step 1



Background information: 
######################


In order for OS2iot to function, there must be an admin user in chirpstack named admin. This is because Chirpstacks API requires an JWT including  ’username’. You can read about JWT here: https://jwt.io/ and decode:  
’ eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJjaGlycHN0YWNrLWFwcGxpY2F0aW9uLXNlcnZlciIsImV4cCI6MTYxODkyMTQ1NiwiaWQiOjEsImlzcyI6ImNoaXJwc3RhY2stYXBwbGljYXRpb24tc2VydmVyIiwibmJmIjoxNjE4ODM1MDU2LCJzdWIiOiJ1c2VyIiwidXNlcm5hbWUiOiJhZG1pbiJ9.H1ildZ6dj3_G-BkI9EPW6QJk_3QowpLUY4lL4vCcvX0’ 

to 

’ {
  "aud": "chirpstack-application-server",
  "exp": 1618921456,
  "id": 1,
  "iss": "chirpstack-application-server",
  "nbf": 1618835056,
  "sub": "user",
  "username": "admin"
}’ 

That is, if the username is changed the JWT will not work anymore.

Chirpstack requires that all its users has an e-mail, when they are created or changed (Including change of password). Nevertheless, when the application is created the first time / the database is created this is not a requirement. This is why you will not be able to change the password using Chirpstacks UI. 

It is possible to change the requirement that the user must be named admin here:
https://github.com/OS2iot/OS2IoT-backend/blob/master/src/services/chirpstack/jwt-token.ts#L17


