# Overview

This playbooks are designed to automate deployment process of cross-chain bridges. It installs bridge as a service and configures .env file.

# Prerequisites

Playbooks automatically installs `Docker`, `docker-compose`, `Python`, `Git` and it dependencies (such as `curl`, `ca-certificates`, `apt-transport-https`, etc.). To launch playbooks [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) should be installed.

## Configuration

Playbooks has two points of configuration - file `hosts.yml` and `group_vars/<bridge_name>.yml`. 

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

You can check `hosts.yml.example` as example. `hosts.yml` can contain multiple hosts and bridge configurations (groups) at once.

`group_vars/<bridge_name>.yml`  should contain public bridge parameters. `group_vars/example.yml` contains example configuration for PoA/Sokol - PoA/Sokol bridge.

## Execution

Once all configuration files and [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) is installed playbooks can be executed:

```yaml
ansible-playbook site.yml
```

Useful arguments:

`--ask-pass` - ask password that will be used to connect to bridge VM.

`--ask-become-pass` - ask `become` password that will be used to execute some commands (such as Docker installation) with root privileges.

`-i <file>` - use specified file as a `hosts.yml` file.

`-e "<variable>=<value>"` - override default variable.

Useful variables:

`bridge_path` - absolute path where bridge will be installed. Defaults to /home/<ansible_user>/bridge.

`bridge_repo` - path to bridge repo.

`docker_compose_version` - specify a version of docker-compose to be installed.

# Working with the Bridge service

Bridge will be installed as a service with the following name: `poabridge`. You can use default `SysVinit` commands to `start`, `stop`, `restart`,`rebuild` service and to check `status` of the service also. Use command in the following format:
```bash
service [start|stop|restart|status|rebuild] poabridge
```
