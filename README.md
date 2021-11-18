### Live patches for AWX and AAP controller

This is a collection of Ansible content to help speed up processes of
modifying and testing live servers.

#### Action - create a superuser

Before anything else, you will need a user. Fill in values at `vars/private_vars.yml`

```yaml
awx_username: admin
awx_password: password
awx_email: "foo@invalid.com"
```

Playbook example:

```
ansible-playbook -i ~/repos/tower-qa/inventory.patch prod_user.yml
```

#### Action - replace receptor binary

Tested on Fedora, go to receptor clone, check out the branch you need

```
sudo dnf install -y golang
make
```

This should populate the receptor binary at the top-level of the receptor checkout.
Assuming checkout is now at `~/repos/receptor/receptor` then

```
ansible-playbook -i ~/repos/tower-qa/inventory.patch -e receptor_bin=~/repos/receptor/receptor receptor_bin_swap.yml
```
