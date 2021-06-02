
# Setting Up a Genesis Osmosis Validator

Thank you for becoming a genesis validator on Osmosis!  This guide will provide instructions on setting up
a node, submitting a gentx, and other tasks needed to participate in the launch of the Osmosis mainnet.

The primary point of communication for the genesis process and future updates will be the #validators
channel on the [Osmosis Discord](https://discord.gg/tPayMvaTT3). This channel is private by default in order
to keep it free of spam and unnecessary noise.  To join the channel, please send a message to @Meow#6669
to add yourself and any team members.

Some important notes on joining as a genesis validator:

1. We highly recommend only experienced validators who have run on past Cosmos SDK chains and participated have participated in a genesis ceremony before become genesis validators at genesis.
2. All Osmosis validators should be expected to be ready to participate active operators of the network. As explained in the [Osmosis: A Hub AMM](https://medium.com/osmosis/osmosis-a-hub-amm-c4c12788f94c) post, Osmosis is intended to be a fast iterating platform that regularly add new features and modules through software upgrades.  A precise timeline for upgrade schedules does not exist, but validators are expected to be ready to upgrade the network potentially as frequently as a monthly basis early on. Furthermore, Osmosis intends adopt many new custom low-level features such as threshold decryption, custom bridges, and price oracles. Some of these future upgrades may require validators to run additional software beyond the normal node software, and validators should be prepared to run these.
3. To be a genesis validator, you must have OSMO at genesis via the fairdrop.  Every address that had ATOMs during the Stargate upgrade of the Cosmos Hub from `cosmoshub-3` to `cosmoshub-4` will have recieve fairdrop OSMO.  You can verify that a Cosmos address has received coins in the fairdrop by inputting an address here: https://airdrop.osmosis.zone/.


## Hardware

You should select an all-purpose server with:
- 4 or more CPU cores
- At least 500GB of SSD disk storage
- At least 16GB of memory
- At least 100mbps network bandwidth

As the usage of the blockchain grows, the server requirements may increase as well, so you should have a plan for updating your server as well.


## Instructions

These instructions are written targeting an Ubuntu 20.04 system.  Relevant changes to commands should be made depending on the OS/architecture you are running on.

## Install Go

Osmosis is built using Go and requires Go version 1.15+. In this example, we will be installing Go on the above Ubuntu 20.04:

```sh
# First remove any existing old Go installation
sudo rm -rf /usr/local/go

# Install the latest version of Go using this helpful script 
curl https://raw.githubusercontent.com/canha/golang-tools-install-script/master/goinstall.sh | bash

# Update environment variables to include go
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
```

To verify that Go is installed:

```sh
go version
# Should return go version go1.16.4 linux/amd64
```

## Install osmosisd

`osmosisd` is the node software for running the osmosis chain.  Install the proper version using these commands:

```sh
# Clone osmosisd repo and checkout correct branch
git clone https://github.com/osmosis-labs/osmosis -b main #todo fix branch
cd osmosis

# Build and install osmosisd
make build
make install
```

To verify that `osmosisd` has been installed correctly, check especially the `version` matches the software version provided here:

```sh
osmosisd version --long

name: osmosis
server_name: osmosisd
version: '"0.0.1"'
commit: 985e04c1e18ef2d130801a621de4ba903e4e9191
build_tags: netgo,ledger
go: go version go1.16.3 darwin/amd64
```

If the software version does not match, then please check your `$PATH` to ensure the correct `osmosisd` is running.

## Install and setup cosmovisor

`osmosisd` is the node software for running the osmosis chain.  Install the proper version using these commands:

```sh
# Clone osmosisd repo and checkout correct branch
git clone https://github.com/osmosis-labs/osmosis -b main
cd osmosis

# Build and install osmosisd
make build
make install
```

To verify that `osmosisd` has been installed correctly, check especially the `version` matches the software version provided here:

```sh
osmosisd version --long

name: osmosis
server_name: osmosisd
version: '"0.0.1"'
commit: 985e04c1e18ef2d130801a621de4ba903e4e9191
build_tags: netgo,ledger
go: go version go1.16.3 darwin/amd64
```

If the software version does not match, then please check your `$PATH` to ensure the correct `osmosisd` is running.


# Configuring Your Node

## Check the Network Parameters

To check the current testnet network parameters:

```sh
# First, get the network config for the current network.
curl https://testnet.agoric.net/network-config > chain.json
# Set chain name to the correct value
chainName=`jq -r .chainName < chain.json`
# Confirm value: should be something like agorictest-N.
echo $chainName
```

**NOTE: If the `$chainName` is out of date, then it means you need to wait until the new Testnet has been bootstrapped before you can continue.**  Please refer to [Network Status](#Network-Status) for when the Testnet corresponding to your software release is scheduled to be live.  Repeat the above step to check if it is ready yet.

## Apply Network Parameters

When the Agoric Testnet is ready in the previous step, you can initialize your validator and download the corresponding genesis file:

If you've never initialized your validator before, use:

```sh
# Replace <your_moniker> with the public name of your node.
# NOTE: The `--home` flag (or `AG_CHAIN_COSMOS_HOME` environment variable) determines where the chain state is stored.
# By default, this is `$HOME/.ag-chain-cosmos`.
ag-chain-cosmos init --chain-id $chainName <your_moniker>
```

Once you have an initialized validator, do:

```sh
# Download the genesis file
curl https://testnet.agoric.net/genesis.json > $HOME/.ag-chain-cosmos/config/genesis.json 
# Reset the state of your validator.
ag-chain-cosmos unsafe-reset-all
```

### Adjust configuration

Next, we want to adjust the validator configuration to add the peers and seeds from the network config:

```
# Set peers variable to the correct value
peers=$(jq '.peers | join(",")' < chain.json)
# Set seeds variable to the correct value.
seeds=$(jq '.seeds | join(",")' < chain.json)
# Confirm values, each should be something like "077c58e4b207d02bbbb1b68d6e7e1df08ce18a8a@178.62.245.23:26656,..."
echo $peers
echo $seeds
# Fix `Error: failed to parse log level`
sed -i.bak 's/^log_level/# log_level/' $HOME/.ag-chain-cosmos/config/config.toml
# Replace the seeds and persistent_peers values
sed -i.bak -e "s/^seeds *=.*/seeds = $seeds/; s/^persistent_peers *=.*/persistent_peers = $peers/" $HOME/.ag-chain-cosmos/config/config.toml
```

# Syncing Your Node

To sync our node, we will use `systemd`, which manages the Agoric Cosmos daemon and automatically restarts it in case of failure. To use `systemd`, we will create a service file:

```sh
sudo tee <<EOF >/dev/null /etc/systemd/system/ag-chain-cosmos.service
[Unit]
Description=Agoric Cosmos daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/ag-chain-cosmos start --log_level=warn
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF

# Check the contents of the file, especially User, Environment and ExecStart lines
cat /etc/systemd/system/ag-chain-cosmos.service
```

If you decide to run from the console, you can just do the following:

```sh
ag-chain-cosmos start --log_level=warn
```

To start syncing:

```sh
# Start the node
sudo systemctl enable ag-chain-cosmos
sudo systemctl daemon-reload
sudo systemctl start ag-chain-cosmos
```

To check on the status of syncing:

```sh
ag-cosmos-helper status 2>&1 | jq .SyncInfo
```

If this command fails with `parse error: ...`, then try just:

```sh
ag-cosmos-helper status
```

## `ag-cosmos-helper status` Errors

If `ag-cosmos-helper status` fails with:
```
Error: failed to parse log level (main:info,state:info,statesync:info,*:error): Unknown Level String: 'main:info,state:info,statesync:info,*:error', defaulting to NoLevel
```

then run:
```sh
sed -i.bak 's/^log_level/# log_level/' $HOME/.ag-cosmos-helper/config/config.toml
```

If `ag-cosmos-helper status` fails with:
```
ERROR: Status: Post http://localhost:26657: dial tcp [::1]:26657: connect: connection refused
```

then you should run `sudo journalctl -u ag-chain-cosmos` and diagnose the failure.

## Status output

If the status command succeeds, this will give output like:

```json
{
  "latest_block_hash": "6B87878277C9164006F2E7C544A27C2A4010D0107F436645BFE35BAEBE50CDF2",
  "latest_app_hash": "010EF4A62021F88D097591D6A31906CF9E5FA4359DC523B535E3C411DC6010B1",
  "latest_block_height": "233053",
  "latest_block_time": "2020-01-31T22:12:45.006715122Z",
  "catching_up": true
}
```

The main thing to watch is that the block height is increasing. Once you are caught up with the chain, `catching_up` will become false. At that point, you can start using your node to create a validator.

## Creating a Validator

**For a chain restart, skip to 

**If you are upgrading, [skip creating a new operator key](#tap-the-faucet).**

## If you don't have an operator key

Are you **really** sure you don't have an operator key?  You should try [recovering it from your mnemonic](#how-do-i-recover-a-key).

First, create a wallet, which will give you a private key / public key pair for your node.

```sh
# Replace <your-key-name> with a name for your operator key that you will remember
ag-cosmos-helper keys add <your-key-name>
# To see a list of wallets on your node
ag-cosmos-helper keys list
```

**NOTE: Be sure to write down the mnemonic for your wallet and store it securely. Losing your mnemonic could result in the irrecoverable loss of Agoric tokens.  Also, recovering pre-`testnet-3` keys from their mnemonic has changed.  Please refer to the [software release notes](/Agoric/agoric-sdk/tree/master/packages/cosmic-swingset/NEWS.md).**

### Tap the Faucet

To request tokens, go to the [Agoric Discord server](https://agoric.com/discord) `#testnet-faucet` channel and send the following chat message with your generated Agoric address (the `address: agoric1...` address, not the `pubkey: agoricpub1...` public key):

```
!faucet delegate agoric1...
```

When you get the ✅ the `uagstake` tokens have been sent, and you can view the tokens in your account.

## Check your balance

```sh
# View the tokens ("coins") currently deposited in your operator account.
# The "uagstake" tokens are millionths of Agoric staking tokens
ag-cosmos-helper query bank balances `ag-cosmos-helper keys show -a <your-key-name>`
```

Verify that you have at least 1 agstake (*1000000uagstake*).

## Catching up

Your node must have caught up with the rest of the chain before you can create the validator.  Here is a shell script loop that will wait for that to happen:

```sh
while sleep 5; do
  sync_info=`ag-cosmos-helper status 2>&1 | jq .SyncInfo`
  echo "$sync_info"
  if test `echo "$sync_info" | jq -r .catching_up` == false; then
    echo "Caught up"
    break
  fi
done
```

The above loop will poll your node every 5 seconds, displaying the `sync_info`.  When you have caught up, it will display a `Caught up` message and stop the loop.

## Get the validator public key

**NOTE: The following command will give incorrect values** if you don't run it under the same machine and user that is currently running your validator:

```sh
# Get the public key from the current node.
ag-chain-cosmos tendermint show-validator
```

This will display something like `agoricvalconspub1...`. Paste this key somewhere you can access it in the below step.

## Submit the create-validator transaction

You can see the options for creating a validator:

```sh
ag-cosmos-helper tx staking create-validator -h
```

An example of creating a validator with 50 agstake self-delegation and 10% commission.  You need the correct `--pubkey=` flag as described in the previous section, or you will **lose your staking tokens**:

```sh
# Set the chainName value again
chainName=`curl https://testnet.agoric.net/network-config | jq -r .chainName`
# Confirm value: should be something like agoricdev-N
echo $chainName
# Replace <key_name> with the key you created previously
ag-cosmos-helper tx staking create-validator \
  --amount=50000000uagstake \
  --broadcast-mode=block \
  --pubkey=<your-agoricvalconspub1-key> \
  --moniker=<your-node-name> \
  --website=<your-node-website> \
  --details=<your-node-details> \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=<your-key-name> \
  --chain-id=$chainName \
  --gas=auto \
  --gas-adjustment=1.4
```

To check on the status of your validator:

```sh
ag-cosmos-helper status 2>&1 | jq .ValidatorInfo
```

# Next steps

After you have completed this guide, your validator should be up and ready to receive delegations. Note that only the top 100 validators by weighted stake (self-delegations + other delegations) are eligible for block rewards. To view the current validator list, check out an Agoric block explorer (in the [Network Status section](#Network-Status)).


## Creating a gentx

see [[Creating a gentx]]

## Frequently Asked Questions

See [[Validator Guide]]

## Caveats

Note that this is a minimal guide and does not cover more advanced topics like [sentry node architecture](https://github.com/bitfishlabs/cosmos-validator-design) and [double-signing protection](https://github.com/tendermint/kms). It is strongly recommended that any parties considering validating do additional research.

## Developing dApps?

**Note that developing dapps for the Agoric chain does not require you to be a validator.**  If you're primarily looking to develop, please head [here](https://agoric.com/documentation/getting-started/alpha.html) instead.  If not, read on!

---
*Disclaimer: This content is provided for informational purposes only, and should not be relied upon as legal, business, investment, or tax advice. You should consult your own advisors as to those matters. References to any securities or digital assets are for illustrative purposes only and do not constitute an investment recommendation or offer to provide investment advisory services. Furthermore, this content is not directed at nor intended for use by any investors or prospective investors, and may not under any circumstances be relied upon when making investment decisions.*

This work, ["Agoric Validator Guide"](https://github.com/Agoric/agoric-sdk/wiki/Validator-Guide), is a derivative of ["Validating Kava Mainnet"](https://medium.com/kava-labs/validating-kava-mainnet-72fa1b6ea579) by [Kevin Davis](https://medium.com/@kevin_35106), used under [CC BY](http://creativecommons.org/licenses/by/4.0/). "Agoric Validator Guide" is licensed under [CC BY](http://creativecommons.org/licenses/by/4.0/) by [Agoric](https://agoric.com/).  It was extensively modified to make relevant to the Agoric Cosmos Chain.