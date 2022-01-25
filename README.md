### Live patches for AWX and AAP controller

This is a collection of Ansible content to help speed up processes of
modifying and testing live servers.

#### Configuration of inventory file

These playbooks require a valid AAP inventory file.
This will have groups like `automationcontroller` and `execution_nodes`.

If using the docker-compose inventory file, there is an inventory script floating
around which will build an inventory of this format from `docker ps -a` data.

Specifying the inventory file is going to get old, so this will pick up a symlink
at the following location `awx_inventory.ini`. For example, to set this up:

```
ln -s ~/repos/tower-qa/playbooks/inventory-docker.py awx_inventory.ini
```

You can still specify a different inventory with `-i` if you need.
All the following commands will assume this is set up.

#### Action - create a superuser

Before anything else, you will need a user. Fill in values at `vars/private_vars.yml`

```yaml
awx_username: admin
awx_password: password
awx_email: "foo@invalid.com"
```

Playbook example:

```
ansible-playbook prod_user.yml
```

#### Action - turn on debugging settings

You want to run a job and have the server preserve the job folders and work unit.

```
ansible-playbook -e debug_state=present debug_on.yml
```

Switch `debug_state=present` to absent to undo this.

#### Action - replace receptor binary

Before you go changing anything, you probably want to print the current versions
that all of the nodes have installed.

```
ansible-playbook --tags=version receptor_bin_swap.yml
```

Tested on Fedora, go to receptor clone, check out the branch you need

```
sudo dnf install -y golang
make
```

This should populate the receptor binary at the top-level of the receptor checkout.
Assuming checkout is now at `~/repos/receptor/receptor` then

```
ansible-playbook -e receptor_bin=~/repos/receptor/receptor receptor_bin_swap.yml
```

TODO: still need a playbook that will replace the `receptorctl` package.

#### Action - replace the ansible-runner install

This follows a similar pattern to replacing receptor.

```
ansible-playbook --tags=version runner_swap.yml
```

A typical pattern is that you would checkout ansible-runner from within
your awx directory and install it in-place.
Assuming you are doing an in-place install with it at `<awx>/testing/ansible-runner`

```
ansible-playbook -e new_package=file:///awx_devel/testing/ansible-runner runner_swap.yml
```

This will allow you to make changes to ansible-runner and have them take
effect in multiple kinds of containers without re-installing.

#### Action - pre-populate image cache with EE

This allows you to distribute an image to all the nodes in the cluster.
Doing this will avoid a long delay before jobs start due to pulling the image.
Most often, you will want to use this to distribute the control plane EE
or the default EE for jobs / ad hoc commands / inventory updates.

Start by assuring that you have the EE locally (or that it's not old).

```
podman pull quay.io/ansible/awx-ee:latest
```

Then run the playbook:

```
ansible-playbook image_distribute.yml
```

Now, if you run a job, it should start within a few seconds, without a minute
or more of delay.

If you delete the `.tar` file (image archive) then this will trigger re-exporting the image.
You need this if you need to pick up a change in the awx-ee image, but it's
more common that you recycle the dev environment containers.

Using a 3 node cluster and re-exporting the image, this playbook takes about 1.5 minutes.
