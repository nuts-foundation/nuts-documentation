.. _nuts-documentation-sso:

RFC: Single Sign On (SSO)
#########################


Motivation
**********

Care professionals sometimes work with the same patients for several care
organizations using different software vendors. Their daily work can greatly be
improved when they can share their identity between these software applications.
Therefor we here propose a way of Single-Sign-On (SSO) for sharing a universal
identity among different software applications, not necessary from the same employer.
The method uses most of the techniques already present in a standard Nuts node.

This document builds on Login Contracts and the RFC on Session Tokens.

Status of this RFC
******************

This RFC is a work in progress. See the TODO_ section below. To provide comments
create an issue on the `Github repository containing this documentation
<https://github.com/nuts-foundation/nuts-documentation/issues>`_.

Limitations
***********

A party who wants to verify the identity of the user must run a Nuts Node.

The application the user jumps to can only show patient provided within the session.

The application the user jumps to does not get notified about logout operations
from the original application. Sessions should end after closing the browser window/tab.


Mechanics
*********

We have two applications: The source application, and the destination application.
Both run their own Nuts Node (could be the same node). The source application collects
information about the user and patient, acquires an access token from the destination application.
With this token the user jumps to the destination application.


SSO Steps
=========

#. The User loads a page in a patient context
#. The source application checks rights and settings to see if this user is allows to jump (local policy)
#. The source application checks if the patient has any external care providers with jumpable applications (Nuts registry)
   This requires 2 calls.

   #. A call to the consent store, with the relevant Actor (care provider
      Identifier) and Subject (patient identifier) This results in a result with
      a list of consent records. The endpoint is described in the :ref:`nuts-consent-store-api`
   #. For each given custodian, query the Nuts Registry for the Nuts-SSO endpoint.
      The endpoint is described in the :ref:`nuts-registry-api`

#. The source application renders a SSO button
#. The User clicks the button
#. The source application collects the users identity using a login contract. If not already present, it lets the user sign one using IRMA
   To create an IRMA session, make a call to the Nuts Auth server as described in the :ref:`nuts-consent-auth-api`

#. The source application requests a session token providing user identity, patient BSN, Care Provider AGB.
   This is described in the :ref:`nuts-documentation-session-tokens`

#. The source application redirects the user to the jump url with the session token in the URL
   The jump URL was retrieved in a previous step from the registry.
#. The destination Authorization Server performs all the checks:

   #. Retrieve the session token from the access token: `rfc7662 <https://tools.ietf.org/html/rfc7662>`_ (This endpoint has yet to be implemented)
   #. Validate the identity from the session using the validate endpoint in Nuts Auth
   #. Check if there is consent for the subject, custodian and actor

#. The destination Authorization Server creates an internal URL and session and redirects the user
#. The destination application shows the page with the client context from the session token


Endpoints
=========

The an registry endpoint should use the following identifier:
::

    urn:oid:1.3.6.1.4.1.54851.1:nuts-sso

Jump
====

The user gets a status 303 or 303 with the jump sso url including the session token.

.. code-block:: http

  HTTP/1.1 302 Found
  Location: https://auth.destination-application.nl?session_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c


TODO
****

Determine which IRMA attributes are needed.

Do we need a separate Jump contract?

Error handling

Specify the contents of the session token
