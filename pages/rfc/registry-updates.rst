.. _nuts-documentation-registry-updates:

.. sectnum::

RFC: Registry Updates
#####################

.. note::

    This document is going to be replaced by Nuts Specification RFC003, RFC004 and RFC005.

Motivation
**********

The Nuts Registry contains information about who (vendors, organizations) participates in the Nuts network and where
to find the resources they're exposing through endpoint registration. Parties also use it to register their certificates
so that other parties can verify the integrity and authenticity of exposed data.

The registry is modelled as an append-only event log which makes it easy to add new data (e.g. register an endpoint).
However, there is currently no way to alter an data (e.g. rename an organization). This is an essential feature for any
Nuts network which is managed by more than a single party, since altering data currently requires coordinated data
pruning across all participating Nuts nodes.

This document proposes a mechanism for updating data in the registry.

Status of this RFC
******************

DRAFT

Terminology
***********

TODO

- Event: an event in the registry event log, e.g. the RegisterVendor event
- Event payload: the actual contents of the event, describing an object.
- Object: the thing an event describes; either a vendor, organization or endpoint.
- Event header: top-level properties of an event describing its payload.

Operations
**********

The mutate operations that can be performed (by human operators or systems) on the registry are as follows:

**Vendors**

* Register vendor (currently supported)
* Update vendor name
* Reissue CA certificate
* Rotate keys (and reissue associated certificates)
* Deactivate/reactivate vendor

**Organizations**

* Register (claim) organization as vendor (currently supported)
* Update organization name
* Reissue CA certificate
* Rotate keys (and reissue associated certificates)
* Migrate organization to another vendor

**Endpoints**

* Register endpoint for organization (currently supported)
* Update endpoint (URL, type, properties)
* Deactivate/reactivate endpoint

.. note::

    There are other operations that are somewhat related to the registry like issuing a TLS certificate to another vendor
    or revoking a certificate, but these don't mutate the registry and thus aren't relevant for this RFC.

Eventing style
**************

From an operator's perspective (be it a human or system) they will be executing one of the operations described above.
However, this isn't necessarily how it's they're mapped to events. These 2 styles have been taken into consideration,
*domain driven* and *state driven*:

**Domain Driven**

Events are modeled in such a way that they resemble the domain (the operations described earlier) as close as possible
e.g.: ``RegisterVendor``, ``UpdateVendorName``, ``IssueVendorCertificate``, ``UpdateOrganizationName``,
``DeactivateVendor``, ``ReactivateVendor``, ``MigrateOrganization``, ``UpdateEndpoint``, etc.

Events itself are very expressive and it is immediately clear what the event's intention. It makes it easy for Nuts nodes
and auxiliary systems to rationalize or report about the data.

Determining the current state of an object is done by *replaying* all events, from the very first until the very last.
When there are many events this can be an computationally expensive task. There's also the challenge of legacy: since
every event is processed again and again long after it was published, every event needs to be supported indefinitely.

.. note::

    Calculating objects' state by replaying cumulative events is known as *event sourcing*.

**State Driven**

The registry objects' (vendor, organization, endpoint) state are fully described by their respective "register"-events
(``RegisterVendorEvent``, ``VendorClaimEvent``, ``RegisterEndpointEvent``). Since they describe the object's complete state,
mutation could be done by simply replacing the entire state.

The data model of such a system is simple, since mutation is a matter of publishing the new state. Ceasing support for
certain properties is simply a matter of ignoring them, making legacy data not much of a problem.

A downside is that a system needs to inspect differences between subsequent events (E\ :sub:`n` \ Δ E\ :sub:`n+1`\)
to find out what changed and what operation was performed. Such system often implement a separate (audit) log for tracking changes.

Event sequencing
****************

This section describes how events will be sequenced.

TODO

Challenges
==========

Given the registry being an event driven distributed system, we see the following challenges:

Conflict detection and resolution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A lot can go wrong with an event driven distributed system, causing conflicting events which are distributed in the network, e.g.:

- a node publishes the same event twice (duplicate data),
- a node publishes 2 events with the same payload (redundant data),
- a node publishes 2 events with the same header but with a different payload (conflicting data).
- etc.

Because this threatens the integrity of the network, nodes must be able to detect and correct conflicting events.

Guaranteeing order
^^^^^^^^^^^^^^^^^^

Since mutations should be applied in order

Actuality
^^^^^^^^^

Nodes use the information in the registry for communication with other nodes. Therefore it's important that nodes can
verify the extend to which their view (on the distributed registry) is up-to-date. If a node assumes their view is
up-to-date while it isn't, data exchange might fail. In worst case, data might accidentally be shared with another party (data leakage).

How do nodes know that their data set is valid and actual?


Solution
========

Conflict detection is relatively straightforward since every node should receive all data that is published to the network,
so every node is able to perform checks to detect conflicts. For conflict resolution there are two styles taken into consideration:

1. Resolution which requires communication between nodes to decide which events to accept and which should be rejected.
   This often uses a form of voting to establish consensus.
2. Communicationless (as much as possible) resolution by structuring data in such a way that nodes can resolve errors
   on their own using a fixed set of rules.

Since communication (especially in a distributed system) adds many new potential fault modes a communicationless
solution is preferred (option 2). This is generally known as a
`conflict-free resolution data type <https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type>`_ (CRDT). Since
our events are basically an ordered append-only log, a set-based CRDT matches best. ``G-Set`` (Grow-only set)

Mechanics
=========

Longest chain
Garbage collecting tombstone set

Since events itself are immutable, only new events can be added to describe an object's new state. A subsequent event
completely replaces the object's state described by the preceding event. The subsequent event must refer to the event
it revises by its hash.

Event referral
^^^^^^^^^^^^^^

An event refers to a preceding event (which it replaces) by its hash. There can only be one event referring to a particular
event. In other words, subsequent events form a single stream without branching/merging:

.. code-block:: console

    +----+ replaces +----+ replaces +----+
    |e(1)+<---------+e(2)+<---------+e(3)|
    +--+-+          +----+          +--+-+
       ^                               ^
       |                               |
    initial                         current

An event's hash is calculated by hashing its payload. The hash itself is specified the event's header in the ``hash`` field.
If there's a preceding event its hash is specified in the ``prev`` field:

.. code-block:: json

    {
      "hash": "...",
      "prev": "...",
      "payload": {
        "identifier": "a09039da-f96b-4fd1-8925-fd28cb833159",
        "prop1": "Hello",
        "prop2": "World"
      }
    }

Events should specify the ``hash`` header as an extra consistency check. When processing an event, its hash should be
calculated and compared to the ``hash`` header. If it doesn't match the event must be rejected.
For historic events which do not contain the ``hash`` header this check is skipped.

Hashing
^^^^^^^

An event is hashed by taking its payload and calculating a SHA-1 hash over its stringified payload. To maintain a
consistent, deterministic hash across implementations and platforms the payload needs to be canonicalized before hashing.
The `Rundgren JSON Canonicalization Scheme (17) <https://www.ietf.org/id/draft-rundgren-json-canonicalization-scheme-17.html>`_
draft will be used for canonicalization since there's no industry standard or accepted RFC at the moment of writing.
It also refers to compliant implementations, helping Nuts implementations for different platforms.

Integrity
^^^^^^^^^

To assure that updates are authentic (and not made by an attacker), ownership of the object must be verified before
processing the event. This is done by asserting that the event was signed by the entity owning the object. In other words,
vendors can only update their own organizations and organizations can only update their own endpoints, and so on.

Furthermore, to maintain functional integrity a subsequent event can only describe the same object as its preceding event.
In other words given ``event(1)`` describing ``vendor(1)``, ``event(3)`` referring to ``event(1)`` can only describe ``vendor(1)``.
This constraint is typically implemented by comparing object identifiers.


registry status (last update) op de status page