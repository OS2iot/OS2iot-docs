Maintenance Guide
=========================

Purpose
-------

The purpose of this document is to enable the maintainers of this project to develop changes for the
solution.

Development
-----------

Configuration of developer machine
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following should be installed in order to develop on OS2iot. It is assumed a Windows laptop is used.

1. Docker Desktop

2. Visual Studio Code with the following extensions:

   a. ESLint

   b. Npm

   c. Jest

   d. Prettier

3. Pgadmin

4. Git/Git Extensions/Sourcetree/Sublime Merge

Mac:
In order to run os2iot-backend outside docker and connect to docker (run it via vs code) one must follow the steps below:

* Add docker to hosts on mac

* Run: sudo vim /etc/hosts

* In vim type i to insert

* Add line with ip and hosts.docker.internal e.g. 127.0.0.1 hosts.docker.internal

* type: esc :wq to save and exit

* start os2iot-backend in vs code via the terminal: npm run start


Source Code
~~~~~~~~~~~

Github is used to store the source code for the OS2iot project. It uses
the following repositories:

-  OS2IoT-frontend: https://github.com/OS2iot/OS2IoT-frontend

-  OS2IoT-backend: https://github.com/OS2iot/OS2IoT-backend

-  OS2IoT-docker: https://github.com/OS2iot/OS2IoT-docker


Requirements
~~~~~~~~~~~~

-  **Docker** – sign up for an account and download docker at
   https://docs.docker.com/get-docker/

-  **NodeJS** – download the latest version at https://nodejs.org/en/

-  **PGAdmin** – download at https://www.pgadmin.org/download/

Setup developement
~~~~~~~~~~~~~~~~~~

To setup your dev environment please follow these instructions:

1. Get the solution from Github. Clone the OS2IoT-docker, OS2IoT-backend, and OS2IoT-frontend.
2. Install typescript and angular globally on your computer.
   a. Angular 9 – install by open terminal and type :code:`npm install -g @angular/cli`
   b. Typescript – install by open terminal and type :code:`npm install -g typescript`
3. For OS2IoT-frontend install dependencies and start
   a. Navigate terminal to the OS2IoT-frontend folder and type :code:`npm install`
4. For OS2IoT-backend install dependencies and start
   a. Navigate terminal to the OS2IoT-backend folder and type :code:`npm install`

Starting dependencies for development
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. (windows) Add os2iot-docker to file share in docker 

   a. Open Docker Desktop -> Settings -> Resources -> FILE SHARING -> add os2iot-docker as shared folder.

2. Open terminal and go to path of OS2IoT-docker and write

   .. code-block:: bash

      docker-compose up chirpstack-network-server postgresql chirpstack-gateway-bridge chirpstack-application-server os2iot-outbound-mosquitto mosquitto redis os2iot-inbound-mosquitto os2iot-kafka os2iot-postgresql os2iot-zookeeper

3. To quit: In your terminal you can press Ctrl + C twice. This will safely shut down docker.

Troubleshooting
^^^^^^^^^^^^^^^

For troubleshooting help see the `installation guide <../installation-guide/installation-guide.html#troubleshooting>`_.

Starting the frontend and backend for development
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Open the OS2IoT-frontend in visual studio code and start angular using: :code:`npm start`
2. Open the OS2IoT-backend in another instance of visual studio code and start it using :code:`npm start`

Database
^^^^^^^^

To access the database, using PGAdmin is recommended.
The default credential can be seen in `Users Notes <../users-notes/users-notes.html>`_.

Map
~~~
OS2IoT maps are running on the Leafletjs framework: https://leafletjs.com/. The tiles are current presented using OpenStreetMap: https://www.openstreetmap.org.

The tiles can be changed by following the steps listed below: 

1. Find "map.component.ts" in the frontend project

2. Within the initMap() method, replace L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png') with the desired tiles

   a. Make sure you also change the attribution attribute.

Note that the solution must be deployed before the changes takes presence.

Debugging 
^^^^^^^^^^

Debugging the frontend
''''''''''''''''''''''

Use the `debugger for chrome <https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome>`_ plugin to enable debugging in Chrome, follow the upto-date instruction in their readme.

Debugging the backend
'''''''''''''''''''''

Use the built in debugging, after launching using the `launch.json` configuration.

Branching strategy
~~~~~~~~~~~~~~~~~~

OS2IoT uses git and GitFlow
https://datasift.github.io/gitflow/IntroducingGitFlow.html for source
code version control.

Naming Convention of branches
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Naming feature branches follows standard: **feature/branch-name**, i.e. :code:`feature/IOT-1337`.

Naming the hotfix branches follows standard: **hotfix/branch-name**, i.e. :code:`hotfix/IOT-1337`.

Format of commit message
^^^^^^^^^^^^^^^^^^^^^^^^

A commit must follow the format: [Story ID]: Message. For example: "IOT-1337: Update CreateUserDto to validate birthdays"

By starting commit messages with [Story ID], traceability is obtained
from the code and to the case.

Tools
-----

Chirpstack
~~~~~~~~~~

Chirpstack is bundled as part of OS2IoT-docker.

The Chirpstack API documentation will be exposed at http://localhost:8080/api . 
More information can be found at https://www.chirpstack.io/application-server/integrate/rest/

For installation configuration of Chirpstack see: https://www.chirpstack.io/application-server/install/config/

Migrations
-----------

This project is using TypeORM migrations when changes are applied to the database. This is recommended by TypeORM when the project is used for production.

Generate migrations
~~~~~~~~~~~~~~~~~~~

If you modify or adds an entity (or more than one entity) and/or relationships you then need to generate a migration. In the console you write: :code:`npm run generate-migration <your name of the migration>`. Then a migration file 
will be created in a migration folder. The folder path is specified in :code:`ormconfig.json`. A timestamp will be added to the name to indicate when the migration has been generated. If no changes have been made to the entity classes, no migrations are generated.

Run migrations
~~~~~~~~~~~~~~~~
When the project is starting, a new command will be called automatically. This happens every time you run the program because of a prestart script.
The command is :code:`run-migrations` which runs all the pending generated migrations in the Migrations folder, starting from the oldest migration. 
In the pending migrations, the :code:`up` block will be executed.
If there are no pending migrations then no migrations will be runned.

It will happen in both debug and prod mode.

Revert migration
~~~~~~~~~~~~~~~~~~
If you later on wish to revert a migration you can write :code:`npm run typeorm migration:revert`. Then the latest runned migration will be reverted. What happens is that the :code:`down` block in the latest runned migration will be executed. 
You can continue to do this until you reach the desired migration.
The generated migrations will not be deleted when you are reverting so when you run the project again, the migrations will be runned with the :code:`up` block unless you manually deletes them.

Show migration
~~~~~~~~~~~~~~~~
If you are in doubt which migrations has been run, then you have the possibility to write :code:`npm run typeorm migration:show` in the console. Then the migrations will be shown in the console,
and if [X] is marked at a migration it means that it has been run. Otherwise it will be an empty [] which means that is has NOT been run. 

Maintaining the docs
--------------------

To update the documentation, i.e. these pages you are reading now, you must edit the OS2IoT-docs Git repository: https://github.com/OS2iot/Os2iot-docs 

The documentation is written in reStructuredText, see https://docutils.sourceforge.io/rst.html for an intro.

Building locally
~~~~~~~~~~~~~~~~

1. Make sure that you have Python installed, and that it is available in PATH
2. Run: :code:`pip install -r requirements.txt`
3. Run: :code:`make html` to generate the docs once or :code:`sphinx-autobuild source build` to rebuild contentiously.
   a. The generated documentation can be found in /build/html/index.html or on the link shown in the CLI if using sphinx-autobuild
4. For spelling check you can install "Code Spell Checker" assuming that you run Visual Code.


ReadTheDocs will automatically pull changes pushed to the :code:`master` branch and build it.

.. |search for Docker app| image:: ./media/image5.png
.. |whale on taskbar| image:: ./media/image6.png
.. |Docker Quick Start tutorial| image:: ./media/image7.png
.. |Download Visual Studio-kode| image:: ./media/image8.png
.. |Opsætningsguide| image:: ./media/image9.png
.. |IDE| image:: ./media/image10.png
.. |Sti til krydsfiler| image:: ./media/image11.png
.. |Download og kør installationsprogram| image:: ./media/image12.png
.. |Verify Installation| image:: ./media/image13.png

.. |image1| image:: ./media/image5.png
.. |image2| image:: ./media/image6.png
.. |image3| image:: ./media/image7.png
.. |image4| image:: ./media/image8.png
.. |image5| image:: ./media/image9.png
.. |image6| image:: ./media/image10.png
.. |image7| image:: ./media/image11.png
.. |image8| image:: ./media/image12.png
.. |image9| image:: ./media/image13.png
.. |image10| image:: ./media/image14.png
.. |Debugging diagram| image:: ./media/image15.png
.. |Simplified initial Run and Debug view| image:: ./media/image16.png
.. |image13| image:: ./media/image17.png
.. |A successful Git branching model » nvie.com| image:: ./media/image18.png
