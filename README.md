# ansible-playbooks

Playbooks contained in this repository can be used as configuration scaffolding for your workloads running in any environment. Configuration is split into various roles that must be installed as dependencies prior to running a play. At a high level, the following must be accomplished prior to running a play from this repo:

- Base OS installation of new server
- Network configuration of new server (including bond, bridge, DNS and routing)
- Configuration of password-less (public key) SSH authentication from the Ansible host (your laptop) to the new server

## Using this repository

Simply clone the repository and start modifying files according to your needs.

```
git clone https://github.com/williamsmt/ansible-playbooks.git myPlaybooks/
```

Prerequisites:
- Ansible and its dependencies must be installed
- Inventory available with ssh password-less authentication (these plays could be run with imaging tool such as [Packer](https://www.packer.io/))
- Role and collection dependencies must be installed prior to running plays
- Role-specific configuration files must be updated to suit your environment, e.g. `banner.txt` contained in [ansible-role-common](https://github.com/williamsmt/ansible-role-common/blob/main/files/banner.txt)

Notes:
- This playbook collection does not include an inventory file. It is intended that the plays will be run from an automation runtime, namely Terraform and Packer. You can supply an inventory file and amend the below usage commands to accommodate a static or dynamic inventory use case.

## Usage

As mentioned above, any playbook contained in this repository can be run once the requisite collections and roles have been installed. It is primarily intended for these playbooks to be included in ancillary automation routines, e.g. Terraform modules, and as such does not contain an inventory file. The commands below assume you are supplying a host IP address as the inventory object for each play.

The following command block outlines necessary steps in order to prepare your local clone of this repository for running playbooks:

```
# Install prerequisite roles
ansible-galaxy role install -r requirements.yml

# Install prerequisite collections
ansible-galaxy collection install -r requirements.yml
```

Make any necessary configuration file and variable changes within the installed roles and playbooks (see below configuration file table). Once satisfied with defined settings to be applied, the plays can be run.

### Independent Run

For a VMware environment, run the playbook as:

`ansible-playbook -i {ip_address_of_host_or_inventory_file}, deployBase.yml`

For all other environments, run the playbook as:

`ansible-playbook -i {ip_address_of_host_or_inventory_file}, deployBase.yml --skip-tags vmware`

If you are running the playbook on production infrastructure, run the playbook as:

`ansible-playbook -i {ip_address_of_host_or_inventory_file}, deployBase.yml --skip-tags volatile`

### Packer Provisioner

```
"provisioners": [
    {
      "collections_path": "./ansible/collections",
      "extra_arguments": [
        "-v"
      ],
      "galaxy_file": "./ansible/requirements.yml",
      "playbook_file": "./ansible/playbook.yml",
      "roles_path": "./ansible/roles",
      "type": "ansible",
      "user": "ansible-admin"
    }
]
```

### Terraform Run

`TBD`

## Repository Contents

The following index outlines the file and directory structure of this repository, inclusive of playbooks, roles, configuration files, and tags. Links to dependent roles are included which highlight documentation regarding required variables and expected behavior. Please carefully review prior to running a playbook referencing said roles.

### Playbooks

A list of playbooks and their description.

Playbook | Description
-------- | -----------
`deployBase.yml` | Base configuration - installs dependent packages, users, configuration and proxy settings if required
`deployDocker.yml` | Includes all configuration from `deployBase.yml` and Docker runtime with docker-compose utility

### Roles

An example directory structure of required roles:

```
roles/
├── ansible-role-common/        Installs base configuration
│   ├── defaults/                   Default variable values
│   ├── files/                      Static config files to be installed
│   ├── handlers/                   Used in main tasks as triggered actions
│   ├── meta/                       Meatdata about the role
│   ├── tasks/                      Includes for main task
│   │   └ main.yml                      Bundles included task files
│   ├── templates/                  Jinja2 templated config files to be installed
│   └── vars/                       Local variables set here
└── ansible-role-docker/        Installs application on host
    ├── defaults/
    ├── files/
    ├── handlers/
    ├── meta/
    ├── tasks/
    │   └ main.yml
    ├── templates/
    └── vars/
```

Role | Repo | Description
---- | ---- | -----------
`ansible-role-common` | [Repo](https://github.com/williamsmt/ansible-role-common) |  Common configuration for new linux hosts
`ansible-role-docker` | [Repo](https://github.com/williamsmt/ansible-role-docker) | Deploys Docker host and docker-compose utility

### Collections

A list of Ansible Collections and their description.

Collection | Repo | Description
---------- | ---- | -----------
`ansible-posix` | [Repo](https://github.com/ansible-collections/ansible.posix) | An Ansible Collection of modules and plugins that target POSIX UNIX/Linux and derivative Operating Systems

### Configuration files

A list of configurable files included in dependent roles for installation.

File | Role | Description
---- | ---- | -----------
`banner.txt` | `ansible-role-common` | Installed as `/etc/issue` login banner

### Tags

Use the following command on any playbook in this repository to show a list of available tags:

```
ansible-playbook deployBase.yml --list-tags
```

Role | Tags | Description
--- | --- | ---
`ansible-role-common` | `vmware` | Designates target cloud for workload as vSphere environment
`ansible-role-common` | `volatile` | Includes invasive tasks such as `apt-get dist-upgrade`

## License

Released under the [Apache-2.0 license](LICENSE)

## Author Information

https://github.com/williamsmt

This is not an officially supported Google product