This repository is **deprecated and no longer maintained**. To get a high-level overview of how Transfer Gateway works, head over to the [Transfer Gateway Documentation Page](https://loomx.io/developers/docs/en/extdev-transfer-gateway.html). Next, make sure to check our [Transfer Gateway Testnet Tutorial](https://loomx.io/developers/docs/en/extdev-transfer-gateway.html) and this step-by-step [video walk-through](https://www.youtube.com/watch?v=e1dJTfDnPPE&t=161s).

---

# Token Gateway Example

This project is built using the [loom-js][] Transfer Gateway API, and demonstrates how to transfer
ERC20 tokens between Ethereum and Loom DAppChains via a browser frontend built with React.

## Interface in action

![](https://uc348d71394e7eb6263b22d38ccb.previews.dropboxusercontent.com/p/orig/AAKPvjgnGhOcy-e53PADMffBK9BC_N_Yy6RHC80V3EbhOfKF6see8hO4uzCE07EKwHruLfY8pbHS19a2_FZuXeNOF9rOvPLLCtGMT5Kcvk9xQjcmr_HqbxwtzQEp4z39W5e1qFns-KX3iACtXJSwWR1sEI-Kqp-SenAb6LEEDODvu68RhLuiUgtv25DblzhmK82Qs3vshmRITN0Nz9qBXhZL/p.gif?size=2048x1536&size_mode=3)

## Quick Start

### Requirements

- node >= 8
- wget (used to download Loom DAppChain binary)
- nc (used to check required ports aren't in use)
- [MetaMask][] browser extension

### Install

After cloning this repository install all the necessary dependencies use the `transfer_gateway`
shell script to automatically download & install all the required dependencies:

```bash
./transfer_gateway setup
```

### Running

After everything has been installed use the `transfer_gateway` script to spin up all the required
services:

```bash
./transfer_gateway start
```

### Stop

When you're done playing around with the example you should stop all the running services using:

```bash
./transfer_gateway stop
```

### Status

To check which services are currently running:

```bash
./transfer_gateway status
```

### Cleanup

If you want to start with a clean slate you can remove all the `node_modules` directories, and the
loom binary by running:

```bash
./transfer_gateway cleanup
```

### Web Browser

After `./transfer_gateway start` and starts the necessary services you can access the web frontend
at `localhost:8080`.

### MetaMask

This example requires [MetaMask][] to be installed, you must also set MetaMask to use the
`Private Network` at `localhost:8545`.

You will also need to [import the private key](https://consensys.zendesk.com/hc/en-us/articles/360004176551-Importing-Accounts-New-UI-) `0xbb63b692f9d8f21f0b978b596dc2b8611899f053d68aec6c1c20d1df4f5b6ee2` generated by Ganache (account `0x5194b63f10691e46635b27925100cfc0a5ceca62`) into MetaMask.

#### Troubleshooting

- If you stop the example and start it up again you may encounter a MetaMask error like this:
  > "rpc error with payload {…nonce. account has nonce of: 0 tx has nonce of: 5"

> This happens because MetaMask keeps track of the last nonce that was used to send a transaction with
> a particular account, and when all the chains are reset the nonce needs to be reset back to zero.
> You can force MetaMask to reset the nonce by [reseting the imported account](https://consensys.zendesk.com/hc/en-us/articles/360004177531-Resetting-an-Account-New-UI-).

- Restart the entire service can help to clean cache also `./transfer_gateway restart`, after restart you should clean MetaMask cache.

- Sometimes when run `./transfer_gateway start` and a message like `Ganache port 8545 is already in use` appears it's because the service `ganache` is running (obvious), however if you never ran `./transfer_gateway start` then it's because another agent started `ganache` and this example starts it self `ganache` version.

- In order to `stop all` services (used by token-gateway-example) you should run `./transfer_gateway stop`, however if services like `ganache` or `webpack` didn't halt then you need to stop then by the process id or `pid`.

## Example "in a nutshell"

The `token-gateway-example` directory contains five sub directories:

```bash
├── dappchain
├── truffle-ethereum
├── truffle-dappchain
├── transfer-gateway-scripts
└── webclient
```

In the following sections we'll explain what's in each of these directories.

> NOTE: In this doc we use the term Mainnet to refer to an Ethereum-like network, as a matter of
> convenience. In this example the only Ethereum network we actually interact with is Ganache, but
> you could also hook everything up to use the Ethereum Mainnet, or the Rinkeby Testnet by changing
> a few configuration settings.

### 📁 dappchain

This directory contains the `loom` binary (after setup), DAppChain config files, and some key pairs.

> The keys have been generated specifically for this example, don't reuse them anywhere else.
> If you wish to use this project as a starting point for your own please don't forget to replace
> all the keys with your own.

Let's take a closer look at `loom.yaml`, this is a configuration file that can be used to configure
both the DAppChain node and the Transfer Gateway Oracle. You can find documentation on the various
settings in this config file in the [Loom SDK docs](https://loomx.io/developers/docs/en/loom-yaml.html),
but we'll highlight some of the settings here that are relevant to this example.

#### loom.yaml

```yaml
# This is a unique identifier for the DAppChain used in this example
ChainID: 'default'
# This enables the latest version of the contract registry maintained by the DAppChain, which is
# required in order for the Transfer Gateway to be able verify DAppChain contract ownership.
# This setting must not be changed after the DAppChain starts for the first time!
RegistryVersion: 2
# Transfer Gateway Settings
TransferGateway:
  # Enables the Gateway Go contract (DAppChain Gateway), must be the same on all nodes.
  ContractEnabled: true

  # Enables the in-process Gateway Oracle.
  OracleEnabled: true

  # Address of the Ganache network where the Mainnet Solidity contracts are deployed
  # (take a look at the truffle-ethereum directory).
  EthereumURI: 'http://127.0.0.1:8545'

  # Address of the Mainnet Gateway contract deployed to the Ganache network above.
  MainnetContractHexAddress: '0xf5cad0db6415a71a5bc67403c87b56b629b4ddaa'

  # Private key that will be used by the Gateway Oracle to sign pending withdrawals.
  MainnetPrivateKeyPath: 'oracle_eth_priv.key'

  # Private key that will be used by the Gateway Oracle to send txs to the DAppChain Gateway.
  DAppChainPrivateKeyPath: 'oracle_priv.key'

  # Address of DAppChain node the Gateway Oracle should interact with.
  DAppChainReadURI: 'http://localhost:46658/query'
  DAppChainWriteURI: 'http://localhost:46658/rpc'

  # These control how frequently the Gateway Oracle will poll the DAppChain and Ganache.
  DAppChainPollInterval: 1 # seconds
  MainnetPollInterval: 1 # seconds

  # Number of seconds to wait before starting the Gateway Oracle, this allows the DAppChain node
  # to spin up before the Gateway Oracle tries to connect to it.
  OracleStartupDelay: 5

  # Number of seconds the Gateway Oracle should wait between reconnection attempts if it encounters
  # a network error of some kind.
  OracleReconnectInterval: 5
```

Now let's take a look at the `genesis.json` file (or `genesis.example.json` if you haven't setup the
example yet), this file specifies which Go contracts should be deployed to the DAppChain when the
DAppChain starts up for the first time. You can read more about the `coin` and `dpos` contracts in the
[Built-in Contracts docs][], but these two contracts are not relevant to this particular example.
The two contracts we're interested in are the built-in `addressmapper`, and `gateway` contracts.

#### genesis.json

```javascript
{
    "contracts": [
        {
            "vm": "plugin",
            "format": "plugin",
            "name": "addressmapper",
            "location": "addressmapper:0.1.0",
            "init": null
        },
        {
            "vm": "plugin",
            "format": "plugin",
            "name": "gateway",
            "location": "gateway:0.1.0",
            "init": {
                "owner": {
                    "chain_id": "default",
                    "local": "c/IFoEFkm4+D3wdqLmFU9F3t3Sk="
                },
                "oracles": [
                    {
                        "chain_id": "default",
                        "local": "22nuyPPZ53/qAqFnhwD2EpNu9ss="
                    }
                ]
            }
        }
    ]
}
```

The Address Mapper contract is responsible for mapping owned DAppChain accounts to Mainnet accounts,
and vice versa, you can read more about its purpose in the [Transfer Gateway docs][].

The Gateway contract is responsible for contract mapping, and token transfers to and from the
DAppChain. It relies on the Gateway Oracle (though there may be more than one) to transfer relevant
data between the DAppChain and Mainnet.

### 📁 truffle-ethereum

This directory contains the contracts that are deployed to Mainnet, primarily the `Gateway` contract
(Mainnet Gateway), and the `GameToken` ERC20 token contract.

The `GameToken` ERC20 contract implements a convenient function for depositing tokens into the
`Gateway` contract:

```solidity
function depositToGateway(uint amount) public {
  safeTransferFrom(msg.sender, gateway, amount);
}
```

The `Gateway` contract implements the `withdrawERC20` function, which should be called by a user
who wishes to withdraw a token from the Mainnet Gateway back to their own Mainnet account.

> NOTE: The Mainnet Gateway will only allow a user to withdraw a token if they can provide proof
> that it's no longer in circulation on the DAppChain, you can read more about it in the [Transfer Gateway docs][].

### 📁 truffle-dappchain

This directory contains the `GameTokenDAppChain` ERC20 contract that's deployed to the DAppChain,
this contract will keep track of any tokens sent from the `GameToken` ERC20 contract on Mainnet
to the DAppChain via the Transfer Gateway.

When a user deposits their `GameToken` tokens into the Mainnet Gateway contract the DAppChain
Gateway contract will `mint` the corresponding tokens in the `GameTokenDAppChain` contract.

Note that the DAppChain Gateway must be authorized to mint tokens in the `GameTokenDAppChain`
contract, otherwise token transfers from Mainnet to the DAppChain won't work.

```solidity
function mint(uint256 _uid) public {
  require(msg.sender == gateway);
  _mint(gateway, _uid);
}
```

### 📁 transfer-gateway-scripts

This directory contains a single `index.js` script file which is responsible for mapping the
`GameToken` ERC20 contract on Mainnet to the `GameTokenDAppChain` ERC20 contract on the
DAppChain. Without this contract mapping the Transfer Gateway won't know what to do when it receives
ERC20 tokens from either contract. You can read more about contract mapping in the
[Transfer Gateway docs][].

### 📁 webclient

This directory contain the web frontend, which requires `MetaMask` to be installed in a compatible
browser (Chrome / Firefox). The web frontend can be reached at `http://localhost:8080` after running
the `./transfer_gateway start`.

The frontend consists of four pages:

- `Home` - where you must map your Mainnet account to a DAppChain account by signing a message via
  `MetaMask` with your Mainnet account. This must be done before attempting to transfer
  any tokens.
- `Ethereum Account` - where you can see the tokens currently owned by you on Mainnet (yeah we just
  gave 5 tokens to you 😉).
- `Ethereum Gateway` - where you can see the tokens in the Mainnet Gateway that are waiting to be
  withdrawn by you to your Mainnet account.
- `DAppChain Account` - where you can see the tokens that have been transferred to the DAppChain from
  your Mainnet account.

## Loom Network

[https://loomx.io](https://loomx.io)

## License

BSD 3-Clause License

[loom-js]: https://github.com/loomnetwork/loom-js
[metamask]: https://metamask.io/
[built-in contracts docs]: https://loomx.io/developers/docs/en/builtin.html
[transfer gateway docs]: https://loomx.io/developers/docs/en/transfer-gateway.html
