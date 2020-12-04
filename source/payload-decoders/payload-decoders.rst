Payload Decoders
================================

Payload Decoders in OS2IoT is used to decode the payloads sent from devices.
These are often raw bytes to minimize payload size, and not JSON or some other more 'human readable' language.

To convert these raw bytes to JSON which will be sent to the data-target, the users of OS2IoT can write payload decoders.

The payload decoders are written in Javascript and executed in a sandbox (https://github.com/patriksimek/vm2) for security and isolation reasons.

The backend expects that the following function prototype to be defined in the payload decoder:

.. code-block:: javascript 
      
   function decode(payload, metadata) {
      // ...
   }

The `payload` parameter is a javascript object containing the payload as received by OS2IoT.
The `metadata` parameter is a javascript object containing the IoTDevice details saved in OS2IoT.

It must return an object, this object will be serialized using JSON.stringify() before it is sent to the data-target.

The user interface provides a testing environment for payload decoder, which can take in data.

Examples of Payload Decoders
-----------------------------

Sensit 3.0
^^^^^^^^^^

Source: https://stackoverflow.com/questions/52534844/decoding-sensit-3-payload-data

.. literalinclude :: ./decode-sensit-payload.js
   :language: javascript

Elsys ERS
^^^^^^^^^

.. literalinclude :: ./decode-elsys-ers-payload.js
   :language: javascript
