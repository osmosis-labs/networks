# Setting Up a Genesis Osmosis Validator Part 2

Thank you for submitting a gentx!  We had 40 gentxs submitted!  This guide will provide instructions
on the next stage of getting ready for the Osmosis launch.  


**The Chain Genesis Time is 17:00 UTC on June 18, 2021.**

Please have your validator up and ready by this time, and be available for further instructions if necessary
at that time.

The primary point of communication for the genesis process will be the #validators
channel on the [Osmosis Discord](https://discord.gg/FAarwSC8Tr). It is absolutely critical that you
and your team join the Discord during launch, as it will be the coordination point in case of any hiccups
or issues during the launch process.  The channel is private by default in order to keep it free of spam
and unnecessary noise.  To join the channel, please send a message to Meow#6669 to add yourself and any
team members.

This guide assumes that you have completed the tasks involved in [Part 1](genesis-validators.md).  You should
be running on a machine that meets the [hardware requirements specified in Part 1](genesis-validators.md#hardware)
with [Go installed](validators.md#install-go).  We are assuming you already have a daemon home ($HOME/.osmosisd)
setup.

These instructions are for creating a basic setup on a single node. Validators should modify these instructions
for their own custom setups as needed (i.e. sentry nodes, tmkms, etc).

## Instructions

These i is itnstructions are written targeting an Ubuntu 20.04 system.  Relevant changes to commands should be made depending on the OS/architecture you are running on.

### Update osmosisd to v1.0.0

For the gentx creation, we used the `gentx-launch` branch of the [Osmosis codebase](https://github.com/osmosis-labs/osmosis).

For launch, please update to the `v1.0.0` tag and rebuild your binaries.

```sh
git clone https://github.com/osmosis-labs/osmosis
cd osmosis
git checkout v1.0.0

make install
```

### Verify Your Installation

Verify that everything is OK. If you get something *like* the following, you've successfully installed Osmosis on your system. (scroll up to see above the list of dependencies)

```sh
osmosisd version --long

name: osmosis
server_name: osmosisd
version: '"1.0.0"'
commit: TODO
build_tags: netgo,ledger
go: go version go1.16.3 darwin/amd64
```

If the software version does not match, then please check your `$PATH` to ensure the correct `osmosisd` is running.

### Save your Chain ID in osmosisd config

Osmosis reintroduces the client-side config that was removed in earlier Stargate versions of the Cosmos SDK.

If you haven't done so already, please save the mainnet chain-id to your client.toml. This will make it so you do not have to manually pass in the chain-id flag for every CLI command.

```sh
osmosisd config chain-id osmosis-1
```

### Install and setup Cosmovisor

We highly recommend validators use cosmovisor to run their nodes. This will make low-downtime upgrades more smoother,
as validators don't have to manually upgrade binaries during the upgrade, and instead can preinstall new binaries, and
cosmovisor will automatically update them based on on-chain SoftwareUpgrade proposals.

You should review the docs for cosmovisor located here: https://docs.cosmos.network/master/run-node/cosmovisor.html

If you choose to use cosmovisor, please continue with these instructions:

Cosmovisor is currently located in the Cosmos SDK repo, so you will need to download that, build cosmovisor, and add it
to you PATH.

```
git clone https://github.com/cosmos/cosmos-sdk
cd cosmos-sdk
git checkout v0.42.5
make cosmovisor
cp cosmovisor/cosmovisor $GOPATH/bin/cosmovisor
cd $HOME
```

After this, you must make the necessary folders for cosmosvisor in your daemon home directory (~/.osmosisd).

```
mkdir -p ~/.osmosisd
mkdir -p ~/.osmosisd/cosmovisor
mkdir -p ~/.osmosisd/cosmovisor/genesis
mkdir -p ~/.osmosisd/cosmovisor/genesis/bin
mkdir -p ~/.osmosisd/cosmovisor/upgrades
```

Cosmovisor requires some ENVIRONMENT VARIABLES be set in order to function properly.  We recommend setting these in
your `.profile` so it is automatically set in every session.

```
echo "# Setup Cosmovisor" >> ~/.profile
echo "export DAEMON_NAME=osmosisd" >> ~/.profile
echo "export DAEMON_HOME=$HOME/.osmosisd" >> ~/.profile
source ~/.profile
```

Finally, you should copy the osmosisd binary into the cosmovisor/genesis folder.
cp $GOPATH/bin/osmosisd ~/.osmosisd/cosmovisor/genesis/bin

This will create a new `.osmosisd` folder in your HOME directory.

### Download Genesis File

You can now download the "genesis" file for the chain.  It is pre-filled with the entire genesis state and gentxs.

```sh
curl https://raw.githubusercontent.com/osmosis-labs/networks/main/osmosis-1/genesis.json > ~/.osmosisd/config/genesis.json
```

### Updates to config files

You should review the config.toml and app.toml that was generated when you ran `osmosisd init` last time.

A couple things to highlight especially:

- In the `launch-gentxs` branch, we defaulted the tendermint fast-sync to be "v2".  However, thanks to testing with
partners from [Skynet](http://skynet.paullovette.com/) and [Akash Network](https://akash.network/), we've determined
that "v2" is too unstable for use in production, and so we recommend everyone downgrade to "v0".  In your config.toml,
in the [fastsync] section, change `version = "v2"` to `version = "v0"`.
- We've defaulted nodes to having their gRPC and REST endpoints enabled.  If you do not want his (especially for validator
nodes), please turn these off in your app.toml
- We have defaulted all nodes to maintaining 2 recent statesync snapshots.
- When it comes the min gas fees, our recommendation is to leave this blank for now (charge no gas fees), to make the UX as seamless as possible
for users to be able to pay with whichever IBC asset they bridge over.  Then you can return to this in ~1 week and include
min-gas-price costs denominated in multiple different IBCed assets.  We're aware this is quite clunkly right now, and we will be working
on better mechanisms for this process.  Here's to interchain UX finally becoming a reality!

### Reset Chain Database

There shouldn't be any chain database yet, but in case there is for some reason, you should reset it.

```sh
osmosisd unsafe-reset-all
```

### Start your node

Now that everything is setup and ready to go, you can start your node.

```
osmosisd start
```

You will likely need some way to keep the process always running.  If you're on linux, you can do this by creating a 
service.

```
DAEMON_PATH=$(which osmosisd)

echo "[Unit]
Description=osmosisd daemon
After=network-online.target
[Service]
User=${USER}
ExecStart=${DAEMON_PATH} start
Restart=always
RestartSec=3
LimitNOFILE=4096
[Install]
WantedBy=multi-user.target
" >osmosisd.service
```

Then update and start the node

```sh
sudo mv osmosisd.service /lib/systemd/system/osmosisd.service
sudo -S systemctl daemon-reload
sudo -S systemctl enable osmosisd
sudo -S systemctl start osmosisd
```

## Conclusion

See you all at launch!  Join the discord!


---
*Disclaimer: This content is provided for informational purposes only, and should not be relied upon as legal, business, investment, or tax advice. You should consult your own advisors as to those matters. References to any securities or digital assets are for illustrative purposes only and do not constitute an investment recommendation or offer to provide investment advisory services. Furthermore, this content is not directed at nor intended for use by any investors or prospective investors, and may not under any circumstances be relied upon when making investment decisions.*
