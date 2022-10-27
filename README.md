## Prerequisite
Please go through the [whitepaper](https://drive.google.com/file/d/17EScDNUlaYT1Xiera20x8rYsmI3ejggj/view) to understand the mechanics

---

## Mande testnet-1

### Managing Validator

While setting up a rudimentary validating node is easy, running a production-quality validator node with a robust architecture and security features requires an extensive setup.

The Mande node is powered by the Tendermint consensus. Validators run full nodes, participate in consensus by broadcasting votes, commit new blocks to the blockchain, and participate in governance of the blockchain. Anyone can cast votes on Validators which in turn gives them some credibility based on x(t) at that point of time. A validator’s voting power is weighted according to their total credebility. The top 175 validators make up the Active Validator Set and are the only validators that sign blocks and receive revenue.

Validators earn the following fees:
- Gas: Fees added on to each transaction to avoid spamming and pay for computing power. Validators set minimum gas prices and reject transactions that have implied gas prices below this threshold.
- Validators get the inflated coins as rewards proportional to their shares

`If validators double sign or frequently offline, their credibility will be slashed. Penalties can vary depending on the severity of the violation.`

---

### Joining Testnet


#### System Requirements

- Two or more CPU cores
- At least 100 GB of disk storage
- At least 4 GB of memory

** HDD not recommended **

#### Binaries

Download our binaries from [here](https://github.com/mande-labs/testnet-1/blob/main/mande-chaind) and place it in your bin path. Ex: `/usr/local/bin/`

#### Generate keys
```bash
mande-chaind keys add [key_name]
```
or
```bash
mande-chaind keys add [key_name] --recover  
```  
 to regenerate keys with your BIP39 mnemonic
 
#### Claim testnet coins
```bash
curl -d '{"address":"mande...<mande wallet address>"}' -H 'Content-Type: application/json' http://35.224.207.121:8080/request
```

#### Setting up a Node
Following steps  are  rudimentary way of setting up a validator, For production we advise your [sentry architecture](https://forum.cosmos.network/t/sentry-node-architecture-overview/454) to create well defined process

* Initialize node
	```shell
	mande-chaind init {{NODE_NAME}} --chain-id mande-testnet-1
	```
* Replace the contents of your `${HOME}/.mande-chaind/config/genesis.json` with that of [genesis file](https://github.com/mande-labs/testnet-1/blob/main/genesis.json) on this repo
* Verify checksum `jq -S -c -M "" genesis.json | sha256sum` matches `def6850afe2cb311b7909cdc9bfb6dd436b36a6fc015c3d524270a5cff050dfe`
* Inside file `${HOME}/.mande-chaind/config/config.toml`, 
  * set `seeds` to `"cd3e4f5b7f5680bbd86a96b38bc122aa46668399@34.171.132.212:26656"`.
  * set `persistent_peers` to `"ee8a1b98e931e81d32c52f0b489fa22b52778d7c@34.171.132.212:26656,6780b2648bd2eb6adca2ca92a03a25b216d4f36b@34.170.16.69:26656"`
  * If your node has a public ip, set it in `external_address = "tcp://<public-ip>:26656"`, else leave the filed empty.
* Set `minimum-gas-prices` in `${HOME}/.mande-chaind/config/app.toml` with the minimum price you want (example `0.005mand`) for the security of the network.
* Start node
	```bash
	mande-chaind start
	```
* Make sure you have some $MAND in your address claimed from the faucet previously.
* Wait for the blockchain to sync. You can check the sync status using the command
	```bash
	curl http://localhost:26657/status sync_info "catching_up": false
	```
* Once `"catching_up"` is `false`, the sync is complete.


#### Register the validator

Run the following command to register the validator  
```bash
mande-chaind tx staking create-validator \
--from {{KEY_NAME}} \
--amount 0cred \
--pubkey "$(mande-chaind tendermint show-validator)" \
--chain-id mande-testnet-1 \
--moniker="{{VALIDATOR_NAME}}" \
```

---


#### Setup a Metadata profile
To Improve  communication with community and core team, please add the following metadata to your validator. These  helps us  send security and chain upgrade announcement in a clear manner

```bash
mande-chaind tx staking edit-validator \
    --identity="keybase identity"
    --security-contact="XXXXXXXX" \
    --website="XXXXXXXX"
```

#### Some helpful commands
##### Query outstanding rewards:
`mande-chaind query distribution validator-outstanding-rewards mandevaloper...`
##### Withdraw rewards:
`mande-chaind tx distribution withdraw-rewards mandevaloper... --commission --from {{KEY_NAME}} --chain-id mande-testnet-1`
##### Cast/Uncast vote:
- `mande-chaind --from {{KEY_NAME}} --chain-id mande-testnet-1 tx voting create-vote [validator_address_to_vote] [amount] [mode]`

- `amount` can be positive or negative (make sure, -1000000000 < amount < 1000000000, otherwise there might be unexpected behaviour which will be fixed in next phase), `mode` - 1 for cast, 0 for uncast.

- Cast Ex: `mande-chaind --from {{KEY_NAME}} --chain-id mande-testnet-1 tx voting create-vote mande... 10000000 1`

- Uncast Ex: `mande-chaind --from {{KEY_NAME}} --chain-id mande-testnet-1 tx voting create-vote mande... 10000000 0`

#### Additional node performance config
System requirements:
- Four or more CPU cores
- At least 100 GB of disk storage
- At least 4 GB of RAM

Increase file limit
```bash
ulimit -n 65536
```

Update ~/.mande-chain/config/config.toml
* log_level = "error"
* send_rate = 20000000
* recv_rate = 20000000
* max_packet_msg_payload_size = 10240
* flush_throttle_timeout = "50ms"
* mempool.size = 10000
* create_empty_blocks = false
* indexer = "null" [If you are not running explorers using same rpc]
* persistent_peers = "ee8a1b98e931e81d32c52f0b489fa22b52778d7c@34.171.132.212:26656,6780b2648bd2eb6adca2ca92a03a25b216d4f36b@34.170.16.69:26656" [Verify]

Delete ~/.mande-chain/config/addrbook.json [It will be generated freshly]
