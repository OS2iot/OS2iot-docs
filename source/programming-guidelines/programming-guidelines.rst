Programming Guidelines
======================

Introduction
------------

This document describes the programming guidelines and procedures for
the solution.

The guidelines and procedures will ensure a uniform code base, which is
easy to understand and maintain and thus is easy to transition to
another vendor should the customer wish to do so. Both the specific
standards for writing code and the more general procedures that apply to
the development process, are described in this document.

Code standards
--------------

Naming conventions
~~~~~~~~~~~~~~~~~~

We follow the same naming conventions as the TypeScript project itself:
https://github.com/Microsoft/TypeScript/wiki/Coding-guidelines#names.

Form and style
~~~~~~~~~~~~~~

-  Each logical component is in its own file.

-  Indentation is 4 spaces

-  Lines should be broken if they exceed 160 characters

-  Only have one declaration pr. Line

Structure of Source folder
~~~~~~~~~~~~~~~~~~~~~~~~~~

The project is separated into three git repositories to separate
concerns between the front-end, the back-end, and the container
orchestration using docker.

Front-end
^^^^^^^^^

Angular 9 is used for the frontend. The file structure described in the
Angular documentation will be used. This can be seen here:
https://angular.io/guide/file-structure

-  The app folder contains projects’ logic and data

-  A shared folder will contain shared components

-  Pages are divided into separate folders (modules)

-  Page elements are divided into components either placed in the
   module/folder of the page or in the shared folder

-  Services are placed at the logical level where they are used. Shared
   services are located higher up in the tree structure

Back-end
^^^^^^^^

| The structure is similar to the one described here:
| https://softwareontheroad.com/ideal-nodejs-project-structure/

Some of the important parts are:

-  Separate business logic from the controllers

-  Separate test code from production code

Docker
^^^^^^

In the root directory we place the docker-compose file. In the
configuration folder, there is another folder for each part of the
project, in which their respective configuration options.

This organization is modelled after the setup ChirpStack [1]_ uses.

Patterns and heuristics
~~~~~~~~~~~~~~~~~~~~~~~

-  Avoid using the “any” type, if at possible

-  Use functional constructions such as map and filter over imperative
   loops.

-  Functions should only be at one level of abstractions (split up to
   ease testing)

-  Avoid side-effects (make functions pure, such that they are easier to
   test)

-  Keep functions short and to the point, have them do one thing.

-  Try to avoid comments. They tend to get outdated and will end up
   causing more confusion the good. Consider rewriting the code and use
   the variables and functions to express the same sentiment.

-  Do not commit dead code, and do not commit commented out code.

Code analysis
~~~~~~~~~~~~~

We use ESLint to lint the code an ensure that we follow best practices.

SIG is used to weekly evaluate code-quality: https://www.softwareimprovementgroup.com/

Front-end – Programming standards
---------------------------------

The frontend code is written in TypeScript [2]_ (Angular 9). In the
project ESLint [3]_ is setup to lint the code for preventable and/or
syntax related errors. To keep code formatting consistent we use
Prettier [4]_ to format the code. We try to follow the do’s and don’ts
of TypeScript
(https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html).
For more examples of keeping the code clean see:
https://labs42io.github.io/clean-code-typescript/

Testing will primarily be done manually in the frontend. If automated
test is needed we will use Karma as is the standard in Angular. See:
https://angular.io/guide/testing

Back-end – Programming standards
--------------------------------

The backend code is written in TypeScript. In the project ESLint will be
setup to lint the code for common, machine fixable mistakes. To keep
code formatting consistent we use Prettier to format the code. We try to
follow the do’s and don’ts of TypeScript
(https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html).
For more examples of keeping the code clean see:
https://labs42io.github.io/clean-code-typescript/

To connect to the database we use TypeORM [5]_, and we use the “Data
Mapper” pattern for all our queries. We determine the datebase schema
code-first, and use TypeORM database migrations to change it.

For testing we use Jest. We attempt to write both
unit-test for each unit in isolation in addition to integration-tests to
ensure that the whole is functioning. We strive to do Test Driven
Development (TDD) and keep the code coverage as high as reasonably
possible.

To minimize differences between how the code runs on each pc, we use
docker for each part, and docker-compose to combine the containers.

.. [1]
   https://github.com/brocaar/chirpstack-docker

.. [2]
   https://www.typescriptlang.org/

.. [3]
   https://eslint.org/
   https://github.com/typescript-eslint/typescript-eslint

.. [4]
   https://prettier.io/

.. [5]
   https://typeorm.io/#/

.. |image0| image:: ./media/image3.emf
