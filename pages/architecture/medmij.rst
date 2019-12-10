Medmij authentication flow using Irma
#####################################

.. warning::

    the proposal below is just that, a proposal. A prototype still has to be built.

The image below is a first attempt to solve the Medmij authentication flow using Irma instead of Digid. This is strictly a proposal and is being discussed with Medmij.

.. figure:: /_static/images/nuts-medmij-irma-auth-flow.png
    :width: 600px
    :align: center
    :alt: Nuts Medmij auth flow
    :figclass: align-center

    Nuts Medmij auth flow

The flow above does two things: register the PGO user at a care organisation, this will link the BSN with a unique e-mail.
Once registered, the second flow demonstrates how an Irma signature can be used to retrieve data from that care organisation.
The fact that signatures are used is important because the information within the signature text is used to limit rights and to restrict the use of the signature between the PGO and the care organisation (linked to TLS connection).
The main idea is that a unique attribute with a low security level combined with other attributes with a high security level creates a set that is both unique and secure.