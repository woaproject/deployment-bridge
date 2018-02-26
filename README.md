## Ansible playbooks for deploying cross-chain bridges

**This is work in progress**
At the current moment trying to replicate functionality of https://github.com/poanetwork/parity-bridge-research

At the present moment this playbook is to perform initial deployment of the bridge and parity services. If you want to run it against existing installation some files might be overwritten, please at least set `initial_deployment: no` in `hosts.yml`.

### Requirements
* on local machine: python, ansible
* on remote: ubuntu server 16.04, python 3

### Deployment process
1. create `hosts.yml` file based on the provided example:
```
cp example-hosts.yml hosts.yml
```
fill all required fields, replace `192.0.2.1` with ip address of your remote ubuntu server.
The following file can be used with parity-bridge-research repository mentioned above:
```
---
all:
  hosts:
    192.0.2.1:
      ansible_user: ubuntu

      ### parity configs
      home_chain_json: "https://raw.githubusercontent.com/poanetwork/parity-bridge-research/master/erc20/PoA_home.json"
      home_chain_name: "PoA_home"
      home_signer_address: "0x842eb2142c5aa1260954f07aae39ddee1640c3a7"
      home_signer_keyfile: '{"id":"7586acc4-a46d-1a69-5615-13596cceb982","version":3,"crypto":{"cipher":"aes-128-ctr","cipherparams":{"iv":"d5bf770f4b18845f67bad272a50720ad"},"ciphertext":"e97c949c28f0e30778359d7ad4ebf023b10a8bd20b023a0dbdae99a89159f96b","kdf":"pbkdf2","kdfparams":{"c":10240,"dklen":32,"prf":"hmac-sha256","salt":"c61d0e8e2d8c74a676eaf32745eb54db0cf75f973d2e2364121fca32a144a605"},"mac":"a61278fdcbcabc19c90fb52a5022bb246024f4ff1660cf961135af51da1b5296"},"address":"842eb2142c5aa1260954f07aae39ddee1640c3a7","name":"","meta":"{}"}'
      home_signer_password: "11"

      foreign_chain_json: "https://raw.githubusercontent.com/poanetwork/parity-bridge-research/master/erc20/PoA_foreign.json"
      foreign_chain_name: "PoA_foreign"
      foreign_signer_address: "0xf3ee321df87781864f46f6464e764c2827fca73b"
      foreign_signer_keyfile: '{"id":"7f1aae19-e63f-fdab-e917-ca01dbffa25a","version":3,"crypto":{"cipher":"aes-128-ctr","cipherparams":{"iv":"f568bfdfc7c4346997c55685b97856bd"},"ciphertext":"a253fb25e484551fa90e50c503cf2b288082e6138efc39d9b95214f2e6b451dd","kdf":"pbkdf2","kdfparams":{"c":10240,"dklen":32,"prf":"hmac-sha256","salt":"8c884d52dd3d573716938a0c0db95a2a89aad77e6c13f82a0bb1e36c72860c9c"},"mac":"fea53c70a3a265a7b618a5839cd51affce4120781cda6a7011ade73b8bb34c45"},"address":"f3ee321df87781864f46f6464e764c2827fca73b","name":"","meta":"{}"}'
      foreign_signer_password: "11"

      ### bridge config
      initial_deployment: yes
      db_toml_location: ""

      home_required_confirmations: 0
      home_poll_interval: 2
      home_request_timeout: 360

      foreign_required_confirmations: 0
      foreign_poll_interval: 2
      foreign_request_timeout: 360

      authorities:
        - "0xf3ee321df87781864f46f6464e764c2827fca73b"
      required_signatures: 1

      ### tests
      actor_address: "0x37a30534da3d53aa1867adde26e114a3161b2b12"
      actor_keyfile: '{"id":"8437ae14-1e28-75d3-4512-a60d63dbfb64","version":3,"crypto":{"cipher":"aes-128-ctr","cipherparams":{"iv":"6ac9cafc4afa4ec408c9e72d99d77b7d"},"ciphertext":"d4c72972233cadf95e3fda4598692cb08461807ebcfa153dc123b48de438bbdb","kdf":"pbkdf2","kdfparams":{"c":10240,"dklen":32,"prf":"hmac-sha256","salt":"c1fa041558f35b0ff17aa1e0ab49b71ecceff739a2f6f87d7abc8eb0661df3c5"},"mac":"d968ac0bad6361192c4ab1c2f96fade3eab05de3c7edd7dfb86b686de2a6429c"},"address":"37a30534da3d53aa1867adde26e114a3161b2b12","name":"","meta":"{}"}'
      actor_password: "11"

      ### bridge_ui
      ### !!! exposes unsafe apis!!! don't use with prod networks (mainnet, core)
      install_bridge_ui: no
```

2. run the playbook (if `ansible_user` cannot execut `sudo` without password, add `--ask-sudo-password` flag below)
```
ansible-playbook -i hosts.yml site.yml [--ask-sudo-password]
```

3. (optional) run tests on the remote
```
ansible-playbook -i hosts.yml tests.yml [--ask-sudo-password]
```

### Deployment details
1. the following folders and files are created by the playbook in `ansible_user`'s home directory (`*` means executable file):
```
.
└── poa-bridge
    ├── bridge
    │   ├── bridge*
    │   ├── config.toml
    │   ├── contracts
    │   │   ├── bridge.sol
    │   │   ├── ERC20.abi
    │   │   ├── ERC20.bin
    │   │   ├── ForeignBridge.abi
    │   │   ├── ForeignBridge.bin
    │   │   ├── foreign_tokenreg.py
    │   │   ├── Helpers.abi
    │   │   ├── Helpers.bin
    │   │   ├── HelpersTest.abi
    │   │   ├── HelpersTest.bin
    │   │   ├── HomeBridge.abi
    │   │   ├── HomeBridge.bin
    │   │   ├── Message.abi
    │   │   ├── Message.bin
    │   │   ├── MessageSigning.abi
    │   │   ├── MessageSigning.bin
    │   │   ├── MessageSigningTest.abi
    │   │   ├── MessageSigningTest.bin
    │   │   ├── MessageTest.abi
    │   │   └── MessageTest.bin
    │   ├── db.toml
    │   ├── solc*
    │   ├── tests
    │   │   ├── home_deposit.py
    │   │   ├── test_env_db.toml
    │   │   ├── token_balance.py
    │   │   └── token_withdraw.py
    │   └── token
    │       ├── ApproveAndCallFallBack.abi
    │       ├── ApproveAndCallFallBack.bin
    │       ├── BridgeableToken.abi
    │       ├── BridgeableToken.bin
    │       ├── BridgeableToken.sol
    │       ├── ERC20.abi
    │       ├── ERC20Basic.abi
    │       ├── ERC20Basic.bin
    │       ├── ERC20.bin
    │       ├── MintableToken.abi
    │       ├── MintableToken.bin
    │       ├── Ownable.abi
    │       ├── Ownable.bin
    │       ├── SafeMath.abi
    │       ├── SafeMath.bin
    │       ├── SafeMathLib.abi
    │       ├── SafeMathLib.bin
    │       ├── StandardToken.abi
    │       ├── StandardToken.bin
    │       └── token_foreign.py
    ├── bridge-ui
    │   ├── build
    │   │   ├── asset-manifest.json
    │   │   ├── favicon.ico
    │   │   ├── index.html
    │   │   ├── manifest.json
    │   │   ├── service-worker.js
    │   │   └── static
    │   ├── example.png
    │   ├── package.json
    │   ├── package-lock.json
    │   ├── public
    │   │   ├── favicon.ico
    │   │   ├── index.html
    │   │   └── manifest.json
    │   ├── README.md
    │   ├── src
    │   │   ├── App.css
    │   │   ├── App.js
    │   │   ├── App.test.js
    │   │   ├── erc20.abi.json
    │   │   ├── foreignAbi.json
    │   │   ├── homeAbi.json
    │   │   ├── index.css
    │   │   ├── index.js
    │   │   ├── logo.svg
    │   │   └── registerServiceWorker.js
    │   └── yarn.lock
    ├── foreign
    │   ├── chain.json
    │   ├── node.toml
    │   ├── parity*
    │   ├── parity_data
    │   │   ├── cache
    │   │   ├── chains
    │   │   ├── dapps
    │   │   ├── jsonrpc.ipc
    │   │   ├── keys
    │   │   ├── network
    │   │   └── signer
    │   └── pass.pwd
    ├── helpers
    │   └── bridge-ui-env
    │       ├── make-env.js
    │       └── package-lock.json
    └── home
        ├── chain.json
        ├── node.toml
        ├── parity*
        ├── parity_data
        │   ├── cache
        │   ├── chains
        │   ├── dapps
        │   ├── jsonrpc.ipc
        │   ├── keys
        │   ├── network
        │   └── signer
        └── pass.pwd
```

`bridge` and `solc` binary are placed in `bridge` subfolder
each parity subfolder has its own copy of `parity` binary (in case different version are required by different networks)

2. if `initial_deployment` in `hosts.yml` is set to `yes` token contract is compiled and deployed to the foreign network using `token_foreign.py` and then registered with `foreign_tokenreg.py`. If `initial_deployment` is set to `no` then `db.toml` is copied from local machine (`db_toml_location`)

3. if `install_bridge_ui` is set to `yes` playbook also installs bridge-ui https://github.com/poanetwork/bridge-ui (branch can be configured with `bridge_ui_branch`). React files are process via `npm run build` and then served as static files using `http-server` and `pm2`.
**NOTE**: installing bridge-ui exposes some unsafe apis to public, so do NOT install bridge-ui if you plan to use prod networks like mainnet or core in your bridge setup.

4. playbook installs 3 services: `parity-home`, `parity-foreign` and `bridge`. Status of a service can be checked with
```
sudo systemctl status [service-name]
```
Services can be restarted with
```
sudo systemctl restart [service-name]
```
Logs are collected by systemd to `/var/log/syslog` and can be viewed with
```
tail -F /var/log/syslog | grep [service-name]
```

5. `tests` run the following scenario
    1. deposit @ home: `home_deposit.py`
    2. check token balance after deposit: `token_balance.py`
    3. withdraw from foreign: `token_withdraw.py`
    4. check token balance after withdraw: `token_balance.py`

ansible's output is unfortunately not too readable
