# Ansible Role for Patroni on Debian

This Ansible playbook allows to deploy a Patroni cluster on Debian hosts,
using the official Debian repositories. It is a stripped down and then extended
adoption of https://github.com/credativ/ansible-playbook-patroni-debian as a role,
focusing on a setup with Etcd + VIP-Manager, simplifying the options on one hand,
but extending the capabilities on the other hand. It also deals with UFW firewall
rules and thus expects UFW to be used.

The target topology consists of three nodes acting as DCS
(Distributed Consensus Store) nodes as well as PostgreSQL/Patroni nodes.

## Install collections

How you install the present role in your ansible setup is up to you. The role also depends on the community.general.ufw module from the community.general collection (https://galaxy.ansible.com/ui/repo/published/community/general), which needs to be installed. With ansible-galaxy this could be done as follows:

`ansible-galaxy collection install community.general`

## Prepare your inventory

In your inventory you need to define the variable(s) `patroni_hosts` for the hosts that should be part of the Sentinel cluster(s). It is an array of the members of a particular cluster, e.g.:
```
patroni_hosts:
  - db-host-01
  - db-host-02
  - db-host-03
```
Note that these names are meant to be the inventory names, not the hostnames necessarily.

Furthermore, you need to specify in `patroni_vip` the virtual IP to be used. All other options are basically optional.

Nevertheless, please consider setting the `patroni_replication_pass` and `patroni_rewind_pass` variables, because the role uses placeholders as defaults.
If you don't want to store the passwords in plaintext in your ansible repository, consider using ansible-vault. You could vault a new password like so:

`ansible-vault encrypt_string -J -n 'patroni_replication_pass' 'p4ssw0rd'` (it will ask you for a vault password)

and then store the result in your inventory. Remember to use `--ask-vault-password` when running your playbooks.

## Final remarks

The *main* task has a few checks in place to prevent accidents. Most notably, the setup part will be skipped if any postgres processes are already running.
If you really want to rerun the setup, please stop these processes beforehand. Alternatively, you can ignore the check for running services by using `-e ignore_running=true` when running the play.
Nevertheless, the *main* task is really only meant as a oneshot setup. For further and repeated configuration changes, the `update_config` task should suffice. It uses a bunch of additional, optional, inventory variables, which are documented in `defaults/main.yml`.

A combined example playbook using both tasks could look like the following:

```
- name: Setup Patroni
  hosts: db
  tasks:
  - name: Setup Patroni
    ansible.builtin.include_role:
      name: patroni
  - name: Update configuration
    ansible.builtin.include_role:
      name: patroni
      tasks_from: update_config
```

A playbook for repeated config changes would then only contain the `update_config` task to update `pg_hba`, PSQL parameters and Patroni tags.
Helpers to create actual databases, users and other things within PostgreSQL are **not** provided by this role. Please consider the `community.postgresql`
collection, for example. However, what *is* provided is a simple task `find_leader`, which sets the boolean variable `is_patroni_leader`, indicating whether
a host is the cluster leader and thus able to write the database.
