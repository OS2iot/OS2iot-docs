Installation guide
==================

Purpose
-------

This page is intended for audiences who wants to run OS2iot on their own machine to test it out, or deploy to a server for a production environment.

Running locally using docker-compose
------------------------------------

Prerequisites
^^^^^^^^^^^^^

1. docker version version 19.03.13 (or above)
2. docker-compose version 1.27.4 (or above)

Configuration
^^^^^^^^^^^^^

1. If the application needs to be accessable from non-localhost machines then modify the :code:`environment.prod.ts` file in :code:`OS2IoT-frontend/src/environments` folder.

   a. This changes the location the frontend accesses the backend on, so this URL should be the way another computer would reach the computer running docker-compose.

2. If callbacks from Sigfox is required then the backend needs to know what URL to configure Sigfox to callback to.

   a. This is done by setting the environment variable :code:`BACKEND_BASEURL` in the runtime environment for the backend.

   b. In docker-compose this can be set in the :code:`docker-compose.yml` file. 

      i. I.e. :code:`BACKEND_BASEURL: "https://os2iot.dk"`

Steps
^^^^^

1. Clone the three repositories into the same folder, such that the structure is as follows:

   a. OS2IoT

       ├── OS2IoT-backend (https://github.com/OS2iot/OS2IoT-backend)

       ├── OS2IoT-docker (https://github.com/OS2iot/OS2IoT-docker)
       
       ├── OS2IoT-frontend (https://github.com/OS2iot/OS2IoT-frontend)

2. Make sure that Chirpstacks configuration files for initializing the postgres database is in unix format, such that the docker container can run it

   a. This can be done with dos2unix or another tool 

         .. code-block:: bash
         
            dos2unix OS2IoT-docker/configuration/postgresql/initdb/001-init-chirpstack_ns.sh OS2IoT-docker/configuration/postgresql/initdb/002-init-chirpstack_as.sh OS2IoT-docker/configuration/postgresql/initdb/003-chirpstack_as_trgm.sh OS2IoT-docker/configuration/postgresql/initdb/004-chirpstack_as_hstore.sh

3. Build the docker containers using docker-compose

   a. navigate to the OS2IoT-docker folder in your terminal

   b. Run :code:`docker-compose build`

4. Run the docker containers using docker-compose

   a. Run :code:`docker-compose run`

      1. Use the :code:`-d` flag to start it in the background.

5. Access the frontend on http://localhost:8081/auth'

6. If this instance of OS2IoT should be accessible from other machines, then naturally the relevant ports should be opened in the firewall(s).

   a. Relevant ports: 

      i. Frontend: 8081

      ii. Backend: 3000

      iii. Chirpstack Application Server: 8080

      iv. Chirpstack Gateway (UDP from gateways to Chirpstack): 1700

Troubleshooting
^^^^^^^^^^^^^^^

Error: Issue connecting to Chirpstacks PostgreSQL.

Fix:
Goto os2iot-docker in your a terminal

-  Run: :code:`docker volume ls`

Located the image name os2iot-docker_postgresqldata and delete it by running:

-  :code:`docker-compose down`

-  :code:`docker volume rm os2iot-docker_postgresqldata`

Add os2iot-docker to docker files share (Described in the quick setup).
Once the path is added run:

-  :code:`docker-compose up`

More docker related troubleshooting can be found at: https://github.com/OS2iot/OS2IoT-docker#troubleshooting-faq

Running in Kubernetes
---------------------

During the development of OS2IoT, Azure was used for hosting the solution.

`Helm <https://helm.sh/>`_ was used for configuration, the Helm charts used can be found in the :code:`OS2IoT-docker/helm` folder.

Disclaimer
^^^^^^^^^^

This configuration is not intended for a production setup. 
There is no persistent volumes for the database(s), Redis, Kafka etc. so data will not be persisted when using this configuration.
There is no log gathering in this setup.
These should at least be resolved before deploying OS2iot in a production setup. 

Setting up an environment
^^^^^^^^^^^^^^^^^^^^^^^^^

During development the following was used:

   1. Azure AKS to run Kubernetes.

   2. Azure ACR to hold our docker images.

   3. Azure DNS to manage DNS.

   4. Azure Load balancer to load balancing the traffic to Azure AKS.

      a. Both TCP (HTTP) traffic for web-browsers, and HTTP callbacks.

      b. ... and UDP traffic to chirpstack-gateway-bridge on port 1700 in a separate loadbalancer.

   5. Azure VM to host Jenkins.

The exact steps will depend on the requirements for the exact deployment, and therefore it is left as an exercise for the reader. 

Deployment
^^^^^^^^^^

Jenkins was used for deployment, the deploy used the following shell script to perform the deploy.
Sensitive information have been redacted.

.. code-block:: bash 

   #!/bin/sh
   set -xe

   az login --service-principal --username redacted --password redacted --tenant redacted
   az acr login --name os2iot

   # Build containers
   sed -i "s/baseUrl: 'http:\/\/localhost:3000\/api\/v1\/'/baseUrl: 'https:\/\/${namespace}-os2iot-backend.os2iot.dk\/api\/v1\/'/" OS2IoT-frontend/src/environments/environment.prod.ts
   # Replace BACKEND_BASEURL for backend:
   sed -i "s/'https:\/\/test-os2iot-backend.os2iot.dk'/'https:\/\/${namespace}-os2iot-backend.os2iot.dk'/" OS2IoT-docker/helm/charts/os2iot-backend/templates/deployment.yaml

   if $USE_DOCKER_BUILD_CACHE; then export OPTIONAL_ARGS=""; else export OPTIONAL_ARGS="--no-cache"; fi

   docker build $OPTIONAL_ARGS -t os2iot-backend:${BUILD_NUMBER} ./OS2IoT-backend
   docker build $OPTIONAL_ARGS -t os2iot-frontend:${BUILD_NUMBER} -f ./OS2IoT-frontend/Dockerfile-prod ./OS2IoT-frontend

   # Tag and push to ACR
   docker tag os2iot-backend:${BUILD_NUMBER} os2iot.azurecr.io/os2iot-backend:${BUILD_NUMBER}
   docker push os2iot.azurecr.io/os2iot-backend:${BUILD_NUMBER}

   docker tag os2iot-frontend:${BUILD_NUMBER} os2iot.azurecr.io/os2iot-frontend:${BUILD_NUMBER}
   docker push os2iot.azurecr.io/os2iot-frontend:${BUILD_NUMBER}

   # Setup  right private key for KOMBIT
   if [ "${namespace}" = "test" ]; then export PRIVATEKEY="-----BEGIN PRIVATE KEY-----\nM-REDACTEDoP\n-----END PRIVATE KEY-----"; fi
   if [ "${namespace}" = "demo" ]; then export PRIVATEKEY="-----BEGIN PRIVATE KEY-----\nM-REDACTEDoP\n-----END PRIVATE KEY-----"; fi

   if [ "${namespace}" = "test" ]; then export ENTRYPOINT="https://adgangsstyring.eksterntest-stoettesystemerne.dk/runtime/saml2/issue.idp"; fi
   if [ "${namespace}" = "demo" ]; then export ENTRYPOINT="https://adgangsstyring.stoettesystemerne.dk/runtime/saml2/issue.idp"; fi

   # Create namespace or not
   NOT_EXISTS=`kubectl get po -n ${namespace} 2>&1 | grep "No resources" | wc -l`
   if [ "$NOT_EXISTS" = "1" ]; then kubectl create namespace ${namespace}; fi

   # Helm deploy
   cat <<EOT >> OS2IoT-docker/helm/values.yaml
   os2iot-backend:
     DOCKER_IMAGE_TAG: $BUILD_NUMBER
     KOMBIT_CERTIFICATEPRIVATEKEY: '$PRIVATEKEY'
     KOMBIT_ENTRYPOINT: '$ENTRYPOINT'
   os2iot-frontend:
     DOCKER_IMAGE_TAG: $BUILD_NUMBER
   EOT

   helm upgrade --install os2iot ./OS2IoT-docker/helm --namespace ${namespace}

Configuration
-------------

OS2IoT-backend
^^^^^^^^^^^^^^

OS2IoT-backend takes several environment variables as configuration, if these are not set a default will be used.

+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| Environment variable          | Purpose                                                                                              | Default value                                                                           |
+===============================+======================================================================================================+=========================================================================================+
| PORT                          | Port to run the backend on.                                                                          | :code:`3000`                                                                            |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| DATABASE_HOSTNAME             | Hostname to connect to Postgresql on                                                                 | :code:`host.docker.internal`                                                            |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| DATABASE_PORT                 | Port to connect to Postgresql on                                                                     | :code:`5433`                                                                            |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| DATABASE_USERNAME             | Username for Postgresql                                                                              | :code:`os2iot`                                                                          |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| DATABASE_PASSWORD             | Password for Postgresql                                                                              | :code:`toi2so`                                                                          |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| DATABASE_ENABLE_SSL           | Enable SSL for database connection                                                                   | :code:`false`                                                                           |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| JWT_SECRET                    | Secret value to sign JWT (THIS SHOULD BE CHANGED!)                                                   | :code:`secretKey-os2iot-secretKey`                                                      |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| JWT_EXPIRESIN                 | Time to expiry for the JWT tokens used                                                               | :code:`9h`                                                                              |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| BACKEND_BASEURL               | URL for external services to connect to the backend (THIS SHOULD BE CHANGED!)                        | :code:`https://test-os2iot-backend.os2iot.dk`                                           |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| FRONTEND_BASEURL              | URL for the frontend, used when building email links (THIS SHOULD BE CHANGED!)                       | :code:`http://localhost:8081`                                                           |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| EMAIL_HOST                    | URL for the SMTP server, used when sending emails (THIS SHOULD BE CHANGED!)                          | :code:`"smtp.ethereal.email"`                                                           |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| EMAIL_PORT                    | Port for the email server, used when sending emails (THIS SHOULD BE CHANGED!)                        | :code:`587`                                                                             |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| EMAIL_USER                    | Username for email server authentification, used when sending emails (THIS SHOULD BE CHANGED!)       | :code:`"ara.kertzmann8@ethereal.email"`                                                 |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| EMAIL_PASS                    | Password for email server authentification, used when sending emails (THIS SHOULD BE CHANGED!)       | :code:`"KzRSyYReEygpFPPZdd"`                                                            |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| EMAIL_FROM                    | Email sender address. Either a plain address or a display name and the address (CHANGE IT!)          | :code:`"OS2iot ara.kertzmann8@ethereal.email"`                                          |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| KOMBIT_ENTRYPOINT             | The context broker URL for KOMBIT adgangsstyring                                                     | :code:`https://adgangsstyring.eksterntest-stoettesystemerne.dk/runtime/saml2/issue.idp` |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| KOMBIT_CERTIFICATEPRIVATEKEY  | The certificate  private key for KOMBIT adgangsstyring                                               | :code:`null`                                                                            |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| KOMBIT_CERTIFICATEPUBLICKEY   | Public certificate from the KOMBIT idp for verifying SAML response                                   | :code:`"INSERT_KOMBIT_CERT"`                                                            |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| KOMBIT_ROLE_NAME              | This string must be a substring of the brugersystemrolle you grant users for them to be given access | :code:`http://os2iot.dk/roles/usersystemrole/adgang/`                                   |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| CHIRPSTACK_JWTSECRET          | Secret to generate JWT for Chirpstack                                                                | :code:`verysecret`                                                                      |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| LOG_LEVEL                     | Minimum Log Level. Levels ordered from high to low are: 'log', 'error', 'warn', 'debug', 'verbose'   | :code:`debug`                                                                           |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| METADATA_SAVED_COUNT          | Maximum number of message metadata to store from an IoT device                                       | :code:`20`                                                                              |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| MQTT_BROKER_HOSTNAME          | The hostname of the MQTT broker.                                                                     | :code:`localhost`                                                                       |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| MQTT_SUPER_USER_PASSWORD      | The password for the internal MQTT listener.                                                         | :code:`SuperUser`                                                                       |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| ENCRYPTION_SYMMETRIC_KEY      | A symmetric key that is used for encrypting                                                          | :code:`SecretKey`                                                                       |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| CA_KEY_PASSWORD               | The password for the Certificate Authority key.                                                      | :code:`os2iot`                                                                          |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+

Logs levels
"""""""""""""""
Specifying a LOG_LEVEL makes sure that only logs with that level or higher are included. Using 'debug' or 'verbose' LOG_LEVEL in a production environment is not recommended.


OS2IoT-frontend
^^^^^^^^^^^^^^^

The frontend can also be configured using environment variables. If these are not set a default will be used.
Defaults are set in :code:`OS2IoT-frontend/src/environments/environment.ts`

+-------------------------------+--------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| Environment variable          | Purpose                                                                                                      | Default value                                                                           |
+===============================+==============================================================================================================+=========================================================================================+
| PRODUCTION                    | If true, then Angular is set in production mode, disabling debugging features                                | :code:`false`                                                                           |
+-------------------------------+--------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| BASE_URL                      | The Url which users will connect to the backend from. This must be changed for the system to work externally | :code:`http://localhost:3000/api/v1/`                                                   |
+-------------------------------+--------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| TABLE_PAGE_SIZE               | Default page size of tables                                                                                  | :code:`25`                                                                              |
+-------------------------------+--------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+

OS2IoT-Mosquitto broker
^^^^^^^^^^^^^^^^^^^^^^^

To get the mosquitto broker working, you have to create some certificates and update some values. These following steps is done with Windows and it's required that you have openssl installed. If you use linux, then write :code:`sudo` before the commands (you still have to install openssl).

   1. Open the command prompt in administrator mode.
   
   2. The following certificates and keys HAS to be placed in the folder "OS2IoT-docker/configuration/mosquitto-broker-os2iot", so it's recommended to navigate to that folder from the start.

   3. Create a certificate authority(CA) key with this command: :code:`openssl genrsa -des3 -out ca.key 2048`. You will be prompted to enter a password. It's very important that you save this password, since it will be used later.

   4. Create the CA certificate with this command: :code:`openssl req -new -x509 -days 1826 -key ca.key -out ca.crt`. You will be asked to enter the password from the step before. After this, you will be prompted to enter informations. These values are not important, except one: "Common name". Common name HAS to be the ip/hostname of your broker.

   5. Create the server key (for the broker) with the command: :code:`openssl genrsa -out server.key 2048`

   6. Create the server signing request with the command: :code:`openssl req -new -out server.csr -key server.key`. You will be prompted to enter some informations. These values are not important, except one: "Common name". Common name HAS to be the ip/hostname of your broker. The rest of the values should not be exact the same as in step 4.

   7. Create the server certificate (that is signed by the CA) with this command: :code:`openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 360`. You will be prompted to enter the password from step 3.

   8. If the ca.crt, ca.key, server.crt and server.key aren't placed in the folder "OS2IoT-docker/configuration/mosquitto-broker-os2iot" then place them in that folder.

   9. Open the mosquitto-os2iot.conf file placed in OS2IoT-docker/configuration/mosquitto-broker-os2iot in a text editor and update the values about your database.

   10. Copy the files ca.crt and ca.key and place them in OS2IoT-backend/resources.

   11. Update the :code:`MQTT_BROKER_HOSTNAME` with the ip/hostname that you used for step 4 and 6, and :code:`CA_KEY_PASSWORD` with the password that you entered in step 3 in the docker-compose.yml file placed in OS2IoT-docker.

If you want to use kubernetes then you need some futher steps. First you have to install kubectl.

   1. Open a command prompt in administrator mode.

   2. Create a secret with the server.key and server.crt with the command: :code:`kubectl create secret generic server-keys --from-file=server.key=path/to/server.key --from-file=server.crt=path/to/server.crt`. Replace path/to/ with the path to your server.key and server.crt, created in the steps above.

   3. Create a secret with the ca.crt and ca.key with the command: :code:`kubectl create secret generic ca-keys --from-file=ca.crt=path/to/ca.crt --from-file=ca.key=path/to/ca.key`. Replace path/to/ with the path to your server.key and server.crt, created in the steps above.

   4. Update the empty values in OS2IoT-docker/helm/charts/mosquitto-os2iot/values.yaml

   5. Update the :code:`MQTT_BROKER_HOSTNAME` with the ip/hostname that you used for step 4 and 6 in the steps above, and :code:`CA_KEY_PASSWORD` with the password that you entered in step 3 in the steps above, in the file "OS2IoT-docker/helm/charts/os2iot-backend/deployment.yaml".