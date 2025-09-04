# dnsmasq Role

Basic Dnsmasq installation and configuration role.

This role provides the includes all the other Dnsmasq roles in the collection.

It will always include the `dnsmasq_install` role.
It will include the `dnsmasq_dhcp` role when `dnsmasq_dhcp_interfaces` is defined.
It will include the `dnsmasq_dhcp_db` role when `dnsmasq_dhcp_db` is defined.
It will include the `dnsmasq_dns` role when `dnsmasq_dns_options` or `dnsmasq_dns_servers` are defined.
It will include the `dnsmasq_web` role when `dnsmasq_web_binary` is defined.

## Example Playbook

A DNS resolver that passes queries through to the host resolver:

```yaml
- hosts: dnsmasq
  roles:
    - amigus.dnsmasq.dnsmasq
```

## License

See the collection LICENSE file.
