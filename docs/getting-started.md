# Getting Started

## Prerequisites

### Ansible controller

A Linux system with [Ansible](https://docs.ansible.com/ansible/latest/installation_guide)
and [amigus.dnsmaq](installation.md) installed.

### Target system(s)

**Must have Python installed** and use the `apk`, `dnf`, or `zypper` package manager.

#### Supported distros

- **RedHat**-based: Rocky Linux, AlmaLinux
- **OpenSUSE**-based: Leap or Tumbleweed
- **Alpine** Linux

#### Root access

The user running Ansible needs **SSH access to the target system(s) as root** or a non-root user with _sudo root_ access.

#### Static IP address

For DHCP to work, the target system(s) **_must_ already have a static IP address** on the interface.

## Create an Inventory

Ansible targets systems from an _inventory_.
Inventories are lists of systems and groupings that Ansible targets en masse.

!!! tip
    The Ansible [documentation](https://docs.ansible.com/ansible/latest)
    includes a [guide](https://docs.ansible.com/ansible/latest/inventory_guide)
    on creating and managing inventories.

The collection's default playbook targets the `dnsmasq` group.
So it is best to add target systems to that group in the inventory:

```yaml {title=inventory.yaml}
---
dnsmasq:
  hosts:
    # Use a DNS resolvable name
    server1:
    # Or set ansible_host to a resolvable name or IP address:
    # nodns1:
    #   ansible_host: 192.168.100.2
  vars:
    # Use when SSH access to the target is as a non-root user with sudo root access:
    ansible_become: yes
    # Use when SSH access uses a different username than the user calling Ansible:
    # ansible_user: adam
```

!!! important
    Ensure that the user can log in as root or as a non-root user with sudo _root_ access, without prompts.

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

The collection includes this one:

```yaml {title=dnsmasq.yaml}
---
- hosts: dnsmasq
  roles:
    - amigus.dnsmasq.dnsmasq
```

Use `ansible-playbook` to run it on the inventory:

```bash
ansible-playbook -i inventory.yaml amigus.dnsmasq.dnsmasq
```

It will install Dnsmasq on the target(s) if everything is set up correctly.
:tada:

!!! tip
    The Ansible [documentation](https://docs.ansible.com/ansible/latest)
    includes a very useful [guide](https://docs.ansible.com/ansible/latest/playbook_guide)
    on using playbooks.

## Configuration

!!! Note
    Ansible variables can be in the inventory or the playbook.
    When they are in both, the inventory takes precedence.

### DNS

The [dnsmasq_dns](roles/dnsmasq_dns.md) role sets DNS options and resolvers (servers).

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

The [dnsmasq_dhcp](roles/dnsmasq_dhcp.md) role runs when `dnsmasq_dhcp_interfaces` is defined.

```yaml
dnsmasq_dhcp_interfaces:
- device: eth0
  router: 1
  start: 100
  end: 199
```

Only the `device` is required:

- Adding `router` tells Dnsmasq to offer that server as the network gateway instead of itself by default.
- Adding `start` tells Dnsmasq to dynamically lease a range of subnet IP addresses starting at this one.
- Adding `end` tells Dnsmasq to extend the range only to this IP instead of extending it through the last IP in the subnet.

Thus, to configure a _static_ server that only serves reserved IP addresses, define `dnsmasq_dhcp_hosts`:

```yaml
dnsmasq_dhcp_hosts: |
  11:22:33:44:55:66,192.168.1.11,server1
  server2,192.168.1.12
dnsmasq_dhcp_interfaces: [{ device: eth0 }]
```

!!! note
    Dnsmasq allows reservations based on name instead of MAC address,
    so in the example above, any client sending DHCP requests with the hostname `server2`,
    will get the IP address `192.168.1.12`.

#### Database

The [dnsmasq_dhcp_db](roles/dnsmasq_dhcp_db.md) role runs when `dnsmasq_dhcp_db` is defined.

```yaml
---
dnsmasq_dhcp_db: /var/lib/misc/dnsmasq.leases.db
dnsmasq_dhcp_interfaces: [{ device: eth0, start: 100, end: 199 }]
dnsmasq_dhcp_db_script: /usr/sbin/dnsmasq-leasesdb
```

!!! note
    The role will _create_ the database and install the script so neither need to exist initially.

#### dnsmasq-web

The [dnsmasq_web](roles/dnsmasq_web.md) role runs when `dnsmasq_dhcp_web_binary` is defined.

```yaml
dnsmasq_dhcp_db: /var/lib/misc/dnsmasq.leases.db
dnsmasq_dhcp_hosts_dir: /var/lib/misc/dnsmasq.hosts.d
dnsmasq_dhcp_interfaces: [{ device: eth0, start: 100, end: 199 }]
dnsmasq_dhcp_db_script: /usr/sbin/dnsmasq-leasesdb

dnsmasq_dhcp_web_binary: /usr/sbin/dnsmasq-web
```

It installs the latest release of [dnsmasq-web](https://github.com/amigus/dnsmasq-web).
Once installed, it accepts HTTP requests on port `867` by default.

The REST API [endpoints](https://github.com/amigus/dnsmasq-web#endpoints)
enable management of DHCP clients, leases and client IP reservations.

There is also a [CLI](https://github.com/amigus/dnsmasq-web/tree/main/cli)
that is handy for viewing the DHCP leases table remotely.

## Takeaways

Typically, DHCP configuration boils down to three (3) things:

1. Whether the server is static or dynamic
1. What DNS server(s) does the server offer clients
1. Does the server include DNS records from a hosts file

If `start` is defined, the server is dynamic; otherwise, it's static.

The DNS server(s) can be the underlying system resolver or a list of servers.

!!! important
    Add `no-resolv` to `dnsmasq_dns_options` when defining `dnsmasq_dns_servers`.
    Otherwise, Dnsmasq will use _both_ as DNS forwarders.

    Also, add `bogus-priv` to avoid spamming the upstream server(s) with requests for local resources.

    ```yaml
    dnsmasq_dns_options: [bogus-priv, no-resolv]
    dnsmasq_dns_servers: [{ address: 1.1.1.1 }]
    ```

!!! important
    The DHCP server offers itself as the DNS server, then forwards requests.
    However, it can also offer another DNS server.

    ```yaml
    dnsmasq_dhcp_client_options:
      - option:dns-server,192.168.1.9
    ```

    In this case, add `no-hosts` and `no-resolv` to `dnsmasq_dns_options`
    without defining `dnsmasq_dns_servers`, to disable the DNS resolver.

    ```yaml
    dnsmasq_dns_options: [no-hosts, no-resolv]
    ```

!!! important
    Add `no-hosts` to dnsmasq_dns_options when defining `dnsmasq_dns_hosts`.
    Otherwise, Dnsmasq will _also_ serve records from `/etc/hosts`.

    ```yaml
    dnsmasq_dns_hosts: |
      192.168.1.6 mco.lbsg.net
    dnsmasq_dns_options: [bogus-priv, no-hosts]
    ```
