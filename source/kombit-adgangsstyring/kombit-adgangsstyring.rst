KOMBIT adgangsstyring
====================================

OS2IoT supports authentification using "KOMBIT Adgangsstyring".
This is a OIOSAML based solution, this is implemented in OS2IoT using the `passport-saml library <https://github.com/node-saml/passport-saml>`__.

Configuration
-------------

In order to use KOMBIT adgangsstyring in your OS2IoT installation, it is required that you do some configuration.

Prerequisites:
^^^^^^^^^^^^^^

1. A service agreement with KOMBIT, and access to "serviceplatformen", or another way of configuring an "It-system".

2. A NemID FOCES or VOCES (FOCES is preferred) for production use (Issued by: TRUST2408 OCES CA). If the OS2IoT installation is a TEST system, and the test environment for KOMBIT adgangsstyring is being used, then a FOCES/VOCES for the NemID integration environment is sufficent (Issued by: TRUST2408 Systemtest).


Once the prerequisites are in order the configuration can begin.

Steps:
^^^^^^

1. Extract the private key and certificate from the FOCES/VOCES certificate.
    a. Using openssl:
        i. Certificate: :code:`openssl pkcs12 -in "my-foces.p12" -clcerts -nokeys -out os2iot-certificate.cer`
        ii. Private key: :code:`openssl pkcs12 -in "my-foces.p12" -nocerts -nodes -out os2iot-private-key.pem`

2. Make a local copy of the SAML EntityDescriptor Metadata file, located in the `resources folder of the OS2IoT-backend repository <https://raw.githubusercontent.com/OS2iot/OS2IoT-backend/master/resources/os2iot-kombit-adgangsstrying-metadata.xml.sample>`__.
    a. Edit the file with a suiteable text editor, such as: VSCode, SublimeText or notepad++.
    b. Replace all instances of :code:`{YOUR_BASE_URL}` with your actual base url, i.e. :code:`test.os2iot.dk`.
    c. Replace all instances of :code:`{YOUR_PUBLIC_KEY_AS_BASE64}` with the base64 encoded part of the certificate (without newlines), i.e. :code:`<X509Certificate>MIIGJDCCBQygAwIBAgIEXd/ZTjANBgkqhkiG9w0BAQsFADBAMQswCQYDVQQGEwJESzESMBAGA1UECgwJVFJVU1QyNDA...=</X509Certificate>`
    (Tip: `{YOUR_PUBLIC_KEY_AS_BASE64}` is the extracted certificate)
    

3. Create an IT-system in KOMBIT adgangsstrying:
    a. Stamdata:
        i. Name: A suitable name, i.e. "OS2IoT-Aarhus-Kommune"
        ii. E-mail: The e-mail of the operations responsible for hosting.
        iii. Type: "Anvendersystem" and "Brugervendt system"
    b. Anvendersystem:
        i. Upload the "os2iot-certificate.cer" file from earlier. (You might have to remove the bag attributes of the file, using a text editor)
    c. Brugervendt system:
        i. "Krævet assurance level": "Niveau 3"
        ii. "SAML metadatafiler": Upload the file created in step two.
        iii. Check that the info shown in the pop-up is correct for your certificate.
    d. Accept terms and save.

4. Create a pair of brugersystemroller og jobfunktionsroller.
    a. In IT-systemer choose the IT-system created above and go to the Brugervendt System tab. Under Brugersystemroller create a new one.
        i. Navn: os2iot-adgang-brugersystemrolle
        ii. Domæne: os2iot.dk
        iii. Rollenavn: adgang
        iv. Version: Choose a unique number (If its already in use you will get an error)
        v. This must yield the URI: :code:`http://os2iot.dk/roles/usersystemrole/adgang/`, if another one is wished then pass the environment variable :code:`KOMBIT_ROLE_NAME` to the backend (default: :code:`http://os2iot.dk/roles/usersystemrole/adgang/`).
        vi. Save the IT-system
        
        |image1|
        
    b. Create a Jobfunktionsrolle (or update existing)
        i. Navn and Rollenavn is decided by you.
        ii. Press "Tilknyt brugersystemrolle" and locate the IT-system and brugersystemrolle created above and save it.
        iii. Save the jobfunktionsrolle.
    c. Update your IdP to give the relevant users the jobfunktionsrolle created here.


5. Configure OS2IoT-backend to support KOMBIT adgangsstrying:
    a. You need to add two environment variables to the command running the backend server in your environment.
        i. This can be using the :code:`.env` file, or by setting them in some other manner.
        ii. The variable :code:`BACKEND_BASEURL` must be set to the URL a users browser will access the backend at, including the protocol, i.e. :code:`https://test.os2iot.dk`. The domain must be identical to the value you put into the metadata file above: `{YOUR_BASE_URL}`.
            a. An example for :code:`.env` could be: 
            
            .. code-block:: javascript

                BACKEND_BASEURL="https://test.os2iot.dk"

        iii. The variable :code:`KOMBIT_CERTIFICATEPRIVATEKEY` must be set to the private key from earlier. Since this file contains newlines, it might be easier to use `\\n` inside a string to set this variable.

            b. An example for :code:`.env` could be: 

            .. code-block:: javascript

                KOMBIT_CERTIFICATEPRIVATEKEY="-----BEGIN RSA PRIVATE KEY-----\nMIIEoQIBAFAKEAQEAlgq4JESby9DF7l73hViKZJ1/l9iIjCndQdjXNf0mOe9uMrWJ\nrDi0few9jFAKEIb0v33UmH20yFe7FiozjRBAgvml+lfZP2DN583evs6rGfHPNQHLb\nLP2g/2cehFAKE4asddasdsadsadX+hnYVJjnzOYmiPAAK418Tnq6g1tk4upPx9O\nlHgWWaDMwFAKEuKczbx/ALy9FxDk7x25Mpxqi3pUg35sMy76/JrdlEfuQzdjpaxp5\n4j29LqjPoFAKElpBJ6DjZotIcV9BL9rjNgZTb4N6jqHUqbyYOGfHAydFnJmeMYRMX\nViYkxag0WFAKEJ/P5YP9bCA3eYIbwJgyi6srT+wIDAQABAoIBAQCUmz1SvplIPxkr\nROgHLHC1wFAKEFoX3vSclpq1Rasdasda+7IJa9LF1v6z9VJWSCz9ZBnuIM\nngoiSY8EyFAKEj8X5LtLkb3CYlNZOQSvTX27xmqsxC2NRSTCt+wi3zpcqzqXXIZiX\n+asddasdsFAKEsadsdasdasdsadsdas+000hVqfokMxOyQ5ao0VECGXIokw+LSx\nlFhDvhRaJFAKEKhWL6nXiZC1QKGJdFsSZ+TdIemZoFaur8/C67Ih10AGH2wUnyoqy\naygdVg4WcFAKE5kDkAEYCpYtrXv0uqjGekSVeplYAaNdz1RXfklu7/k+PwJTv7mje\n15c5PABhAFAKEAPy80MzPKqm1SElhyRUbMx01yJEp6jouygHKLuoUlQu32ZntkjhY\nZjPE2+GmYFAKEXcocirmCpPf6MhbTJvvV4hDh4vmiuNjTWpudqK65UByFhzlnBuIZ\niNIZVHXyWFAKEwkd2fb0A59918LlERArVDYHXmTRVVjEyBgl8yTjIQSiDAoGBAJf6\nilrugZ8i1FAKEUgj606Ng5LBkW3ADgn9yz9PvpPXD3EiCEpSVKz7PxDa9xKjSrWqZ\n8EEYq6Z83FAKE7Gna/Ur97NfSPJDUtDbAw9m+9dDryNFqEbUrfxRAffAaq41xGjaY\nzp5t9wRsTFAKEkqCsDj+CChwSrCxc/TnffY4+AZ0pAoGAeftrz54hmj1LwVc35T72\ngZ+mySFw0FAKE14pM8F+0vC4lEV0PmLBZy5y0/4j8lTtPjTAPaI/8rU8Ng+SvyRan\nALz1fsUh8FAKES6dhdcstNkbgSD1InjHyzmy5TiAFYlGLxFAVSfa48yqKX/Dp4kyI\nM2XqpM6XRFAKEprCashQ9Fp8CgYB8uLBIVYlspnIk6P4cvnNmlcK3e3SKpWa33unt\nnLI8uoKRwFAKE6Iv33RvCbNPyVAra+//t/CgJ1lk8osayTHQn0eFHcJIhrV1Dvcg8\nbvlefdFtAFAKEzulYHmD75Xc5+UKw4ZBW9hmMuK/Jfz+lue1rcvWG+k/vjFSH5QAt\n9nbumQJ/RFAKEUqHciYXc+Q4lUSN3yvY5Ae6m1CvjmTg4Lzuc+0N7lnsp/FLDUg7P\nLbF7dgOw9FAKE40+sLhxAf8/b86LDVANUlfiN4JMUQYr6xZ1Ts1dCN9wRgZ4cbdU2\nT5XZL2YlXFAKEW5IcI8RKEYCHsNJIkPlh6LfNyrdAj56x2/w9Ew==\n-----END RSA PRIVATE KEY-----"

        iiii. The variable :code:`KOMBIT_ENTRYPOINT` must be set to the entrypoint of KOMBIT. If unset, it will be set to: :code:`https://adgangsstyring.eksterntest-stoettesystemerne.dk/runtime/saml2/issue.idp`, which is the test environment.
            c. An example for :code:`.env` could be:

            .. code-block:: javascript

                KOMBIT_ENTRYPOINT="https://adgangsstyring.eksterntest-stoettesystemerne.dk/runtime/saml2/issue.idp"


Test:
^^^^^

To test the functionality, press the "Login med KOMBIT Adgangsstyring" button from the login page (:code:`/auth`).
