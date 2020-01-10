# Nuts Registry

## High-level goals
- Let vendors register endpoints (FHIR, ...?) for their client organisations so that they can be looked up by other vendors(?)
  so that their (client) organisations can exchange data.
- Let organisations register their practitioners so that ... (why?)


## Processes
### Register vendor
Prerequisites:
- Corda node has been registered

Steps:
1. ...

### Process: register/update endpoint

Prerequisites:
- Vendor has been registered.
- Vendor administrator has been registered.
- Organisation has been registered as a client of the vendor.

Steps:
1. ...

## Considerations

### Migrating endpoints
When an endpoint is relocated (e.g. from https://my-org/foo/consent to https://my-org/bar/consent) nodes should know this in advance, otherwise XIS's might be calling old endpoints.
We can achieve this by adding 2 timestamp properties to the endpoint; 'validFrom' (mandatory) and 'validUntil' (optional). The calling XIS should always look for the 'youngest' valid endpoint which hasn't expired yet.

### Multi-tenancy
Since organisations might use multiple vendors, each vendor should only be able to mutate their own organisation endpoints.
In other words, if an organisation X uses vendor A and vendor B, vendor B shouldn't be able to mutate vendor A's endpoints for
 organisation X.

### Outstanding issues
- Why are Corda endpoints registered in the registry? They can also be read from the network map?
 If we register them in the network map, they should be kept in sync from there out.
- How do we keep the identity of the vendor administrator protected outside of the vendor? Does the administrator sign with his own certificate or the vendor's?
- What if 2 vendors define the same endpoint for the same organisation?

### Ideas
- New changes have to be accepted by 