Let users authenticate themselves to the Nuts network
-----------------------------------------------------

The Nuts network is based upon encryption instead of trust. This means that instead
of trusting all parties in the network to provide correct data, we ensure all data
is cryptographically verifiable. This includes the identity of people requesting
data of patients.

Therefor in order to make a request for data by another party, a user should provide
identity information that everyone in the network can verify. For that we use the
`IRMA <https://irma.app/docs/>`_ framework developed by the `privacy by design
foundation <https://privacybydesign.foundation/>`_ (a spin off of the Radboud university).

In this tutorial we will show you how to include a user interface inside your
application were users can authenticate themselves using IRMA.
The end result will look like this:

.. figure:: ../../_static/images/irma_flow.gif
    :width: 400px
    :align: center
    :alt: Nuts IRMA login flow
    :figclass: align-center

    Nuts IRMA login flow

+-------------------------------------------------------------------------------------+-----------------------------------------------------------------+
| Useful links                                                                        | Description                                                     |
+=====================================================================================+=================================================================+
| `IRMA web frontend <https://github.com/nuts-foundation/irma-web-frontend>`_         | Repository of the Nuts irma styleguide                          |
+-------------------------------------------------------------------------------------+-----------------------------------------------------------------+
| `IRMA styleguide <https://nuts-foundation.github.io/irma-web-frontend/index.html>`_ | Styleguide on how to embed the IRMA screens in your application |
+-------------------------------------------------------------------------------------+-----------------------------------------------------------------+
| `Nuts auth-js <https://www.npmjs.com/package/@nuts-foundation/auth>`_               | Package for easy nuts-auth irma flow integration                |
+-------------------------------------------------------------------------------------+-----------------------------------------------------------------+

Please wait while our automatic tutorial generator enjoys his coffee/weekend. We will expand this section soon.
