---
tags: [Build, Subnets]
description: This tutorial demonstrates how to deploy a permissioned Subnet on Fuji Testnet.
sidebar_label: On Fuji Testnet 
pagination_label: Deploy a Permissioned Subnet on Fuji Testnet
sidebar_position: 1
---

# Deploy a Permissioned Subnet on Fuji Testnet

:::note

This document describes how to use the new Avalanche-CLI to deploy a Subnet on `Fuji`.

:::

After trying out a Subnet on a local box by following [this tutorial](/build/subnet/deploy/local-subnet.md),
next step is to try it out on `Fuji` Testnet.

In this article, it's shown how to do the following on `Fuji` Testnet.

- Create a Subnet.
- Deploy a virtual machine based on Subnet-EVM.
- Join a node to the newly created Subnet.
- Add a node as a validator to the Subnet.

All IDs in this article are for illustration purposes. They can be different in your own
run-through of this tutorial.

## Prerequisites

- 1+ nodes running and fully bootstrapped on `Fuji` Testnet. Check out the section
[Nodes](/nodes/README.md) on how to run a node and become a validator.
- [`Avalanche-CLI`](https://github.com/ava-labs/avalanche-cli) installed

## Virtual Machine

Avalanche can run multiple blockchains. Each blockchain is an instance of a
[Virtual Machine](/learn/avalanche/subnets-overview.md#virtual-machines), much like an object in
an object-oriented language is an instance of a class.
That's, the VM defines the behavior of the blockchain.

[Subnet-EVM](https://github.com/ava-labs/subnet-evm) is the VM that defines the Subnet Contract
Chains. Subnet-EVM is a simplified version of [Avalanche C-Chain](https://github.com/ava-labs/coreth).

This chain implements the Ethereum Virtual Machine and supports Solidity smart contracts as well as
most other Ethereum client features.

## Fuji Testnet

For this tutorial, it's recommended that you follow
[Run an Avalanche Node Manually](/nodes/run/node-manually.md#connect-to-fuji-testnet)
and this step below particularly to start your node on `Fuji`:

_To connect to the Fuji Testnet instead of the main net, use argument `--network-id=Fuji`_

Also it's worth pointing out that
[it only needs 1 AVAX to become a validator on the Fuji Testnet](/nodes/validate/what-is-staking.md#fuji-testnet)
and you can get the test token from the [faucet](https://faucet.avax.network/).

To get the NodeID of this `Fuji` node, call the following curl command to [info.getNodeID](/reference/avalanchego/info-api.md#infogetnodeid):

```text
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.getNodeID"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

The response should look something like:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "nodeID": "NodeID-5mb46qkSBj81k9g9e4VFjGGSbaaSLFRzD"
  },
  "id": 1
}
```

That portion that says, `NodeID-5mb46qkSBj81k9g9e4VFjGGSbaaSLFRzD` is the NodeID, the entire thing.
The user is going to need this ID in the later section when calling [addValidator](#add-a-validator).

:::info

With more data on `Fuji`, it may take a while to bootstrap `Fuji` Testnet from scratch.
You can use [State-Sync](/nodes/configure/chain-config-flags.md#state-sync-enabled-boolean)
to shorten the time for bootstrapping.

:::

## Avalanche-CLI

If not yet installed, install `Avalanche-CLI` following the tutorial at [Avalanche-CLI installation](/build/subnet/deploy/local-subnet.md#installation)

### Private Key

All commands which issue a transaction require either a private key loaded into the tool, or
a connected ledger device.

This tutorial focuses on stored key usage and leave ledger operation details for the `Mainnet`
deploy one, as `Mainnet` operations requires ledger usage, while for `Fuji` it's optional.

`Avalanche-CLI` supports the following key operations:

- create
- delete
- export
- list

:::warning

You should only use the private key created for this tutorial for testing operations on `Fuji` or
other testnets. Don't use this key on `Mainnet`. CLI is going to store the key on your file
system. Whoever gets access to that key is going to have access to all funds secured by that
private key. To deploy to `Mainnet`, follow [this tutorial](/build/subnet/deploy/mainnet-subnet.md).

:::

Run `create` if you don't have any private key available yet. You can create multiple named keys.
Each command requiring a key is going to therefore require the appropriate key name you want to use.

```bash
avalanche key create mytestkey
```

This is going to generate a new key named `mytestkey`. The command is going to then also print addresses
associated with the key:

<!-- markdownlint-disable MD013 -->

```bash
Generating new key...
Key created
+-----------+-------------------------------+-------------------------------------------------+---------------+
| KEY NAME  |             CHAIN             |                     ADDRESS                     |    NETWORK    |
+-----------+-------------------------------+-------------------------------------------------+---------------+
| mytestkey | C-Chain (Ethereum hex format) | 0x86BB07a534ADF43786ECA5Dd34A97e3F96927e4F      | All           |
+           +-------------------------------+-------------------------------------------------+---------------+
|           | P-Chain (Bech32 format)       | P-custom1a3azftqvygc4tlqsdvd82wks2u7nx85rg7v8ta | Local Network |
+           +                               +-------------------------------------------------+---------------+
|           |                               | P-fuji1a3azftqvygc4tlqsdvd82wks2u7nx85rhk6zqh   | Fuji          |
+-----------+-------------------------------+-------------------------------------------------+---------------+
```

<!-- markdownlint-enable MD013 -->

You may use the C-Chain address (`0x86BB07a534ADF43786ECA5Dd34A97e3F96927e4F`) to
fund your key from the [faucet](https://faucet.avax.network/). The command also prints P-Chain
addresses for both the default local network and `Fuji`. The latter
(`P-fuji1a3azftqvygc4tlqsdvd82wks2u7nx85rhk6zqh`) is the one needed for this tutorial.

The `delete` command of course deletes a private key:

```bash
avalanche key delete mytestkey
```

Be careful though to always have a key available for commands involving transactions.

The `export` command is going to **print your private key** in hex format to stdout.

```bash
avalanche key export mytestkey
21940fbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb5f0b
```

_this key is intentionally modified_.

You can also **import** a key by using the `--file` flag with a path argument and also providing a
name to it:

```bash
avalanche key create othertest --file /tmp/test.pk
Loading user key...
Key loaded
```

Finally, the `list` command is going to list all your keys in your system and their associated addresses
(CLI stores the keys in a special directory on your file system, tampering with the directory is
going to result in malfunction of the tool).

<!-- markdownlint-disable MD013 -->

```bash
avalanche key list
+-----------+-------------------------------+-------------------------------------------------+---------------+
| KEY NAME  |             CHAIN             |                     ADDRESS                     |    NETWORK    |
+-----------+-------------------------------+-------------------------------------------------+---------------+
| othertest | C-Chain (Ethereum hex format) | 0x36c83263e33f9e87BB98D3fEb54a01E35a3Fa735      | All           |
+           +-------------------------------+-------------------------------------------------+---------------+
|           | P-Chain (Bech32 format)       | P-custom1n5n4h99j3nx8hdrv50v8ll7aldm383nap6rh42 | Local Network |
+           +                               +-------------------------------------------------+---------------+
|           |                               | P-fuji1n5n4h99j3nx8hdrv50v8ll7aldm383na7j4j7q   | Fuji          |
+-----------+-------------------------------+-------------------------------------------------+---------------+
| mytestkey | C-Chain (Ethereum hex format) | 0x86BB07a534ADF43786ECA5Dd34A97e3F96927e4F      | All           |
+           +-------------------------------+-------------------------------------------------+---------------+
|           | P-Chain (Bech32 format)       | P-custom1a3azftqvygc4tlqsdvd82wks2u7nx85rg7v8ta | Local Network |
+           +                               +-------------------------------------------------+---------------+
|           |                               | P-fuji1a3azftqvygc4tlqsdvd82wks2u7nx85rhk6zqh   | Fuji          |
+-----------+-------------------------------+-------------------------------------------------+---------------+
```

<!-- markdownlint-enable MD013 -->

#### Funding the Key

:::danger

Do these steps only to follow this tutorial for `Fuji` addresses. To access the wallet for `Mainnet`,
the use of a ledger device is strongly recommended.

:::

1. A newly created key has no funds on it. Send funds via transfer to its correspondent addresses
if you already have funds on a different address, or get it from the faucet at
[`https://faucet.avax.network`](https://faucet.avax.network/) using your **C-Chain address**.

2. **Export** your key via the `avalanche key export` command, then paste the output when selecting
"Private key" while accessing the [web wallet](https://wallet.avax.network). (Private Key is the
first option on the [web wallet](https://wallet.avax.network)).
3. Move the test funds from the C-Chain to the P-Chain by clicking on the `Cross Chain` on the left
side of the web wallet (find more details on
[this tutorial](https://support.avax.network/en/articles/6169872-how-to-make-a-cross-chain-transfer-in-the-avalanche-wallet)).

After following these 3 steps, your test key should now have a balance on the P-Chain on `Fuji` Testnet.

## Create an EVM Subnet

Creating a Subnet with `Avalanche-CLI` for `Fuji` works the same way as with a
[local network](/build/subnet/deploy/local-subnet.md#create-a-custom-subnet-configuration). In fact,
the `create` commands only creates a specification of your Subnet on the local file system. 
Afterwards the
Subnet needs to be _deployed_. This allows to reuse configs, by creating the config with the
`create` command, then first deploying to a local network and successively to `Fuji` - and
eventually to `Mainnet`.

To create an EVM Subnet, run the `subnet create` command with a name of your choice:

```bash
avalanche subnet create testsubnet
```

This is going to start a series of prompts to customize your EVM Subnet to your needs. Most prompts have
some validation to reduce issues due to invalid input. The first prompt asks for the type of the
virtual machine (see [Virtual Machine](#virtual-machine)).

```bash
Use the arrow keys to navigate: ↓ ↑ → ←
? Choose your VM:
  ▸ SubnetEVM
    Custom
```

As you want to create an EVM Subnet, just accept the default `Subnet-EVM`.
Next, CLI asks for the ChainID. You should provide your own ID. Check
[chainlist.org](https://chainlist.org/) to see if the value you'd like is already in use.

```bash
✔ Subnet-EVM
creating subnet testsubnet
Enter your subnet's ChainId. It can be any positive integer.
ChainId: 3333
```

Now, provide a symbol of your choice for the token of this EVM:

```bash
Select a symbol for your subnet's native token
Token symbol: TST
```

At this point, CLI prompts the user for the fee structure of the Subnet, so that he can tune the fees
to the needs:

```bash
Use the arrow keys to navigate: ↓ ↑ → ←
? How would you like to set fees:
  ▸ Low disk use    / Low Throughput    1.5 mil gas/s (C-Chain's setting)
    Medium disk use / Medium Throughput 2 mil   gas/s
    High disk use   / High Throughput   5 mil   gas/s
    Customize fee config
    Go back to previous step
```

You can navigate with the arrow keys to select the suitable setting. Use
`Low disk use / Low Throughput 1.5 mil gas/s` for this tutorial.

The next question is about the airdrop:

```bash
✔ Low disk use    / Low Throughput    1.5 mil gas/s
Use the arrow keys to navigate: ↓ ↑ → ←
? How would you like to distribute funds:
  ▸ Airdrop 1 million tokens to the default address (do not use in production)
    Customize your airdrop
    Go back to previous step
```

You can accept the default -again, NOT for production-, or customize your airdrop. In the latter
case the wizard would continue. Assume the default here.

The final question is asking for precompiles. Precompiles are powerful customizations of your EVM.
Read about them at [precompiles](/build/subnet/upgrade/customize-a-subnet.md#precompiles).

```bash
✔ Airdrop 1 million tokens to the default address (do not use in production)
Use the arrow keys to navigate: ↓ ↑ → ←
? Advanced: Would you like to add a custom precompile to modify the EVM?:
  ▸ No
    Yes
    Go back to previous step
```

For this tutorial, assume the simple case of no additional precompile. This finalizes the
prompt sequence and the command exits:

```bash
✔ No
Successfully created genesis
```

It's possible to end the process with Ctrl-C at any time.

At this point, CLI creates the specification of the new Subnet on disk, but isn't deployed yet.

Print the specification to disk by running the `describe` command:

```bash
avalanche subnet describe testsubnet
 _____       _        _ _
|  __ \     | |      (_) |
| |  | | ___| |_ __ _ _| |___
| |  | |/ _ \ __/ _  | | / __|
| |__| |  __/ || (_| | | \__ \
|_____/ \___|\__\__,_|_|_|___/
+----------------------------+----------------------------------------------------+
|         PARAMETER          |                       VALUE                        |
+----------------------------+----------------------------------------------------+
| Subnet Name                | testsubnet                                         |
+----------------------------+----------------------------------------------------+
| ChainID                    | 3333                                               |
+----------------------------+----------------------------------------------------+
| Token Name                 | TST                                               |
+----------------------------+----------------------------------------------------+
| VM ID                      | tGBrM2SXkAdNsqzb3SaFZZWMNdzjjFEUKteheTa4dhUwnfQyu  |
+----------------------------+----------------------------------------------------+
| Fuji SubnetID              | XTK7AM2Pw5A4cCtQ3rTugqbeLCU9mVixML3YwwLYUJ4WXN2Kt  |
+----------------------------+----------------------------------------------------+
| Fuji BlockchainID          | 5ce2WhnyeMELzg9UtfpCDGNwRa2AzMzRhBWfTqmFuiXPWE4TR  |
+----------------------------+----------------------------------------------------+
| Local Network SubnetID     | 2CZP2ndbQnZxTzGuZjPrJAm5b4s2K2Bcjh8NqWoymi8NZMLYQk |
+----------------------------+----------------------------------------------------+
| Local Network BlockchainID | oaCmwvn8FDuv8QjeTozGpHeczk1Kv2651j2jhm6sR1nraGwVW  |
+----------------------------+----------------------------------------------------+

  _____              _____             __ _
 / ____|            / ____|           / _(_)
| |  __  __ _ ___  | |     ___  _ __ | |_ _  __ _
| | |_ |/ _  / __| | |    / _ \| '_ \|  _| |/ _  |
| |__| | (_| \__ \ | |___| (_) | | | | | | | (_| |
 \_____|\__,_|___/  \_____\___/|_| |_|_| |_|\__, |
                                             __/ |
                                            |___/
+--------------------------+-------------+
|      GAS PARAMETER       |    VALUE    |
+--------------------------+-------------+
| GasLimit                 |     15000000 |
+--------------------------+-------------+
| MinBaseFee               | 25000000000 |
+--------------------------+-------------+
| TargetGas (per 10s)      |    20000000 |
+--------------------------+-------------+
| BaseFeeChangeDenominator |          36 |
+--------------------------+-------------+
| MinBlockGasCost          |           0 |
+--------------------------+-------------+
| MaxBlockGasCost          |     1000000 |
+--------------------------+-------------+
| TargetBlockRate          |           2 |
+--------------------------+-------------+
| BlockGasCostStep         |      200000 |
+--------------------------+-------------+

          _         _
    /\   (_)       | |
   /  \   _ _ __ __| |_ __ ___  _ __
  / /\ \ | | '__/ _  | '__/ _ \| '_ \
 / ____ \| | | | (_| | | | (_) | |_) |
/_/    \_\_|_|  \__,_|_|  \___/| .__/
                               | |
                               |_|
+--------------------------------------------+------------------------+---------------------------+
|                  ADDRESS                   | AIRDROP AMOUNT (10^18) |   AIRDROP AMOUNT (WEI)    |
+--------------------------------------------+------------------------+---------------------------+
| 0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC |                1000000 | 1000000000000000000000000 |
+--------------------------------------------+------------------------+---------------------------+


  _____                                    _ _
 |  __ \                                  (_) |
 | |__) | __ ___  ___ ___  _ __ ___  _ __  _| | ___  ___
 |  ___/ '__/ _ \/ __/ _ \| '_   _ \| '_ \| | |/ _ \/ __|
 | |   | | |  __/ (_| (_) | | | | | | |_) | | |  __/\__ \
 |_|   |_|  \___|\___\___/|_| |_| |_| .__/|_|_|\___||___/
                                    | |
                                    |_|

No precompiles set
```

Also you can list the available Subnets:

```bash
avalanche subnet list
go run main.go subnet list
+-------------+-------------+----------+---------------------------------------------------+------------+-----------+
|   SUBNET    |    CHAIN    | CHAIN ID |                       VM ID                       |    TYPE    | FROM REPO |
+-------------+-------------+----------+---------------------------------------------------+------------+-----------+
| testsubnet  | testsubnet  |     3333 | tGBrM2SXkAdNsqzb3SaFZZWMNdzjjFEUKteheTa4dhUwnfQyu | Subnet-EVM | false     |
+-------------+-------------+----------+---------------------------------------------------+------------+-----------+
```

List deployed information:

```bash
avalanche subnet list --deployed
go run main.go subnet list --deployed
+-------------+-------------+---------------------------------------------------+---------------+-----------------------------------------------------------------+---------+
|   SUBNET    |    CHAIN    |                       VM ID                       | LOCAL NETWORK |                          FUJI (TESTNET)                         | MAINNET |
+-------------+-------------+---------------------------------------------------+---------------+-----------------------------------------------------------------+---------+
| testsubnet  | testsubnet  | tGBrM2SXkAdNsqzb3SaFZZWMNdzjjFEUKteheTa4dhUwnfQyu | Yes           | SubnetID: XTK7AM2Pw5A4cCtQ3rTugqbeLCU9mVixML3YwwLYUJ4WXN2Kt     | No      |
+             +             +                                                   +               +-----------------------------------------------------------------+---------+
|             |             |                                                   |               | BlockchainID: 5ce2WhnyeMELzg9UtfpCDGNwRa2AzMzRhBWfTqmFuiXPWE4TR | No      |
+-------------+-------------+---------------------------------------------------+---------------+-----------------------------------------------------------------+---------+

```

## Deploy the Subnet

To deploy the new Subnet, run

```bash
avalanche subnet deploy testsubnet
```

This is going to start a new prompt series.

```bash
Use the arrow keys to navigate: ↓ ↑ → ←
? Choose a network to deploy on:
  ▸ Local Network
    Fuji
    Mainnet
```

This tutorial is about deploying to `Fuji`, so navigate with the arrow keys to `Fuji` and hit enter.
The user is then asked to provide which private key to use for the deployment. The deployment basically
consists in running a
[createSubnet transaction](/reference/avalanchego/p-chain/api.md#platformcreatesubnet). Therefore the
key needs to have funds.

Also, this tutorial assumes that a node is up running, fully bootstrapped on `Fuji`, and runs
from the **same** box.

```bash
✔ Fuji
Deploying [testsubnet] to Fuji
Use the arrow keys to navigate: ↓ ↑ → ←
? Which private key should be used to issue the transaction?:
    test
  ▸ mytestkey
```

Subnets are currently permissioned only. Therefore, the process now requires the user to provide
_which keys can control the Subnet_. CLI prompts the user to provide one or more **P-Chain addresses**.
Only the keys corresponding to these addresses are going to be able to add or remove validators.
Make sure to provide **Fuji P-Chain** addresses -`P-Fuji....`-.

```bash
Configure which addresses may add new validators to the subnet.
These addresses are known as your control keys. You are going to also
set how many control keys are required to add a validator.
Use the arrow keys to navigate: ↓ ↑ → ←
? Set control keys:
  ▸ Add control key
    Done
    Cancel
```

Enter at `Add control key` and provide at least one key. You can enter multiple addresses, just use
 one here. When finishing, hit `Done`. (The address provided here is
intentionally invalid. The address has a checksum and the tool is going to make sure it's a valid address).

```bash
✔ Add control key
Enter P-Chain address (Ex: `P-...`): P-fuji1vaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaasz
Use the arrow keys to navigate: ↓ ↑ → ←
? Set control keys:
    Add control key
  ▸ Done
    Cancel
```

Finally, there is a need to define the threshold of how many keys to require for a change to be valid
-there is some input validation-. For example, if the is one control key, as preceding, just enter 1.
The threshold _could_ be arbitrary depending on the needs, for example 2 of 4 addresses,
1 of 3, 3 of 5, etc., but currently this tool only works if _the private key used here owns at least
one control key and the threshold is 1_.

```bash
✔ Enter required number of control key signatures to add a validator: 1
```

Here the wizard completes, and CLI attempts the transaction.

If the private key isn't funded or doesn't have enough funds, the error message is going to be:

```bash
Error: insufficient funds: provided UTXOs need 100000000 more units of asset "U8iRqJoiJm8xZHAacmvYyZVwqQx6uDNtQeP3CQ6fcgQk3JqnK"
```

If the private key has funds, but the **control key** is incorrect (not controlled by the private
key), the CLI is going to create the Subnet, but _not the blockchain_:

```bash
Subnet has been created with ID: 2EkPnvnDiLgudnf8NjtxaNcVFtdAAnUPvaoNBrc9WG5tNmmfaK. Now creating blockchain...
Error: insufficient authorization
```

Therefore the user needs to provide a control key which he has indeed control of, and then it succeeds.
The output (assuming the node is running on `localhost` and the API port set to standard `9650`)
is going to look something like this:

<!-- markdownlint-disable MD013 -->

```bash
Subnet has been created with ID: 2b175hLJhGdj3CzgXENso9CmwMgejaCQXhMFzBsm8hXbH2MF7H. Now creating blockchain...
Endpoint for blockchain "2XDnKyAEr1RhhWpTpMXqrjeejN23vETmDykVzkb4PrU1fQjewh" with VM ID "tGBrMADESojmu5Et9CpbGCrmVf9fiAJtZM5ZJ3YVDj5JTu2qw": http://127.0.0.1:9650/ext/bc/2XDnKyAEr1RhhWpTpMXqrjeejN23vETmDykVzkb4PrU1fQjewh/rpc
```

<!-- markdownlint-enable MD013 -->

Well done. You have just created your own Subnet with your own Subnet-EVM running on `Fuji`.

To get your new Subnet information, visit the
[Avalanche Subnet Explorer](https://subnets-test.avax.network/). The
search best works by blockchain ID, so in this example, enter `2XDnKyAEr1RhhWpTpMXqrjeejN23vETmDykVzkb4PrU1fQjewh`
into the search box and you should see your shiny new blockchain information.

## Request to Join a Subnet as a Validator

The new Subnet created in the previous steps doesn't have any dedicated validators yet. 
To request permission to validate a Subnet, the following steps are required:

:::info

Before a node can be a validator on a Subnet, the node is required to already be a validator on the
primary network, which means that your node has **fully bootstrapped**.

See [here](/nodes/validate/add-a-validator.md#add-a-validator-with-avalanche-wallet) on how to
become a validator.

:::

First, request permission to validate by running the `join` command along with the Subnet name:

```bash
avalanche subnet join testsubnet
```

Note: Running `join` does not guarantee that your node is a validator of the Subnet! The owner of 
the Subnet must approve your node to be a validator afterwards by calling `addValidator` as 
described in the next section.

When you call the `join` command, you are first prompted with the network selection:

```bash
Use the arrow keys to navigate: ↓ ↑ → ←
? Choose a network to validate on (this command only supports public networks):
  ▸ Fuji
    Mainnet
```

Next, there are two setup choices: Automatic and Manual configurations. As mentioned earlier,
"Automatic" is going to attempt at editing a config file and setting up your plugin directory, while
"Manual" is going to just print the required config to the screen. See what "Automatic" does:

```bash
✔ Automatic
✔ Path to your existing config file (or where it's going to be generated): config.json
```

Provide a path to a config file. If executing this command on the box where your
validator is running, then you could point this to the actually used config file, for example
`/etc/avalanchego/config.json` - just make sure the tool has **write** access to the file. Or you
could just copy the file later. In any case, the tool is going to either try to edit the existing file
specified by the given path, or create a new file. Again, set write permissions.

Next, provide the plugin directory. The beginning of this tutorial contains VMs description
[Virtual Machine](#virtual-machine). Each VM runs its own plugin, therefore AvalancheGo needs to
be able to access the correspondent plugin binary. As this is the `join` command, which doesn't
know yet about the plugin, there is a need to provide the directory where the plugin resides. Make
sure to provide the location for your case:

```bash
✔ Path to your avalanchego plugin dir (likely avalanchego/build/plugins): /home/user/go/src/github.com/ava-labs/avalanchego/build/plugins
```

The tool doesn't know where exactly it's located so it requires the full path. With the path given,
it's going to copy the VM binary to the provided location:

```shell
✔ Path to your avalanchego plugin dir (likely avalanchego/build/plugins): /home/user/go/src/github.com/ava-labs/avalanchego/build/plugins█
VM binary written to /home/user/go/src/github.com/ava-labs/avalanchego/build/plugins/tGBrMADESojmu5Et9CpbGCrmVf9fiAJtZM5ZJ3YVDj5JTu2qw
This is going to edit your existing config file. This edit is nondestructive,
but it's always good to have a backup.
Use the arrow keys to navigate: ↓ ↑ → ←
? Proceed?:
  ▸ Yes
    No
```

Hitting `Yes` is going to attempt at writing the config file:

<!-- markdownlint-disable MD013 -->

```shell
✔ Yes
The config file has been edited. To use it, make sure to start the node with the '--config-file' option, e.g.

./build/avalanchego --config-file config.json

(using your binary location). The node has to be restarted for the changes to take effect.
```

<!-- markdownlint-enable MD013 -->

It's **required to restart the node**.

If choosing "Manual" instead, the tool is going to just print _instructions_. The user is going to have
to follow these instructions and apply them to the node. Note that the IDs for the VM and Subnets is
going to be different in your case.

```bash
✔ Manual

To setup your node, you must do two things:

1. Add your VM binary to your node's plugin directory
2. Update your node config to start validating the subnet

To add the VM to your plugin directory, copy or scp from /tmp/tGBrMADESojmu5Et9CpbGCrmVf9fiAJtZM5ZJ3YVDj5JTu2qw

If you installed avalanchego manually, your plugin directory is likely
avalanchego/build/plugins.

If you start your node from the command line WITHOUT a config file (e.g. via command
line or systemd script), add the following flag to your node's startup command:

--track-subnets=2b175hLJhGdj3CzgXENso9CmwMgejaCQXhMFzBsm8hXbH2MF7H
(if the node already has a track-subnets config, append the new value by
comma-separating it).

For example:
./build/avalanchego --network-id=Fuji --track-subnets=2b175hLJhGdj3CzgXENso9CmwMgejaCQXhMFzBsm8hXbH2MF7H

If you start the node via a JSON config file, add this to your config file:
track-subnets: 2b175hLJhGdj3CzgXENso9CmwMgejaCQXhMFzBsm8hXbH2MF7H

TIP: Try this command with the --avalanchego-config flag pointing to your config file,
this tool is going to try to update the file automatically (make sure it can write to it).

After you update your config, you are going to need to restart your node for the changes to
take effect.
```

## Add a Validator

:::warning

If the `join` command isn't successfully completed before `addValidator` is completed, the Subnet
could experience degraded performance or even halt.

:::

Now that the node has joined the Subnet, a Subnet control key holder must call `addValidator` to 
grant the node permission to be a validator in your Subnet.

To whitelist a node as a recognized validator on the Subnet, run:

```bash
avalanche subnet addValidator testsubnet
```

As this operation involves a new
[transaction](/reference/avalanchego/p-chain/api.md#platformaddsubnetvalidator), you will need to specify
which private key to use:

```bash
Use the arrow keys to navigate: ↓ ↑ → ←
? Which private key should be used to issue the transaction?:
    test
  ▸ mytestkey
```

Choose `Fuji`:

```bash
Use the arrow keys to navigate: ↓ ↑ → ←
? Choose a network to deploy on. This command only supports Fuji currently.:
  ▸ Fuji
    Mainnet
```

Now use the **NodeID** of the new validator defined at the beginning of this tutorial. For best
results make sure the validator is running and synced.

```bash
What is the NodeID of the validator you'd like to whitelist?: NodeID-BFa1paAAAAAAAAAAAAAAAAAAAAQGjPhUy
```

-this ID is intentionally modified-

The next question requires a bit of thinking. A validator has a weight, which defines how often
consensus selects it for decision making. You should think ahead of how many validators you want
initially to identify a good value here. The range is 1 to 100, but the minimum for a Subnet without
any validators yet is 20. The structure is a bit described at
[addSubnetValidator](/reference/avalanchego/p-chain/api.md#platformaddsubnetvalidator) under the
`weight` section.

Just select 30 for this one:

```bash
Use the arrow keys to navigate: ↓ ↑ → ←
? What stake weight would you like to assign to the validator?:
    Default (20)
  ▸ Custom
```

```bash
✔ What stake weight would you like to assign to the validator?: 30
```

Then specify when the validator is going to start validating. The time must be in the future. Custom
option is going to require to enter a specific date in `YYYY-MM-DD HH:MM:SS` format. Just take the
default this time:

```bash
Use the arrow keys to navigate: ↓ ↑ → ←
? Start time:
  ▸ Start in one minute
    Custom
```

Finally, specify how long it's going to be validating:

```bash
✔ Start in one minute
Use the arrow keys to navigate: ↓ ↑ → ←
? How long should your validator validate for?:
  ▸ Until primary network validator expires
    Custom
```

If choosing `Custom` here, the user must enter a **duration**, which is a time span expressed in
hours. For example, could say `200 days = 24 \* 200 = 4800 hours`

```bash
✔ How long should this validator be validating? Enter a duration, e.g. 8760h: 4800h
```

CLI shows an actual date of when that's now:

```bash
? Your validator is going to finish staking by 2023-02-13 12:26:55:
  ▸ Yes
    No
```

Confirm if correct. At this point the prompt series is complete and CLI attempts the transaction:

```bash
NodeID: NodeID-BFa1padLXBj7VHa2JYvYGzcTBPQGjPhUy
Network: Fuji
Start time: 2022-07-28 12:26:55
End time: 2023-02-13 12:26:55
Weight: 30
Inputs complete, issuing transaction to add the provided validator information...
```

This might take a couple of seconds, and if successful, it's going to print:

```bash
Transaction successful, transaction ID :EhZh8PvQyqA9xggxn6EsdemXMnWKyy839NzEJ5DHExTBiXbjV
```

This means the node is now a validator on the given Subnet on `Fuji`!

## Subnet Export

This tool is most useful on the machine where a validator is or is going to be running. In order to
allow a VM to run on a different machine, you can export the configuration. Just need to provide a path
to where to export the data:

```bash
avalanche subnet export testsubnet
✔ Enter file path to write export data to: /tmp/testsubnet-export.dat
```

The file is in text format and you shouldn't change it. You can then use it to import the
configuration on a different machine.

## Subnet Import

To import a VM specification exported in the previous section, just issue the `import` command with
the path to the file after having copied the file over:

```bash
avalanche subnet import /tmp/testsubnet-export.dat
Subnet imported successfully
```

After this the whole Subnet configuration should be available on the target machine:

```bash
avalanche subnet list
+---------------+---------------+----------+-----------+----------+
|    SUBNET     |     CHAIN     | CHAIN ID |   TYPE    | DEPLOYED |
+---------------+---------------+----------+-----------+----------+
| testsubnet    | testsubnet    |     3333 | SubnetEVM | No       |
+---------------+---------------+----------+-----------+----------+
```

## Appendix

### Connect with Core

To connect Core (or MetaMask) with your blockchain on the new 
Subnet running on your local computer, 
you can add a new network on your Core wallet 
with the following values:

```text
- Network Name: testsubnet
- RPC URL: <http://127.0.0.1:9650/ext/bc/2XDnKyAEr1RhhWpTpMXqrjeejN23vETmDykVzkb4PrU1fQjewh/rpc>
- Chain ID: 3333
- Symbol: TST
```

:::note

Unless you deploy your Subnet on other nodes, you aren't going to be able to use other nodes,
including the public API server `https://api.avax-test.network/`, to connect to Core.

If you want to open up this node for others to access your Subnet, you should set it up properly
with `https//node-ip-address` instead of `http://127.0.0.1:9650`, however, it's out of scope for
this tutorial on how to do that.

:::
