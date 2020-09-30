
#############
Release notes
#############

Whats has been changed, and how to update between versions.

*******
v0.15.0
*******

This release adds the p2p registry. Github is still used as backup registry but registry updates should now be instant!

Project board: https://github.com/orgs/nuts-foundation/projects/9
Required migrations: :ref:`migration`

=======================
Features / improvements
=======================

- Add Nuts network, a p2p network using grpc. (https://github.com/nuts-foundation/nuts-registry/pulls/119)
- Process events received through Nuts network. (https://github.com/nuts-foundation/nuts-registry/issues/141)
- GenerateKeyPair should only overwrite existing key pair if specified. (https://github.com/nuts-foundation/nuts-crypto/issues/83)
- Added support for Prometheus format diagnostics.
- Added client interface for remote crypto administration. (https://github.com/nuts-foundation/nuts-crypto/issues/18)
- Add baseline truststore. (https://github.com/nuts-foundation/nuts-crypto/issues/61)
- Add certificate expiration monitoring. (https://github.com/nuts-foundation/nuts-crypto/issues/65)

========
Bugfixes
========

- Vendor ID was parsed incorrectly from <0.15 certificates. (https://github.com/nuts-foundation/nuts-registry/issues/142)
- Irma scheme didn't update for validator. (https://github.com/nuts-foundation/nuts-auth/pulls/92)

*******
v0.14.0
*******

This release added additional information to the Corda states. This allows vendor software to diagnose any problems and have a better understanding what the current state of a consent request is.

For specific issues also see: https://github.com/orgs/nuts-foundation/projects/12

======================
Upgrading from v0.13.0
======================

Starting version 0.14 vendors and organizations will have an X.509 certificate (encoded in the JWK) associated with their key pairs. These certificates are used to identify the holder of the key pair used to sign events, which is also introduced in this version.

Action required: This migration can be performed by the Nuts node; use the newly introduced `registry verify` command with the `–fix` flag to generate key pairs (if necessary), issue certificates and sign events. **Don’t forget to publish these changes to the central registry.**
See :ref:`5. Verifying and fixing registry data` for the command.

See :ref:`migration` for a complete overview of all migrations.

=======================
Features / improvements
=======================

- Support update events in registry. (https://github.com/nuts-foundation/nuts-registry/issues/41)
- Remove vendor Id from vendor CLI commands. (https://github.com/nuts-foundation/nuts-registry/issues/47)
- Auto generate vendor and organisation certificates. (https://github.com/nuts-foundation/nuts-registry/pull/94)
- Make HTTP client time-out configurable. (https://github.com/nuts-foundation/nuts-registry/issues/101)
- Add cancel flow for ConsentRequestState. (https://github.com/nuts-foundation/nuts-consent-cordapp/issues/16)
- Support cancellation event (https://github.com/nuts-foundation/nuts-consent-bridge/issues/7)
- Fixed readEvent() to not break Ref() calclations. (https://github.com/nuts-foundation/nuts-registry/issues/100)
- Register initiating node and legalEntity in branch. (https://github.com/nuts-foundation/nuts-consent-cordapp/issues/47)
- Allow administrators to reissue vendor and organization certificates (https://github.com/nuts-foundation/nuts-registry/issues/97)
- Remove legacy_auth_token (https://github.com/nuts-foundation/nuts-auth/issues/53)
- publish errors to normal channel to initiate cancellation. (https://github.com/nuts-foundation/nuts-consent-logic/issues/63)
- Add extra origin information for a consentRequest (https://github.com/nuts-foundation/nuts-consent-logic/pulls/62)
- Removed decodeURI, not needed for latest echo and updated all modules (https://github.com/nuts-foundation/nuts-go/pulls/43)
- Update event diagram with 'closed' event (https://github.com/nuts-foundation/nuts-event-octopus/pulls/35)

========
Bugfixes
========

All bugs were fixed in 0.13.x versions.

- Missing endpoint does not result in errored event or retry. (https://github.com/nuts-foundation/nuts-consent-bridge/issues/61)
- Already uploaded attachment gave error. (https://github.com/nuts-foundation/nuts-consent-bridge/issues/52)
- Incorrect starting point of Nats subscription. (https://github.com/nuts-foundation/nuts-consent-bridge/issues/56)
- Corda connection monitoring not working properly (https://github.com/nuts-foundation/nuts-consent-bridge/issues/54)
- Not enough memory allocated for Corda in Docker image. (https://github.com/nuts-foundation/nuts-consent-cordapp/issues/42)
- Endpoint deduplication in API removing too much endpoints. (https://github.com/nuts-foundation/nuts-registry/issues/90)
- Contract validation incomplete. (https://github.com/nuts-foundation/nuts-auth/issues/56)
- Retry event doesn't increment retryCount. (https://github.com/nuts-foundation/nuts-consent-bridge/issues/53)
- DemoEHR endpoints have the wrong port (https://github.com/nuts-foundation/nuts-network-local/issues/11)
- Fix sphinx python script to allow for semantic versioning (https://github.com/nuts-foundation/nuts-documentation/issues43)
- Diagnostics shows Nats DOWN state (https://github.com/nuts-foundation/nuts-consent-bridge/issues/63)
- Diagnostics shows Corda DOWN state (https://github.com/nuts-foundation/nuts-consent-bridge/issues/64)
- Records can't be recorded twice: dup record/unique constraint (https://github.com/nuts-foundation/nuts-consent-store/issues/55)
- Registering vendor doesn't survive restarts: (https://github.com/nuts-foundation/nuts-registry/issues/103)

*******
v0.13.0
*******

Focus of this relase was on robustness; automatic resumption of listeners/handlers, no more lost events or duplicate events. Developer Happiness by providing scripts to setup the nuts-local-network. Adding a Demo-EHR so nuts becomes clickable. Improving the registry by making it event based and adding signatures. Provide a convenient API authorization method by providing an easy to use OAuth 2 flow which accepts IRMA identity tokens.

For specific issues also see: https://github.com/orgs/nuts-foundation/projects/8

======================
Upgrading from v0.12.0
======================

The registry now also contains vendors and since a node must be linked to a vendor, the `nuts.yaml` must now contain an `identity`. Checkout :ref:`nuts-go-config` for the details.

See :ref:`migration` for a complete overview of all migrations.

=======================
Features / improvements
=======================

- Added chaos testing setup which runs tests while pausing different parts of the Nuts node. (https://github.com/nuts-foundation/nuts-chaos-testing)
- Updated Corda to 4.4 (https://github.com/nuts-foundation/nuts-consent-cordapp/issues/39)
- Querying endpoints now also returns the organisation ID for the endpoint. (https://github.com/nuts-foundation/nuts-registry/issues/64)
- Added OAuth2 flow using Irma contracts as zero-knowledge-proof tokens to get access token. (https://github.com/nuts-foundation/nuts-auth/issues/31)
- Better reliability of the bridge due to better connection handling. (https://github.com/nuts-foundation/nuts-consent-bridge/issues/46)
- Added docker container health check for nuts-cordapp image. (https://github.com/nuts-foundation/nuts-consent-cordapp/issues/38)
- Added docker container health check for nuts-bridge image. (https://github.com/nuts-foundation/nuts-consent-bridge/issues/47)
- Added docker container health check for nuts-service-space image. (https://github.com/nuts-foundation/nuts-go/issues/15)
- The starting point for Corda events is now persisted so the bridge will resume after restart. (https://github.com/nuts-foundation/nuts-consent-bridge/issues/21)
- Registry entries for vendor now have a signed certificate. (https://github.com/nuts-foundation/nuts-registry/issues/24)
- Registry entries for organisation now have a signed certificate. (https://github.com/nuts-foundation/nuts-registry/issues/26)
- Registry entries are now signed. (https://github.com/nuts-foundation/nuts-registry/issues/60)
- Registry entries are validated and errors are shown when the certificate hierarchy is incorrect. (https://github.com/nuts-foundation/nuts-registry/issues/25)
- Added CLI mode to the nuts executable.
- Added *registerVendor*, *registerOrganization* and *registerEndpoint* commands tro CLI mode. (https://github.com/nuts-foundation/nuts-registry/issues/30)
- Authorization server type endpoint has been added to the registry. (https://github.com/nuts-foundation/nuts-registry/issues/44)
- Consent bridge now also publishes to the retry and error queue when things go wrong. (https://github.com/nuts-foundation/nuts-consent-bridge/issues/41)
- Added identity parameter to nuts-go config. (https://github.com/nuts-foundation/nuts-registry/issues/43)
- Added persistence to discovery service, CSR's, signed certificates, the network config and node information now survive restarts.
- Prepared for change in endpoint type identifier for bridge. (https://github.com/nuts-foundation/nuts-consent-bridge/issues/39)

========
Bugfixes
========

- It wasn't possible to add a legal base for data exchange when the custodian and actor were serviced by the same node. (https://github.com/nuts-foundation/nuts-consent-logic/issues/30)
- Disabled Corda auto-reconnect. According to Corda it is experimental and indeed it did not function well. (https://github.com/nuts-foundation/nuts-consent-bridge/issues/38)

*******
v0.12.0
*******

See `github project <https://github.com/orgs/nuts-foundation/projects/7>`_ for more details

=======================
Features / improvements
=======================

- Added status endpoint for consent-bridge available under /status
- Added status endpoint for service executable available under /status
- Added diagnostics endpoint for consent-bridge available under /status/diagnostics giving information about the service health. Things like connection status, disk status etc.
- Added diagnostics endpoint for service executable available under /status/diagnostics giving information about the service health. Things like connection status, disk status etc.
- Added docs about service monitoring
- JWK's are now internally used for key representation
- Added Ping flow to Corda to check if nodes can contact each other. Available via diagnostics
- Corda contract now also checks if old consent records are re-offered
- When creating a session, the existence of the given legal entity is checked
- The registry files have changed from state-based to event-based.

========
Bugfixes
========

- The public key JWT check was broken (nuts-foundation/nuts-auth#29)
- The return value for the consent check was wrong (nuts-foundation/nuts-consent-store#30)
- Path variables in http service are now decoded correctly (nuts-foundation/nuts-go-core#7)
- Fix for consent query when no validTo was given (nuts-foundation/nuts-consent-store#31)

*******
v0.11.2
*******

See `github project <https://github.com/orgs/nuts-foundation/projects/11>`_ for more details

========
Bugfixes
========

* Consent conversion from and to the internal FHIR record was broken due to missing namespacing. (https://github.com/nuts-foundation/nuts-fhir-validation/issues/8)
  Additionally the dataClass format is also checked in the consent POST call. (https://github.com/nuts-foundation/nuts-consent-logic/issues/23)
* The validity period now uses DateTime values instead of LocalDates. This is needed to end a particular consent immediately. (https://github.com/nuts-foundation/nuts-consent-cordapp/issues/32)
* Searching and checking active consent could result in the wrong answer when a newer version ended consent. (https://github.com/nuts-foundation/nuts-consent-store/issues/24)
* ValidTo is now optional in a validity period. There was a mismatch between different parts of the system.
* Searching for consent with a validAt parameter used string comparison and not date comparison. (https://github.com/nuts-foundation/nuts-consent-store/issues/22)
* RFC3339 time notation is now used for all dateTime values. https://github.com/nuts-foundation/nuts-consent-store/issues/25)

======================
Upgrading from v0.11.0
======================

Because of the corrupted dataClasses, all data has to be wiped. Both the `persistence.mv` for Corda and the sqlite DB for the consent store have to be deleted.

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
* Changed consent on the level of individual FHIR resources (Patient, Observation, etc) to data classes (Medical, Social, Mental) across all modules.
  Mapping individual FHIR resources to and from classes is future work.
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
