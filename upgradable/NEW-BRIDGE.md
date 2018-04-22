## How to create group_vars file for a new bridge deployment

If you deployed a new bridge and want it to be preconfigured for the future, you need to create an `.yml` file in `group_vars/` folder of this playbook.

Basically, you can use `sokol-kovan.yml` as a template:
```
### parity home side configs
home_name: "sokol"
home_bootnodes_url: "https://raw.githubusercontent.com/poanetwork/poa-chain-spec/sokol/bootnodes.txt"
home_chain_json_url: "https://raw.githubusercontent.com/poanetwork/poa-chain-spec/sokol/spec.json"
home_chain_name: "Sokol"
home_chain_folder_name: "Sokol"

### parity foreign side configs
foreign_name: "kovan"
foreign_chain_name: "kovan"
foreign_chain_folder_name: "kovan"

### bridge configs
bridge_estimated_gas_cost_of_withdraw: 0
bridge_deposit_relay_gas: 3000000
bridge_withdraw_relay_gas: 3000000
bridge_withdraw_confirm_gas: 3000000

bridge_authorities:
  - "0x006E27B6A72E1f34C626762F3C4761547Aff1421"
  - "0x006E27B6A72E1f34C626762F3C4761547Aff1421"
bridge_authorities_requires_signatures: 1

bridge_home_required_confirmations: 0
bridge_home_poll_interval: 2
bridge_home_request_timeout: 360
bridge_foreign_required_confirmations: 0
bridge_foreign_poll_interval: 2
bridge_foreign_request_timeout: 360

bridge_home_contract_bytecode: "6060..."
bridge_foreign_contract_bytecode: "6060..."
bridge_home_contract_address: "0x9A3EE574D3945201064A4CfcE56deed0c576422e"
bridge_foreign_contract_address: "0x4c4C8BB1ea9d81543764e1e59bEc7773b1Df298e"
bridge_home_contract_deploy: 2079636
bridge_foreign_contract_deploy: 6964652
```

Let's examine available options:
* `*_name` - value of this variable is not actually used anywhere, it's just for convenience of log messages
* `*_bootnodes_url` - url to a list of reserved peers that should be used when connecting to the network. Can be empty `''` in which case no reserved peers are used
* `*_chain_json_url` - url to chain genesis file of the network. When using a built-in network like kovan or foundation, this value should be empty `''`
* `*_chain_name`, `*_chain_folder_name` - network name and network folder name (may be set in genesis file in `dataDir` property). `chain_name` is the one used with `--chain` option, `chain_folder_name` is the one that is used by parity to store chain files. It is possible that these values are different, this is why there are two separate options. For example, look at [foundation genesis file](https://github.com/paritytech/parity/blob/master/ethcore/res/ethereum/foundation.json): while you can use `--chain foundation` or `--chain mainnet` to specify the network, actual folder created to store chain files is called `ethereum`. So for foundation you should use
```
foreign_chain_name: foundation
foreign_chain_folder_name: ethereum
```
If `dataDir` property is omitted in genesis file, use value from `name` (case sensitive!), so for POA Core network you should
```
home_chain_json_url: 'https://raw.githubusercontent.com/poanetwork/poa-chain-spec/core/spec.json'
home_bootnodes_url: 'https://raw.githubusercontent.com/poanetwork/poa-chain-spec/core/bootnodes.txt'
home_chain_name: core
home_chain_folder_name: Core
```
* `bridge_*` options are directly related to the values in `config.toml` and `db.toml`, see `roles/bridge/templates/config.toml.j2` and `roles/bridge/templates/db.toml.j2` for more details.
* `bridge_authorities` should be set to an array of authorities addresses, one per line like this:
```
bridge_authorities:
  - "0x123..."
  - "0xabc..."
```
Example config above assumes that test kovan-sokol bridge uses `home_signer_address` and `foreign_signer_address` as authorities, this is not a general case, so correct values should be provided here
* `bridge_*_contract_bytecode` - contracts bytecode should be copied here

When one of the two networks is large (e.g. foundation), it is reasonable to prevent playbook from waiting for parity nodes to sync. This can be done adding
```
wait_sync: no
```
Note that in this case bridge service is not started by the playbook and needs to be started later
