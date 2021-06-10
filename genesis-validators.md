# Setting Up a Genesis Osmosis Validator

Thank you for becoming a genesis validator on Osmosis!  This guide will provide instructions on setting up
a node, submitting a gentx, and other tasks needed to participate in the launch of the Osmosis mainnet.

The primary point of communication for the genesis process and future updates will be the #validators
channel on the [Osmosis Discord](https://discord.gg/FAarwSC8Tr). This channel is private by default in order
to keep it free of spam and unnecessary noise.  To join the channel, please send a message to @Meow#6669
to add yourself and any team members.

Some important notes on joining as a genesis validator:

1. **Gentxs must be submitted by End of Day UTC on June 11.**
2. We highly recommend only experienced validators who have run on past Cosmos SDK chains and have participated in a genesis ceremony before become genesis validators on Osmosis.
3. All Osmosis validators should be expected to be ready to participate active operators of the network. As explained in the [Osmosis: A Hub AMM](https://medium.com/osmosis/osmosis-a-hub-amm-c4c12788f94c) post, Osmosis is intended to be a fast iterating platform that regularly add new features and modules through software upgrades.  A precise timeline for upgrade schedules does not exist, but validators are expected to be ready to upgrade the network potentially as frequently as a monthly basis early on. Furthermore, Osmosis intends to adopt many new custom low-level features such as threshold decryption, custom bridges, and price oracles. Some of these future upgrades may require validators to run additional software beyond the normal node software, and validators should be prepared to learn and run these.
4. To be a genesis validator, you must have OSMO at genesis via the fairdrop.  Every address that had ATOMs during the Stargate upgrade of the Cosmos Hub from `cosmoshub-3` to `cosmoshub-4` will have recieve fairdrop OSMO.  You can verify that a Cosmos address has received coins in the fairdrop by inputting an address here: https://airdrop.osmosis.zone/.

## Hardware

We recommend selecting an all-purpose server with:

- 4 or more physical<sup>[1]</sup> CPU cores
- At least 500GB of SSD disk storage
- At least 16GB of memory
- At least 100mbps network bandwidth

As the usage of the blockchain grows, the server requirements may increase as well, so you should have a plan for updating your server as well.

<sup>[1]</sup>:
You'll often see 4 distincy physical cores as a machine with 8 logical cores due to hyperthreading.
The distinct logical cores are helpful for things that are I/O bound,
but threshold decryption will have validators running significant, non-I/O bound, computation,
hence the need for physical cores.
We are not launching with this parallelism, but we include the requirement as we expect parallelism in some form to be needed by validators in a not-so-distant future.

## Instructions

These instructions are written targeting an Ubuntu 20.04 system.  Relevant changes to commands should be made depending on the OS/architecture you are running on.

### Install Go

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

### Get Osmosis Source Code

Use git to retrieve Osmosis source code from the [official repo](https://github.com/osmosis-labs/osmosis), and checkout the `gentx-launch` tag, which contains the latest stable release.

```sh
git clone https://github.com/osmosis-labs/osmosis
cd osmosis
git checkout gentx-launch
```

## Install osmosisd

You can now build Osmosis node software. Running the following command will install the executable osmosisd (Osmosis node daemon) to your GOPATH.

```sh
make install
```

### Verify Your Installation

Verify that everything is OK. If you get something *like* the following, you've successfully installed Osmosis on your system.

```sh
osmosisd version --long

name: osmosis
server_name: osmosisd
version: '"0.0.1"'
commit: 197171b8fcb364bd2c5c2fbb2532eab3f5e8517c
build_tags: netgo,ledger
go: go version go1.16.3 darwin/amd64
```

If the software version does not match, then please check your `$PATH` to ensure the correct `osmosisd` is running.

### Save your Chain ID in osmosisd config

We recommend saving the mainnet `chain-id` into your `osmosisd`'s client.toml.  This will make it so you do not have to manually pass in the chain-id flag for every CLI command.

```sh
osmosisd config chain-id osmosis-1
```

### Initialize your Node

Now that your software is installed, you can initialize the directory for osmosisd.

```sh
osmosisd init --chain-id=osmosis-1 <your_moniker>
```

This will create a new `.osmosisd` folder in your HOME directory.

### Download Pregenesis File

You can now download the "pregenesis" file for the chain.  This is a genesis file with the chain-id and airdrop balances.

```sh
cd $HOME/.osmosisd/config/
curl https://raw.githubusercontent.com/osmosis-labs/networks/main/osmosis-1/pregenesis.json > $HOME/.osmosisd/config/genesis.json
```

### Import Validator Key

The create a gentx, you will need the private key to an address that received an allocation in the airdrop.

There are a couple options for how to import a key into `osmosisd`.

You can import such a key into `osmosisd` via a mnemonic or exporting and importing a keyfile from an existing CLI.

#### Import Via Mnemonic

To import via mnemonic, you can do so using the following command and then input your mnemonic when prompted.

```sh
osmosisd keys add <key_name> --recover
```
#### Import From Another CLI

If you have the private key saved in the keystore of another CLI (such as gaiad), you can easily import it
into `osmosisd` using the following steps.

1. Export the key from an existing keystore.  In this example we will use gaiad.  When prompted, input a password to encrypt the key file with.

```sh
gaiad keys export <original_key_name>
```

2. Copy the output starting from the line that says `BEGIN TENDERMINT PRIVATE KEY` and ending with the line that says `END TENDERMINT PRIVATE KEY` into a txt file somewhere on your machine.
3. Import the key into `osmosisd` using the following command.  When prompted for a password, use the same password used in step 1 to encrypt the keyfile.

```sh
osmosisd keys import <new_key_name> ./path/to/key.txt 
```
4. Delete the keyfile from your machine.

#### Import via Ledger

To import a key stored on a ledger, the process will be exactly the same as adding a ledger key to the CLI normally.
You can connect a Ledger device with the Cosmos app open and then run:

```sh
osmosisd keys add <key_name> --ledger
```

and follow any prompts.

#### Get your Tendermint Validator Pubkey

You must get your validator's consensus pubkey as it will be necessary to include in the transaction to create your validator.

If you are using Tendermint's native `priv_validator.json` as your consensus key, you display your validator public key using the following command

```
osmosisd tendermint show-validator
```

The pubkey should be formatted with the bech32 prefix `osmovalconspub1`.

If you are using a custom signing mechanism such as `tmkms`, please refer to their relevant docs to retrieve your validator pubkey.


### Create GenTx

Now that you have you key imported, you are able to use it to create your gentx.

To create the genesis transaction, you will have to choose the following parameters for your validator:

- moniker
- commission-rate
- commission-max-rate
- commission-max-change-rate
- min-self-delegation (must be >1)
- website (optional)
- details (optional)
- identity (keybase key hash, this is used to get validator logos in block explorers. optional)
- pubkey (gotten in previous step)

Note that your gentx will be rejected if you use an amount greater than what you have as liquid from the fairdrop. Recall only 20% of your fairdrop allocation is liquid at genesis. Also, note that Osmosis has a chain-mandated minimum commission rate of 5%.

If you would like to override the memo field, use the `--ip` and `--node-id` flags.

An example genesis command would thus look like:

```sh
osmosisd gentx <key_name> 1000000uosmo \
  --chain-id="osmosis-1" \
  --moniker=osmosiswhale \
  --website="https://osmosis.zone" \
  --details="We love Osmossis" \
  --commission-rate="0.1" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --identity="5B5AB9D8FBBCEDC6" \
  --pubkey="osmovalconspub1zcjduepqnxl4ntf8wjn0275smfll4n4lg9cwcurz2qt6dkhrjzf94up8g4cspyyzn9"
```

It will show an output something similar to:

```sh
Genesis transaction written to "/Users/ubuntu/.osmosisd/config/gentx/gentx-eb3b1768d00e66ef83acb1eee59e1d3a35cf76fc.json"
```

The result should look something like this [sample gentx file]("/Users/sunnya97/.osmosisd/config/gentx/gentx-eb3b1768d00e66ef83acb1eee59e1d3a35cf76fc.json). 

### Submit Your GenTx

To submit your GenTx for inclusion in the chain, please upload it to the [github.com/osmosis-labs/networks](https://github.com/osmosis-labs/networks) repo by End of Day, June 10.

To upload the your genesis file, please follow these steps:

1. Rename the gentx file just generated to gentx-{your-moniker}.json (please do not have any spaces or special characters in the file name)
2. Fork this repo by going to https://github.com/osmosis-labs/networks, clicking on fork, and choose your account (if multiple).
3. Clone your copy of the fork to your local machine
```sh
git clone https://github.com/<your_github_username>/networks
```
4. Copy the gentx to the networks repo (ensure that it is in the correct folder)

```sh
cp ~/.osmosisd/config/gentx/gentx-<your-moniker>.json networks/osmosis-1/gentxs/
```

5. Commit and push to your repo.
 
```sh
cd networks
git add osmosis-1/gentxs/*
git commit -m "<your validator moniker> gentx"
git push origin master
```

6. Create a pull request from your fork to master on this repo.
7. Let us know on Discord when you've completed this process!
8. Stay tuned for next steps which will be distributed after June 11

---
*Disclaimer: This content is provided for informational purposes only, and should not be relied upon as legal, business, investment, or tax advice. You should consult your own advisors as to those matters. References to any securities or digital assets are for illustrative purposes only and do not constitute an investment recommendation or offer to provide investment advisory services. Furthermore, this content is not directed at nor intended for use by any investors or prospective investors, and may not under any circumstances be relied upon when making investment decisions.*

This work, ["Osmosis Genesis Validators Guide"](https://github.com/osmosis-labs/networks/genesis-validators.md), is a derivative of ["Agoric Validator Guide"](https://github.com/Agoric/agoric-sdk/wiki/Validator-Guide) used under [CC BY](http://creativecommons.org/licenses/by/4.0/). The Agoric validator gudie is itself is a derivative of ["Validating Kava Mainnet"](https://medium.com/kava-labs/validating-kava-mainnet-72fa1b6ea579) by [Kevin Davis](https://medium.com/@kevin_35106), used under [CC BY](http://creativecommons.org/licenses/by/4.0/). "Osmosis Validator Guide" is licensed under [CC BY](http://creativecommons.org/licenses/by/4.0/) by [Osmosis Labs](https://osmosis.zone/).  It was extensively modified to be relevant to the Osmosis Chain.
