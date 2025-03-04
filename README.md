Ansible Role for Patroni on Debian
======================================

This Ansible playbook allows to deploy a Patroni cluster using the `patroni`
packages provided by Debian repository. It is a stripped down adaption of
https://github.com/credativ/ansible-playbook-patroni-debian as a role,
focusing on a setup with Etcd (v3) + VIP-Manager and thus simplifying the options a bit.

Those packages have been integrated into Debian's `postgresql-common`
framework and will look very similar to a regular stand-alone PostgreSQL
install on Debian. This is done by using `pg_createconfig_patroni` creating
setting the `manual` flag in the `/etc/postgresql/$PG_VERSION/$CLUSTER_NAME/start.conf`
config file. That will cause the postgres init script/systemd unit not to
start that cluster automatically. Instead `patroni` will take care to start
and manage that cluster.

The default topology consists of three nodes acting as DCS
(Distributed Consensus Store) nodes as well as PostgreSQL/Patroni nodes.
