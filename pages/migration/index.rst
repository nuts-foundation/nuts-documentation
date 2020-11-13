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

0.15
^^^^
Starting 0.15 the node will default to vendor CA certificates issued by the Nuts Network Authority instead of self-signing
them. See :ref:`register-vendor-label` how to register (or update) your vendor. There is backwards compatibility by supporting
self-signed certificates issued before version 0.15.

In 0.15 the Nuts Network is introduced, an experimental P2P network decentralized document transport layer which is
intended as a replacement candidate for Nuts Registry data on Github and Consents over Corda.

**Manual action required**

While the Nuts Network engine is enabled by default it needs configuration to function optimally:

1. Configure bootstrap node(s) using which peer nodes can be discovered; since there's no dedicated bootstrap node
   (at time of writing) it has to be configured with the Nuts Network address of one (or more) of the node's peer vendor nodes.
   This is done using the `NUTS_NETWORK_BOOTSTRAPNODES` which takes a space-separated list of nodes
   (e.g. `vendor-b:5555 vendor-c:9876`) to initially connect to.
2. Configure `NUTS_NETWORK_GRPCADDR` which is the local interface address gRPC (which powers Nuts Network) will bind on.
   Defaults to `:5555` but should be changed if applicable. This interface should be publicly reachable (but can be behind
   a TLS forwarding proxy). TLS termination for incoming connections using a proxy in front of the node is currently not possible.
3. Configure `NUTS_NETWORK_PUBLICADDR` which is the network address (`host:port`) which will be advertised on the network
   so that your node can be discovered by other nodes you're not directly connected to. This will typically be used to
   support resolving by public DNS entries (e.g. `nuts.vendor-A.nl:5555`), when the Nuts node is behind a proxy/load balancer
   or NAT.

Refer to :ref:`nuts-network-configuration` for configuration guidelines for your particular infrastructure layout.

Notes for v0.15.2: fixes a bug that causes old registry events that are received through the new experimental Nuts network
to have a zeroed timestamp. It removes invalid documents (zeroed timestamp) from the local storage and disallows adding
new documents with a zeroed timestamp.

0.14
^^^^
Starting version 0.14 vendors and organizations will have an X.509 certificate (encoded in the JWK) associated with their key pairs.
These certificates are used to identify the holder of the key pair used to sign events, which is also introduced in this version.

**Manual action required**: This migration can be performed by the Nuts node; use the newly introduced `registry verify` command with the `--fix` flag
to generate key pairs (if necessary), issue certificates and sign events. Don't forget to publish these changes to the central registry.

0.13
^^^^

In 0.13 the Node *identity* configuration parameter is introduced, which identifies the vendor operating the Node.
It is used to determine which registry entries it owns an should manage.

**Manual action required**: While this is a mandatory parameter, operators should configure it in either nuts.yaml or through environment variables. To learn how to configure this parameter
please refer to :ref:`nuts-go-config`.
