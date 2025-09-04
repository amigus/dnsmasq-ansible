# dnsmasq_dhcp Role

Manages DHCP configuration including:

- dhcp-hostsdir
- dhcp-hostsfile
- dhcp-ignore
- dhcp-range
- dhcp-options
- dhcp-mac
- domain

## DHCP Configuration

The DHCP role is written to assume that the interface already has a static IP.
It then uses the network information of the device to configure the server.
Thus are are no explicit subnet declarations and nodes are integers.

### DHCP interfaces

The `start`, `end`, and `router` variables are integers representing the node.
E.g., `router=1`, `start=2` and `end=10` on an interface with subnet `192.168.1.0/24`,
will set the subnet gateway to `192.168.1.1` and the range to `192.168.1.2-10`.

#### device and router values

Each interface must specify at least a device name, e.g., `eth0`.
It can also specify a router, e.g., `1`.
If the router is not set then Dnsmasq supplies its own IP as the gateway by default.

#### start and end values

If no `start` or `end` are set then the inteface will be `static` in Dnsmasq.
If a `start` but no `end` is set then the range will run to the last address in the subnet.

### Ethernet address (MAC) tags

The role sets the Dnsmasq `dhcp-mac` parameter which enables ethernet address prefix-based tagging.

### DHCP hosts

The role manages a file containing `dhcp-host` configuration values.
It passes the file to Dnsmasq in the `dhcp-hostsfile` parameter.
It also sets the `dhcp-hostsdir` from the `dnsmasq_dhcp_hosts_dir` variable.

## Role Variables

### Required

- `dnsmasq_dhcp_interfaces`: List of interfaces to configure DHCP on

### Optional

- `dnsmasq_dhcp_client_options`: A list of Dnsmasq `dhcp-option` parameters,
  e.g., option:domain-search,wifi.lan
- `dnsmasq_dhcp_domain`: Sets the Dnsmasq `dhcp-domain`
- `dnsmasq_dhcp_hosts`: DHCP host reservations
- `dnsmasq_dhcp_hosts_dir`: Directory for DHCP host files, i.e., `dhcp-hostsdir`
- `dnsmasq_dhcp_options`: A list of Dnsmasq DHCP-related options, e.g., `log-dhcp`
- `dnsmasq_dhcp_mac_tags`: A mapping of Dnsmasq tags to ethernet addresses

## Dependencies

- amigus.dnsmasq.dnsmasq_install

## Example Playbooks

Minimal DHCP server on `eth0` that sets itself as the gateway and leases the entire subnet from `.5`.
It also sets itself as the DNS server by forwarding everything through system resolver:

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dhcp_interfaces: [{ device: eth0, start: 5 }]
  roles:
    - amigus.dnsmasq.dnsmasq
```

A static (reservation-only) DHCP server that serves two clients that use `.1` as the gateway:

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dhcp_hosts: |
      00:15:5F:09:03:01,192.168.1.11,server1
      00:15:5F:09:03:02,192.168.1.12,server2
    dnsmasq_dhcp_interfaces: [{ device: eth0, router: 1 }]
  roles:
    - amigus.dnsmasq.dnsmasq
```

A DHCP server that uses tags to manage device settings:

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dhcp_client_options:
      - tag:hyperv,option:dns-server:192.168.1.5
      - tag:wizlb,option:dns-server,192.168.1.4
    dnsmasq_dhcp_domain: wifi.lan
    dnsmasq_dhcp_interfaces: [{ device: eth0, router: 1, start: 100, end: 199 }]
    dnsmasq_dhcp_mac_tags:
      - hyper-v: 00:15:5F:*:*:*
      - wizbulb: [6c:29:90:*:*:*, 44:4f:8e:*:*:*]
  roles:
    - amigus.dnsmasq.dnsmasq
```

## License

See the collection LICENSE file.
