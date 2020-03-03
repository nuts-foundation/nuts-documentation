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

If certificates are issued for any other environment than production, it is indicated in the common name. If the
certificate is issued for the production environment, the environment part is omitted.

Vendor CA Certificate
=====================

The Vendor CA Certificate is the vendor's root certificate which is only used to issue its intermediate CA certificate.
The key should be kept offline.

The certificate contains the 'Nuts domain' extension, indicating domain the vendor operates in. It also
contains a subject alternative name containing the vendor ID (1.3.6.1.4.1.54851.4) corresponding with the vendor's
ID in the registry.

Vendor CA Intermediate Certificate
==================================

The Vendor CA Intermediate certificate is used by the vendor on a day-to-day basis to issue certificates.

.. note::
    The Nuts domain extension indicates what kind of node it is. It can be medical, social, insurance or private.
    Some of these domains are not allowed to process Social Security Numbers (BSN).

=================================  ==========  =========================================
Extension                          Critical?   Value
=================================  ==========  =========================================
Subject                                        CN=<Vendor> CA <Environment>,O=<Vendor>,C=NL
BasicConstraints                   Yes         CA=true pathLenConstraint=1
KeyUsage                           Yes         digitalSignature & keyCertSign & crlSign
SubjectAltName                     No          Vendor identifier, otherName: 1.3.6.1.4.1.54851.4=UTF8String.<number>
CRLDistributionPoints              No          Distribution point (URL) of the CRL
Nuts domain (1.3.6.1.4.1.54851.3)  No          UTF8String=healthcare | social | pgo | insurance
=================================  ==========  =========================================

Vendor Certificate
------------------

This certificate is issued by the vendor to itself to sign events. It's validity period should be as short as possible,
just long enough to allow the event to be timestamped and signed. In any case the validity period should not be longer
than 1 minute (60 seconds).

=================================  ==========  =========================================
Extension                          Critical?   Value
=================================  ==========  =========================================
Subject                                        CN=<Vendor> <Environment>,O=<Vendor>,C=NL
KeyUsage                           Yes         digitalSignature
SubjectAltName                     No          Vendor identifier, otherName: 1.3.6.1.4.1.54851.4=UTF8String.<number>
CRLDistributionPoints              No          Distribution point (URL) of the CRL
=================================  ==========  =========================================

Organisation Certificates
-------------------------

The organisation certificate is issued by the vendor when it claims a care organisation as their client. It is used to
sign/encrypt JWTs and as KEK for encrypted data exchange with other vendors.

=================================  ==========  =========================================
Extension                          Critical?   Value
=================================  ==========  =========================================
Subject                                        CN=<Organisation>,O=<Vendor>,C=NL
KeyUsage                           Yes         digitalSignature, keyEncipherment, dataEncipherment
SubjectAltName                     No          Organisation identifier, otherName: 2.16.840.1.113883.2.4.6.1=UTF8String.<number>
CRLDistributionPoints              No          Distribution point (URL) of the CRL
Nuts domain (1.3.6.1.4.1.54851.3)  No          UTF8String=healthcare | social | pgo | insurance
=================================  ==========  =========================================


TLS Certificates
----------------

TLS certificates are issued by the vendor itself and are used to validate incoming requests. The implementation details
are therefore a recommendation, since non-compliance imposes a risk limited to the vendor's software, rather than the
whole Nuts network. Adhering to these recommendations also helps to establish a common denominator for implementations
across the network.

Vendor TLS CA Certificate
^^^^^^^^^^^^^^^^^^^^^^^^^

This is a intermediate CA certificate, used to issue TLS client certificates. The certificate is issued by the Vendor CA.

=================================  ==========  =========================================
Extension / Field                  Critical?   Value
=================================  ==========  =========================================
Subject                                        CN=<Vendor> TLS CA <Environment>,O=<Vendor>,C=NL
BasicConstraints                   Yes         CA=true pathLenConstraint=0
KeyUsage                           Yes         digitalSignature, keyCertSign, crlSign
SubjectAltName                     No          Vendor identifier, otherName: 1.3.6.1.4.1.54851.4=UTF8String.<number>
CRLDistributionPoints              No          Distribution point (URL) of the CRL
=================================  ==========  =========================================

TLS Client Certificate
^^^^^^^^^^^^^^^^^^^^^^

When a vendor A's software wants to retrieve data from vendor B, the TLS connection is secured with certificates 2-way:
vendor B presents a server certificate and vendor A with a client certificate which is issued by vendor B. That way,
vendor B can easily authenticate incoming clients by testing them against its own certificate chain.

Issued certificates should contain the 'Nuts domain' extension, which can be used to determine whether vendor (A) is
allowed to receive Social Security Numbers or that they should be masked. The 'Nuts domain' of the issued certificate
should match that of the vendor's (A) Vendor CA Certificate.

Issued certificates should also contain the vendor's (A) ID as subject alternative name.

=================================  ==========  ====================================================================
Extension / Field                  Critical?   Value
=================================  ==========  ====================================================================
Subject                                        CN=<Vendor A> <Environment>,O=<Vendor A>,C=NL
KeyUsage                           Yes         digitalSignature
ExtendedKeyUsage                   Yes         clientAuth
SubjectAltName                     No          Vendor identifier, otherName: 1.3.6.1.4.1.54851.4=UTF8String.<number>
CRLDistributionPoints              No          Distribution point (URL) of the CRL
Nuts domain (1.3.6.1.4.1.54851.3)  No          UTF8String=healthcare | social | pgo | insurance
=================================  ==========  ====================================================================

.. note::
    The certificate is issued by vendor B to vendor A through an out-of-band process, meaning the network does not
    provide means to transport the certificate after issuance.

CN
**

The common name of the certificates used in two-way ssl must conform to the following specification:

    nuts-[network]-[app_name]

where *nuts* is static, *[network]* must be replaced by ``development``, ``test`` or ``production`` and *[app_name]* needs to be replaced with the name also used in the login contracts.

.. note::

    The specification of the CN might be changed to a certificate extension in the future, which will allow the CN to be freely choosen.

.. note::

    The HTTP over TLS specification (RFC2818_) mentions in section 3.1 that usage of Common Name is deprecated and that
    subject alternative names should be used. This only concerns the validation of a server's identity and does not
    imply that the use of Common Names should be avoided altogether. Therefore, we'll still use Common Names in our
    distinguished names without specifying it as subject alternative names as long as it's not a server certificate.

.. _RFC2818: https://tools.ietf.org/html/rfc2818#section-3.1