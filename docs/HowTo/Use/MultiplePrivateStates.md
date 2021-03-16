# Multi-tenancy via MPS

## Prerequisites

* Set isMPS to true in Quorum genesis????
* Configure the [JSON RPC Security plugin](JSON-RPC-API-Security.md#configuration)
* Use [Tessera] version `20.10.3` or later and configure `residentGroups`

## Network Topology

A network can consist of multi-tenant nodes and single-tenant nodes. One or more independent
authorization servers can be used to protect multi-tenant nodes, however one multi-tenant node can
only be protected by one authorization server.

## Sample Network Setup

This section outlines an example of how multi-tenancy can be set up. A network operator must
configure [scope values] for each user in an authorization server, for each tenant.
This example network contains 4 nodes, 2 of which are multi-tenant nodes. The multi-tenant nodes are
`Node1` and `Node2`.

!!! note
    A node consists of GoQuorum client and Tessera Private Transaction Manager.

    We name Privacy Manager key pairs for easy referencing, for example: `J_K1` or `G_K1`. In
    reality, their values are the pubic keys used in `privateFor` and `privateFrom` fields.

Privacy Manager key pairs are allocated as follows:

* `Node1` manages `J_K1`, `J_K2`, `G_K1`, and `G_K3`
* `Node2` manages `G_K2` and `D_K1`.

Tenants are assigned to multi-tenant nodes as follows:

* `J Organization` owns `J_K1` and `J_K2`, and it's tenancy is on `Node1`
* `G Organization` owns `G_K1`, `G_K2`, and `G_K3`, and its tenancy is on `Node1` and `Node2`
* `D Organization` owns `D_K1`, and its tenancy is on `Node2`.

In practice, `J Organization`, `G Organization` and `D Organization` may decide to allocate keys to
their departments, therefore the security model could be as below:

* `J Organization` has:
    * `J Investment` owning `J_K1`
    * `J Settlement` owning `J_K2`
    * `J Audit` having READ access to contracts in which `J_K1` and `J_K2` are participants
* `G Organization` has:
    * `G Investment` owning `G_K1`
    * `G Settlement` owning `G_K2`
    * `G Research` owning `G_K3`
    * `G Audit` having READ access to contracts in which `G_K1`, `G_K2` and `G_K3` are participants
* `D Organization` has:
    * `D Investment` owning `D_K1`

Each authorization server has its own configuration steps and client onboarding process.
A network operator's responsibility is to implement the above security model in the authorization
server by defining [custom scopes](../../Concepts/Multitenancy/Overview.md#access-token-scope) and
granting them to target clients.

A custom scope representing __`J Investment` owning `J_K1`__,  
where `J_K1=8SjRHlUBe4hAmTk3KDeJ96RhN+s10xRrHDrxEi1O5W0=`  
would be:

```text
private://0x0/_/contracts?owned.eoa=0x0&from.tm=8SjRHlUBe4hAmTk3KDeJ96RhN%2bs10xRrHDrxEi1O5W0%3d
```

Custom scopes representing __`J Audit` having READ access to contracts in which `J_K1` and `J_K2` are participants__,  
where `J_K1=8SjRHlUBe4hAmTk3KDeJ96RhN+s10xRrHDrxEi1O5W0=`  
and `J_K2=2T7xkjblN568N1QmPeElTjoeoNT4tkWYOJYxSMDO5i0=`,  
would be:

```text
private://0x0/read/contracts?owned.eoa=0x0&from.tm=8SjRHlUBe4hAmTk3KDeJ96RhN%2bs10xRrHDrxEi1O5W0%3d&from.tm=2T7xkjblN568N1QmPeElTjoeoNT4tkWYOJYxSMDO5i0%3d
```

!!! important
    Clients must also be granted scopes which specify access to the JSON RPC APIs.

    Refer to the [JSON RPC Security plugin](../../Reference/Plugins/security/For-Users.md#oauth2-scopes)
    for more information.

In summary, to reflect the above security model, typical scopes being granted to `J Investment`
would be the following:

```text
rpc://eth_*
private://0x0/_/contracts?owned.eoa=0x0&from.tm=8SjRHlUBe4hAmTk3KDeJ96RhN%2bs10xRrHDrxEi1O5W0%3d
```

### Tessera Setup

In addition to configuring the Authorization Server, the Tessera config file must be updated to contain residentGroups.

In the above setup, the residentGroup configuration would be:

``` json
"residentGroups": [
 {
   "name": "PS1",
   "members": ["publicKey1", "publicKey2"],
   "description": "Private state 1"
 },
 {
   "name": "PS2",
   "members": ["publicKey3", "publicKey4"],
   "description": "Private State 2"
 }
]
```

During Tessera startup, residentGroups are validated to check that each key is part of a single resident group.
If a key is not configured to be a part of a residentGroup it is automatically added to the default "private" resident group.
Once a key is added to a residentGroup, it should remain in that group.

## Adding a new Tenant to MPS Node

Network Admin executes Tessera keygen to generate the new key
The Tessera config file must be updated to include the new key in a residentGroup.
Tessera needs to be restarted to load the new key. When Tessera starts, if the new key was generated but not added to a residentGroup it will be put in the default "private" residentGroup.
Updates to Authorization Server???

## Removing a Tenant from an MPS Node

It may be the case that a tenant on an MPS node wants to move to its own standalone node. What happens here????

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


<!--links-->
[scope values]: ../../Concepts/Multitenancy/Overview.md#access-token-scope
[Tessera]: https://docs.tessera.consensys.net/en/stable/
