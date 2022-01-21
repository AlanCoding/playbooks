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
ansible-playbook -i <inventory> -e debug_state=present debug_on.yml
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
ansible-playbook -i <inventory> image_distribute.yml
```

Now, if you run a job, it should start within a few seconds, without a minute
or more of delay.

If you delete the `.tar` file (image archive) then this will trigger re-exporting the image.
You need this if you need to pick up a change in the awx-ee image, but it's
more common that you recycle the dev environment containers.

Using a 3 node cluster and re-exporting the image, this playbook takes about 1.5 minutes.
