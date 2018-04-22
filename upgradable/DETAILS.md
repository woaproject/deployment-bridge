Bridge-specific parameters should be stored in `group_vars/$BRIDGE_NAME.yml` so that they can be easily reused by selecting suitable bridge name in `hosts.yml`. However, for one-time testing purposes, these parameters can also be set directly in `hosts.yml` alongside other node-specific parametrs.

## Details of the authority node setup
Installation consists of 4 parts:

### 1. Preparing
1. A new user without sudo access is created. By default it's named `bridgeuser`, but can be controlled by `service_user` variable in `authority_node.yml`

2. UFW is configured to allow inbound tcp connections only on ssh port (`22` by default) and tcp/udp connections on p2p ports:
    * `30303` (so that external ethereum nodes could communicate with bridge node, assuming they use default port)
    * `home_p2p_port` (default value `30303`)
    * `foreign_p2p_port` (default value `40303`)

3. All outbound connections are allowed

4. If NTP is not installed, then Chrony is installed to sync time. Playbook tries to determine if it runs on AWS node, in which case it uses Amazon time server

5. Syslog forwarding to remote server is setup by editing `/etc/rsyslog.conf` file. This is done only if `syslog_server_port` is not empty

6. Binaries and configuration files will be stored in the bridgeuser's home directory in `poa-bridge` folder, with the following structure:
```
.
├── poa-bridge/
│   ├── bridge
│   │   ├── bridge*
│   │   ├── config.toml
│   │   ├── db.toml
│   │   ├── ForeignBridge.bin
│   │   └── HomeBridge.bin
│   ├── foreign-node/
│   │   ├── logrotate.conf
│   │   ├── logs/
│   │   ├── node.toml
│   │   ├── parity*
│   │   ├── parity_data/
│   │   └── pass.pwd
│   └── home-node/
│       ├── bootnodes.txt
│       ├── logrotate.conf
│       ├── logs/
│       ├── node.toml
│       ├── parity*
│       ├── parity_data/
│       ├── pass.pwd
│       └── sokol.json
```
here `*` means executable file, `/` means folder. Parity binary is downloaded both to home-node folder and foreign-node folder in case different versions might be required.

### 2. Setting up home-side parity node
1. Parity logs are stored in `logs/parity.log`, rotated based on size (200 MB threshold) to `old` subfolder. They can also be found in `/var/log/syslog`

2. Parity binary is downloaded from url specified in `parity_bin_url` variable (default value is set in `roles/parity/defaults/main.yml`). Binary's sha256-checksum is validated against the value from `parity_bin_sha256`. So when changing default parity version **both** `parity_bin_url` and `parity_bin_sha256` should be updated

3. Authority keys are put into `parity_data/keys/{{ home_chain_folder_name }}/{{ home_signer_address }}.json`. Content of keyfile is read from `home_signer_keyfile` variable, password is read from `home_signer_password`

4. If a custom network is used (not built-in like kovan or foundation), genesis file is downloaded from `chain_json_url` and (optionally) list of reserved peers from `bootnodes_url`

5. Parity configuration file `node.toml` is created based on template `roles/parity/templates/node.toml.j2`, example:
```
[parity]
chain = "sokol.json"
base_path = "parity_data"

[network]
port = 30303
discovery = true
nat = "extip:52.170.238.8"
reserved_peers = "bootnodes.txt"

[rpc]
port = 8545

[websockets]
port = 8546

[account]
unlock = ["0x842eb2142c5aa1260954f07aae39ddee1640c3a7"]
password = ["pass.pwd"]

[misc]
log_file = "logs/parity.log"
```
rpc and websockets are left open from local node, by default they only accept connections from `localhost`, also ufw denies incoming connections on `8545` and `8546` ports

6. Parity service is installed for `systemd` in `/etc/systemd/system/parity-home.service` based on `roles/parity/templates/parity.service.j2`, example:
```
[Unit]
Description=parity-home
After=network.target

[Service]
User=bridgeuser
Group=bridgeuser
WorkingDirectory=/home/bridgeuser/poa-bridge/home-node
ExecStart=/home/bridgeuser/poa-bridge/home-node/parity --config node.toml
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```
this makes parity-home service auto-start on startup and auto-restart if parity process fails. By default, delay between restarts is 3 seconds, this can be controlled by `restart_delay_sec` variable

### 3. Setting up foreign-side parity node
Analogous to home-side, except for variable namings change `*home* -> *foreign*`. Actually, the same `roles/parity` is used by the playbook twice with different sets of variables

### 4. Setting up bridge service
1. Bridge binary is downloaded from url specified in `bridge_bin_url` variable (default value is set in `roles/bridge/defaults/main.yml`). Binary's sha256-checksum is validated against the value from `bridge_bin_sha256`. So when changing default version **both** `bridge_bin_url` and `bridge_bin_sha256` should be updated

2. Contract bytecodes are read from `bridge_foreign_contract_bytecode` and `bridge_home_contract_bytecode` variables and stored in `ForeignBridge.bin` and `HomeBridge.bin`

3. Bridge `config.toml` is created based on `roles/bridge/templates/bridge.service.j2`, example:
```
estimated_gas_cost_of_withdraw = 0

[home]
account = "0x842eb2142c5aa1260954f07aae39ddee1640c3a7"
ipc = "/home/bridgeuser/poa-bridge/home-node/parity_data/jsonrpc.ipc"
required_confirmations = 0
poll_interval = 2
request_timeout = 360

[home.contract]
bin = "HomeBridge.bin"

[foreign]
account = "0x842eb2142c5aa1260954f07aae39ddee1640c3a7"
ipc = "/home/bridgeuser/poa-bridge/foreign-node/parity_data/jsonrpc.ipc"
required_confirmations = 0
poll_interval = 2
request_timeout = 360

[foreign.contract]
bin = "ForeignBridge.bin"

[authorities]
accounts = [
  "0x842eb2142c5aa1260954f07aae39ddee1640c3a7",
"0x842eb2142c5aa1260954f07aae39ddee1640c3a7"
]
required_signatures = 1

[transactions]
deposit_relay = { gas = 3000000 }
withdraw_relay = { gas = 3000000 }
withdraw_confirm = { gas = 3000000 }
```

4. Database `db.toml` file is created based on `roles/bridge/db.toml.j2`, example (in this case it will be the same for all newly-created nodes):
```
home_contract_address = "0xad1dae0320717a288912ff7bae766ac87e7d14a5"
foreign_contract_address = "0xfd03be9947cbecb14a1ae8729936e23af7a0b50b"
home_deploy = 1768893
foreign_deploy = 6715777
checked_deposit_relay = 1768893
checked_withdraw_relay = 6715777
checked_withdraw_confirm = 6715777
```
**OR** `db.toml` can be copied from local machine, in this case `db_toml_location` variable should be set in `hosts.yml` to absolute path of the file

5. Bridge service is installed for `systemd` so that it auto-start on startup and auto-restarts if bridge process fails. Bridge service starts after parity services are started. Example of `/etc/systemd/system/bridge.service`
```
[Unit]
Description=bridge
After=network.target

[Service]
User=bridgeuser
Group=bridgeuser
WorkingDirectory=/home/bridgeuser/poa-bridge/bridge
Environment=RUST_LOG=info
ExecStart=/home/bridgeuser/poa-bridge/bridge/bridge --config /home/bridgeuser/poa-bridge/bridge/config.toml --database /home/bridgeuser/poa-bridge/bridge/db.toml
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```
By default, restart delay is 3 seconds, this can be controlled by `restart_delay_sec` variable

6. Logs are stored in `/var/log/syslog`

7. During installation, playbook waits for parity nodes to sync before starting bridge for the first time. Sync-check is done by envoking `eth_syncing` rpc call against home and then against foreign nodes every 30 seconds. There is a limit of 120 attempts per node which is approximately 1 hour (so 1 hour for home and 2 hours for foreign). If a node is not synced within this limit, playbook will fail. This check can be turned off by setting `wait_sync` variable to `no` (**useful if syncing with foundation**), in this case playbook will continue, but bridge service won't be started. So it is recommended to periodically check sync status and manually start bridge service when sync is completed by
```
sudo systemctl start bridge
```

## Useful commands
1. Restart services:
```
sudo systemctl restart parity-home
sudo systemctl restart parity-foreign
sudo systemctl restart bridge
```
Replace `restart` with `start` or `stop` if needed

2. Get quick status of a service:
```
sudo systemctl status parity-home
sudo systemctl status parity-foreign
sudo systemctl status bridge
```
note if it's reported `active`, `running` or `dead`

3. Tail bridge logs from `/var/log/syslog`:
```
tail -F /var/log/syslog | grep bridge
```

4. Tail parity logs:
```
tail -F /home/bridgeuser/poa-bridge/home-node/logs/parity.log
tail -F /home/bridgeuser/poa-bridge/foreign-node/logs/parity.log
```

## URLs of precompiled binaries
1. for parity binary, update `parity_bin_url` and `parity_bin_sha256` from `roles/parity/defaults/main.yml`

2. for bridge binary, update `bridge_bin_url` and `bridge_bin_sha256` from `roles/bridge/defaults/main.yml`

