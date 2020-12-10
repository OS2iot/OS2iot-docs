Internal Interface Design
====================================

This document describes the internal interface design of OS2iot. 


Backend REST API
----------------

The primary way of communicating with the back end is through the RESTful API.

Swagger documentation
^^^^^^^^^^^^^^^^^^^^^

The documentation of the endpoints, their parameters and so forth is documented using Swagger. 
You can access Swagger through the endpoint: ``/api/v1/docs/`` of the backend. So for the test environment, 
it would be: `https://test-os2iot-backend.os2iot.dk/api/v1/docs/ <https://test-os2iot-backend.os2iot.dk/api/v1/docs/>`__.

Authentication
^^^^^^^^^^^^^^

The API is protected using JSON Web Token (JWT), see `https://jwt.io/introduction/ <https://jwt.io/introduction/>`__ for an introduction.
To get a JWT you must perform a login using username and password to the ``/api/v1/auth/login`` endpoint.
In response to a valid login, a JWT is sent, inside a JSON object. 

To call protected endpoints the ``Authorization`` header must be set to ``Bearer xxx.yyy.zzz`` where ``xxx.yyy.zzz`` is a valid JWT.

JWT Payload
~~~~~~~~~~~

The JWT payload currently contains two pieces of information: 

- userId of the logged in user.
- email (username) of the logged in user.

JWT Expiration
~~~~~~~~~~~~~~

The JWT is by default set to expire after nine hours. upon which it is invalid. To acquire a new one, 
a new login must be performed as described above.

Using Swagger with authorization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To use Swagger with protected endpoints, you must first login, and then add the JWT to Swagger.
This is done by pressing the "Authorize" button near the top right. 
To test your login, the ``/api/v1/auth/profile`` endpoint can be called, it will return the contents of the JWT payload, 
which is explained below.


Authorization (Permissions)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There is four levels of permissions in OS2IoT:


- Global Admin

 - Can do anything

- Organization Admin 

 - Is scoped to a single organization
 - Can do anything to that organization
 - Can add new users

- Write 

 - Is scoped to a single organization and zero or more applications
 - Can write/create/delete entities within an organization on certain applications

- Read

 - Is scoped to a single organization and zero or more applications
 - Can read (view) entities within certain applications within an organization

The permissions are hieratical, meaning that you implicitly have all lesser permissions than the ones you have explicitly.
For instance, if a user is an Organization Admin for an Organization, then that user also have the Write and Read permissions.
