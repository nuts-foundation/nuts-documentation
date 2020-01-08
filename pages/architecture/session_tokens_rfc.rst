.. _nuts-documentation-session-tokens:

RFC: Session Tokens
###################

Session tokens are tokens which should be provided along an API request which refer
to contextual information of the request.
A session is scoped to an actor, subject and custodian. Optionally a reference
to a specific consent record can be provided.

Motivation
**********

Originally the plan was to only provide the Actor's
:ref:`nuts-documentation-login-contracts` along the API request, assuming this
was sufficient. However we encountered situations where more information was
needed.

Size
####

The more attributes are requested to sign the IMRA login contract, the bigger
the resulting JSON hash gets. So big that when base64 encoded and signed as a
JWTit exceeds the often used http header size of 8kb. Although header sizes are
not officially limited, approaching this limit will likely result in unforeseen
problems.

Subject scoping
###############

How does the custodians server knows about which subject the data request is
about? Some protocols contain a patient id in the URL, others require inspecting
the body or headers. In order for Nuts to have a minimal implementation effort,
is is important to refer to the patient in a uniform way.

Custodian scoping
#################

Some protocols run on a multi tenant environment which need a way of specifying
the custodian the request is about. Just like the subject, in order for Nuts to
have a minimal implementation effort, is is important to refer to the custodian
in a uniform way.

Cost of checking consent and Identity
#####################################

When performing multiple request for a triple of custodian|subject|actor, the
:ref:`nuts-documentation-login-contracts`, and consent must be verified each
time, increasing response time.

Limitations
###########

Because it is necessary to request a new session for every combination of subject
custodian and actor, bulk operations, like getting reports for many patients
at once, is not possible. We could consider allowing list of subjects during the
token requests.

Mechanics
*********

To resolve above issues, we'll introduce sessions. A session lives on the
custodian side and contains information about the subject, actor and custodian.
Additional information can be provided like, session expiration and consent reference.

To refer to a session during an API call, a reference to the session must be provided.
For example, in the case of a HTTP REST call in the form of a bearer token in the
authorization header.

The session token can be provided using a OAuth 2.0 flow with the
`urn:ietf:params:oauth:grant-type:jwt-bearer` grant_type. This profile is described in `RFC7523 <https://tools.ietf.org/html/rfc7523>`_.

The returned session token can be a reference to a session or a signed JWT
containing all the relevant information needed by the API server.


.. raw:: html
  :file: ../../_static/images/nuts_session-tokens.svg

The endpoint
************

A vendor can implement this flow using its own existing infrastructure or use
a standard and minimal nuts implementation.
The address of the OAuth 2.0 server endpoint must be announced in the Nuts
registry under the `urn:oid:1.3.6.1.4.1.54851.2:session-tokens` type.

The JWT
*******

The JWT used to obtain the token should consists of the following fields:

.. code-block:: json

  {
    "iss": "urn:oid:2.16.840.1.113883.2.4.6.1:48000000",
    "sub": "urn:oid:2.16.840.1.113883.2.4.6.1:12481248",
    "sid": "urn:oid:2.16.840.1.113883.2.4.6.3:9999990",
    "aud": "https://target_token_endpoint",
    "usi": {...irma based signature...},
    "osi": {...hardware token sig...},
    "con": {...additional context...},
    "exp": max(time_from_irma_sign, some_limited_time),
    "iat": "",
    "jti": {unique-identifier}
  }


Iss
---
The issuer in the JWT is always the actor, thus the care organization doing the request.
This iss used to find the public key of the issuer from the Nuts registry.

.. note::
Since the nuts token is signed with the private key of the requester, it is not trivial to verify the signature of the token.
When recieving a request, any token signature verification steps must be postponed until it is clear a token is not a nuts token.

Sub
---
The subject (not a Nuts subject) contains the urn of the custodian. The custodian information is used to find the relevant consent (together with actor and subject).

Sid
---
The Nuts subject id, patient identifier in the form of a BSN.

Aud
---
As per `rfc7523 <https://tools.ietf.org/html/rfc7523>`_, the aud must be the token endpoint. This can be taken from the Nuts registry.

Usi
---
User signature. This is the Irma signature presented to the user. Base64 encoded.

Osi
---
Ops signature, optional signature coming from a hardware token, indicating the user belongs to the issuer organization. Can be linked to the Nuts registry.

Con
---
Base64 encoded json representing key-value pairs for additional context for the requested access token. Such as task flow selection.

Exp
---
Expiration, should be set relatively short since this call is only used to get an access token. Must not be bigger than the validity of the Nuts signature validity.

Iat
---
Issued at

Jti
---
Unique identifier, secure random number to prevent replay attacks. The authorization server must check this!
