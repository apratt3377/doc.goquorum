# Multiple Private States

## Migration Guides

### GoQuorum

MPS introduces improvements to how GoQuorum handles the private state, so upgrading requires re-syncing a node after Tessera is upgraded.

In the future, we will provide a migration tool that will assist with updating the GoQuorum database to be MPS compatible.

#### Backwards Compatibility

In the event that a user wants to upgrade the version of GoQuorum, but not use the MPS feature, the user does not need to do anything special.  GoQuorum will continue to operate in "legacy" mode on a single private state.

### Tessera

Tenants own one or more Privacy Manager key pairs. Public keys are used to address private transactions.
Please refer to [Tessera keys configuration documentation](https://docs.tessera.consensys.net/en/stable/HowTo/Configure/Keys/)
for more information about how Tessera manages multiple key pairs.

#### MPS node upgrade

Tessera will need to be rebuilt from the privacy managers of the standalone nodes it will now support. All transactions from the privacy managers will need to be merged into the new Tessera storage. Provide specific details to how this is achieved????

The Tessera configuration file needs to be updated to contain the relevant residentGroups. The residentGroups should be configured to provide an experience equivalent to when the tenants were running as standalone nodes.

Example residentGroup configuration scenario:

#### Standalone node upgrade

It may be the case that a tenant of a node wants to upgrade their node to support MPS but continue running as the only tenant.

In this case, the Tenant's Tessera version will need to be upgraded. No residentGroups will need to be added to the configuration, since there is only one Tenant on the node. All of the Tenant's public keys will be automatically added to the default "private" residentGroup upon Tessera startup.

## MPS APIs

### quorum_privateStates

Returns the list of private states accessible to the user
On a standalone node it returns the complete list of private states available on the node
On a multitenant node it only returns the private state(s) the current user is authorized to access

### quorum_currentState

Returns the private state the user is operating on
