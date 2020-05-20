Migration
#########

.. toctree::
    :maxdepth: 1
    :caption: Contents:
    :hidden:
    :glob:

    *

This chapter describes which migrations are performed when upgrading your Nuts node. Most of them will be automatic, but sometimes
manual action is required.

0.14
^^^^
Starting version 0.14 vendors and organizations will have an X.509 certificate (encoded in the JWK) associated with their key pairs.
These certificates are used to identify the holder of the key pair used to sign events, which is also introduced in this version.

This migration can be performed by the Nuts node automatically; use the newly introduced `registry verify` command with the `--fix` flag
to generate key pairs (if necessary), issue certificates and sign events. Don't forget to publish these changes to the central registry.

0.13
^^^^

In 0.13 the Node *identity* configuration parameter is introduced, which identifies the vendor operating the Node.
It is used to determine which registry entries it owns an should manage. While this is a mandatory parameter,
operators should configure it in either nuts.yaml or through environment variables. To learn how to configure this parameter
please refer to :ref:`nuts-go-config`.
