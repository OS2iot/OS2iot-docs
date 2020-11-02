Maintenance Guide
=========================

Introduction
------------

Purpose
~~~~~~~

The purpose of this document is to enable the application management
team to service the solution in production and develop changes for the
solution.

The target audience for the deliverable is either Netcompany AMC or the
customers application management team that supports the solution in
production.

Development
-----------

Requirements
~~~~~~~~~~~~

-  **Docker** – sign up for an account and download docker at
   https://docs.docker.com/get-docker/

-  **NodeJS** – download the latest version at https://nodejs.org/en/

-  **PGAdmin** – download at https://www.pgadmin.org/download/

For futher instructions for installation of standard software go to
`O0100 – installation
guide <https://goto.netcompany.com/cases/GTE720/ERHIO2/Deliverables/Migreret%20til%20Git%20(DONT%20MODIFY!)/D0100%20-%20User-Interface%20Guidelines.docx?web=1>`__

Setup developement
~~~~~~~~~~~~~~~~~~

To setup your dev environment please follow these instructions:

**Step 1** - Get solution from Github. See 4.14.1 Source control and
pull the OS2IoT-docker, OS2IoT-backend, and OS2IoT-frontend.

**Step 2 -** Install typescript and angular globally on your computer .

-  **Angular 9** – install by open command prompt and type

npm install -g @angular/cli

-  Typescript – install by open command prompt and type

npm install -g typescript

Quick 
^^^^^^

Important! Prepare by setting up the repositories, OS2IoT-docker,
OS2IoT-backend, and OS2IoT-frontend, in the same folder/path of your
choosing

Open command prompt and go to path of OS2IoT-docker and write

Docker-compose up

Hereafter the docker will install the backend and frontend and you can
see the UI result on http://localhost:4200/

To quite, you can press Ctrl + C two times and docker will shut down
safely.

Advanced 
^^^^^^^^^

Important! Prepare by setting up the repositories, OS2IoT-docker,
OS2IoT-backend, and OS2IoT-frontend, in the same folder/path of your
choosing

1 Open command prompt and go to path of OS2IoT-docker and write

docker-compose up chirpstack-network-server postgresql
chirpstack-gateway-bridge chirpstack-geolocation-server
chirpstack-application-server os2iot-outbound-mosquitto mosquitto redis
os2iot-inbound-mosquitto os2iot-kafka os2iot-postgresql os2iot-zookeeper
os2iot-backend

|image1|

Docker will install all dependencies to the backend, and you can go to
`http://[::1]:3000/api/v1/docs/ <http://[::1]:3000/api/v1/docs/>`__ and
the result should look like the below picture.

|image2|

2 Open command prompt and go to path/folder of OS2IoT-frontend and write

npm install

|image3|

It will then install all the npm dependencies to the solution.

1 Open command prompt and go to path of OS2IoT-docker and write

Npm start

|image4|

Database
^^^^^^^^

To see the data you have to install PGAdmin as described in 2.1
Requirements.

Open PGAdmin

|image5|

Add a new server

|image6|

Name the server

|image7|

Fill in the connection tab with the following information

|image8|

You can checkout the information(password, port, username) to the server
setup in C:\repos\OS2IoT\OS2IoT-docker\docker-compose.yml. and scroll
down to os2iot-postgresql

|image9|

Configuration of developer laptop
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following must be installed in order to develop on OS2iot. It is
assumed a Windows laptop is used.

1. Docker Desktop

2. Visual Studio Code with the following extensions:

   a. ESLint

   b. Npm

   c. Jest

   d. Prettier

3. Pgadmin

4. Git/Git Extensions/Sourcetree/Sublime Merge

Map
~~~
OS2IoT maps are running on the Leafletjs framework: https://leafletjs.com/. The tiles are current presented using OpenStreetMap: https://www.openstreetmap.org.

The tiles can be changed by following the steps listed below: 

1. Find "map.component.ts" in the frontend project

2. Within the initMap() method, replace L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png') with the desired tiles

   a. Make sure you also change the attribution attribute.

Note that the solution must be deployed before the changes takes presence.

Development procedures
~~~~~~~~~~~~~~~~~~~~~~

Service tier
^^^^^^^^^^^^

Front tier
^^^^^^^^^^

.. _database-1:

Database
^^^^^^^^

The database is created code first using TypeORM.

Database changes are done using the TypeORM migrations.

Batch jobs
^^^^^^^^^^

Debugging 
^^^^^^^^^^

Debug VSCode 
'''''''''''''

One of the key features of Visual Studio Code is its debugging support.
VS Code's built-in debugger helps accelerate edit, compile and debug
loop. The solution is setup to debug on a firefox browser and therefore
you have to install the **Debugger for Firefox** extension. Go to
extension and search for **Debugger for Firefox and install it.**

|image10|

Afterwards you can start debugging the code by adding a breakpoint
somewhere.

|Debugging diagram|

If running and debugging is not yet configured (no launch.json has been
created) VSCode show the Run start view.

|Simplified initial Run and Debug view|

To run or debug a simple app in VS Code, press F5 and VS Code will try
to run your currently active file.

However, for most debugging scenarios, creating a launch configuration
file is beneficial because it allows you to configure and save debugging
setup details. VS Code keeps debugging configuration information in
a launch.json file located in a .vscode folder in your workspace
(project root folder) or in your \ `user
settings <https://code.visualstudio.com/docs/editor/debugging#_global-launch-configuration>`__ or `workspace
settings <https://code.visualstudio.com/docs/editor/multi-root-workspaces#_workspace-launch-configurations>`__.

To create a launch.json file, open your project folder in VS Code
(File > Open Folder) and then select the Configure gear icon on the Run
view top bar.

Debug VSCode with Chrome
''''''''''''''''''''''''

If you want to use Chrome as the default browser for debugging you have
to install **debugger for chrome** in the extension menu. Afterwards got
to launch.json in the .vscode folder and add the following configuration

|image13|

Management of common data
~~~~~~~~~~~~~~~~~~~~~~~~~

Building the solution
~~~~~~~~~~~~~~~~~~~~~

Continuous integration
~~~~~~~~~~~~~~~~~~~~~~

Configuration of the solution
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configuration of security
~~~~~~~~~~~~~~~~~~~~~~~~~

Branching strategy
~~~~~~~~~~~~~~~~~~

OS2IoT uses git and GitFlow
https://datasift.github.io/gitflow/IntroducingGitFlow.html for source
code version control.

|A successful Git branching model » nvie.com|

GitFlow involves the following branches:

-  "master" - The main industry with the current code in production.

-  “develop” - The main development industry. Created from the master
   industry and merges back into master cutting which often only through
   and frees branch. Contains latest development work, but changes can
   not push directly to this branch - we use Instead pull requests
   through Github. If development from a feature branch is not to be
   included in the next release, this should not be merged to develop,
   men instead wait for the correct release branch to be set up.

-  "function" branches - contains code for individual new functions.
   Created from develop block and merge to develop branch via a pull
   request when the new feature is complete.

   -  CRM depot prefixes, industries are with OS2, eg OS2feature /
      somenewfeature

-  “hotfix” branches - contains quick changes to master / release
   branch. Will be merged for master / release via a pull request. After
   a hotfix is ​​merged, downstream branches need to be updated, in most
   cases developed.

-  “release” branches - new releases candidates and is used to deploy a
   version to the test, pre-production and production environments. Each
   gang and liberating branch is created, "required" policies must be
   configured for it, such as:

   -  Reviewers

   -  Low validation

Naming Convention of branches
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Naming feature branches follows standard: **feature / Branch name.**

Naming the publishing branches follows standard: **release / Branch
name**

Naming the hotfix branches follows standard: **hotfix / Branch name**

Format of commit message
^^^^^^^^^^^^^^^^^^^^^^^^

A commit must follow the format: [Story ID] Message

By starting commit messages with [Story ID], traceability is obtained
from the code and to the case.

Configuration of deployment
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Deploy to DEV
~~~~~~~~~~~~~

Deploy to TEST
~~~~~~~~~~~~~~

Tools
-----

Source code control
~~~~~~~~~~~~~~~~~~~

Github is used to store the source code for the OS2iot project. It uses
the following repositories:

-  OS2IoT-frontend: https://github.com/OS2iot/OS2IoT-backend

-  OS2IoT-backend: https://github.com/OS2iot/OS2IoT-frontend

-  OS2IoT-docker: https://github.com/OS2iot/OS2IoT-docker

Jira
~~~~

Jenkins
~~~~~~~

Jenkins is used for CI and CD. It can be accessed here:

https://jenkins.os2iot.dk/

Enterprise Architect
~~~~~~~~~~~~~~~~~~~~

The project uses a database hosted in Azure for storing the Enterprise
Architect model. Perform the following steps to establish connection:

1. Open Enterprise Architect

2. Open Server connection

3. Choose "Microsoft OLE DB Provider for SQL Server"

   a. Server name: os2iot-ea.database.windows.net

   b. User name: ea-admin

   c. Password: Found in KeePass

   d. Database: OS2iot

.. |image0| image:: ./media/image4.emf
   :width: 1.51111in
   :height: 0.23194in
.. |image1| image:: ./media/image5.png
   :width: 6.56806in
   :height: 0.51319in
.. |image2| image:: ./media/image6.png
   :width: 5.16806in
   :height: 2.70861in
.. |image3| image:: ./media/image7.png
   :width: 6.56806in
   :height: 0.48542in
.. |image4| image:: ./media/image8.png
   :width: 6.56806in
   :height: 0.6in
.. |image5| image:: ./media/image9.png
   :width: 6.56528in
   :height: 3.55625in
.. |image6| image:: ./media/image10.png
   :width: 6.56806in
   :height: 3.55764in
.. |image7| image:: ./media/image11.png
   :width: 6.56806in
   :height: 3.58056in
.. |image8| image:: ./media/image12.png
   :width: 6.56806in
   :height: 3.57569in
.. |image9| image:: ./media/image13.png
   :width: 3.30254in
   :height: 3.08376in
.. |image10| image:: ./media/image14.png
   :width: 3.9851in
   :height: 1.31123in
.. |Debugging diagram| image:: ./media/image15.png
   :width: 3.74783in
   :height: 2.10335in
.. |Simplified initial Run and Debug view| image:: ./media/image16.png
   :width: 4in
   :height: 2.68681in
.. |image13| image:: ./media/image17.png
   :width: 3.97411in
   :height: 2.26479in
.. |A successful Git branching model » nvie.com| image:: ./media/image18.png
   :width: 3.8087in
   :height: 5.04698in
