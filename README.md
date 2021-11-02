# Ansible EventStoreDB Role

An Ansible role that installs and configures a Linux machine to be used as an [EventStoreDB server](https://eventstore.com).

[See usage examples here.](https://github.com/gsoft-inc/ansible-role-eventstore-example)

## Requirements

Ubuntu 14.04 (Trusty) LTS and up.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
eventstore_version: 20.10.4
eventstore_admin_password: changeit
eventstore_admin_new_password: null
eventstore_config_file: "{{ role_path }}/templates/eventstore.conf.j2"
eventstore_wait_for_http: true
eventstore_wait_for_http_timeout: 30
eventstore_generate_node_certitificate: false
eventstore_certificate_path: /etc/eventstore/certs
eventstore_ca_key_path: ./ca/ca.key
eventstore_ca_cert_path: ./ca/ca.crt
eventstore_node_cert_days: 365
eventstore_node_cert_out: ./node
eventstore_node_cert_ip_addresses:
  - 127.0.0.1
  - "{{ ansible_default_ipv4.address }}"
eventstore_node_cert_dns_names:
  - localhost
  - "{{ inventory_hostname }}"
```

## Configuration

By default, the configuration file (_eventstore.conf_) is barebone as follows:
```yaml
---
ClusterSize: 1
```

## Security

Unlike previous versions, EventStoreDB v20+ is secure by default. It means that you have to supply valid certificates and configuration for the database node to work.

**This role assumes the CA certificate is already present our your machine.**  [See this repo for examples](https://github.com/gsoft-inc/ansible-role-eventstore-example) on how you could provision your certs before executing this role.

### Node certificates

If you wish to generate the node certificates with the role, set `eventstore_generate_node_certitificate: true`.
Certificate generation will follow the default values as follows (overridable):
```yaml
eventstore_certificate_path: /etc/eventstore/certs
eventstore_ca_key_path: ./ca/ca.key
eventstore_ca_cert_path: ./ca/ca.crt
eventstore_node_cert_days: 365
eventstore_node_cert_out: ./node
eventstore_node_cert_ip_addresses:
  - 127.0.0.1
  - "{{ ansible_default_ipv4.address }}"
eventstore_node_cert_dns_names:
  - localhost
  - "{{ inventory_hostname }}"
```
This will in turn generate the following files:
```
/etc/eventstore/certs/node/node.key
/etc/eventstore/certs/node/node.crt
```
with SANs matching loopback IP address/DNS, default IP address (eth0 interface) and hostname DNS.

## Example Playbook

Modifying the configuration is as easy as creating your own YAML file and specifying the path with the `eventstore_config_file` variable. For example:

Example folder structure:

```
- playbook.yml
- files/eventstore.conf.j2
```

Example contents for `eventstore.conf.j2`:

** See [the official configuration documentation](https://developers.eventstore.com/server/v20.10/introduction/#getting-started) for all the possible options. **

```yaml
# Certificates configuration
CertificateFile: /etc/eventstore/certs/node/node.crt
CertificatePrivateKeyFile: /etc/eventstore/certs/node/node.key
TrustedRootCertificatesPath: /etc/eventstore/certs/ca

# Network configuration
IntIp: {{ ansible_default_ipv4.address }}
ExtIp: {{ ansible_default_ipv4.address }}
EnableExternalTcp: true
EnableAtomPubOverHTTP: true

# Cluster gossip
ClusterSize: 3
DiscoverViaDns: true
ClusterDns: eventstore.dns

# Projections configuration
RunProjections: All
```

Example playbook.yml

```yaml
- hosts: all
  roles:
    - eventstore

  vars:
    eventstore_generate_node_certitificate: true
    eventstore_config_file: ./files/eventstore.conf.j2
    eventstore_ca_key: "{{ lookup('file', 'ca/ca.key') }}"
    eventstore_ca_cert: "{{ lookup('file', 'ca/ca.crt') }}"
    eventstore_node_cert_ip_addresses:
      - 127.0.0.1
      - "{{ ansible_default_ipv4.address }}"
    eventstore_node_cert_dns:
      - localhost
      - "{{ inventory_hostname }}"
      - eventstore.dns
  pre_tasks:
    - name: Ensure eventstore group exists
      ansible.builtin.group:
        name: eventstore
        state: present

    - name: Ensure eventstore user exists
      ansible.builtin.user:
        name: eventstore
        state: present

    - name: Create eventstore ca cert folder
      ansible.builtin.file:
        path: /etc/eventstore/certs/ca
        state: directory

    - name: Add eventstore ca cert files
      ansible.builtin.copy:
        content: "{{ item.content }}"
        dest: "{{ item.dest }}"
        owner: eventstore
        group: eventstore
        mode: '0600'
      no_log: true
      loop:
        - { content: "{{ eventstore_ca_key }}", dest: /etc/eventstore/certs/ca/ca.key }
        - { content: "{{ eventstore_ca_cert }}", dest: /etc/eventstore/certs/ca/ca.crt }

    - name: Trust eventstore ca cert
      ansible.builtin.copy:
        content: "{{ eventstore_ca_cert }}"
        dest: /usr/local/share/ca-certificates/eventstore-ca.crt
        mode: '0644'
      notify:
        - update trusted ca

  handlers:
    - name: update trusted ca
      shell: /usr/sbin/update-ca-certificates
```

## License

Copyright © 2021, GSoft inc. This code is licensed under the Apache License, Version 2.0. You may obtain a copy of this license at https://github.com/gsoft-inc/gsoft-license/blob/master/LICENSE.