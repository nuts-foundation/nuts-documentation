Identifiers
===========

The main Nuts components rely heavily on identifiers. In most of the source code these are referred to as ``Identifier`` and are coded as strings.
This makes the code reusable for all kind of use-cases. Within the Nuts *spaces* identifiers are just used to link different records.
On the boundary of *service space* and *vendor space*, however, some choices have to be made in order for vendors to understand the different identifiers and connect them to their own logic.

Nuts identifiers use **URN** style representation and use existing URN's where available. An identifier consists of two parts, the URN and the unique representation within the URN namespace. See https://en.wikipedia.org/wiki/Uniform_Resource_Name or https://tools.ietf.org/html/rfc8141 for how URN's are constructed.

The complete Nuts identifiers thus consists of the URN followed by a colon and then the identifying value.

.. code-block::

    urn:<NID>:<NSS>:<id>

=====================================   ====================    =====================   =============================================================================
Identifier                              Known as                Used in                 Description
=====================================   ====================    =====================   =============================================================================
urn:oid:2.16.840.1.113883.2.4.6.3       BSN                     Consent                 'Burgerservicenummer' used within the Netherlands for identifying a patient.
urn:oid:2.16.840.1.113883.2.4.6.1       AGBCode                 Consent/Registry/Auth   The AGBCode as maintained by Vektis for both Organizations and professionals.
urn:oid:1.3.6.1.4.1.54851               Nuts root OID                                   Root OID used for mounting point of Nuts classifiers.
urn:oid:1.3.6.1.4.1.54851.1             Nuts consent classes    Consent                 Consent classifier ID, eg: urn:oid:1.3.6.1.4.1.54851.1:MEDICAL.
urn:oid:1.3.6.1.4.1.54851.2             Nuts endpoint types     Registry                The type of endpoint a URL points to, eg: Consent, Registry or FHIR.
urn:oid:1.3.6.1.4.1.54851.3             Nuts domain             Registry                Domain supported by Nuts in which a vendor/organization operates.
                                                                                        Supported values: "healthcare", "social", "pgo" or "insurance".
urn:oid:1.3.6.1.4.1.54851.4             Nuts vendor ID          Registry                Identifies a registered vendor. Contains the Chamber of Commerce registration
                                                                                        number, encoded as a string.
urn:ietf:rfc:1779                       X500Name                Registry                Name notation used in X509 Certificates. Identifies a Consent Corda node.
urn:ietf:rfc:3986                       URI                     Consent/Registry/Auth   Indication the value is of type URI, should be used to indicate an OID
=====================================   ====================    =====================   =============================================================================

The choice (for now) has been made to use URN and OID style identifiers.
URL style identifiers commonly used as namespaces seem to be all over the place and each new project or initiative declares its own namespace for the same identifier.
We'd like the identifiers to be more static.

Examples
--------

BSN
...
::

    urn:oid:2.16.840.1.113883.2.4.6.3:999999990

AGBCode
.......
::

    urn:oid:2.16.840.1.113883.2.4.6.1:00000007

Hospital OID
............
In this case the hospital has its own OID.
::

    urn:ietf:rfc:3986:2.16.840.1.113883.2.4.3.55

X500Name
........
::

    urn:ietf:rfc:1779:O=Nuts, OU=Healthcare, C=NL, ST=Gelderland, L=Eibergen, CN=nuts_corda_development_local

Consent Endpoint
........
Endpoint to connect with to synchronise consents
::

    urn:oid:1.3.6.1.4.1.54851.2:consent
