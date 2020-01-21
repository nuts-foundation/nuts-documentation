.. _nuts-documentation-architecture-certificates:

RFC: Distributed Registry
#########################

The goal of this system is to provide a registry in which parties can:

* Lookup endpoints which can be used to exchange data.
* Search for care organisations connected to the Nuts network (through their vendor).

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
* Connections (between instances) are secured using 2-way SSL. Vendor instance identify and authenticate themselves
  to their peers using their node identity certificate.

Maintainability
---------------

* Changes should be backwards compatible so that new features don't break historical data.
* New instances must be able to get the full history after joining.

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

The data contained in the registry is modelled as a chain of immutable events, where every event refers to its
preceding event. Since events are immutable, every change (or deletion) is recorded as a new event. Each event is signed
so that every participant can validate the integrity of the event chain.

Every event contains the following fields, aside from its event-specific data:

======================  =============  ====
Field                   Format         Description
======================  =============  ====
ID                      Hash           Uniquely identifies this event
Previous event          Hash           Refers to the previous event in this stream
Date of issue           Timestamp      Instant the event was emitted
Signature               RFC 7515 JWS   Signature which authenticates the event
Payload                 JSON           Payload of the event.
======================  =============  ====

Processes
*********

.. raw:: html
    :file: ../../_static/images/distributed-registry-processes.svg

Registering a vendor
--------------------
This process makes a vendor known to the network. Afterwards:

* The vendor is known to the network.
* The vendor can claim their relationships with care organisations.

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
    #. A Nuts administrator accepts the CSR and issues the vendor CA certificate.
    #. The vendor issues its vendor certificate using its CA certificate. This certificate is used to sign events.
    #. The vendor creates the registration event and signs it using the vendor certificate.

Vendor CA certificate fields:

====================  =====
Field                 Value
====================  =====
Validity              3 years
Key usage             Digital signature
CA?                   true
Issuer                Nuts Vendor CA
Subject.CN            Vendor's name or unit within the organisation
Subject.O             Vendor's name
Subject.ST            Province in which the vendor is located
Subject.C             Vendor's country
Subject.serialNumber  Chamber of Commerce registration number of the vendor
====================  =====

Vendor Registration Event Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

======================  =====
Field                   Format
======================  =====
Vendor CA certificate   JWK encoded X.509
======================  =====

.. note::
  We could add the vendor CA certificate to the X.509 trust chain when a certificate is referenced,
  making this event unnecessary. However, this bloats the event header so it's desirable to register it once using
  this event so it can be looked up every time a certificate (issued by this CA certificate) is validated.

Registering a vendor - care organisation relation
-------------------------------------------------

Through this process a vendor registers a care organisation as its client.
Afterwards, the vendor can register endpoints for the care organisation it serves through its node.

Steps:

#. The vendor issues a certificate for the care organisation under its vendor CA certificate
   (see the table below for the prescribed fields of the certificate).
#. The vendor creates the claim event and signs it using the (just issued) .

Care organisation certificate fields:

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

Vendor Claim Event Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^

======================  =====
Field                   Format
======================  =====
Care organisation       OID
Vendor                  OID
Start date              RFC 3339 timestamp
End date                RFC 3339 timestamp (optional)
Certificate             JWK encoded X.509
======================  =====

.. note::
  In the ideal situation, the relationship is claimed 2-way: the vendor claims its care organisation as client,
  and the care organisation claims its vendor. This 2nd claim is authorised by a (digitally signed) registration
  of the Chamber of Commerce (Kamer van Koophandel) which identifies the care organisation.
  However, in the current state IRMA doesn't provide the Chamber of Commerce registration attribute (to identify
  organisation's authorised signatories). We expect this attribute to be available in the (near) future, so for now the
  care organisation's claim will be omitted. This will suffice for now, since every participating vendor is trusted
  since they're known by name and face by the Nuts Foundation.

Registering/updating an endpoint
--------------------------------

Through this process a vendor can register (or update) an endpoint on which they use to serve data for one of their clients.
Afterwards, other vendors can lookup the endpoint for data exchange.

Endpoint Registration Event Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
======================  =====

Requesting a vendor client certificate
--------------------------------------

This process is used by vendors to request a TLS certificate to authenticate themselves at other vendors' endpoints.

TODO