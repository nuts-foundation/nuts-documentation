.. _nuts-documentation-architecture-certificates:

Distributed Registry
#########################

The goal of this system is to provide a registry in which parties can lookup endpoints which can be used to exchange data.

Non-functional Requirements
***************************

Availability
------------

* Every consumer of the registry should have its own instance, because:
  * End users will be using the registry to search for care organisations, probably using full-text search. Having a local copy gives the best performance and user experience.
  * Remote systems and networks can be unreliable, having a local copy ensures availability.
* Every instance should connect to other instances and be open for connections from other instances, which provides
  better availability than relying on a centralized instance.

Durability
----------

* Every instance should maintain a complete copy of the registry (including the complete history), and be willing
  to share it with other instances. This helps to mitigate the risk of data loss.

Security
--------

* Data in the registry should be digitally signed so that the identity of the producing party and the integrity
  of the data can be reassured.

Maintainability
---------------

* Changes should be backwards compatible so that new features don't break historical data.

Roles
*****

===========================  ======================================================
Role / Entity                Description
===========================  ======================================================
Care organisation            Healthcare organisation (e.g. hospital, insurance company,
                             who wishes to exchange data with other care organisations
                             through their respective vendors using Nuts.
Care organisation signatory  Person within a care organisation who is authorised to
                             sign in the name of their organisation.
Vendor                       Organisation who supplies and operates a care organisation's
                             software systems containing medical data.
Nuts administrator           Person within the Nuts Foundation who is authorised to
                             accept registrations from vendors.
Node                         Vendor's system used to communicate with the network,
                             conform the Nuts specification.
===========================  ======================================================

Architecture
************

The data contained in the registry is modelled as streams of events. Every stream starts with a genesis event,
every successive event refers to its preceding event. Events are immutable, so every change (or deletion) is
recorded as a new event.
For now, only the registration of a new vendor starts a new stream. The minimal event stream leading to an
endpoint looks as following:

.. raw:: html
    :file: ../../_static/images/distributed-registry-vendor-stream.svg

Every event contains the following fields, aside from its event-specific data:

======================  ===========  ====
Field                   Format       Description
======================  ===========  ====
ID                      UUID / hash  Uniquely identifies this event
Previous event ID       UUID / hash  Refers to the previous event in this stream
Date of issue           timestamp    Timestamp the event was emitted
Signature               TBD          Signature which authenticates the event
======================  ===========  ====

Processes
*********

Registering a vendor
--------------------
This process makes a vendor known to the network. Afterwards:

* The vendor is known to the network.
* The vendor van register their contracts with care organisations.

Limitations:

* For now, we only support 1 node per vendor.
* For now, we only support 1 vendor per care organisation (but a vendor can serve multiple care organisations).

Steps:

#. Node registration:
    #. The vendor generates a CSR (for the node's CA certificate) and submits it to Nuts Discovery.
    #. A Nuts administrator accepts the CSR and issues the node CA certificate.
    #. The vendor issues node identity certificate (for its node) using its CA.
    #. The vendor submits its node information (node identity certificate, node address, ...) to Nuts Discovery.
    #. The node information is published so that every (other) node in the network knows about the vendor's node.
#. Vendor registration:
    #. The vendor generates a CSR (for the vendor's CA certificate) and submits it to Nuts Registry.
       TODO: Nuts Registry shouldn't be responsible for issuing certificates, since we're also issuing them
       from Nuts Discovery? Or maybe they're RA's in front of the Nuts Foundation CA?
    #. A Nuts administrator accepts the CSR and issues the vendor CA certificate.
    #. The vendor creates the registration event and signs it using the issued CA certificate.

VendorRegistration Event
^^^^^^^^^^^^^^^^^^^^^^^^

======================  =====
Field                   Format
======================  =====
Vendor CA certificate   PEM encoded X.509
======================  =====

Validation
""""""""""

#. Is the vendor CA certificate signed by the Nuts Foundation?


Registering a contract (vendor - care organisation)
---------------------------------------------------

Through this process a vendor registers a care organisation as its client.
Afterwards, the vendor can register endpoints for the care organisation it serves through its node.

Steps:

#. The vendor generates a CSR for the vendor's contract with the care organisation and signs it
   using its vendor CA certificate (see the table below for the prescribed fields of the certificate).
#. The vendor creates the contract proposal and signs it using the (just issued) contract certificate.
#. The vendor creates the contract approval and signs it (again) using the contract certificate (see note below).

Contract certificate fields:

====================  =====
Field                 Value
====================  =====
Validity              3 years
Key usage             Digital signature
CA?                   false
Issuer                Vendor CA certificate
Subject.CN            Care organisation's name or unit within the organisation
Subject.O             Care organisation's name
Subject.ST            Province in which the care organisation is located
Subject.C             Care organisation's country
Subject.serialNumber  Chamber of Commerce registration number of the care organisation
====================  =====

ContractProposal Event
^^^^^^^^^^^^^^^^^^^^^^

This event is emitted (and thus signed) by the vendor and proposes the contract to the care organisation.

======================  =====
Field                   Format
======================  =====
Contract ID             UUID type 4
Care organisation       OID
Vendor                  OID
Start date              RFC 3339 timestamp
End date                RFC 3339 timestamp (optional)
Certificate             PEM encoded X.509
======================  =====

Validation
""""""""""

#. Is this event referring to an event stream of the same vendor?
#. Is the start date on or after the date of issue of the event?
#. Is the end date, if present, chronologically after the start date?

ContractApproval Event
^^^^^^^^^^^^^^^^^^^^^^

This event is emitted (and thus signed) by the care organisation and approves the contract.

======================  =====
Field                   Format
======================  =====
Contract ID             UUID type 4
======================  =====

Validation
""""""""""

#. Is the contract ID referring to a proposed contract?

.. note::
  In the ideal situation, the contract is digitally signed 2-way: the vendor appoints its client (signature 1),
  which is approved by the client (signature 2). This 2nd signature is authorised by a (digitally signed) registration
  of the Chamber of Commerce (Kamer van Koophandel) which identifies the care organisation.
  However, in the current state IRMA doesn't provide the Chamber of Commerce registration attribute (to identify
  organisation's authorised signatories). We expect this attribute to be available in the (near) future, so for now the
  2nd signature will also be the vendor's. This will suffice for now, since every vendor is known by name and face by
  the Nuts Foundation.

Registering/updating an endpoint
--------------------------------

Through this process a vendor can register (or update) an endpoint on which they use to serve data for one of their clients.
Afterwards, other vendors can lookup the endpoint for data exchange.

EndpointRegistration Event
^^^^^^^^^^^^^^^^^^^^^^^^^^

======================  =====
Field                   Format
======================  =====
Care organisation       OID
Vendor                  OID
Endpoint ID             URN
Endpoint type           URN
Endpoint location       URL
Start date              RFC 3339 timestamp
End date                RFC 3339 timestamp (optional)
Signature               (TBD)
======================  =====

Validation
""""""""""

#. Is there an approved contract for this vendor / care organisation?
#. Is the end date, if present, chronologically after the start date?
#. Is the endpoint type one of the supported types?
#. Is the endpoint location format supported by the endpoint type?
#. Is the start date on or after the date of issue of the event?