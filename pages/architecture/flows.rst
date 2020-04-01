.. _nuts-documentation-architecture-flows:

Application flows
#################

Application flows are the main flows of interaction between vendor application software and the different parts of the Nuts nodes.
The paragraphs below will explain what happens internally and why things work as they do. There are two main flows: the consent flow and the data flow.

Registering consent
*******************

The diagram below shows the flow between two parties. The *actor* is the party that wants to request some data about a patient and the *custodian* is the care organization holding the data.
Both parties have their own information system (*XIS*) and both are running a Nuts node within their infrastructure.
The Nuts registry has been omitted from the flow, but is used internally by the node for translating an organization identifier to endpoint.

.. raw:: html
    :file: ../../_static/images/nuts_consent_app_flow.svg

The flow starts at the top-right of the diagram (yes we like to be different) by a user registering consent via a UI.
The XIS is responsible for contacting the Nuts node via the :ref:`nuts-consent-logic-api` which starts the :ref:`Create Consent Branch` flow.
The result is a handle to the transaction which can be used to follow the state of the transaction.
It's unwise to make this a synchronous process since there's no limit on how long the transaction can take.

All nodes that are involved in the transaction must validate the transaction. The transaction must be validated technically (syntax, references, timestamps) and contextually.
Each node must validate that the `BSN` indeed belongs to a patient registered in the local XIS (due to legislation, ID card check and such) or will be in the near future.
If a patient is not in care at one of the nodes, the transaction will be cancelled. An `ack` by a node is represented as a :ref:`Sign Consent Branch` flow in Corda.

.. note::

    When a node starts too many invalid transactions, it might be an indication that the specific node is misbehaving and must be isolated in the network.
    A symptom might be a lot of negative responses to the `patient_in_care()` call.

Transaction validation by a node is done by adding a signature. In the diagram this is shown by the `sign()` call and `Sig` abstraction.
The initiating node does this as well, but most likely does this in an automated way.
The other nodes can choose to do it automatically by searching for a patient (by bsn) in their own records or manually validate it through an UI.
For most of the implicit consents, a patient record will not be available yet at the *XIS actor* part.
In that case the validation will most likely happen through an UI where the consent request can be linked to an out-of-band communication effort.

The `ConsentBranch` synchronization in the flow diagram is handled by Corda.

When all nodes have validated the transaction, the initiating Nuts node will finalize the transaction (`final()`) by calling the :ref:`Merge Branch` flow.
A result will be a synchronized state at all involved nodes. The Nuts node can emit an event for the XIS to notify users or update some state.

Data retrieval
**************

.. warning::

    The security for the data request is currently in flux. If the *works-for* relation has to be proven as well, this will no longer fit in http headers.
    Also when additional context like a patient identifier or a specific task number is required at the custodian end, you don't want the token to be created at request time.
    Therefore an access token will be used. The token can be requested by an out-of-band call, similar to Open ID Connect, Saml and OAuth2.
    This documentation will be updated when available.

The diagram below shows the flow for retrieving data. Most important to note is that the Nuts nodes are not responsible for the data exchange.

.. raw:: html
    :file: ../../_static/images/nuts_auth_app_flow.svg

An actor requests data from another system from the XIS it's working in. The XIS needs the identity of the actor before the request can be made.
The Nuts node has convenience services for generating a QR-code which can be scanned by the IRMA app. The resulting signature/token can be used in the data call.
This can be done by calling the correct :ref:`nuts-auth-api` call. Specification on the IRMA signatures can be found at :ref:`nuts-documentation-login-contracts`.

Given the identity of the current user and the context of the current care organization, the Nuts node can be used to query available consent by calling the :ref:`nuts-consent-store-api`.
This can be done with or without a patient context. The result will be a class of data that can be retrieved for custodians. The translation of class to resources is future work...

Given a custodian from the resulting consent API call, the technical endpoint can be retrieved from the :ref:`nuts-registry-api`.

Now all requirements are met for doing the data request (endpoint, user identity, actor identity, XIS identity, patient identity, custodian identity).

The data request that arrives at the custodian XIS endpoint will have to be validated in the reverse order is sort of the same manner:

- is it a secure connection?
- are the given identities valid?
- has consent been given?

The later two can be checked by two API calls on the Nuts node. The return values are nothing more than a yes/no response. More details can be found on the API pages:
:ref:`nuts-consent-store-api`, :ref:`nuts-auth-api` and how to implement it: :ref:`nuts-documentation-authenticate`.
The two-way TLS connection will be established with vendor specific certificates coming from a CA specified in :ref:`nuts-documentation-architecture-certificates`.

.. note::

    Some more references to other pages to add when they come available.
