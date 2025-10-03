# dnsmasq_dhcp_db

Manages the optional lease database, associated script, and configuration including:

- dhcp-script
- dhcp-scriptuser
- script-on-renewal
- leasefile-ro

The role contains a shell script and an SQLite3 database schema.
It configures Dnsmasq to use the shell script to manage leases.
The shell script uses the SQLite3 database to manage them.
It also keeps accumulates request and client data over time.
It will run when `dnsmasq_dhcp_db` is defined.

## Database Schema

The role creates an SQLite3 database three tables:

- requests
- leases
- clients

The requests table grows infinitely(!).
The leases and clients tables are maintained by the script.
The leases table contains current leases.
The clients table contains a list of current and past clients.

## Role Variables

### Required

- `dnsmasq_dhcp_db`: Where to store the database (e.g., `/var/lib/misc/dnsmasq.leases.db`)
- `dnsmasq_dhcp_db_script`: Where to install the script (e.g., `/usr/sbin/dnsmasq-leasesdb`)

### Optional

- `dnsmasq_dhcp_db_script_sqlite_bin`: The location of the sqlite3 binary
- `dnsmasq_dhcp_db_user`: User to run the script as (default, `dnsmasq_user`)

## Examples

A DHCP server that uses the SQLite lease database:

```yaml
- hosts: dnsmasq
  vars:
    dnsmasq_dhcp_db: /var/lib/misc/dnsmasq.leases.db
    dnsmasq_dhcp_db_script: /usr/sbin/dnsmasq-leasesdb
    dnsmasq_dhcp_interfaces: [{ device: eth0, router: 1, start: 100, end: 199 }]
  roles:
    - amigus.dnsmasq.dnsmasq
```

A more complex example with MAC address tagging:

```yaml
- hosts: servers
  vars:
    dnsmasq_dhcp_db: /var/lib/misc/dnsmasq.leases.db
    dnsmasq_dhcp_client_options:
      - tag:hyperv,option:dns-server:192.168.1.5
      - tag:wizlb,option:dns-server,192.168.1.4
    dnsmasq_dhcp_domain: wifi.lan
    dnsmasq_dhcp_interfaces: [{ device: eth0, router: 1, start: 100, end: 199 }]
    dnsmasq_dhcp_mac_tags:
      - hyperv: 00:15:5F:*:*:*
      - wizlb: [6c:29:90:*:*:*, 44:4f:8e:*:*:*]
    dnsmasq_dhcp_db_script: /usr/sbin/dnsmasq-leasesdb
  roles:
    - amigus.dnsmasq.dnsmasq
```
