# Quick Reference - osmo-testnet-1
Seed
* f5051996db0e0df69c55c36977407a9b8f94edb4@159.203.100.232:26656

Persistent Peers
* edd2b4968f012148641205b8ddd29f1beae8ab09@68.183.153.16:26656
* e159391f00e8127d8e6ec1319b04633ffc33ed1a@165.227.122.46:26656

RPC
* http://165.227.122.46:26657/

Must use v6.2.0

# Guide (osmo-testnet-1)

This guide will show you how to statesync to the new testnet. While you can sync from genesis, a current tendermint peer bug will make this process more painful than it should be, so we therefore recommend using statesync. This guide will not go extremely in depth as a majority of this is covered in https://docs.osmosis.zone

```
git clone https://github.com/osmosis-labs/osmosis.git
git checkout v6.2.0
make install
```

reload terminal
set up testnet genesis

```
osmosisd init NODENAME --chain-id osmo-testnet-1
cd ~/.osmosis/config/
wget https://github.com/osmosis-labs/networks/raw/adam/v2testnet/osmo-testnet-1/genesis.tar.bz2
tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
```

Modify the config.toml

Set up statesync by modifying the RPCs, TRUST_HEIGHT, and TRUST_HASH

RPCs = "165.227.122.46:26657,165.227.122.46:26657"
LATEST_HEIGHT = curl -s http://osmo-sync.blockpane.com:26657/block | jq -r .result.block.header.height
TRUST_HEIGHT = LATEST_HEIGHT - 1000
TRUST_HASH = curl -s http://osmo-sync.blockpane.com:26657/block?height=TRUST_HEIGHT | jq -r .result.block_id.hash

Add the seed node
```
f5051996db0e0df69c55c36977407a9b8f94edb4@159.203.100.232:26656
```

Add the persistent peers
```
edd2b4968f012148641205b8ddd29f1beae8ab09@68.183.153.16:26656,e159391f00e8127d8e6ec1319b04633ffc33ed1a@165.227.122.46:26656
```

then start the daemon with
```
osmosisd start
```

no patch needed, you will then be syncing blocks!
