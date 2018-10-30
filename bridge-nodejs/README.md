### Prerequisites
1. Launch an Ubuntu 16.04 server on your favorite hosting provider and note its IP address. You should setup ssh access to your node via public+private keys (using passwords is less secure). When creating the node, set a meaningful `hostname` that can identify you (e.g. `validator-0x...`).

2. On your local machine install
    * Python 2 (v2.6-v2.7)/Python3 (v3.5+)
    * Ansible v2.3+
    * Git

### Configuration

1. Clone this repository and go to the `bridge-nodejs` folder
```
git clone https://github.com/poanetwork/deployment-bridge.git
cd bridge-nodejs
```
2. Create the file `hosts.yml` from `hosts.yml.example`
```
cp hosts.yml.template hosts.yml
```

`hosts.yml` should have the following structure:

```yaml
<bridge_name>:
    hosts:
        <host_ip>:
            ansible_user: "<user>"
            VALIDATOR_ADDRESS: "<hex address>" 
            VALIDATOR_ADDRESS_PRIVATE_KEY: "<private_key>"
            #syslog_server_port: "<protocol>://<ip>:<port>" # When this parameter is set all bridge logs will be redirected to <ip>:<port> address.
```

`hosts.yml` can contain multiple hosts and bridge configurations (groups) at once.


3. Create a file for the public bridge parameters. 
   1. Go to the group_vars folder. 
   `cd group_vars`
   2. Copy the example file and rename with your <bridge_name>. For example, if the bridge name in hosts.yml is sokol-kovan, you would name the file sokol-kovan.yml
   `cp example.yml <bridge_name>.yml` 


`group_vars/<bridge_name>.yml` contains the public bridge parameters. `group_vars/example.yml` shows an example configuration for the POA/Sokol - POA/Sokol bridge. Parameter values should match values from the .env file for the token-bridge. See https://github.com/poanetwork/token-bridge#configuration-parameters for details.


## Execution

Playbooks can be executed once [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) is installed and all configuration variables are complete. 

```yaml
ansible-playbook -i hosts.yml site.yml
```

**Useful arguments:**

Used with ansible-playbook command, for example:

```yaml
 `ansible-playbook -i hosts.yml site.yml --ask-become-pass`
```

* `--ask-pass` - ask for the password used to connect to the bridge VM.

* `--ask-become-pass` - ask for the `become` password used to execute some commands (such as Docker installation) with root privileges.

* `-i <file>` - use specified file as a `hosts.yml` file.

* `-e "<variable>=<value>"` - override default variable.

* `--private-key=<file_name>` - if private keyfile is required to connect to the ubuntu instance.

**Useful variables:**

* `bridge_path` - absolute path where bridge will be installed. Defaults to /home/<ansible_user>/bridge.

* `bridge_repo` - path to the bridge repo.

* `docker_compose_version` - specify a version of docker-compose to be installed.


## Bridge service commands

The Bridge service is named `poabridge`. Use the default `SysVinit` commands to `start`, `stop`, `restart`, and `rebuild` the service and to check the `status` of the service. 

Commands format:
```bash
sudo service poabridge [start|stop|restart|status|rebuild]
```

## Logs

If the `syslog_server_port` option in the hosts.yml file was not set, all logs will be stored in containers and can be accessed via the default `docker-compose logs` command. To obtain logs, `cd` to the directory where the bridge is installed (`~/bridge` by default) and execute the `docker-compose logs` command.

If the `syslog_server_port` was set, logs can be obtained from the specified server.

```yaml 
syslog_server_port: "<protocol>://<ip>:<port>" # When this parameter is set all bridge logs will be redirected to <ip>:<port> address.
```