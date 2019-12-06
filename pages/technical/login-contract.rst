.. _nuts-documentation-login-contracts:

#######################
Login Contracts
#######################

In the Nuts network, the identity of a user is universally valid.
This means the whole network trusts authorities like the BIG, CIBG and BRP instead of the unknown sysadmin and processes of a random care organisation.

When a user wants to access data stored in a different EHR system than they are currently using, they must give their own system consent to request data.

So, we have several parties:
 * A user, probably a care professional or patient (User)
 * A trusted registry which administers and issues attributes about the user such as an identity or medical qualification (Issuer)
 * The users own software system (Acting Party)
 * Another system containing data the user wants to access. (Verifier)

The consent must:
 * be human readable
 * be machine readable
 * be final in time
 * only be usable by the users software provider
 * temper proof
 * versionable
 * signable with attributes
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


Using regular expressions, a machine can extract all the relevant variables::

  /(language):(Contract-name):(version) Ondergetekende geeft hierbij toestemming aan
  (Acting Party) om uit naam van ondergetekende (Scope) ten behoeven van (Purpose).
  Deze toestemming is geldig (period)./

Examples:

NL:Login:v1 Ondergetekende geeft hierbij toestemming aan *SoftwareApplicatie* om uit naam van ondergetekende *data op te vragen, data aan te passen* ten behoeven van *het verlenen van goede zorg*. Deze toestemming is geldig tot *2019-02-06T21:00*.

NL:Gegevensverwerking:v3 Ondergetekende geeft hierbij toestemming aan *Medians* om uit naam van ondergetekende *data op te vragen, data te verzamelen* ten behoeven van *het doen van medisch onderzoek*. Deze toestemming is geldig tot *2020-02-06T21:00*.

NL:Inzage:v2 Ondergetekende geeft hierbij toestemming aan *Alle cardiologen* om *data op te vragen* ten behoeven van het *verlenen van goede zorg*. Deze toestemming is geldig van 2018-01-01T00:00 tot 2030-12-31T23:59

You can try this simple example over here: https://regex101.com/r/sHfsKq/1


Matching groups
===============

Language
--------
We can use human readable `Language tags <https://www.w3.org/International/articles/language-tags/>`_.

Contract-name
-------------
We can have multiple contracts like login, giving consent for treatment, authorization etc.
The contract name is localized.

Version
-------
We should be able to update contracts as they might change over time.
Since the version indicator will be displayed in the contract, it is important to choose a form which does distract or confuse the user.
Think of a scheme like v1, v2 etc.


Acting Party
------------
Purpose: Only the acting party is given the permission.

In case of a Nuts party, We can use the X.509 CommonName used in the Directory service for software names.
In case of people, we can use relevant attributes such as a name, medical expertise or medical registration number (BIG, AGB).

Period
------
Purpose: limiting the time frame a contract is valid.

We should agree on a human and machine readable time format like ISO 8601: https://xkcd.com/1179/

Consent scope
-------------
Purpose: limit the possible operations the acting party can perform on the data.

Actions like: collect, access, use, disclose or correct: https://www.hl7.org/fhir/valueset-consent-action.html

Purpose
-------
What will be the reason for performing the operations on the information?
We can encode our *Purpose* field based on the FHIR consent scope set: https://www.hl7.org/fhir/v3/PurposeOfUse/vs.html

Signing with attributes
=======================

Nuts is made for many different user groups: GPs, patients, guardians, informal care givers etc.
All these people have different relevant properties (attributes).
Therefor we need an universal authentication system which can handle many different
attributes.
https://privacypatterns.org/patterns/attribute-based-credentials

Attribute Based Credentials (ABC)
---------------------------------

In order to sign a contract we need an application which support these attributes.
Currently in the Netherlands there is one system which gains in popularity: IRMA (I reveal my attributes) from the privacy by design foundation: https://privacybydesign.foundation/.


Using ABC decouples the issuer from the verifier.
This improves the privacy of the user, eliminates the need for 1-to-1 API implementations and the need for complicated legal documents between the parties since the users decides what they will do with their own data.

.. Note::
  It is important to note that the workings of Nuts are not limited to IRMA.


Every node in the Nuts network sets up its own trusted IRMA server with the
`Dutch IRMA key chain <https://github.com/privacybydesign/pbdf-schememanager>`_
so it can independently validate claims and contracts.


.. figure:: /_static/images/irma-consent-login.png
    :width: 600px
    :align: center
    :alt: Irma consent login
    :figclass: align-center
