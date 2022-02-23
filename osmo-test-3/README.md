# Quick Reference - (osmo-test-3)
Seed
* 0f9a9c694c46bd28ad9ad6126e923993fc6c56b1@137.184.181.105:26656

Persistent Peers
* 4ab030b7fd75ed895c48bcc899b99c17a396736b@137.184.190.127:26656
* 3dbffa30baab16cc8597df02945dcee0aa0a4581@143.198.139.33:26656

RPC
* https://rpc.osmo-test.ccvalidators.com/ (hosted by CryptoCrew)
* https://osmosistest-rpc.quickapi.com/ (hosted by ChainLayer)
* https://testnet-rpc.osmosis.zone

LCD
* https://osmosistest-lcd.quickapi.com/ (hosted by ChainLayer)
* https://lcd.osmo-test.ccvalidators.com/ (hosted by CryptoCrew)
* https://testnet-rest.osmosis.zone

DO NOT USE v6.2.0

# Bash Installer

Simply run the bash installer at https://get.osmosis.zone and follow the on screen instructions for `osmo-test-3`

# Sync From Genesis Guide (osmo-test-3)

This guide will show you how to sync from genesis to the new testnet. PLEASE NOTE, the first block may take a while (up to an hour) to get passed. If you need all transaction data, it is recommended to download an archive snapshot from ChainLayer instead. This guide will not go extremely in depth as a majority of this is covered in https://docs.osmosis.zone

Clone the osmosis repo and install v6.4.0

```
git clone https://github.com/osmosis-labs/osmosis.git
git checkout v6.4.0
make install
```

Set up testnet genesis

```
osmosisd init NODENAME --chain-id osmo-test-3
cd ~/.osmosisd/config/
wget https://github.com/osmosis-labs/networks/raw/adam/osmo-test-3/osmo-test-3/genesis.tar.bz2
tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
```

Start osmosisd with --x-crisis-skip-assert-invariants flag.

```
osmosisd start --x-crisis-skip-assert-invariants
```

You only need to use this flag once. Your daemon will run the first block and then get stuck searching for peers. This can take an hour or more. After an hour or so, you will then sync blocks. If you cancel this process before finding two or more blocks, you will have to use `osmosisd unsafe-reset-all` and then start with the skip assert invariants flag again. After finding two or more blocks you are in the clear. You can cancel the daemon at any point you want and simply use `osmosisd start` to get it running again.
