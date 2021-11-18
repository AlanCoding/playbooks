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
