# Quick Reference - osmo-test-3
Seed
* 

Persistent Peers
* 4ab030b7fd75ed895c48bcc899b99c17a396736b@137.184.190.127:26656
* 3dbffa30baab16cc8597df02945dcee0aa0a4581@143.198.139.33:26656

RPC
* http://143.198.139.33:26657/

DO NOT USE 6.2.0

# Statesync Guide (osmo-test-3)

This guide will show you how to statesync to the new testnet. This guide will not go extremely in depth as a majority of this is covered in https://docs.osmosis.zone

Clone the osmosis repo and install v6.3.1

```
git clone https://github.com/osmosis-labs/osmosis.git
git checkout v6.3.1
make install
```

Set up testnet genesis

```
osmosisd init NODENAME --chain-id osmo-test-3
cd ~/.osmosis/config/
wget https://github.com/osmosis-labs/networks/raw/adam/osmo-test-3/osmo-test-3/genesis.tar.bz2 
tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
```

Set up statesync by modifying the RPCs, TRUST_HEIGHT, and TRUST_HASH in the config.toml

- RPCs = "143.198.139.33:26657,143.198.139.33:26657"
- LATEST_HEIGHT = curl -s http://143.198.139.33:26657/block | jq -r .result.block.header.height
- TRUST_HEIGHT = LATEST_HEIGHT - 2000
- TRUST_HASH = curl -s http://143.198.139.33:26657/block?height=TRUST_HEIGHT | jq -r .result.block_id.hash

Add the seed node
```
f5051996db0e0df69c55c36977407a9b8f94edb4@159.203.100.232:26656 
```

Add the persistent peers
```
4ab030b7fd75ed895c48bcc899b99c17a396736b@137.184.190.127:26656,3dbffa30baab16cc8597df02945dcee0aa0a4581@143.198.139.33:26656
```

Then start the daemon with

```
osmosisd start
```

no patch needed, you will then be syncing blocks!

# Sync From Genesis Guide (osmo-test-3)

This guide will show you how to sync from genesis to the new testnet. PLEASE NOTE, the first block may take a while (up to an hour) to get passed. If you need all transaction data, it is recommended to download an archive snapshot from ChainLayer instead. This guide will not go extremely in depth as a majority of this is covered in https://docs.osmosis.zone

Clone the osmosis repo and install v6.3.1

```
git clone https://github.com/osmosis-labs/osmosis.git
git checkout v6.3.1
make install
```

Set up testnet genesis

```
osmosisd init NODENAME --chain-id osmo-test-3
cd ~/.osmosis/config/
wget https://github.com/osmosis-labs/networks/raw/adam/v2testnet/osmo-test-3/genesis.tar.bz2 
tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
```

Start osmosisd with --x-crisis-skip-assert-invariants flag.

```
osmosisd start --x-crisis-skip-assert-invariants
```

You only need to use this flag once. Your daemon will run the first block and then get stuck searching for peers. This can take an hour or more. After an hour or so, you will then sync blocks. If you cancel this process before finding two or more blocks, you will have to use `osmosisd unsafe-reset-all` and then start with the skip assert invariants flag again. After finding two or more blocks you are in the clear. You can cancel the daemon at any point you want and simply use `osmosisd start` to get it running again.