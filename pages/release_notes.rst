
#############
Release notes
#############

Whats has been changed, and how to update between versions.

*******
v0.11.0
*******

See `github project <https://github.com/orgs/nuts-foundation/projects/5>`_ for more details

=======================
Features / improvements
=======================

* A version number has been added to the FHIR consent record (:ref:`nuts-fhir-validation-requirements`) which is also visible in the consent-store.
Currently, the API's will only return the latest version. The version is mainly for forwards compatibility and for viewing changes in consent in future releases.
* The consent-store query API has been changed to return a `PatientConsent` model instead of a `SimplifiedConsent` model , ref: :ref:`nuts-consent-store-api`.
* Changed consent on the level of individual FHIR resources (Patient, Observation, etc) to data classes (Medical, Social, Mental) across all modules. Mapping individual FHIR resources to and from classes is future work.
* Public keys in registry can now be stored in JWK format. All api's that request or return public keys can handle JWK format.
* Period dates in the consent store have been changed to datetime objects instead of dates.
This is mainly done for when consent is withdrawn, it should not be active for the rest of the day.
* Corda has been updated to 4.3.

========
Bugfixes
========

* Fix incorrect return values for hash and ID in the consent-store api
* Fix usage of validAt query param on consent-store query api
* Fix period adherence in login contract creation
* Fix technical error when validating login contract

*******
v0.10.0
*******

See `github project <https://github.com/orgs/nuts-foundation/projects/4>`_ for more details

=======================
Features / improvements
=======================

* Signed JWTs with private key of requestor. This allows the custodian to check if
  JWT has been created by the requestor instead of being reused from another party.
* Add strictmode flag which forbids unsafe config options.
* Add IRMA schememanager config flag which allows setting demo or production attributes
* Recover events on startup
* Purge completed events at startup
* Add retry queues for failed events by a temporary cause
* Make nats subscription durable
* Updates all the modules to go 1.13, allowing for the new encapsulating errors
* Compare public keys by object instead of by string

========
Bugfixes
========

* Fix 500 on createConsent API call when body is incomplete / empty
* Fix nullpointer error on incorrect legalName in cordapp
