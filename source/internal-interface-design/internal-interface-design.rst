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
Permissions are given on specific applications to users and API keys through UserGroups. A UserGroup can have multiple permissions. 

There are five levels of permissions in OS2IoT:

- Global Admin

 - Can do everything for all organizations and applications

- Application Admin

 - Is scoped to a single organization and zero or more applications
 - Can access and modify applications and Sigfox devices within the user group in that organization

- Gateway Admin 

 - Is scoped to a single organization
 - Can access and modify gateways within that organization

- User Admin 

 - Is scoped to a single organization
 - Can access and modify users and permissions within that organization

- Read

 - Is scoped to a single organization and zero or more applications
 - Can read (view) entities within certain applications within an organization

Each of the admin permissions is part of a hierarchy with the read permission. If you have an Admin permission within an organization, with zero or more applications, you have an
implicit read permission within that scope.
For instance, if a user has Application Admin within an Organization, then that user also has Read permission within it.

Global Admin is at the top of the hierarchy and can thus do what any of the other permissions provide access to.

.. include:: api-key-access.rst