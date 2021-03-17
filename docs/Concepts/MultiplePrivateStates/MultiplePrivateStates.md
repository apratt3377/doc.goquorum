# Multiple Private States

Multiple Private States is a feature that allows a GoQuorum node to have more than one private state. The private states are separated logically in the shared database.  This functionality lays the foundation for [supporting multiple tenants on a single node].

## Key Changes

### PSI

A private state is identified by a sequence of bytes, referred to as the PSI (Private State Identifier). The PSI is used to determine the specific private state a user can operate on.

### Trie of Private States

A Trie of Private States is introduced into GoQuorum to keep track of all private states managed by a node. The PSI is the key in the trie of Private States that maps to the root hash of the corresponding private state. At each block, all affected private states are updated, the trie of private states is updated with any new private state root hashes at their PSI, and a new root of the trie of private states is calculated and mapped to the public block hash.

### Private State Manager

The purpose of the Private State Manager in GoQuorum is to resolve the PSI based on input parameters.

#### Applying the Transaction

When executing a private transaction, it may be the case that the transaction is addressed to multiple locally managed parties and as a result it may be necessary to apply the transaction to multiple private states.  The Private State Manager resolves each managed party tessera public key to one locally managed private state identifier.

#### User invokes an RPC API

Any RPC API call must be accompanied by a state identifier or authorization token. From the token, the Private State Manager must be able to derive the private state the user is attempting to access.

### Tessera Resident Groups

MPS uses the concept of [Tessera Resident Groups] (is the group concept going to be added to the Tessera docs???) to map tenants to private states. 
During Tessera startup, residentGroups are validated to check each tessera key is part of a single resident group. Any key not part of a configured residentGroup is added to the default "private" resident group.

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

The above `residentGroups` defines 2 resident groups (each with 2 members) that map to 2 GoQuorum private states.

``` text
name - the PSI used to identify the private state the group has access to
members - the public keys that have access to private state
```

View how to [setup residentGroups in Tessera]

## Configuration Changes

### GoQuorum

`genesis.json` file has been modified to include `isMPS`.  This value should be set to `true` to enable MPS and `false` to operate as a standalone node. There can be a mix of MPS enabled nodes and standalone nodes in the network.

### Tessera

The new field `residentGroups` can be added to the Tessera config to define group access to specific private states.  View how to [setup residentGroups in Tessera] for further information.  If the node is running as a standalone node (`isMPS=false`), configuration of `residentGroups` is not necessary.

## Upgrading a Node to MPS

If a user would like to upgrade an existing GoQuorum node to an node with MPS functionality, there are additional steps to follow to make sure the database is compatible with MPS.  View the [steps for migration].

## Enable Multiple Private States

MPS requires [Tessera] version `20.10.3` or later.

For any given node the privacy manager (Tessera) is started first and for that reason we allow the Tessera node to be upgraded with MPS support ahead of the GoQuorum upgrade.  But when the GoQuorum node is upgraded and Geth is reinitialized with `isMPS=true`, the GoQuorum node will validate the version of Tessera running and will fail to start if Tessera is not running an upgraded version.  The GoQuorum node reports an appropriate error message in the console suggesting users to upgrade Tessera first.

If a node wants to upgrade it's Tessera to MPS release (or further) to have other features and fixes but not ready to upgrade GoQuorum, it can do so by running GoQuorum as usual. GoQuorum will continue to function as a standalone node.

If both Tessera and GoQuorum are upgraded but not configured to run MPS, the node will continue to function as a standalone node.

## Tessera Q2T Communication Changes

MPS introduces a new QT2 endpoint `/residentGroups` to return the resident groups defined in Tessera.
This endpoint is invoked at GoQuorum startup to retrieve all resident groups.  These details are kept in memory in quorum, so the Private State Manager is able to resolve these resident groups to the corresponding private state.

## Accessing a Private State

Users will need to specify the private state they wish to operate on. For backwards compatibility, if a user connects without specifying the private state, the default "private" identifier will be used.  If a "private" state is not configured, the user will only be operating on an empty read only private state.

In order to specify a private state to operate on the user has 3 options:

### Precedence

1. privacyGroupID in specific API calls (to be implemented)
2. HTTP Header
3. URL Parameter

### URL Parameter

PSI query param can be added to the API URI:
```
geth attach http://localhost:22000/?PSI=PS1
```
### HTTP Header

Every RPC request must have an HTTP Header "PSI" attached that specifies the private state to use

### IPC and inproc connections

Prepend the PSI to the ID field of the jsonrpcMessage


<!--links-->
[supporting multiple tenants on a single node]: Multitenancy.md
[setup residentGroups in Tessera]: ../../HowTo/Use/MultitenancyMPS.md#Tessera-Setup
[steps for migration]: ../../HowtoUse/MultiplePrivateStates.md#Migration-Guides
[Tessera]: https://docs.tessera.consensys.net
[Tessera Resident Groups]: https://docs.tessera.consensys.net
