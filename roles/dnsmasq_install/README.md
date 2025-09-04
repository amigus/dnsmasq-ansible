# dnsmasq_install Role

Installs and configures Dnsmasq on the target system using _apk_, _dnf_ and _zypper_.

- Installs the Dnsmasq package(s)
- Installs SQLite3 when using the lease database
- Ensures that the user, default `dnsmasq`,
  usergroup, default `dnsmasq`,
  and configuration directory, default `/etc/dnsmasq.d` all exist.
- Configures Dnsmasq to read the `.conf` files from the configuration directory.
- Configures _SELinux_ (when present) to permit Dnsmasq to use `dhcp-hostsdir`.
- Configures _firewalld_ (when present) to permit DHCP, DNS or both.

## Role Variables

### Required

None.

### Optional

- `dnsmasq_etc`: The Dnsmasq configuration directory;
  default is `/etc/dnsmasq.d`
- `dnsmasq_user`: The user that Dnsmasq runs as;
  default is `dnsmasq`
- `dnsmasq_usergroup`: The usergroup that the Dnsmasq user belongs to;
  default is `dnsmasq`

## Dependencies

None.

## Example Playbook

```yaml
- hosts: dnsmasq
  roles:
    - amigus.dnsmasq.dnsmasq_install
```

## License

See the collection LICENSE file.
