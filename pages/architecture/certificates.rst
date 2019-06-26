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