# dnsmasq_dns

Manages DNS host and server configuration including:

- interface
- addn-hosts
- server
- rev-server

## DNS Configuration

The DNS roles are written to configure Dnsmasq as a resolver.
There is no support for authoratative domains.
However `auth-server` like any other option, can be added to `dnsmasq_dns_options`.

### DNS interfaces

The DNS role are written to set the `interface` Dnsmasq configuration.
The value `default` tells the role to use the name of the gateway interface of the target.
Otherwise it should list the interfaces that Dnsmasq should listen for DNS requests on.

### DNS hosts

The role can maintain a hosts file which is passed to Dnsmasq via the `addn-hosts` parameter.
Add `no-hosts` to `dnsmasq_dns_options` to avoid also reading `/etc/hosts` on the target system.

### servers

The role creates `server` and `rev-server` configuration parameters for Dnsmasq.
Add `no-resolv` to `dnsmasq_dns_options` to avoid also using the resolver on the target system.

#### address values

Each server must have _one_ `address` value which must be an IP _address_.
Each `address` will become a Dnsmasq `server` parameter.
A server can contain one or more `domain` and `network` values.

#### domain and network values

Adding a `domain` or list of `domains` to a server configures Dnsmasq to forward queries for it/them to the server.
Adding a `network` or list of `networks` forwards reverse-DNS queries, i.e., IP to name queries to the corresponding server.

## Variables

### Optional

- `dnsmasq_dns_servers`: List of DNS servers to configure
- `dnsmasq_dns_hosts`: DNS host entries
- `dnsmasq_dns_interfaces`: Interfaces to listen on for DNS requests
- `dnsmasq_dns_options`: DNS-related Dnsmasq options

## Playbooks

A DHCP and DNS server running on the gateway that uses `1.1.1.1` and `8.8.8.8` for DNS:

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dns_options: [bogus-priv, domain-needed, no-resolv]
    dnsmasq_dns_servers: [{address: 1.1.1.1}, {address: 8.8.8.8}]
  roles:
    - amigus.dnsmasq.dnsmasq
```

NOTE: the `bogus-priv` and `domain-needed` parameters are recommended to avoid spamming them.
`no-resolv` avoids also using whatever server(s) exist in `/etc/resolv.conf`

A DNS server with a hosts file and a non-default local resolver:

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dns_hosts: |
      192.168.1.11 storage
      192.168.1.12 games
    dnsmasq_dns_options: [bogus-priv, domain-needed, no-hosts, no-resolv]
    dnsmasq_dns_interfaces: default
    dnsmasq_dns_servers:
      - address: 192.168.1.2
        domain: wifi.lan
        network: 192.168.1.0/24
      - address: 1.1.1.1
      - address: 8.8.8.8
  roles:
    - amigus.dnsmasq.dnsmasq
```

It ignores /etc/hosts and /etc/resolv.conf.
It forwards queries for `wifi.lan` and `192.168.1.0/24` to `192.168.1.2`,
and uses `1.1.1.1` and `8.8.8.8` to resolve anything else.
