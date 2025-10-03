# dnsmasq

This is the main role of the collection.
It conditionally includes all the other roles:

| Role                | Included by this role                                            |
|---------------------|------------------------------------------------------------------|
| `dnsmasq_install`   | always                                                           |
| `dnsmasq_dhcp`      | when `dnsmasq_dhcp_interfaces` is defined                        |
| `dnsmasq_dhcp_db`   | when `dnsmasq_dhcp_db` is defined                                |
| `dnsmasq_dns`       | when `dnsmasq_dns_options` or `dnsmasq_dns_servers` are defined  |
| `dnsmasq_web`       | when `dnsmasq_web_binary` is defined                             |

## Examples

A DNS resolver that forwards queries to the host resolver:

```yaml
- hosts: dnsmasq
  roles:
    - amigus.dnsmasq.dnsmasq
```

A resolver that resolves two hosts and forwards all other queries to `1.1.1.1`:

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dns_hosts: |
      192.168.1.11 storage
      192.168.1.12 games
    dnsmasq_dns_options: [bogus-priv, domain-needed, no-hosts, no-resolv]
    dnsmasq_dns_servers: [{ address: 1.1.1.1 }]
  roles:
    - amigus.dnsmasq.dnsmasq
```
