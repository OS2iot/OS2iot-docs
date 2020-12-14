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
| JWT_SECRET                    | Secret value to sign JWT (THIS SHOULD BE CHANGED!)                                                   | :code:`secretKey-os2iot-secretKey`                                                      |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| JWT_EXPIRESIN                 | Time to expiry for the JWT tokens used                                                               | :code:`9h`                                                                              |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| BACKEND_BASEURL               | URL for external services to connect to the backend (THIS SHOULD BE CHANGED!)                        | :code:`https://test-os2iot-backend.os2iot.dk`                                           |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| KOMBIT_ENTRYPOINT             | The context broker URL for KOMBIT adgangsstyring                                                     | :code:`https://adgangsstyring.eksterntest-stoettesystemerne.dk/runtime/saml2/issue.idp` |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| KOMBIT_CERTIFICATEPRIVATEKEY  | The certificate  private key for KOMBIT adgangsstyring                                               | :code:`null`                                                                            |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| KOMBIT_ROLE_NAME              | This string must be a substring of the brugersystemrolle you grant users for them to be given access | :code:`http://os2iot.dk/roles/usersystemrole/adgang/`                                   |
+-------------------------------+------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+

OS2IoT-frontend
^^^^^^^^^^^^^^^

The frontend can be configured by modifying :code:`environment.prod.ts` file in :code:`OS2IoT-frontend/src/environments` folder.

 .. code-block:: javascript

   export const environment = {
      production: true,
      baseUrl: 'http://localhost:3000/api/v1/',
      tablePageSize: 20,
   };

+-------------------------------+--------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| Configuration variable        | Purpose                                                                                                      | Default value                                                                           |
+===============================+==============================================================================================================+=========================================================================================+
| production                    | If true, then Angular is set in production mode, disabling debugging features                                | :code:`true`                                                                            |
+-------------------------------+--------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| baseUrl                       | The Url which users will connect to the backend from. This must be changed for the system to work externally | :code:`http://localhost:3000/api/v1/`                                                   |
+-------------------------------+--------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
| tablePageSize                 | Default page size of tables                                                                                  | :code:`20`                                                                              |
+-------------------------------+--------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------+
