# Home

This is the documentation for the _amigus.dnsmasq_ Ansible collection.
It is [available](https://galaxy.ansible.com/ui/repo/published/amigus/dnsmasq/)
on Ansible Galaxy.
It contains roles that automate [Dnsmasq](https://dnsmasq.org/doc.html).
They are modular and create their own separate Dnsmasq configuration files.

## Roles

[`dnsmasq`](roles/dnsmasq.md)
:   runs `dnsmasq_install` then other roles based on which variables are present.

[`dnsmasq_install`](roles/dnsmasq_install.md)
:   installs Dnsmasq and configures the system to run it.

[`dnsmasq_dhcp`](roles/dnsmasq_dhcp.md)
:   configures DHCP options and tags, IPv4 ranges, and reservations.

[`dnsmasq_dhcp_db`](roles/dnsmasq_dhcp_db.md)
:   adds an [SQLite3](https://sqlite.org/) DHCP lease management.

[`dnsmasq_dns`](roles/dnsmasq_dns.md)
:   configures DNS resolver options, including upstream servers, and a hosts file.

[`dnsmasq_web`](roles/dnsmasq_web.md)
:   installs the [dnsmasq-web](https://github.com/amigus/dnsmasq-web) REST API.

## Prerequisites

This is an Ansible Collection so it requires [Ansible](https://www.ansible.com)
which requires [Python](https://www.python.org).
It also requires the Python [netaddr](https://pypi.org/project/netaddr/) libarary.

## Quick Start

Install the collection from Ansible Galaxy:

```bash
ansible-galaxy collection install amigus.dnsmasq
```

Target the `dnsmasq` group in your Ansible inventory:

```ini {title=/etc/ansible/hosts}
[dnsmasq]
192.168.100.2
```

Run the default playbook:

```bash
ansible-playbook amigus.dnsmasq.dnsmasq
```

The initial installation will include no configuration.
See the [Configuration](getting-started.md#configuration)
section of the [Getting Started](getting-started.md)
guide for details.
