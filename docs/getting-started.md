# Getting Started

## Prerequisites

### Ansible controller

A Linux system with [Ansible](https://docs.ansible.com/ansible/latest/installation_guide)
and [amigus.dnsmaq](../installation).

### Target system(s)

Target Linux system(s) with Python already installed,
that use the `apk`, `dnf` or `zypper` package manager.

The user running Ansible needs SSH access to the target system(s),
**as root or a non-root user with _sudo root_ access**.

For DHCP, the systems _must_ **already have a static IP address** on the interface.

## Create an Inventory

Ansible targets systems from an _inventory_.
Inventories are lists of systems and groupings that Ansible targets en masse.

!!! tip
    The Ansible [documentation](https://docs.ansible.com/ansible/latest)
    includes a [guide](https://docs.ansible.com/ansible/latest/inventory_guide)
    to creating and managing inventories.

The collection's default playbook targets the `dnsmasq` group.
So it is best to add target systems to that group in the inventory:

```yaml {title=inventory.yaml}
---
dnsmasq:
  hosts:
    server1:
    # Use a DNS resolvable name, or set ansible_host to a resolvable name or IP address:
    # noname_server:
    #   ansible_host: 192.168.100.2
  vars:
    # Required when SSH access to the target is as a non-root user with sudo root access:
    ansible_become: yes
    # Required when using a user with a different username than the user calling Ansible:
    # ansible_user: admin
```

!!! important
    Ensure that the user can log as root or a non-root user with sudo _root_ access without any prompting

!!! tip
    It is not typical but demonstration purposes,
    the same system can be function as the controller and the target.
    **To target localhost (only):**

    ```yaml
    ---
    dnsmasq:
      hosts:
        localhost:
          ansible_connection: local
    ```

    !!! note
        The default `ansible_connection` is `ssh`.
        Using `local` simplifies the demonstration environment by not using it.

## Run the playbook

Ansible executes _tasks_ most often organized into _playbooks_.

There is also a [guide](https://docs.ansible.com/ansible/latest/playbook_guide)
however, **the collection includes this one**:

```yaml {title=dnsmasq.yaml}
---
- hosts: dnsmasq
  roles:
    - amigus.dnsmasq.dnsmasq
```

To run it on the inventory:

```bash
ansible-playbook -i inventory.yaml amigus.dnsmasq.dnsmasq
```

If everything is set up correctly, it will install Dnsmasq on the target(s). :tada:

## Configuration

!!! Note
    Ansible variables can be added to either the inventory or playbook,
    so adding the YAML below to either will work.

### DNS

The [dnsmasq_dns](../roles/dnsmasq_dns) role sets DNS options and resolvers (servers).

For example, to use `1.1.1.1` _instead of_ the system resolver:

1. Add `1.1.1.1` as a server
1. Add the Dnsmasq `no-resolv` to the options
1. Add `bogus-priv` and `domain-needed` as well if the resolver is external

```yaml
dnsmasq_dns_options:
- bogus-priv
- domain-needed
- no-resolv
dnsmasq_dns_servers
- address: 1.1.1.1
```

### DHCP

The [dnsmasq_dhcp](../roles/dnsmasq_dhcp) role runs when `dnsmasq_dhcp_interfaces` is defined.

```yaml
dnsmasq_dhcp_interfaces:
- device: eth0
- router: 1
- start: 100
- end: 199
```

Only the `device` is required:

- Add `router` to tell Dnsmasq use that sever (not itself) as the gateway.
- Add `start` to lease a range of IP addresses from the subnet dynamically,
- Add `end` to extend the range less than the last IP in the subnet.

So, if `start` is not defined then `dnsmasq_dhcp_hosts` should be or the server will not have any clients:

```yaml
dnsmasq_dhcp_hosts: |
  11:22:33:44:55:66,192.168.1.11,server1
  server2,192.168.1.12
dnsmasq_dhcp_interfaces: [{ device: eth0 }]
```

!!! tip
    Dnsmasq allows reservations based on name instead of MAC address,
    so in the example above, any server sending DHCP requests with the client hostname `server2`,
    will get the IP address `192.168.1.12`.
