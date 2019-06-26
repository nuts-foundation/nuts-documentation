.. _nuts-documentation-architecure-overview:

#######################
High level architecture
#######################

The image below is a first attempt to identify all the required components for making Nuts work. All interfaces between components will have a specification. Each vendor has the choice to implements their own components following the spec or just use the components provided by Nuts (mix-and-match).

.. raw:: html
    :file: ../../_static/images/high_level_architecture.svg

At first glance, the different components are divided into four different *spaces*.
This is done to distinguish between different levels of trust and (probably) different non-functional requirements like scaling.
*Vendor space* has the highest level of trust. It includes both personal and medical data.
Any components required to make Nuts work **must** be implemented by the vendor. *Service space* is also a trusted space since it'll include decrypted personal data, but not medical data.
Since it consists of quite a few components a vendor can buy this as part of a service package from a different vendor.
For example, a simple PGO might want get a subscription to a SaaS service which includes both *Service space* and *Nuts space* which only requires the PGO to implement the *Patient callback*. *Service space* also handles the administration of all the certificate logic for TLS connections.
*Nuts space* only contains encrypted personal data and decrypted personal data that has already been made public.
The primary goal of *Nuts space* is to make sure all required data for running the Nuts network is distributed across all nodes without any centrally controlled component.
This notion of different service levels allows smaller vendors to connect to Nuts and at the same time provide an opportunity for the current network providers to develop new business.
*Nuts foundation* is a separate entity which has the responsibility for adding the nodes. Essentially it controls the network.
Although we don't like centrally controlled components, a central authority or root is required. In this case the *Nuts foundation* will control the root certificate.
This is a direct result of the available technology, in the future other technology might become available or with enough Nuts participants, we can create something of our own.

Communication between the different components is done via REST services and ZMQ. In the future different protocols might be supported. Different security measures are/will be supported between the different components.

************
Vendor space
************

*Vendor space* contains all the components that **must** be implemented by each vendor with exception of the PGO UI. Most of the components will already be available since they are part of the EHR. Each of the components will be explained further in the following sections.

Public EHR API
==============

The *Public EHR API* is the primary point of data exchange for a care provider. It'll be based on international standards, for example: FHIR or CDA. This component is placed in vendor space because the underlying TLS connection will be secured with two-way SSL. The control over the certificates should lie with the entity responsible for the data: the care provider. Ultimately, it'll be the software vendor who will probably offer this responsibility as a service.

The *Public EHR API* will most likely consist of multiple components in an average production environment. The main responsibilities are:

* Authentication
* Consent authorisation
* Auditing
* Return data in requested format

Authentication
--------------

Authentication within Nuts will be provided by an `Irma signature <https://credentials.github.io/docs/irma.html#attribute-based-signatures>`_. The signature will have a signature text and attached attributes used to sign the text. Within the signature text, the requesting application name, the requesting care provider, the goal binding and a timestamp must be included. The attributes used to sign the text will identify the user. The requesting application name will also be in the CN used in the two-way TLS connection, this ensures that the care provider/vendor receiving the request can not use the same signature to request the Nuts network (man-in-the-middle). The timestamp ensures a short lived session. All the other information will be used to lookup the required consent record.

Consent authorisation
---------------------

The consent check is basically the authorization check for the request. Every consent record basically consists of a quadruple: (care provider, patient, care professional, scope), or 'May the care professional access the specific scoped data for the patient managed by the care provider?' The patient can be acquired by analysing the request, for example within FHIR the BSN will be included in the request, the care professional can be determined by the attributes used to generate the signature. The query to the consent registry within Nuts will then result in a list of care providers which can be used to query the correct data (or not found). Access to the specific resource is determined by the scope of the consent.

Data
----

If all previous steps have succeeded, the API component can then proceed to fetch and convert the data as requested. The initial versions will probably use FHIR as standard of choice.

Patients DB
===========

This *database* represents the patient's personal information and specifically the BSN record. All EHRs will contain this information since it's required by law. The data needs to be accessible by the *patient callback* component.

Patient callback
================

This component will mainly be used to check if a patient is receiving care for the given care provider. If, for example, this care provider receives consent to access certain patient records from another care provider it can only accept this consent if the patient is really a patient there. If not, the BSN may not be stored and the consent request can not be accepted. This check can also be used to detect faulty or corrupt Nuts nodes, since a lot of negative results from this component may indicate fraud. In a later version of Nuts this can be used for automatic blacklisting of Nuts nodes.

EHR UI
======

The EHR UI represents the piece of software the user interacts with. The part that is particular interesting is the consent UI. In the early stages of Nuts, the care providers will probably do all the work for gathering the patient consent. This means that the EHR needs to have a UI capable of recording this consent.

PGO UI
======

This component represents the UI needed for the PGO-inclusion flow. An idea exists where a patient is redirected by a PGO to this component to link their PGO identity to a BSN. The vendor can then use the Nuts network to update the consent records with the added PGO identity for all existing consent records for that patient. The UI needs to be in vendor or service space, otherwise the BSN can not be used. The difference between putting it in vendor or service space would be if it's embedded or not. Nuts will provide a reference implemention for placing it in the *service space*.

*************
Service space
*************

Consent store
=============

All consent within *Nuts space* is encrypted. The store will have a unencrypted copy of the records in memory to support querying from, for example, the *API*.
The attached *encrypted storage* will ensure that this sensitive data is encrypted-at-rest.

Consent Logic
=============

The logical component does most of the heavy lifting and depends on all the other components in *service space*.
For example, when creating a new consent request, this is send to the component it then checks if it's valid by using the validation component.
Next it has to find the correct organizations and encrypt the record with the right public keys.
Then it has to send the encrypted record to the Consent bridge for synchronization.

Also when a new consent event is received by the component from the consent bridge, it needs to decrypt it and check its validity.
If valid it has to be send to vendor space to check if the subject is really a patient for that care organization.

Consent validation
==================

This component handles all logic regarding validating the FHIR consent record. It checks the content via different rules predetermined by Nuts.

Crypto
======

The crypto component is an abstraction layer for the encryption/decryption process and the storage for pub/priv key-pairs. The abstraction is needed to support the different use-cases. A PGO might choose for file-storage since it'll only have a single key-pair. A service provider might choose for a Vault installation because it handles thousands of keys.

Irma
====

Generic Irma server for checking Irma proofs.

Nuts auth
=========

This component is responsible for checking the different Irma signatures used like login and connect (PGO).
It connects to the *Irma* server for checking Irma proofs if those are used to sign a consent record. This can't be done in Nuts space since it will then be encrypted.

**********
Nuts space
**********

The *Nuts space* consists of two main components: *Consent Cordapp* and the *Nuts registry*. The other components are requirements coming from technology choices for these two components. The funky figure within these components indicate that they use distributed technology. They basically are a data store without a single owner and the single truth is constructed from mutual approved contracts.

Nuts registry
=============

The registry contains mostly relational and identifying information. It must be able to answer questions like:

* What is the FHIR endpoint for this care provider?
* Which Nuts nodes serve a particular Care Provider?
* To which care provider does this care professional belong to?
* and others

The consensus about the data is constructed by a few different rules:

* It'll probably contain a tree structure, where a lower level node can only be **added** by a higher level node.
* Only the **owner** of a piece of data can update that data.

Which can be translated to things like:

* Only a Nuts node can add a care provider/application/service to a that Nuts Node.
* A care professional can only be added to a care provider by the care provider.
* The personal data of a care professional can only be updated by that care professional.

To guarantee these constraints, cryptographic rules have to be used. Nuts will probably use a combination of Irma signatures and digital signatures (PGP) for this.

Since the data within the registry is useful for everybody using Nuts, it can use a mesh network to keep in sync.

Registry UI
===========

There'll probably be two UI's: one for administrative purposes and one for care professionals to update their information. The last will then probably be a reference implementation provided by Nuts, since vendors can offer such an interface from within their own products.

Consent Cordapp
===============

The *Nuts Consent Cordapp* (What is a Cordapp?: https://docs.corda.net/cordapp-overview.html) is responsible for creating the decentralised state of consent.
The :ref:`nuts-consent-cordapp-technical-model` therefore consists mainly of encrypted data. Validation of any data specific constraints will be delegated to *Service space* during a Corda transaction.

The Corda node which will store all the consent records. Corda has currently been chosen to store the consent. It's unique ability to only include nodes that are part of the consent in the transaction makes it ideal to synchronize personal information. Although the data itself is encrypted, having it all over the place just isn't a good idea. Another plus is that it requires a third party to also acknowledge the transaction (the notary). It can even use a voting scheme to include multiple random notaries. This means that the control over all transactions lies with the community and not a single party.

For every transaction, each involved node needs to approve the transaction according to the logic in the contract. This will rely on data available in the *Nuts registry* or even the *patient callback*, proxied through *service space* for decryption. This will prevent data to scatter all over the place.

Consent bridge
==============

The bridge is an abstraction layer for translating the Java specific format from the *Consent Cordapp* to something more usefull for different vendors. This will allow different vendors to be able to use their own technology stack.


***************
Nuts foundation
***************

The *Nuts foundation* controls the root certificate, defines which nodes are added to the network and which versions of the Cordapp are allowed. This is needed because Corda requires a CA tree structure. Corda also requires a NetworkMap which must be signed by a single key. The control of this key must lie with a trusted third party. This party can only accept/reject Nuts nodes, it cannot exchange medical or personal data.

Nuts registry
=============
The *Nuts foundation* will also run a *Nuts registry* instance to add the Nuts nodes so they can be found by other nodes. The Nuts nodes can then add new organisations themselves.

Nuts Consent Discovery
======================
The *Nuts Discovery Service* is the Nuts implementation of the Network map service described by *Corda*. The *Corda* specific documentation can be found at https://docs.corda.net/network-map.html. The reason for implementing the network map as a service and not just distributing node information via other means is that this greatly simplifies development, puts the control of the root CA at the right place and creates a bridge to the *Nuts registry*. When a node registers with the discovery service, the service can also add the node to the registry. This will enable node administrators to link care providers to their Nuts node entry in the registry.

Corda often speaks of the *Doorman*. This is the *service* that is responsible for approving nodes, eg: signing certificate requests. The *Doorman* uses the intermediate CA to sign Node CA's. Right now, Nuts combines the *Doorman* service and the *NetworkMap* service in the *Nuts discovery Service*.


