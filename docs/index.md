# About

[amigus.dnsmasq](https://galaxy.ansible.com/ui/repo/published/amigus/dnsmasq/)
is a collection available on [Ansible Galaxy](https://galaxy.ansible.com).

It contains roles to automate the installation and configuration of [Dnsmasq](https://dnsmasq.org/doc.html):

[`dnsmasq`](https://galaxy.ansible.com/ui/repo/published/amigus/dnsmasq/content/role/dnsmasq/)
:   runs `dnsmasq_install` then other roles based on which variables are present.

    The other roles are run in the order listed.

[`dnsmasq_install`](https://galaxy.ansible.com/ui/repo/published/amigus/dnsmasq/content/role/dnsmasq_install/)
:   installs Dnsmasq and configures the system to for it.

    Currently supports three package managers

    - apk (Alpine Linux)
    - dnf (RedHat Linux variants)
    - zypper (OpenSUSE Linux variants)

    Configures SELinux to allow Dnsmasq to manage DHCP lease reservations in `dhcp-hostsdir`.

    Configures firewalld to allow DHCP and/or DNS ports

[`dnsmasq_dhcp`](https://galaxy.ansible.com/ui/repo/published/amigus/dnsmasq/content/role/dnsmasq_dhcp/)
:   configures DHCP options and tags, IPv4 ranges and reservations.

[`dnsmasq_dhcp_db`](https://galaxy.ansible.com/ui/repo/published/amigus/dnsmasq/content/role/dnsmasq_dhcp_db/)
:   adds an [SQLite3](https://sqlite.org/) DHCP lease management.

[`dnsmasq_dns`](https://galaxy.ansible.com/ui/repo/published/amigus/dnsmasq/content/role/dnsmasq_dns/)
:   configures DNS resolver options, including upstream servers, and a hosts file.

[`dnsmasq_web`](https://galaxy.ansible.com/ui/repo/published/amigus/dnsmasq/content/role/dnsmasq_web/)
:   installs the [dnsmasq-web](https://github.com/amigus/dnsmasq-web) REST API.
