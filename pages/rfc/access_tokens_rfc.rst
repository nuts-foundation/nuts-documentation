.. _nuts-documentation-access-tokens:

RFC: Access Tokens
###################

Access tokens are short lived opaque tokens which accommodate an API request and
refer to contextual information of the request.
Such an API request is scoped by the token to an actor, subject and custodian.
Optionally a reference to a specific consent record can be provided.

Status of this RFC
******************

This RFC is a work in progress. See the TODO_ section below. To provide comments
create an issue on the `Github repository containing this documentation <https://github.com/nuts-foundation/nuts-documentation/issues>`_.

Motivation
**********

Originally the plan was to only provide the Actor's
:ref:`nuts-documentation-login-contracts` along the API request, assuming this
was sufficient. However we encountered situations where more information was
needed.

Size
====

The more attributes are requested to sign the IMRA login contract, the bigger
the resulting JSON hash gets. So big that, when base64 encoded and signed,
it exceeds the often used http header max size of 8kb. Although header sizes are
not officially limited, approaching this limit will likely result in unforeseen
problems.

Subject scoping
===============

How does the custodian's server know which subject the data request is
about? Some protocols contain a patient id in the URL, others require inspecting
the body or headers. In order for Nuts to have a minimal implementation effort,
it is important to refer to the patient in a uniform way.

Custodian scoping
=================

Some protocols run on a multi tenant environment which need a way of specifying
the custodian the request is about. Just like the subject, in order for Nuts to
have a minimal implementation effort, it is important to refer to the custodian
in a uniform way.

Cost of checking consent and Identity
=====================================

When performing multiple request for a triple of custodian|subject|actor, the
:ref:`nuts-documentation-login-contracts`, and consent must be verified each
request. The crypto behind IRMA takes some time, adding up to the response time
of these requests.

Limitations
***********

Because it is necessary to request a new access token for every combination of subject
custodian and actor, bulk operations, like getting reports for many patients
at once, is not possible. We could consider allowing list of subjects during the
token requests.

Mechanics
*********

To resolve above issues, we'll introduce *access tokens*. Access tokens refer to
a context of the current request. This information can be embedded in an encrypted
access token or can be stored on the authorization server. This information contains
of the subject, actor and custodian.
Additional information can be stored like, token expiration and consent reference.

To **obtain the users identity**, the users follows a standard Login flow presenting
its attributes using IRMA, signing the LoginContract. The signed contract can
be stored by the users XIS in a session for the validity of the contract.

Once the user wants to request data about a subject, a **access token must be obtained**.
The access token can be acquired using an OAuth 2.0 flow with the
`urn:ietf:params:oauth:grant-type:jwt-bearer` grant_type. This profile is
described in `RFC7523 <https://tools.ietf.org/html/rfc7523>`_.

For Actor identification the requests will be performed over a mutual SSL connection.
The client will use a certificate signed by the CA of the custodian.
This will be described in another RFC.

As the rfc7523 describes, a JWT Bearer token should be build and posted to the
Authorization Server (Nuts node or own implementation). The Authorization Server
checks all requirements like a valid consent, validity of the access token,
match of clientID and Actor etc. If everything is valid, it creates and returns
the access token.

The client can now perform API requests using the same mutual SSL connection
providing the access token along with the request.
For example, in the case of a HTTP REST call in the form of a bearer token in the
Authorization Header. Contrary to the
`original OAuth 2 definition <https://tools.ietf.org/html/rfc6750#section-1.2>`_
of a Bearer token, the token is bound to the client by its two way (mutual) SSL
certificate.

The XIS receiving the access token can retrieve the context by
`token introspection <https://tools.ietf.org/html/rfc7662>`_.

.. raw:: html
  :file: ../../_static/images/nuts_session-tokens.svg

The endpoint
************

A vendor can implement this flow using its own existing infrastructure or use
a by Nuts provided minimalistic implementation.

The address of the Authorization Server endpoint must be provided in the resource server's registry entry in the properties object under the key `authorizationServerURL`.

A complete registry entry for an sso endpoint can look like this:

::

    {
        "URL": "http://sso.custodian.local/land",
        "endpointType": "urn:oid:1.3.6.1.4.1.54851.1:nuts-sso",
        "identifier": "7b8f7852-d218-4242-8406-39cf6abcde58",
        "properties": {
            "authorizationServerURL": "http://nuts.custodian.local
        },
        "status": "active"
    }

The JWT
*******

The JWT used to obtain the token should consists of the following fields:

.. code-block:: json

  {
    "iss": "urn:oid:2.16.840.1.113883.2.4.6.1:48000000",
    "sub": "urn:oid:2.16.840.1.113883.2.4.6.1:12481248",
    "sid": "urn:oid:2.16.840.1.113883.2.4.6.3:9999990",
    "aud": "https://target_token_endpoint",
    "usi": {...Base64 encoded IRMA based signature...},
    "osi": {...hardware token sig...},
    "con": {...additional context...},
    "exp": max(time_from_irma_sign, some_limited_time),
    "iat": 1578910481,
    "jti": {unique-identifier}
  }


iss
===
The issuer in the JWT is always the actor, thus the care organization doing the request.
This is used to find the public key of the issuer from the Nuts registry.

.. note::

    Since the nuts token is signed with the private key of the requester, it is not
    trivial to verify the signature of the token.
    When receiving a request, any token signature verification steps must be
    postponed until it is clear a token is not a nuts token.

sub
===
The subject (not a Nuts subject) contains the urn of the custodian. The
custodian information is used to find the relevant consent (together with actor
and subject).

sid
===
The Nuts subject id, patient identifier in the form of an oid encoded BSN.

aud
===
As per `rfc7523 <https://tools.ietf.org/html/rfc7523>`_, the aud must be the
token endpoint. This can be taken from the Nuts registry.

usi
===
User signature. This is the IRMA signature presented to the user. Base64 encoded.
It contains the users identity and consent for the vendor to use the Nuts
network on its behalf.

osi
===
Ops signature, optional signature coming from a hardware token, indicating the
user belongs to the issuer organization. Can be linked to the Nuts registry.
This mechanism is used to establish an employer relationship without actual
placing personal information into the registry.

con
===
Base64 encoded JSON representing key-value pairs for additional context for the
requested access token. Such as task flow selection.

exp
===
Expiration, should be set relatively short since this call is only used to get
an access token. Must not be bigger than the validity of the Nuts signature validity.

iat
===
Issued at. NumericDate value of the time at which the JWT was issued.

jti
===
Unique identifier, secure random number to prevent replay attacks. The
authorization server must check this!

TODO
****

Some things have to be defined:

* the exact formats of API calls
* the mechanisms of mutual SSL
