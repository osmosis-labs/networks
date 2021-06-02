# Setting Up a Genesis Osmosis Validator

Thank you for becoming a genesis validator on Osmosis!  This guide will provide instructions on setting up
a node, submitting a gentx, and other tasks needed to participate in the launch of the Osmosis mainnet.

The primary point of communication for the genesis process and future updates will be the #validators
channel on the [Osmosis Discord](https://discord.gg/tPayMvaTT3). This channel is private by default in order
to keep it free of spam and unnecessary noise.  To join the channel, please send a message to @Meow#6669
to add yourself and any team members.

Some important notes on joining as a genesis validator:

1. We highly recommend only experienced validators who have run on past Cosmos SDK chains and have participated in a genesis ceremony before become genesis validators on Osmosis.
2. All Osmosis validators should be expected to be ready to participate active operators of the network. As explained in the [Osmosis: A Hub AMM](https://medium.com/osmosis/osmosis-a-hub-amm-c4c12788f94c) post, Osmosis is intended to be a fast iterating platform that regularly add new features and modules through software upgrades.  A precise timeline for upgrade schedules does not exist, but validators are expected to be ready to upgrade the network potentially as frequently as a monthly basis early on. Furthermore, Osmosis intends to adopt many new custom low-level features such as threshold decryption, custom bridges, and price oracles. Some of these future upgrades may require validators to run additional software beyond the normal node software, and validators should be prepared to learn and run these.
3. To be a genesis validator, you must have OSMO at genesis via the fairdrop.  Every address that had ATOMs during the Stargate upgrade of the Cosmos Hub from `cosmoshub-3` to `cosmoshub-4` will have recieve fairdrop OSMO.  You can verify that a Cosmos address has received coins in the fairdrop by inputting an address here: https://airdrop.osmosis.zone/.

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

Use git to retrieve Osmosis source code from the [official repo](https://github.com/osmosis-labs/osmosis), and checkout the `TODO` tag, which contains the latest stable release.

```sh
git clone https://github.com/osmosis-labs/osmosis
cd osmosis
git checkout master # TODO
```

## Install osmosisd

You can now build Osmosis node software. Running the following command will install the executable osmosisd (Osmosis node daemon) to your GOPATH.

```sh
make install
```

### Verify Your Installation

Verify that everything is OK. If you get something like the following, you've successfully installed Osmosis on your system.

```sh
osmosisd version --long

name: osmosis
server_name: osmosisd
version: '"0.0.1"' # TODO
commit: 985e04c1e18ef2d130801a621de4ba903e4e9191
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

You can now download the "pregenesis" file for the chain.  This is the genesis file, with the exception that it is missing the gentxs.

```sh
curl <url> # TODOs
```

### Create GenTx

```shell
myKey=<your key name (the one you specified with `keys add`) or address (agoric1...)>
myMoniker="<the actual value you want to use as your validator's moniker>"
chainName=osmosis-1

# Create the gentx.
# Note, your gentx will be rejected if you use an amount greater than what you have as liquid from the
# fairdrop. Recall only 20% of your fairdrop allocation is liquid at genesis.
# Also recall that since Osmosis has a min-commission-rate of .05, your commission rate
# must be greater than or equal to 0.05
osmosisd gentx $myKey 100uosmo --output-document=gentx.json \
  --chain-id=$chainName \
  --keyring-dir=$HOME/.osmosisd \
  --moniker="$myMoniker" \
  --website=<your-node-website> \
  --details=<your-node-details> \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1"
```

The result should look something like this [sample gentx file](https://gist.github.com/michaelfig/c1976099f28899d0077f2e47dfed04c1). _For reference: [gaia gentx docs](https://github.com/cosmos/gaia/blob/main/docs/validators/validator-setup.md#participate-in-genesis-as-a-validator)._

## Developing AMMs?

**Note that developing AMMs for the Osmosis chain does not require you to be a validator.**  If you're primarily looking to develop, please head [{TODO AMM DEVELOPER DOCS}](https://github.com/osmosis-labs/osmosis) instead.

---
*Disclaimer: This content is provided for informational purposes only, and should not be relied upon as legal, business, investment, or tax advice. You should consult your own advisors as to those matters. References to any securities or digital assets are for illustrative purposes only and do not constitute an investment recommendation or offer to provide investment advisory services. Furthermore, this content is not directed at nor intended for use by any investors or prospective investors, and may not under any circumstances be relied upon when making investment decisions.*

This work, ["Osmosis Genesis Validators Guide"](https://github.com/osmosis-labs/networks/genesis-validators.md), is a derivative of ["Agoric Validator Guide"](https://github.com/Agoric/agoric-sdk/wiki/Validator-Guide) used under [CC BY](http://creativecommons.org/licenses/by/4.0/). The Agoric validator gudie is itself is a derivative of ["Validating Kava Mainnet"](https://medium.com/kava-labs/validating-kava-mainnet-72fa1b6ea579) by [Kevin Davis](https://medium.com/@kevin_35106), used under [CC BY](http://creativecommons.org/licenses/by/4.0/). "Osmosis Validator Guide" is licensed under [CC BY](http://creativecommons.org/licenses/by/4.0/) by [Osmosis Labs](https://osmosis.zone/).  It was extensively modified to be relevant to the Osmosis Chain.
