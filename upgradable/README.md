**WIP**

For development version, do the following:

## Deployment
0. create an Ubuntu 16.04 server and get it's IP address

1. create file `hosts.yml` from `hosts.yml.template`, replace `192.0.2.1` with your server's IP address, `ansible_user` with ssh user, empty strings with values for your node.

If you want to use custom binaries of bridge or parity, you need to append additional parameters:
```
bridge_bin_url: "https://"
bridge_bin_sha256: "0123..."

parity_bin_url: "https://..."
parity_bin_sha256: "0123..."
```
use `sha256sum` (linux) or `shasum -a 256` (mac) to calculate checksums

If you want to connect to a custom bridge contract, append its bytecode, address and block numbers:
```
bridge_home_contract_bytecode: "6060..."
bridge_foreign_contract_bytecode: "6060..."
bridge_home_contract_address: "0x01234..."
bridge_foreign_contract_address: "0x01234..."
bridge_home_contract_deploy: 1768893
bridge_foreign_contract_deploy: 6715777
```
You can alternatively set these values in `group_vars/sokol-kovan.yml`

2. to install an authority node, run:
```
ansible-playbook -i hosts.yml authority-node.yml
```

3. to install a node with bridge ui:
```
ansible-playbook -i hosts.yml ui-node.yml
```
it will be available on http://localhost:3000

## Details
1. Playbook creates folder `poa-bridge` with subfolders `parity-home`, `parity-foreign` and `bridge`.
2. parity logs can be found in `parity-home/logs/parity.log` subfolder, or with journalctl: `journalctl -f -u parity-home`
3. bridge logs can be obtained with journalctl: `journalctl -f -u bridge` or in system logs: `tail -F /var/log/syslog | grep bridge`
