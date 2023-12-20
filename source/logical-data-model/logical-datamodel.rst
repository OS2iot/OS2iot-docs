Logical Datamodel
===============================

Introduction
------------

The purpose of this deliverable is to describe the logical data-entities
that are part of the solutions domain together the relationship between
them.

The target audience of the deliverable are:

-  The developers, who need to understand the design, and prepare the
   detailed design and eventually implement the solution

-  Participants and decision-makers from the customer who are
   responsible for the functional design

Scope
~~~~~

This is not a database model, but a logical model of the relationship
between business objects.

Logical Datamodel
-----------------

|image1|

Entity metadata
---------------

Unique identifier
~~~~~~~~~~~~~~~~~

Each entity has an id attribute, which is a persistent unique identifier
for an object. This will never change for an object. The identifier is
only unique for this entity, meaning that object of different entities
can have the same id value.

Temporal values
~~~~~~~~~~~~~~~

Data in OS2iot is non-temporal. Each entity has "createdAt" and "updatedAt"
attributes which contains the date and time an object was created and
last modified, respectively. If an object has been created but not
modified, "createdAt" and "updatedAt" contain the same values.

Each entity also has "createdBy" and "updatedBy" attributes, which
contain the username of the user that created the object and the last
person to modify an object. If an object has been created but not
modified, "createdAt" and "updatedAt" contain the same values.

Enumerations
------------

This section contains the relevant enumerations in OS2iot.

PermissionLevel
~~~~~~~~~~~~~~~

1. Read
2. OrganizationUserAdmin
3. OrganizationGatewayAdmin
4. OrganizationApplicationAdmin
5. GlobalAdmin

ActionType
~~~~~~~~~~~~~~~
These are the types of action which can be part of the `AuditLog <../audit-log/audit-log.html>`_.

1. DELETE
2. CREATE
3. UPDATE

SearchResultType
~~~~~~~~~~~~~~~~

1. Gateway
2. IoTDevice
3. Application


AuthorizationType 
~~~~~~~~~~~~~~~~~

1. NO_AUTHORIZATION
2. HTTP_BASIC_AUTHORIZATION
3. HEADER_BASED_AUTHORIZATION


DataTargetType
~~~~~~~~~~~~~~~~~

1. HttpPush
2. Fiware
3. MQTT
4. OpenDataDK

IoTDeviceType
~~~~~~~~~~~~~~~~~

1. GenericHttp
2. LoRaWAN
3. SigFox
4. MQTTInternalBroker
5. MQTTExternalBroker

ErrorCodes 
~~~~~~~~~~~~~~~~~

1. IdDoesNotExists
2. IdMissing
3. NameInvalidOrAlreadyInUse
4. IdInvalidOrAlreadyInUse
5. InvalidApiKey
6. InvalidPost
7. WrongLength
8. NotValidFormat
9. BadEncoding
10. MissingOTAAInfo
11. MissingABPInfo
12. UserAlreadyExists
13. OrganizationAlreadyExists
14. OrganizationDoesNotExists
15. OrganizationDoesNotMatch
16. UserInactive
17. NotSameApplication
18. PasswordNotMetRequirements
19. SigFoxBadLogin
20. GatewayIdNotAllowedInUpdate
21. GroupCanOnlyBeCreatedOncePrOrganization
22. DeviceDoesNotExistInSigFoxForGroup
23. DownlinkNotSupportedForDeviceType
24. DownlinkLengthWrongForSigfox
25. OnlyAllowedForLoRaWANAndSigfox
26. DeviceIsNotActivatedInChirpstack
27. QueryMustNotBeEmpty
28. IsUsed
29. CannotModifyOnKombitUser
30. SigfoxError
31. NoData
32. MissingRole
33. DeleteNotAllowedItemIsInUse
34. DeleteNotAllowedHasSigfoxDevice
35. DeleteNotAllowedHasLoRaWANDevices
36. KOMBITLoginFailed

KafkaTopic 
~~~~~~~~~~~~~~~~~

1. RAW_REQUEST
2. TRANSFORMED_REQUEST

ActivationType
~~~~~~~~~~~~~~~~~
1. NONE
2. OTAA
3. ABP

PermissionType 
~~~~~~~~~~~~~~~~~

1. GlobalAdmin
2. OrganizationApplicationAdmin
3. OrganizationGatewayAdmin
4. OrganizationUserAdmin
5. Read

SendStatus
~~~~~~~~~~~~~~~~~

1. OK
2. ERROR

SigFoxDownlinkMode
~~~~~~~~~~~~~~~~~~
1. DIRECT
2. CALLBACK
3. NONE
4. MANAGED

SigFoxPayloadType
~~~~~~~~~~~~~~~~~
1. RegularRawPayload
2. CustomGrammar
3. Geolocation
4. DisplayInASCII
5. RadioPlanningFrame
6. Sensitv2

AuthenticationType
~~~~~~~~~~~~~~~~~~
1. PASSWORD
2. CERTIFICATE

.. |image1| image:: ./media/logical-datamodel.png

