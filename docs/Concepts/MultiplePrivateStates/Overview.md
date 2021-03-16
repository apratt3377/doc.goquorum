# Multiple Private States

In a typical network, each participant (tenant) uses its own GoQuorum and Tessera node. Tessera can
be configured to manage multiple key pairs which are owned by one tenant. This model is costly to
run and scale as more tenants join the network.

[Multiple Private States] allows multiple tenants to use the same GoQuorum node, with each tenant having it's own private state(s). Tenants can perform all operations (create/read/write) on any contract in their private state and a single tenant can have access to multiple private states. MPS allows for a similar user experience to a user running their own managed node.

The public state remains available publicly to all tenants and private states are logically segregated.

Sample diagram of tenants with multiple private states here and use of EAS

[JSON RPC security](../../HowTo/Use/JSON-RPC-API-Security.md) features are used to manage a users access to a private state. link to EAS section below

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

MPS uses the concept of Tessera Resident Groups to map tenants to private states. During Tessera startup, residentGroups are validated to check each tessera key is part of a single resident group. Any key not part of a configured residentGroup is added to the default "private" resident group.  During GoQuorum startup the residentGroups tessera Q2T API is invoked to retrieve all resident groups.  These details are kept in memory in quorum, so the Private State Manager is able to resolve these resident groups to the corresponding private state.

Sample residentGroup definition

name - used to identify the private state this group has access to 
members - public keys that have access to the PSI

View how to setup residentGroups in the Tessera config file here: link(Tessera Resident Groups howto)


## Enable multiple private states

### RPC Security Plugin

To enable mps, configure the [JSON RPC Security plugin](../../HowTo/Use/JSON-RPC-API-Security.md#configuration)


### GoQuorum

After configuring the plugin, isMPS should be set to true in the GoQuorum genesis.config

example of genesis config file...are there other ways to specify mps???

When genesis file is configured to run MPS, can run quorum with the plugin flag
```shell
geth <other parameters> \
    --plugins file:///<path>/<to>/plugins.json
```

In the command, `plugins.json` is the [plugin settings file](../../HowTo/Configure/Plugins.md) that
contains the [JSON RPC Security plugin definition](../../HowTo/Configure/Plugins.md#plugindefinition).
View the [security plugins documentation] for more information about how to configure the JSON RPC
Security plugin.
    
### Tessera

Multiple private states requires [Tessera] version `20.10.3` or later.

Without additional configuration, Tessera will automatically group all public keys it manages into a default 'private' resident group.

View link(how to configure tessera resident groups documentation) for more information about how to configure specific resident groups that map to multiple private states

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

## Enterprise Authorization Server

The authorization server has the ability to grant private state access to clients via a private state identifier
More details here about how this gets setup/works...possibly link to a howto page on how to set it up...provide an example like in the multitenancy howto doc

### Access Token Scope

The JSON RPC Security plugin enables the `geth` JSON RPC API server to be an OAuth2-compliant
resource server. A client must first obtain a pre-authenticated access token from an authorization
server, then present the access token (using an `Authorization` HTTP request header) when calling an
API. Calls to the quorum RPC API without an authenticated token are rejected.

Scope details here???

## Upgrade

In order to upgrade a GoQuorum node to an mps GoQuorum node, ...


## Backwards Compatibility

In the event that a user wants to upgrade the version of GoQuorum, but not use the MPS feature, the user does not need to do anything special.  GoQuorum will continue to operate in "legacy" mode on a single private state.

## MPS Apis

### quorum_privateStates

Returns the list of private states accessible to the user
On a standalone node it returns the complete list of private states available on the node
On a multitenant node it only returns the private state(s) the current user is authorized to access

### quorum_currentState

Returns the private state the user is operating on



<!--links-->
[Multi-tenancy]: ../../HowTo/Use/Multitenancy.md
[scope]: #access-token-scope
[scoped access tokens]: #access-token-scope
[pre-authenticated access tokens with the authorized scope]: #access-token-scope
[security plugins documentation]: ../../Reference/Plugins/security/For-Users.md#configuration
[Tessera]: https://docs.tessera.consensys.net
