#######################
Login Contracts
#######################

In the Nuts network, the identity of a user is universally valid. This means the whole network trusts authorities like the BIG, CIBG and BRP instead of the sysadmin of a random care organisation.

When a user wants to access data stored in a different EHR system than they are currently using, they must give their own system consent to request data.

So, we have several parties:
* A user, probably a care professional.
* A trusted registry which administres traits about the user such as an identity or medical qualification
* The users software own software system
* Another system containing data the user wants to access.

The consent should:
 * be human readable
 * be machine readable
 * be final in time
 * only be usable by the users software provider
 * temper proof
 * versionable
 * support multiple languages
 * should fit in the http AUTHENTICATION header
 * contain the following attributes
    * The identity of the user
    * The software system given consent to
    * The scope of the consent
    * The purpose of the consent
    * Period when the consent is valid
    
Example
=======

Login v1:NL Ondergetekende geeft hierbij toestemming aan *SoftwareApplicatie* om uit naam van ondergetekende *data op te vragen* ten behoeven van *het verlenen van goede zorg*. Deze toestemming is geldig tot *2019-02-06T21:00*.

Using regular expressions, a machine can extract all the relevant variables::

  /(version):(language) Ondergetekende geeft hierbij toestemming aan (Acting Party) om uit naam van ondergetekende (Purpose) ten behoeven van (Goal). Deze toestemming is geldig (period)./


You can try this simple example over here: https://regex101.com/r/sHfsKq/1

Matching groups
===============

Version
-------
We should be able to update the contract. Also we should open up options for other contracts. These versions should not confuse the user to much.

Language
--------
We can use human readable `Language tags <https://www.w3.org/International/articles/language-tags/>`_.

Acting Party
------------
We can use the X.509 CommonName used in the Directory service for software names.

Period
------
We should agree on a human and machine readable time format like ISO 8601: https://xkcd.com/1179/

Purpose
-------
We can encode our *Purpose* field based on the FHIR PurposeOfUse: https://www.hl7.org/fhir/v3/PurposeOfUse/vs.html

Scope
-----
We can encode our *scope* field based on the FHIR consent scope set: https://www.hl7.org/fhir/valueset-consent-scope.html

IRMA Contracts
==============
We can use IRMA to sign the contract using appropriate attributes. Every IRMA server with the `Dutch IRMA key chain <https://github.com/privacybydesign/pbdf-schememanager>`_ should be able to validate the contract.


.. figure:: /_static/images/irma-consent-login.png
    :width: 600px
    :align: center
    :alt: Irma consent login
    :figclass: align-center
