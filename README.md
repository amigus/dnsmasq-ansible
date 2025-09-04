# Dnsmasq Ansible Collection

[![Build and Publish Collection](https://github.com/amigus/dnsmasq-ansible/actions/workflows/release.yml/badge.svg)](https://github.com/amigus/dnsmasq-ansible/actions/workflows/release.yml)

An ansible collection containing a set of roles that configures [Dnsmasq](https://dnsmasq.org/doc.html).
`dnsmasq_dhcp` configures DHCP options and tags, as well as IPv4 ranges and reservations.
`dnsmasq_dhcp_db` adds [SQLite3](https://sqlite.org/) DHCP lease management via `dhcp-script`.
`dnsmasq_dns` configures DNS resolver options, including upstream servers, and hosts file.
`dnsmasq_install` installs Dnsmasq and configures the system to use it.
`dnsmasq_web` adds [dnsmasq-web](https://github.com/amigus/dnsmasq-web) for REST API management.

## Roles

### dnsmasq

Basic Dnsmasq installation and configuration role. This role provides the foundation for other Dnsmasq roles in this collection.

See [roles/dnsmasq/README.md](roles/dnsmasq/README.md) for detailed documentation.

### dnsmasq_install

Installs and configures Dnsmasq on the target system using _apk_, _dnf_ and _zypper_.

See [roles/dnsmasq_install/README.md](roles/dnsmasq_install/README.md) for detailed documentation.

### dnsmasq_dhcp

Manages DHCP configuration including dhcp-hostsdir, dhcp-hostsfile, dhcp-ignore, dhcp-range, dhcp-options, dhcp-mac, and domain settings.

It will run when `dnsmasq_dhcp_interfaces` is defined.

See [roles/dnsmasq_dhcp/README.md](roles/dnsmasq_dhcp/README.md) for detailed documentation.

### dnsmasq_dhcp_db

Manages the optional lease database, associated script, and configuration. Contains a shell script and SQLite3 database schema for lease management.

It will run when `dnsmasq_dhcp_db` is defined.

See [roles/dnsmasq_dhcp_db/README.md](roles/dnsmasq_dhcp_db/README.md) for detailed documentation.

### dnsmasq_dns

Manages DNS host and server configuration including interface, addn-hosts, server, and rev-server settings.

It will run when `dnsmasq_dns_servers` is defined.

See [roles/dnsmasq_dns/README.md](roles/dnsmasq_dns/README.md) for detailed documentation.

### dnsmasq_web

Installs the [dnsmasq-web](https://github.com/amigus/dnsmasq-web) REST API for managing DHCP leases and querying client data.

It will run when `dnsmasq_web_enable` is `true`.

See [roles/dnsmasq_web/README.md](roles/dnsmasq_web/README.md) for detailed documentation.

## Examples

Here are some quick examples to get you started.
For detailed configuration options, see the individual role README files.

### Minimal DHCP and DNS server

It offers itself as the network gateway and leases the entire subnet starting at .10.
It offers recursive DNS by passing queries to the system resolver.

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dhcp_interfaces: [{ device: eth0, start: 10 }]
    dnsmasq_dns_options: [bogus-priv, domain-needed]
  roles:
    - amigus.dnsmasq.dnsmasq
```

### DHCP and DNS server that offers Google DNS to non-light bulbs

It offers .1 as the network gateway and leases addresses 100-200.
It uses Google for DNS but offers a different (local) resolver to WiZ light bulbs.

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dhcp_client_options:
      - tag:wizbulb,option:dns-server,192.168.1.253
    dnsmasq_dhcp_hosts: |
      6c:29:90:02:04:06,192.168.1.200,wiz_020406
    dnsmasq_dhcp_interfaces: [{ device: eth0, router: 1, start: 100, end: 199 }]
    dnsmasq_dhcp_mac_tags:
      - wizbulb: [6c:29:90:*:*:*, 44:4f:8e:*:*:*]
    dnsmasq_dns_options: [bogus-priv, domain-needed, no-resolv]
    dnsmasq_dns_servers: [{address: 8.8.8.8}, {address: 4.4.4.4}]
  roles:
    - amigus.dnsmasq.dnsmasq
```

### DHCP and DNS server with a DHCP lease database and REST API

It uses a lease database and installs dnsmasq-web for remote management.
Queries for `dmz.lan` domain or `192.168.100.*` will forward to `192.168.100.3`.
All other queries will go to `1.1.1.1`.

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dhcp_db: /var/lib/misc/dnsmasq.leases.db
    dnsmasq_dhcp_domain: servers.lan
    dnsmasq_dhcp_hosts_dir: /var/lib/misc/dnsmasq.hosts.d
    dnsmasq_dhcp_interfaces: [{ device: eth0, router: 1, start: 5 }]
    dnsmasq_dns_hosts: |
      192.168.1.2 dhcp.servers.lan
      192.168.1.3 dns.servers.lan
      192.168.1.4 dns2.servers.lan
    dnsmasq_dns_options: [bogus-priv, domain-needed, no-resolv]
    dnsmasq_dns_servers:
      - address: 1.1.1.1
      - domain: dmz.lan
        network: 192.168.100.0/24
        address: 192.168.100.3
    dnsmasq_web_binary: /usr/local/sbin/dnsmasq-web
  roles:
    - amigus.dnsmasq.dnsmasq
```
