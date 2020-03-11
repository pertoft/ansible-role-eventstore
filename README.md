# Ansible EventStore Role

An Ansible role that installs and configures a Linux machine to be used as an [EventStore server](https://eventstore.com).

[See usage examples here.](https://github.com/gsoft-inc/ansible-role-eventstore-example)

## Requirements

Ubuntu 14.04 (Trusty) LTS and up.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

**Important: null values default to the inherent default values defined by EventStore.  See documentation for more information.**
```yaml
# Role specific variables
eventstore_version: 5.0.6-1
eventstore_admin_password: changeit
eventstore_admin_new_password: null

# Cluster options variables
# See https://eventstore.com/docs/server/command-line-arguments/index.html#cluster-options
eventstore_cluster_size: null
eventstore_discover_via_dns: null
eventstore_cluster_dns: null
eventstore_cluster_gossip_port: null
eventstore_gossip_seed: null
eventstore_cluster_gossip_timeout: null
eventstore_cluster_gossip_interval: null

# See https://eventstore.com/docs/server/command-line-arguments/index.html#database-options
eventstore_db: null
eventstore_prepare_timeout: null
eventstore_commit_timeout: null

# See https://eventstore.com/docs/server/command-line-arguments/index.html#interface-options
eventstore_ext_ip: "127.0.0.1"
eventstore_ext_ip_advertise_as: null
eventstore_int_ip: null
eventstore_int_tcp_port: null
eventstore_ext_tcp_port: null
eventstore_int_http_port: 2113
eventstore_ext_http_port: 2113
eventstore_int_http_prefixes: null
eventstore_ext_http_prefixes: null
eventstore_add_interface_prefixes: null
eventstore_int_tcp_heartbeat_timeout: null
eventstore_int_tcp_heartbeat_interval: null
eventstore_ext_tcp_heartbeat_timeout: null
eventstore_ext_tcp_heartbeat_interval: null

# See https://eventstore.com/docs/server/command-line-arguments/index.html#projections-options
eventstore_run_projections: null
eventstore_projection_threads: null
```

## Example Playbook

```yaml
- hosts: all
  roles:
    - eventstore
  vars:
    eventstore_admin_new_password: 'please_use_ansible_vault'
```

## License

Copyright © 2020, GSoft inc. This code is licensed under the Apache License, Version 2.0. You may obtain a copy of this license at https://github.com/gsoft-inc/gsoft-license/blob/master/LICENSE.
