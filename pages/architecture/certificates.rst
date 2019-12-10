.. _nuts-documentation-architecture-certificates:

Certificate structure
#####################

Although Nuts promotes a decentralized architecture and uses individual signatures as proofs, it's still quite hard to secure the network without using two-way TLS.
Corda, for example, requires a certificate tree structure for its permissioned network.
Also vendors should be enabled to block other vendors if they think those vendors are compromised. They are responsible for the security, so they should be able to act on it.
So the need for a root CA can be derived from these requirements. Access to the key will be restricted to the Nuts foundation.
This does not give the foundation super powers since vendors and users (both validated by Irma signatures) are still part of the security scheme to access data.

.. raw:: html
    :file: ../../_static/images/certificates.svg

Extensions
**********

The vendor CA's will get a x509.v3 extension with a specific oid to indicate what kind of node it is.
This is needed to distinguish between nodes in the medical, social, insurance or private domain. Some of these domains are not allowed to process BSN's.
To make sure this is embedded in the security model, it's added to the certificate and must be transferred to issued certificates.

CN
**

The common name of the certificates used in two-way ssl must conform to the following specification:

    nuts-[network]-[app_name]

where *nuts* is static, *[network]* must be replaced by ``development``, ``test`` or ``production`` and *[app_name]* needs to be replaced with the name also used in the login contracts.

.. note::

    The specification of the CN might be changed to a certificate extension in the future, which will allow the CN to be freely choosen.
