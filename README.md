### Live patches for AWX and AAP controller

This is a collection of Ansible content to help speed up processes of
modifying and testing live servers.

#### Configuration of inventory file

These playbooks require a valid AAP inventory file.
This will have groups like `automationcontroller` and `execution_nodes`.

If using the docker-compose inventory file, there is an inventory script floating
around which will build an inventory of this format from `docker ps -a` data.

#### Action - create a superuser

Before anything else, you will need a user. Fill in values at `vars/private_vars.yml`

```yaml
awx_username: admin
awx_password: password
awx_email: "foo@invalid.com"
```

Playbook example:

```
ansible-playbook -i <inventory> prod_user.yml
```

#### Action - turn on debugging settings

You want to run a job and have the server preserve the job folders and work unit.

```
ansible-playbook -i ~/repos/tower-qa/inventory.patch -e debug_state=present debug_on.yml
```

Switch `debug_state=present` to absent to undo this.

#### Action - replace receptor binary

Tested on Fedora, go to receptor clone, check out the branch you need

```
sudo dnf install -y golang
make
```

This should populate the receptor binary at the top-level of the receptor checkout.
Assuming checkout is now at `~/repos/receptor/receptor` then

```
ansible-playbook -i <inventory> -e receptor_bin=~/repos/receptor/receptor receptor_bin_swap.yml
```
