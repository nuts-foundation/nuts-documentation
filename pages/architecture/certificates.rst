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

Vendor CA Certificate
*********************

Common Name: CN=...,O=...,C=NL

=================================  ==========  =========================================
Extension                          Critical?   Value
=================================  ==========  =========================================
BasicConstraints                   Yes         CA=true pathLenConstraint=1
NameConstraints                    ???         ???
KeyUsage                           Yes         digitalSignature & keyCertSign & crlSign
SubjectAltName                     No          Vendor identifier: otherName, 1.3.6.1.4.1.54851.4=IA5String.<number>
Nuts domain (1.3.6.1.4.1.54851.3)  No          IA5String=healthcare | social | pgo | insurance
CRLDistributionPoints              No          Distribution point (URL) of the CRL
=================================  ==========  =========================================

TLS Client Certificate
**********************

Common Name: CN=...,O=...,C=NL

=================================  ==========  ====================================================================
Extension                          Critical?   Value
=================================  ==========  ====================================================================
BasicConstraints                   Yes         CA=false
KeyUsage                           Yes         digitalSignature
SubjectAltName                     No          Vendor identifier: otherName, 1.3.6.1.4.1.54851.4=IA5String.<number>
ExtendedKeyUsage                   Yes         clientAuth
CRLDistributionPoints              No          Distribution point (URL) of the CRL
=================================  ==========  ====================================================================

.. note::
    The Nuts domain extension indicates what kind of node it is. This is needed to distinguish between nodes in the
    medical, social, insurance or private domain. Some of these domains are not allowed to process Social Security Numbers (BSN).
    To make sure this is embedded in the security model, it's added to the certificate and must be transferred to issued certificates.

CN
**

The common name of the certificates used in two-way ssl must conform to the following specification:

    nuts-[network]-[app_name]

where *nuts* is static, *[network]* must be replaced by ``development``, ``test`` or ``production`` and *[app_name]* needs to be replaced with the name also used in the login contracts.

.. note::

    The specification of the CN might be changed to a certificate extension in the future, which will allow the CN to be freely choosen.
