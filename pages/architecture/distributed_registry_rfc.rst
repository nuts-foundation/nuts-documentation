.. _nuts-documentation-architecture-certificates:

RFC: Distributed Registry
#########################

High-level goals

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
Vendor signatory             do we need this???
Nuts administrator           Person within the Nuts Foundation who is authorised to
                             accept registrations from vendors.
Node                         Vendor's system used to communicate with the network,
                             conform the Nuts specification.
Node administrator           Person who's in charge of administering a node (e.g.
                             accepting new nodes to the network).
===========================  ======================================================

Processes
*********

...

Registering a vendor
--------------------
This process makes a vendor known to the network. Afterwards, the vendor can:

* Register their contracts with care organisations (for now, we support 1 vendor/care organisation).
* ...?

Limitations:

* For now, we only support 1 node per vendor.
* For now, we only support 1 vendor per care organisation is supported (but a vendor can serve multiple multiple care organisations).

Steps:

#. Node registration:
    #. The vendor generates a CSR (for the node's CA certificate) and submits it to Nuts Discovery(define???).
    #. A Nuts administrator accepts the CSR and issues the node CA certificate.
    #. The vendor issues node identity certificate (for its node) using its CA.
    #. The vendor submits its node information (node identity certificate, node address, ...) to Nuts Discovery.
    #. The node information is published so that every (other) node in the network knows about the vendor's node.
#. Vendor registration:
    #. The vendor generates a CSR (for the vendor's CA certificate) and submits it to Nuts Registry.
       TODO: Nuts Registry shouldn't be responsible for issuing certificates, since we're also issuing them
       from Nuts Discovery? Or maybe they're RA's in front of the Nuts Foundation CA?
    #. A Nuts administrator accepts the CSR and issues the vendor CA certificate.
    #. The vendor is added to the registry.

TODO: Link to certificate tree

Registering a care organisation
-------------------------------

Registering a vendor - care organisation relation
-------------------------------------------------
(TODO; decide on a better title)

Registering a notary
-------------------------------------------------
TODO

Key Material
************
TODO

Recommendations
---------------

Participating parties should consider the following recommendations:

* Every certificate should use a key pair. In other words, key pairs shouldn't be shared across certificates.
  The only instance where a key pair might be reused is when the certificate is reissued.
* TODO: What about key strength? RSA 4096 vs RSA 2048
* TODO: RSA vs RSASSA-PSS (RSA with salted mask generation function)? eIDAS requires either RSASSA-PSS or elliptic curves.

Interfaces
**********
TODO

Which roles have which interfaces?

Impl:

- CRDTs
- Gossip protocols