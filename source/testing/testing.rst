Testing
=======

The section describes how OS2iot is tested.

Automated testing
-----------------

The primary method for tested the code is through automated integration-tests (called end-to-end) in which (parts of) the application and its dependencies are started. 
Most of the tests involve setting up some data, making one or more RESTful HTTP requests to the application, and then asserting that the application responded as expected, and that the state was changed as expected, i.e. if the application was created.

These tests cover close to 90% of all the code in the backend, from nearly 200 tests.

Test execution prerequisites
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. node (>=v12) and npm (>=6.14)

2. docker (>=19.03) and docker-compose (>=1.27)

Running the tests
^^^^^^^^^^^^^^^^^

1. Clone the OS2iot-docker and OS2iot-backend to seperate folders.

    a. Start the dependencies using docker-compose from te OS2IoT-docker folder: 

        i. :code:`docker-compose up --force-recreate --build -d chirpstack postgresql postgresCsV4 chirpstack-gateway-bridge mosquitto-os2iot mosquitto redis os2iot-kafka os2iot-postgresql os2iot-zookeeper`

2. Install dependencies to run test from the OS2IoT-backend folder:

    a. :code:`npm install`

3. Run the tests using NPM:

    a. :code:`npm run test:e2e`

Manual testing
--------------
In addition to automated tests the solution is also tested manually. There is a large collection of ~122 test-cases to cover all the acceptance criteria and use cases.
