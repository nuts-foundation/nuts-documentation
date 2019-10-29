
#############
Release notes
#############

Whats has been changed, and how to update between versions.

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
