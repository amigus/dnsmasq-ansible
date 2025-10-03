# dnsmasq_web

Installs the [dnsmasq-web](https://github.com/amigus/dnsmasq-web) REST API.
It has methods for managing DHCP leases and querying client and request data.
It also manages DHCP revervations as files in `dhcp-hostsdir`.

## Variables

### Required

- `dnsmasq_web_binary`: Where to the install the dnsmasq-web binary

### Optional

- `dnsmasq_web_user`: User to run the service as;
  default is `dnsmasq_user`
- `dnsmasq_web_group`: Group to run the service as;
  default is `dnsmasq_usergroup`
- `dnsmasq_web_listen_address`: The TCP _address:port_ that the daemon should listen on;
  default is `:867`
- `dnsmasq_web_pid_file`: Path to the daemon's PID file;
  default is `/run/dnsmasq-web.pid`

## Playbook

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dhcp_hosts_dir: /var/lib/misc/dnsmasq.hosts.d
    dnsmasq_dhcp_interfaces: [{ device: eth0 }]
    dnsmasq_web_binary: /usr/sbin/dnsmasq-web
  roles:
    - amigus.dnsmasq.dnsmasq
```
