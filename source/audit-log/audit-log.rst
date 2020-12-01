Audit Log
=========

Whenever an action that creates, updates or deletes an entity in OS2iot, it is logged.
We refer to this as audit logging, since it can be used to audit who have performed changes to certain entities.

Reading the audit-logged
------------------------

The audit log is logged in JSON format like so:

.. code-block:: javascript 

    [Nest] 16976   - 2020-12-01 10:50:37   [AuditLog] {"userId":1,"timestamp":"2020-12-01T09:50:37.950Z","actionType":"CREATE","type":"Application","id":271,"name":"Demo for audit log","completed":true}

Formatting the JSON part to be more readable yields this:

.. code-block:: javascript 

    {
        "userId": 1,
        "timestamp": "2020-12-01T09:50:37.950Z",
        "actionType": "CREATE",
        "type": "Application",
        "id": 271,
        "name": "Demo for audit log",
        "completed": true
    }

Explaination of each part:

1. :code:`userId`
    a. This is the id of the user who performed the action.
2. :code:`timestamp`
    a. This is the time the action was performed, this is given in zulu time (without timezone).
3. :code:`actionType`
    a. This is the type of action which was performed, can be either :code:`CREATE`, :code:`UPDATE`, or :code:`DELETE`.
4. :code:`type`
    a. This is the type of entity which was changed, for instance :code:`Application`, :code:`User`, or :code:`IoTDevice`.
5. :code:`id`
    a. This is the id of the entity which was changed. This can be null, for instance if it was a :code:`CREATE` that failed.
6. :code:`name`
    a. This is the name of the entity if applicable, otherwise it can be null.
7. :code:`completed`
    a. This is a boolean indication if the action was completede or not, in other words if it was successful.
