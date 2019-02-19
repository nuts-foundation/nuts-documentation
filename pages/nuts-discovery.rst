######################
Nuts Discovery Service
######################

tl;dr; to the repository documentation: :ref:`nuts-discovery`

The *Nuts Discovery Service* is the Nuts implementation of the Network map service described by *Corda*. The *Corda* specific documentation can be found at https://docs.corda.net/network-map.html. The reason for implementing the network map as a service and not just distributing node information via other means is that this greatly simplifies development, puts the control of the root CA at the right place and creates a bridge to the *Nuts registry*. When a node registers with the discovery service, the service can also add the node to the registry. This will enable node administrators to link care providers to their Nuts node entry in the registry.

******************
Modes of operation
******************

Currently the *Nuts Discovery Service* can only be started in development mode. In development mode all requests are auto-approved and the requesting node will be added to the network.
