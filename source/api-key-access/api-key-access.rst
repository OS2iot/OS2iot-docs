API key access
====================

OS2IoT supports API key authentification. This allows external clients to use the system without the need of a browser or a user.

Configuration
-------------

In order to use this type of authentication, an administrator must create an API key with one or more user groups.
This can be done on the portal through the API key management pages. It can be accessed through the menu.


Usage
-----
With an API key, you can fetch and manage data in your OS2IoT system just as well as if you were a user on the browser.
What you can access and manage is limited by the user group (-s) tied to the API key.

With that in mind, let's dive into using an API key. Let's make a few assumptions:

- We have an API key with the value :code:`00000000-abcd-ef00-0000-012345678901`
- This API key is tied to a user group with Read priviliges to an organization and all its applications
- There exists a :code:`GET` endpoint for fetching applications. The route ends with :code:`/application`

There's a broad variety of applications, terminals etc. which can perform HTTP requests. To authenticate this request
with the API key, you must provide it as the header "x-api-key". Below, a screenshot is shown of how this is done in Postman.

|api-key-usage|

The bottom half of the screenshot is the response. The request was limited to 2 applications using the :code:`limit` query parameter.
As seen, the backend responded with 2 (collapsed) applications.


Limitations
-----------
While most of the system is accessible using API key authentication, there are some security limitations, however:

- You cannot manage your API key or someone else's
- You cannot manage users nor user groups
- You cannot manage organizations

Currently, there's no expiration date on an API key. It will not expire unless an administrator revokes it on the key management page.

.. |api-key-usage| image:: ./media/api-key-usage.jpg