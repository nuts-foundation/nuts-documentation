
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

* A version number has been added to the FHIR consent record which is also visible in the consent-store
* The consent-store query response has been changed from a simplified model to the complete patient consent model
* Changed consent on the level of individual resources to data classes across all modules
* Public keys in registry can now be stored in JWK format. All api's that request or return public keys can handle JWK format
* Period dates in the consent store have been changed to datetime objects instead of dates
* Corda has been updated to 4.3 which allows using a Java 11 compatible JVM

========
Bugfixes
========

* Fix incorrect return values for hash and ID in the consent-store api
* Fix usage of validAt query param on consent-store query api
* Fix period in login contract creation
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
