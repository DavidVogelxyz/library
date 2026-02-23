Ansible - basic inventory
=========================

[Back to Ansible main page](README.md)

Table of contents
-----------------

- [Inventory](#inventory)
    - [Example basic inventory](#example-basic-inventory)
    - [Notes on inventory](#notes-on-inventory)
- [Group variables](#group-variables)
    - [Example basic group vars](#example-basic-group-vars)
    - [Notes on group vars](#notes-on-group-vars)

Inventory
---------

### Example basic inventory

Refer to this following example file, which could be located at `inventory/upper_tier_1/inventory/main.yml`.

```yaml
---

upper_tier_1:
  children:
    mid_tier_1:
      children:
        host_1:
          hosts: host_1
          vars:
            ansible_user: user_1
```

### Notes on inventory

One good way to sort Ansible inventory is to have the root of the repo to contain a directory named `inventory`.

Within the `inventory` directory should be directories for the "top-level groups" (example: `inventory/upper_tier_1`). This is probably best thought of as different "sites". Each of these "top-level groups" (example: `upper_tier_1`) will have their own `inventory` directory; and, within that directory, there should be a `main.yml` file.

Each "top-level group" can have a multitude of subgroups (example: `mid_tier_1`). Many subgroups can be defined, and each subgroup can contain multiple subgroups.

Eventually, a host (example: `host_1`) will be defined. Instead of that host having `children`, as the groups do, the host instead has a `hosts` definition. For `host_1`, the definition `hosts: host_1` is telling Ansible the hostname at which to reach the host. So long as the `/etc/hosts` and/or `~/.ssh/config` files define these hostnames, then Ansible will be able to reach the host.

Host variables can also be set at this point; but, for ease of reading, host variables are often set elsewhere (see [notes on group vars](#notes-on-group-vars)). In the case of the example, the `ansible_user` (example: `user_1`) is the username that Ansible will attempt to connect to via SSH.

Group variables
---------------

### Example basic group vars

Refer to this following example file, which could be located at `inventory/upper_tier_1/group_vars/upper_tier_1/defaults.yml`.

```yaml
---

ansible_become_method: sudo
ansible_become_pass: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  <ENCRYPTED_CONTENT>
```

### Notes on group vars

The variables set in the example will be referenced by Ansible whenever a playbook is run against `inventory/upper_tier_1`, and a targeted host is included in the group `upper_tier_1`. Since this group encompasses all hosts, this will always happen.

Ansible will begin by reading `inventory/upper_tier_1/inventory/main.yml`, understand that the targeted hosts are in `upper_tier_1` and `mid_tier_1`, and will then read those groups' group variables (in that order). Therefore, any variable defined in `inventory/upper_tier_1/group_vars/upper_tier_1/defaults.yml` will be set for all hosts within `upper_tier_1`, unless overridden by a more specific group or host variable.

In that file, two variables have been set: `ansible_become_method` and `ansible_become_pass`.

`ansible_become_method` is the command that Ansible will call when a task includes `become: true`.

`ansible_become_pass` is the password that should be used when `become: true`.
