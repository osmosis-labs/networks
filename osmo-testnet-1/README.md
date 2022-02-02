# Quick Reference - osmo-testnet-1
Seed
* f5051996db0e0df69c55c36977407a9b8f94edb4@159.203.100.232:26656

Persistent Peers
* b894030ba5cf2dab4fbe092eb659004d70d7dc90@142.93.177.164:26656
* e159391f00e8127d8e6ec1319b04633ffc33ed1a@165.227.122.46:26656

RPC
* http://165.227.122.46:26657/

Must use v6.2.0

# Statesync Guide (osmo-testnet-1)

This guide will show you how to statesync to the new testnet. This guide will not go extremely in depth as a majority of this is covered in https://docs.osmosis.zone

Clone the osmosis repo and install v6.2.0

```
git clone https://github.com/osmosis-labs/osmosis.git
git checkout v6.2.0
make install
```

Set up testnet genesis

```
osmosisd init NODENAME --chain-id osmo-testnet-1
cd ~/.osmosis/config/
wget https://github.com/osmosis-labs/networks/raw/adam/v2testnet/osmo-testnet-1/genesis.tar.bz2
tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
```

Set up statesync by modifying the RPCs, TRUST_HEIGHT, and TRUST_HASH in the config.toml

- RPCs = "165.227.122.46:26657,165.227.122.46:26657"
- LATEST_HEIGHT = curl -s http://165.227.122.46:26657/block | jq -r .result.block.header.height
- TRUST_HEIGHT = LATEST_HEIGHT - 1000
- TRUST_HASH = curl -s http://165.227.122.46:26657/block?height=TRUST_HEIGHT | jq -r .result.block_id.hash

Add the seed node
```
f5051996db0e0df69c55c36977407a9b8f94edb4@159.203.100.232:26656
```

Add the persistent peers
```
b894030ba5cf2dab4fbe092eb659004d70d7dc90@142.93.177.164:26656,e159391f00e8127d8e6ec1319b04633ffc33ed1a@165.227.122.46:26656
```

Then start the daemon with

```
osmosisd start
```

no patch needed, you will then be syncing blocks!

# Sync From Genesis Guide (osmo-testnet-1)

This guide will show you how to sync from genesis to the new testnet. PLEASE NOTE, you will have to get through a tendermint peer bug that might make this process take longer than it normally would. If you need all transaction data, it is recommended to download an archive snapshot from ChainLayer instead. This guide will not go extremely in depth as a majority of this is covered in https://docs.osmosis.zone

Clone the osmosis repo and install v6.2.0

```
git clone https://github.com/osmosis-labs/osmosis.git
git checkout v6.2.0
make install
```

Set up testnet genesis

```
osmosisd init NODENAME --chain-id osmo-testnet-1
cd ~/.osmosis/config/
wget https://github.com/osmosis-labs/networks/raw/adam/v2testnet/osmo-testnet-1/genesis.tar.bz2
tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
```

Start osmosisd with --x-crisis-skip-assert-invariants flag.

```
osmosisd start --x-crisis-skip-assert-invariants
```

You only need to use this flag once. After the genesis has been initialized, you will then run into the peer bug. Your daemon will run the first block and then get stuck searching for peers. This can take an hour or more. After an hour or so, you will then sync blocks. If you cancel this process before finding two or more blocks, you will have to use `osmosisd unsafe-reset-all` and then start with the skip assert inveriants flag again. After finding two or more blocks you are in the clear. You can cancel the daemon at any point you want and simply use `osmosisd start` to get it running again.