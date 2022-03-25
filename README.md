# SubNav Infra

## Fuji validator

## Local test network with Vagrant, Ansible and AvalancheGo

The steps bellow leverages [Ansible Avalanche Collection](https://github.com/Nuttymoon/ansible-avalanche-collection) to deploy a local virtual machines based Avalanche environment.

The main steps are:

- Create 5 virtual machines with [Vagrant](https://www.vagrantup.com/)
- Bootstrap a local Avalanche cluster with [Avalanche Go](https://github.com/ava-labs/avalanchego)
- Create a subnet and add it to the `whitelisted-subnets` of each node.
- Create a [subnet-evm](https://github.com/ava-labs/subnet-evm) blockchain in the subnet.

### With Ansible >= 2.11

```sh
# Install Ansible collection
ansible-galaxy collection install git+https://github.com/Nuttymoon/ansible-avalanche-collection.git

# Bootstrap network
ansible-playbook nuttymoon.avalanche.bootstrap_local_network -i inventories/local

# Create subnet with predefined private key
curl -X POST --data '{
	"jsonrpc": "2.0", "id": 1, "method" :"platform.importKey",
	"params" : {
		"username" :"ewoq", "password": "I_l1ve_@_Endor",
		"privateKey": "PrivateKey-fRj9WBGk1nYJGbY9CTpoyf8Hu58cr24j2WJwThePLas5bu8eU"
	}
}' -H 'content-type:application/json;' 192.168.10.11:9650/ext/bc/P

ansible-playbook nuttymoon.avalanche.create_local_subnet -i inventories/local \
  --extra-vars '{"subnet_control_keys": ["P-local15sjkrydv6nze23uguanupg4plgut9dzuvxq5je"]}'

# Whitelist subnet
ansible-playbook nuttymoon.avalanche.bootstrap_local_network -i inventories/local \
    -e avalanche_whitelisted_subnets=2omKR2BxV5AyuEuQUqEwGBkPdHBGiZFoZPdNSpJ6JbhguGPPVz

# Create blockchain
ansible-playbook nuttymoon.avalanche.create_local_blockchains -i inventories/local -e subnet_id=2omKR2BxV5AyuEuQUqEwGBkPdHBGiZFoZPdNSpJ6JbhguGPPVz
```

### With Ansible <= 2.10

Bellow are the same commands if your Ansible version is <= 2.10.

```sh
# Install Ansible collection
mkdir -p ansible_collections/nuttymoon
git clone https://github.com/Nuttymoon/ansible-avalanche-collection.git ansible_collections/nuttymoon/avalanche

# Bootstrap network
ansible-playbook ansible_collections/nuttymoon/avalanche/playbooks/bootstrap_local_network.yml -i inventories/local

# Create subnet with predefined private key
curl -X POST --data '{
	"jsonrpc": "2.0", "id": 1, "method" :"platform.importKey",
	"params" : {
		"username" :"ewoq", "password": "I_l1ve_@_Endor",
		"privateKey": "PrivateKey-fRj9WBGk1nYJGbY9CTpoyf8Hu58cr24j2WJwThePLas5bu8eU"
	}
}' -H 'content-type:application/json;' 192.168.10.11:9650/ext/bc/P

ansible-playbook ansible_collections/nuttymoon/avalanche/playbooks/create_local_subnet.yml -i inventories/local \
  --extra-vars '{"subnet_control_keys": ["P-local15sjkrydv6nze23uguanupg4plgut9dzuvxq5je"]}'

# Whitelist subnet
ansible-playbook ansible_collections/nuttymoon/avalanche/playbooks/bootstrap_local_network.yml -i inventories/local \
    -e avalanche_whitelisted_subnets=2omKR2BxV5AyuEuQUqEwGBkPdHBGiZFoZPdNSpJ6JbhguGPPVz

# Create blockchain
ansible-playbook ansible_collections/nuttymoon/avalanche/playbooks/create_local_blockchains.yml -i inventories/local \
    -e subnet_id=2omKR2BxV5AyuEuQUqEwGBkPdHBGiZFoZPdNSpJ6JbhguGPPVz
```
