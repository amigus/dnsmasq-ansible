# Installation

The installation guide covers getting the collection installed on an Ansible _controller_.

The [Getting Started](../getting-started) guide covers using the collection to install Dnsmasq.

## Prerequisites

A Linux system with Python installed to serve as the Ansible _controller_.
It will run the roles to configure the _target_ system.

## Install

### Virtual enviornment

Python includes [venv](https://docs.python.org/3/library/venv.html).
This guide will use it to create a "virtual environment."
The collection and it's dependencies will be installed into it.
This avoids adding it to the global/system environment.

```bash
python -m venv dnsmasq.venv.d
. dnsmasq.venv.d/bin/activate
```

!!! note
    This is optional but _recommended_ in most cases.
    However, you can **skip this step if the controller system is dedicated** to running Ansible.

### Dependencies via pip

#### netaddr

The [netaddr](https://pypi.org/project/netaddr/) library may already be installed.
The command below will use pip to install it only when it is not present.

```bash
python3 -c "import netaddr" 2>/dev/null || pip install netaddr
```

#### ansible

Likewise, ansible may already be present.
The command below will install ansible when it is not already present.

```bash
command -v ansible >/dev/null 2>&1 || pip install ansible-core
```

### Collection via Galaxy

Ansible includes the `ansible-galaxy` tool.
It manages roles and collections in the [Ansible Galaxy](https://galaxy.ansible.com/ui/).

```bash
ansible-galaxy collection install amigus.dnsmasq
```

### All-in-one install

```bash
python3 -m venv dnsmasq.venv.d
. dnsmasq.venv.d/bin/activate
python3 -c "import netaddr" 2>/dev/null || pip install netaddr
command -v ansible >/dev/null 2>&1 || pip install ansible-core
ansible-galaxy collection install amigus.dnsmasq
```
