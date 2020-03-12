# Ansible EventStore Role

An Ansible role that installs and configures a Linux machine to be used as an [EventStore server](https://eventstore.com).

[See usage examples here.](https://github.com/gsoft-inc/ansible-role-eventstore-example)

## Requirements

Ubuntu 14.04 (Trusty) LTS and up.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
eventstore_version: 5.0.6-1
eventstore_admin_password: changeit
eventstore_admin_new_password: null
eventstore_config_file: "{{ role_path }}/templates/eventstore.conf.j2"
```

## EventStore Configuration

By default, the configuration is barebone as follows:
```yaml
ClusterSize: 1
```

## Example Playbook

Modifying the configuration is as easy as creating your own YAML file and specifying the path with the `eventstore_config_file` variable. For example:

Example folder structure:

```
- playbook.yml
- files/eventstore.conf.j2
```

Example contents for `eventstore.conf.j2`:

** See [the official configuration documentation](https://eventstore.com/docs/server/command-line-arguments/index.html) for all the possible options. **

```yaml
ClusterSize: 3
RunProjections: All
ExtIp: {{ ansible_default_ipv4.address }}
```

Example playbook.yml

```yaml
- hosts: all
  roles:
    - eventstore
  vars:
    - eventstore_config_file: ./files/eventstore.conf.j2
```

## License

Copyright © 2020, GSoft inc. This code is licensed under the Apache License, Version 2.0. You may obtain a copy of this license at https://github.com/gsoft-inc/gsoft-license/blob/master/LICENSE.
