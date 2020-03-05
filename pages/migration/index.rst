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

0.13
^^^^

In 0.13 the Node *identity* configuration parameter is introduced, which identifies the vendor operating the Node.
It is used to determine which registry entries it owns an should manage. While this is a mandatory parameter,
operators should configure it in either nuts.yaml or through environment variables. To learn how to configure this parameter
please refer to :ref:`nuts-go-config`.
