## How to create group_vars file for a new bridge deployment

If you deployed a new bridge and want it to be preconfigured for the future, you need to create an `.yml` file in `group_vars/` folder of this playbook.

Basically, you can use `sokol-kovan.yml` as a template:
```
### home side rpc
home_rpc_url: https://sokol.poa.network
home_rpc_port: 443

### foreign side rpc
foreign_rpc_url: https://kovan.infura.io/metamask
foreign_rpc_port: 443

### bridge configs
bridge_estimated_gas_cost_of_withdraw: 0
bridge_home_deploy_gas: 3000000
bridge_foreign_deploy_gas: 3000000
bridge_deposit_relay_gas: 3000000
bridge_withdraw_relay_gas: 3000000
bridge_withdraw_confirm_gas: 3000000

bridge_home_deploy_gas_price: 1000000000
bridge_foreign_deploy_gas_price: 1000000000
bridge_deposit_relay_gas_price: 1000000000
bridge_withdraw_relay_gas_price: 1000000000
bridge_withdraw_confirm_gas_price: 1000000000

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

bridge_home_contract_address: "0x9A3EE574D3945201064A4CfcE56deed0c576422e"
bridge_foreign_contract_address: "0x4c4C8BB1ea9d81543764e1e59bEc7773b1Df298e"
bridge_home_contract_deploy: 2079636
bridge_foreign_contract_deploy: 6964652
```

Let's examine available options:
* `*_rpc_url`: url of the rpc endpoint of the home-side of the bridge
* `*_rpc_port`: port to use (for https use 443)
* `bridge_*` options are directly related to the values in `config.toml` and `db.toml`, see `roles/bridge/templates/config.toml.j2` and `roles/bridge/templates/db.toml.j2` for more details.
* `bridge_authorities` should be set to an array of authorities addresses, one per line starting with dash `-`, like this:
```
bridge_authorities:
  - "0x123..."
  - "0xabc..."
```
