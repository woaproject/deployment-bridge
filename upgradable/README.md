1. create file `hosts.yml` from hosts.yml.template, replace empty strings with values for your node
2. to install an authority node install:
```
ansible-playbook -i hosts.yml authority-node.yml
```
3. to install a node with bridge ui:
```
ansible-playbook -i hosts.yml ui-node.yml
```
