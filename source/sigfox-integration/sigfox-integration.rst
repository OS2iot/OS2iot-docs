Manual SigFox Integration
=========================

This document describes how to manually setup a SigFox Integration in OS2IoT.

Pre-requisites:

1. The device is activated in SigFox'es backend.
2. You have administration access to the group the device and devicetype belongs to.

How-to:

1. Add new SigFox device to OS2IoT

  a. Input the SigFox' deviceId from the back of the device
  b. Input the SigFox' deviceTypeId, this can be collected from the SigFox backend.

2. Add data-target and payload decoder if needed.
3. Goto the SigFox backend.

  a. Navigate to the :code:`DEVICE TYPE` in the top menu.
  b. Goto the device type of the device.
  c. Goto :code:`CALLBACKS` in the left menu.
  d. Press :code:`New` in the top right, to add a new callback.
  e. Enter the following:

    i. Type: :code:`DATA`- :code:`UPLINK`
    ii. Channel: :code:`URL`
    iii. Custom payload config: (leave blank)
    iv. Url pattern: :code:`https://<hostname-of-backend>/api/v1/sigfox-callback/data/uplink?apiKey={deviceTypeId}` (replace <hostname-of-backend> with your actual hostname.)
    v. Use HTTP Method: :code:`POST`
    vi. Send SNI: Check the box if SNI is required for your webserver.
    vii. Content type: :code:`application/json`
    viii. Body: 

        .. code-block:: javascript

            {
                "time": {time},
                "deviceTypeId": "{deviceTypeId}",
                "deviceId": "{device}",
                "snr": {snr},
                "rssi": {rssi},
                "station": "{station}",
                "data": "{data}",
                "seqNumber": {seqNumber}
            }

  f. Press "OK" to save.
